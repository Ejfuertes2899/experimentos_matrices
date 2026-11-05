# Benchmark reproducible de multiplicación de matrices NxN

La versión estable de este artifact se archiva en Zenodo. El DOI de esa versión está en la sección «Publicación en Zenodo». La licencia está en `LICENSE` y la descripción de los datos en `DATASHEET.md`.

## Objetivo

Comparar el tiempo de ejecución del algoritmo naive de multiplicación de matrices cuadradas NxN en C++, Go, Java y Python, con implementaciones equivalentes y bajo las mismas condiciones de entrada y de medición.

## Pregunta de investigación

¿Existen diferencias estadísticamente significativas en el tiempo de ejecución del algoritmo naive de multiplicación de matrices NxN entre implementaciones equivalentes desarrolladas en C++, Go, Java y Python, y cómo varían estas diferencias según el tamaño de la matriz?

## Hipótesis

- **H₀:** El lenguaje de implementación no produce diferencias estadísticamente significativas en el tiempo de ejecución del algoritmo de multiplicación de matrices.
- **H₁:** Al menos uno de los lenguajes presenta un tiempo de ejecución significativamente diferente de los demás.

El nivel de significación del contraste es α = 0.05.

## Variables

### Independientes

| Variable                | Niveles               |
| ----------------------- | --------------------- |
| Lenguaje                | C++, Go, Java, Python |
| Tamaño de la matriz (N) | 500, 1000, 1500       |

### Dependiente

Tiempo de ejecución de la multiplicación, en segundos. Cada medición queda identificada por el lenguaje, el valor de N y el número de repetición.

### Controladas

- Algoritmo naive, con el mismo orden de bucles en los cuatro lenguajes.
- Tipo de dato real de 64 bits (`double` en C++ y Java, `float64` en Go, `float` en Python).
- Mismas matrices de entrada, generadas con la misma semilla.
- Misma forma de medir el tiempo.
- Sin librerías de multiplicación de matrices.
- Misma máquina para las cuatro implementaciones.

## Diseño experimental

El diseño es factorial completo. Se cruzan los 4 lenguajes con los 3 tamaños y cada combinación se repite 10 veces. En total son 120 mediciones.

La unidad experimental es una multiplicación, para un lenguaje, un N y una repetición. Las matrices A y B ya están creadas cuando empieza esa medición.

### Algoritmo

```text
para i en 0 … N−1:
    para j en 0 … N−1:
        C[i][j] ← 0
        para k en 0 … N−1:
            C[i][j] ← C[i][j] + A[i][k] × B[k][j]
```

El recorrido es i, j, k en los cuatro lenguajes. La multiplicación se hace con ese bucle, sin BLAS, NumPy, Eigen ni otra librería equivalente.

### Matrices de entrada

La semilla es 42. Los valores se obtienen con el generador de Park–Miller:

```text
x ← (16807 · x) mod 2147483647
valor ← x / 2147483647
```

Primero se llena A por filas y después B, siguiendo con el mismo generador. Así las cuatro implementaciones usan las mismas matrices. A y B quedan terminadas antes de medir.

### Medición

El tiempo empieza justo antes de la multiplicación y termina cuando C ya está calculada. Generar o inicializar las matrices no forma parte del tiempo registrado.

## Implementación

Las cuatro implementaciones están en `cpp/`, `go/`, `java/` y `python/`. Usan el mismo algoritmo, el mismo tipo de 64 bits, la misma semilla y el mismo criterio de reloj. No usan librerías de multiplicación de matrices.

Los resultados de cada lenguaje quedan en:

- `results/cpp.csv`
- `results/go.csv`
- `results/java.csv`
- `results/python.csv`

Cada archivo tiene las columnas `language`, `N`, `repetition` y `time_seconds`.

`results/benchmark.csv` usa esas mismas columnas y reúne las 120 mediciones. Las tablas y las figuras quedan en `results/analysis/`. Esos dos productos, tal como están ahora, se calcularon con la medición local. El pipeline los vuelve a escribir al terminar la medición en contenedores.

Las características de la máquina, las imágenes y las versiones del análisis están en `ENVIRONMENT.md`.

## Cómo ejecutar con Docker

Hace falta Docker y Python 3.12. Cada imagen se construye a mano con su `Dockerfile`. Dentro van el compilador o el intérprete y el programa de ese lenguaje. El pipeline ejecuta esas imágenes locales. No descarga imágenes del experimento desde Docker Hub ni desde otro registro.

Con las cuatro imágenes ya construidas, Snakemake corre los contenedores, arma `results/benchmark.csv` y escribe el análisis.

### Construcción

Desde la raíz del repositorio:

```text
docker build -t matrix-bench-cpp:1.0 cpp
docker build -t matrix-bench-go:1.0 go
docker build -t matrix-bench-java:1.0 java
docker build -t matrix-bench-python:1.0 python
```

### Instalar el análisis

Una vez, desde la raíz del repositorio:

```text
python -m pip install -r requirements.txt
```

Ahí están Snakemake, pandas, statsmodels, SciPy y matplotlib, con la versión usada en este trabajo.

### Corrida completa

Con Docker en marcha y las cuatro imágenes ya construidas:

```text
python -m snakemake --cores 1 --resources machine=1
```

Ese comando es la reproducción del experimento. Hace, en este orden:

1. Un contenedor por lenguaje y por N. Son 12 corridas: C++, Go, Java y Python, cada uno con N = 500, 1000 y 1500, y 10 repeticiones. Cada una escribe `results/parts/<lenguaje>_n<N>.csv`.
2. Junta las tres corridas de un lenguaje en `results/cpp.csv`, `results/go.csv`, `results/java.csv` y `results/python.csv`.
3. Junta esos cuatro archivos en `results/benchmark.csv`.
4. Con ese archivo calcula el ANOVA, Tukey HSD cuando corresponde, y escribe las tablas y las figuras en `results/analysis/`.

Snakemake no arma el benchmark hasta tener los cuatro CSV, y no empieza el análisis hasta tener `results/benchmark.csv`. Si la corrida se corta, el mismo comando sigue desde el archivo que falte. Para repetir una medición ya hecha, se borra su CSV en `results/parts/` y se vuelve a lanzar el comando.

Los contenedores van de a uno. `--cores 1` y el recurso `machine` dejan una sola medición activa. El pipeline además toma un candado mientras corre el contenedor, así dos lenguajes no comparten la CPU. Si la compartieran, los tiempos no serían comparables.

La primera corrida vuelve a medir los cuatro lenguajes aunque `results/cpp.csv` y los otros tres archivos ya existan: esos archivos son la medición local anterior, hecha sin contenedores. Python con N = 1500 es la etapa larga. En esa medición local, solo ese lenguaje llevó cerca de dos horas.

Los niveles de N, las repeticiones, el alfa y el nombre de cada imagen están en `config.yaml`.

### Prueba corta del pipeline

No es el experimento. Comprueba el encadenado con N = 40 y N = 50, dos repeticiones, y escribe en `results/smoke/` sin tocar los CSV del estudio:

```text
python -m snakemake --cores 1 --resources machine=1 --config smoke=true
```

### Correr un solo contenedor

Sirve para repetir un lenguaje a mano. N = 500, 1000 y 1500, con 10 repeticiones. En PowerShell:

```text
New-Item -ItemType Directory -Force -Path results

docker run --rm -v "${PWD}/results:/results" matrix-bench-cpp:1.0 --out /results/cpp.csv
docker run --rm -v "${PWD}/results:/results" matrix-bench-go:1.0 -out /results/go.csv
docker run --rm -v "${PWD}/results:/results" matrix-bench-java:1.0 --out /results/java.csv
docker run --rm -v "${PWD}/results:/results" matrix-bench-python:1.0 --out /results/python.csv
```

Go usa `-out`. Los otros tres usan `--out`. El archivo que sale tiene las columnas `language`, `N`, `repetition` y `time_seconds`. Esta vía no arma `benchmark.csv` ni el análisis: eso lo hace el pipeline.

### Probar un contenedor

Sirve para comprobar que el contenedor genera el CSV, con una sola matriz pequeña. Pisa el CSV de ese lenguaje en `results/`:

```text
docker run --rm -v "${PWD}/results:/results" matrix-bench-cpp:1.0 --n 50 --reps 1 --out /results/cpp.csv
docker run --rm -v "${PWD}/results:/results" matrix-bench-go:1.0 -n 50 -reps 1 -out /results/go.csv
docker run --rm -v "${PWD}/results:/results" matrix-bench-java:1.0 --n 50 --reps 1 --out /results/java.csv
docker run --rm -v "${PWD}/results:/results" matrix-bench-python:1.0 --n 50 --reps 1 --out /results/python.csv
```

## Análisis estadístico

La entrada es `results/benchmark.csv`. Los factores son el lenguaje y el tamaño N. La respuesta es el tiempo en segundos. El modelo es un ANOVA de dos factores con interacción:

```text
tiempo ~ lenguaje + N + lenguaje × N
```

N entra como factor, con los tres niveles del diseño. El diseño está balanceado: 10 repeticiones en cada una de las 12 celdas. Las sumas de cuadrados son de tipo II. El alfa es 0.05, el de la hipótesis, y también está en `config.yaml`.

Si la interacción es significativa, el efecto del lenguaje no es el mismo en todos los N. En ese caso Tukey HSD compara los lenguajes dentro de cada N, y los tamaños dentro de cada lenguaje. Si la interacción no es significativa, Tukey se hace sobre el factor principal que sí lo sea.

El cálculo del ANOVA está en statsmodels. Tukey HSD está en SciPy (`scipy.stats.tukey_hsd`), con varianzas iguales y el mismo número de repeticiones por grupo. Hay dos tablas. `tukey_hsd.csv` está en segundos: `diferencia` es la media del segundo grupo menos la del primero. `tukey_log10.csv` está en log10 del tiempo y agrega `razon`, la media geométrica del segundo grupo dividida por la del primero. Se rechaza la igualdad cuando la p ajustada es menor que 0.05. Si Levene rechaza varianzas iguales en segundos, las parejas entre C++, Go y Java se leen en la tabla de log10: el desvío de Python agranda el error de la escala en segundos.

| Archivo | Contenido |
| --- | --- |
| `results/analysis/anova.csv` | ANOVA del tiempo en segundos |
| `results/analysis/anova_log10.csv` | El mismo ANOVA sobre log10 del tiempo |
| `results/analysis/tukey_hsd.csv` | Tukey HSD del tiempo en segundos |
| `results/analysis/tukey_log10.csv` | Tukey HSD de log10(tiempo), con la razón entre medias geométricas |
| `results/analysis/descriptivos.csv` | Media, desvío y error estándar por celda |
| `results/analysis/supuestos.csv` | Shapiro–Wilk de los residuos y prueba de Levene |
| `results/analysis/resumen.txt` | Lectura de esas tablas |
| `results/analysis/figuras/interaccion.png` | Medias por N y lenguaje, eje logarítmico |
| `results/analysis/figuras/interaccion_segundos.png` | Las mismas medias en segundos |
| `results/analysis/figuras/boxplot.png` | Cajas por lenguaje en escala logarítmica, un panel por N |

`descriptivos.csv` incluye `media_sobre_cpp`, la media de la celda dividida por la media de C++ en ese mismo N.

Los tiempos de Python y de C++ no están en la misma escala, y la dispersión crece con la media. `anova.csv` es el análisis de la respuesta medida, en segundos. `anova_log10.csv` repite el modelo en una escala de proporciones: ahí una diferencia de 1 es un factor de 10. `supuestos.csv` deja Shapiro–Wilk y Levene en las dos escalas. `resumen.txt` dice si los efectos significativos coinciden.

La eta parcial cuadrado de un efecto es su suma de cuadrados dividida por esa suma más la del error.

## Reutilización

El código, los datos y esta documentación se pueden usar bajo la licencia MIT (`LICENSE`). Los datos generados, su procedencia, su estructura, sus limitaciones y las condiciones para volver a usarlos están en `DATASHEET.md`.

Para repetir el estudio en otra máquina hacen falta Docker y Python 3.12. Cada imagen sale del `Dockerfile` de su carpeta. Los parámetros del diseño están en `config.yaml` y las versiones del análisis en `requirements.txt`.

```text
docker build (manual, una vez por lenguaje)
└─ results/parts/<lenguaje>_n<N>.csv
   └─ results/<lenguaje>.csv
      └─ results/benchmark.csv
         └─ results/analysis/
```

El `Snakefile` declara esas salidas como dependencias. `python -m snakemake --dag` imprime el grafo en formato dot. La corrida completa repite el experimento.

### Publicar en Docker Hub

Estas imágenes publicadas no entran en la corrida. El pipeline usa las construidas en la máquina.

Las cuatro imágenes están en la cuenta `94031728605`, etiqueta `1.0`:

- https://hub.docker.com/r/94031728605/matrix-bench-cpp
- https://hub.docker.com/r/94031728605/matrix-bench-go
- https://hub.docker.com/r/94031728605/matrix-bench-java
- https://hub.docker.com/r/94031728605/matrix-bench-python

Para volver a publicarlas, sustituye `usuario` por el nombre de la cuenta:

```text
docker login -u usuario

docker tag matrix-bench-cpp:1.0 usuario/matrix-bench-cpp:1.0
docker tag matrix-bench-go:1.0 usuario/matrix-bench-go:1.0
docker tag matrix-bench-java:1.0 usuario/matrix-bench-java:1.0
docker tag matrix-bench-python:1.0 usuario/matrix-bench-python:1.0

docker push usuario/matrix-bench-cpp:1.0
docker push usuario/matrix-bench-go:1.0
docker push usuario/matrix-bench-java:1.0
docker push usuario/matrix-bench-python:1.0
```

En la página de cada repositorio, el campo de descripción corta puede quedar así:

| Repositorio | Descripción |
| --- | --- |
| `matrix-bench-cpp` | Multiplicación naive de matrices NxN en C++. Mide solo el producto. |
| `matrix-bench-go` | Multiplicación naive de matrices NxN en Go. Mide solo el producto. |
| `matrix-bench-java` | Multiplicación naive de matrices NxN en Java. Mide solo el producto. |
| `matrix-bench-python` | Multiplicación naive de matrices NxN en Python. Mide solo el producto. |

La descripción larga de cada imagen ya viaja en las etiquetas de la imagen (`org.opencontainers.image.title` y `org.opencontainers.image.description`). Se leen con `docker inspect matrix-bench-cpp:1.0`.

## Cómo ejecutar en la máquina local

Desde la raíz del repositorio:

```text
g++ -O2 -o cpp/main.exe cpp/main.cpp
cpp/main.exe --out results/cpp.csv

cd go
go build -o main.exe .
cd ..
go/main.exe -out results/go.csv

javac -d java java/Main.java
java -cp java Main --out results/java.csv

python python/main.py --out results/python.csv
```

Cada corrida completa usa N = 500, 1000 y 1500, con 10 repeticiones por tamaño. Para una prueba corta:

```text
python python/main.py --n 50 --reps 1 --out results/python.csv
```

## Publicación en Zenodo

Esta versión incluye el código de las cuatro implementaciones, este README, los `Dockerfile`, el `Snakefile`, los cuatro CSV, `results/benchmark.csv`, el análisis de `results/analysis/`, la licencia y `DATASHEET.md`.

Los CSV de `results/` son la medición local descrita en `ENVIRONMENT.md` y en el datasheet. El pipeline puede generar otra medición dentro de Docker. Esos tiempos nuevos no reemplazan a los archivados.

DOI: https://doi.org/10.5281/zenodo.23954875
