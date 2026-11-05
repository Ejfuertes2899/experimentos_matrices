# Benchmark reproducible de multiplicación de matrices

**Autor:** Jonathan Alexis Valenzuela Tapia  
**Programa:** Doctorado en Ingeniería en Informática, USACH  
**Curso:** Experimentación Computacional  
**Versión:** 1.0.0 (semana 5; DOI Zenodo pendiente de publicación)

## Descripción general

Este proyecto compara el tiempo de ejecución del algoritmo clásico de multiplicación de matrices cuadradas `NxN` en cuatro lenguajes de programación: C++, Go, Java y Python.

La implementación sigue una estrategia equivalente en los cuatro casos: mismo algoritmo naive, mismo tipo de datos (`double`), misma semilla para la generación de entrada y la misma estructura de validación.

## Pregunta de investigación

¿Existen diferencias estadísticamente significativas en el tiempo de ejecución entre implementaciones equivalentes en C++, Go, Java y Python, y cómo cambian estas diferencias con el tamaño de la matriz?

## Hipótesis

- **H0:** No hay diferencias estadísticamente significativas entre lenguajes.
- **H1:** Al menos uno de los lenguajes presenta diferencias significativas.

## Técnicas y diseño

- **Diseño factorial:** 4 lenguajes × 3 tamaños de matriz (`500`, `1000`, `1500`)
- **Repeticiones:** 10 por condición
- **Semilla:** `42` para generar entradas idénticas en las cuatro implementaciones
- **Validación:** cada ejecución genera un checksum para comprobar que todas las implementaciones producen el mismo resultado

## Lenguajes implementados

- C++
- Go
- Java
- Python (sin NumPy)

## Estructura del repositorio

```text
.
├── README.md
├── src/
│   ├── cpp/
│   ├── go/
│   ├── java/
│   └── python/
├── docker/
│   ├── Dockerfile.cpp
│   ├── Dockerfile.go
│   ├── Dockerfile.java
│   ├── Dockerfile.python
│   ├── Dockerfile.all
│   ├── run_container.sh
│   └── run_lang.sh
├── Snakefile
├── config.yaml
├── requirements.txt
├── requirements-lock.txt     # dependencias exactas para Python 3.12
├── LICENSE                 # código, licencia MIT
├── LICENSE-DATA.md         # resultados y documentación, CC BY 4.0
├── DATASHEET.md            # mini datasheet FAIR
├── CITATION.cff            # metadatos de citación
├── REVIEW-CROSS.md         # formulario de evaluación cruzada
├── .dockerignore           # evita enviar resultados/binarios al build context
├── scripts/
│   ├── run_all.sh
│   ├── gen_order.py
│   ├── record_env.py
│   ├── verify.py
│   ├── summarize.py
│   ├── run_container.py
│   ├── merge_results.py
│   ├── validate_results.py
│   └── analyze.py
├── results/                  # CSV, análisis, gráfico y logs generados
├── results_semana2_windows/
└── .gitignore
```

## Requisitos

- C++17
- Go 1.21+
- JDK 17+
- Python 3.8+
- Python 3.12 en el host para Snakemake y el análisis estadístico (entorno validado)
- Docker Desktop o Docker Engine
- WSL2 con Linux para ejecutar Snakemake en Windows

## Ejecución rápida

Desde la carpeta `mm/`:

```bash
./scripts/run_all.sh
```

También puede ejecutarse con tamaños reducidos:

```bash
SIZES="100 200" REPS=3 ./scripts/run_all.sh
```

## Construcción manual de imágenes Docker

Ejecuta estos comandos desde la carpeta `mm/`. Se construyen imágenes locales con los Dockerfiles del artifact; no descargues imágenes del benchmark desde un registry:

```bash
docker build --pull -f docker/Dockerfile.cpp -t matmul-cpp .
docker build --pull -f docker/Dockerfile.go -t matmul-go .
docker build --pull -f docker/Dockerfile.python -t matmul-python .
docker build --pull -f docker/Dockerfile.java -t matmul-java .
```

`--pull` actualiza la imagen base declarada por cada Dockerfile; cada imagen del experimento se construye localmente. Snakemake comprueba que las cuatro imágenes locales existan y usa `docker run --pull=never`, por lo que nunca descarga ni construye esas imágenes.

## Pipeline automatizado con Snakemake

Instala las dependencias del host una vez. Se probaron Python 3.12.10 y las versiones exactas de `requirements-lock.txt`. En Windows, usa Docker Desktop en modo Linux:

```powershell
py -3.12 -m pip install -r requirements-lock.txt
```

En Linux o WSL2 con Python 3.12, instala los mismos requisitos con `python3 -m pip install -r requirements-lock.txt`. `requirements.txt` contiene las dependencias directas; para reproducir el entorno probado, prefiere el lockfile. Una vez construidas manualmente las cuatro imágenes, ejecuta desde `mm/`:

```bash
snakemake --cores 1
```

En Windows también puedes ejecutar Snakemake desde PowerShell, como en la ejecución validada; si OneDrive impide crear o limpiar los bloqueos de Snakemake, usa WSL2 con Docker Desktop integrado y, de ser posible, una copia del artifact dentro del filesystem de WSL. `--cores 1` ejecuta los contenedores secuencialmente para evitar que compitan por CPU durante la medición. El workflow configura tamaños `500`, `1000` y `1500`, 10 repeticiones y semilla `42` en [config.yaml](config.yaml).

Snakemake declara estas dependencias: cada imagen genera su CSV y checksum; los cuatro CSV se validan e integran en `results/benchmark.csv`; el workflow comprueba estructura, repeticiones y checksums; finalmente ejecuta ANOVA factorial de dos factores con interacción, Tukey HSD cuando corresponde y genera tablas y gráfico.

El script nativo `scripts/run_all.sh` aleatoriza los bloques con la semilla `42`. El workflow Docker de Snakemake ejecuta cada lenguaje como un trabajo independiente y cada contenedor recorre los tamaños en el orden configurado; no aplica aleatorización de bloques.

Resultados automatizados en `results/`:

- `cpp.csv`, `go.csv`, `python.csv`, `java.csv`: mediciones por lenguaje.
- `benchmark.csv` y `checksums.txt`: datos integrados y checksums.
- `descriptive.csv`, `anova.csv`, `tukey_hsd.csv` y `analysis.md`: resultados estadísticos.
- `benchmark_by_language.png`: gráfico de cajas por lenguaje y tamaño.
- `validated.ok`: marca de que pasó la validación del experimento.
- `logs/`: salida de los contenedores, la integración, la validación y el análisis.

Si modificas tamaños, repeticiones, semilla, imágenes o α, edita `config.yaml`. Para reconstruir todos los resultados desde cero, usa `snakemake --cores 1 --forceall`. Esto vuelve a ejecutar las mediciones, que pueden tardar más de una hora debido a Python.

Si cambias una implementación o su Dockerfile, reconstruye manualmente la imagen correspondiente con el comando `docker build` anterior antes de volver a ejecutar Snakemake.

### Evaluación de los requisitos de la semana 4

| Requisito | Evidencia en el artifact |
|---|---|
| Ejecutar las imágenes locales de C++, Go, Python y Java, construidas manualmente desde sus Dockerfiles | `run_language_container` verifica las imágenes locales y las ejecuta con `docker run --pull=never`; el README documenta los cuatro comandos `docker build`. |
| Ejecutar N = 500, 1000 y 1500 con 10 repeticiones por combinación | Valores por defecto en `config.yaml`; validación de 30 filas por lenguaje y tamaño. |
| Generar cuatro CSV independientes e integrarlos en `benchmark.csv` | `run_language_container` genera cada CSV; `merge_results` los valida e integra automáticamente. |
| Analizar ambos factores y su interacción mediante ANOVA de dos factores | `analyze.py` ajusta `time_seconds ~ C(language) * C(N)` y exporta `anova.csv`. |
| Aplicar Tukey HSD cuando corresponde y producir tablas y gráficos | El análisis evalúa significancia; con interacción significativa compara lenguajes dentro de cada N y tamaños dentro de cada lenguaje. Produce `tukey_hsd.csv`, tablas descriptivas y ANOVA, reporte Markdown y gráfico PNG. |
| Declarar dependencias y reproducir con un comando | `Snakefile` define generación → integración → validación → análisis; tras la construcción manual de imágenes basta `snakemake --cores 1`. |

La ejecución completa disponible en `results/` se validó: `benchmark.csv` contiene 120 mediciones (30 por lenguaje) y los checksums coinciden entre lenguajes para los tres tamaños. El ANOVA detectó efectos de lenguaje (p = 3,74 × 10⁻¹⁵³), tamaño (p = 5,04 × 10⁻¹²⁹) e interacción lenguaje × tamaño (p = 2,90 × 10⁻¹⁴⁸). Dada la interacción, el reporte presenta las comparaciones de Tukey por estrato. En esta ejecución Python fue significativamente más lento que los otros lenguajes en cada tamaño; las diferencias entre C++, Go y Java no fueron significativas en los estratos por tamaño.

Esta evaluación confirma el cumplimiento funcional de los requisitos de automatización y análisis de la semana 4, y facilita la reutilización del artifact; no constituye una certificación oficial de ACM. La pauta de esta semana no solicita pruebas de supuestos del ANOVA; por tanto, el análisis generado no incluye diagnósticos de normalidad de residuos ni homogeneidad de varianzas. Las inferencias deben leerse teniendo presente esa limitación.

## Semana 5: versión archivada y evaluación cruzada

### Contenido de esta versión

La versión 1.0.0 incluye el código fuente de los cuatro lenguajes, los Dockerfiles, el workflow y su configuración, dependencias fijadas, scripts de validación/análisis, CSV individuales, `benchmark.csv`, checksums, registro del entorno, resultados estadísticos, tablas y gráfico. También incluye [LICENSE](LICENSE) (MIT para el software), [LICENSE-DATA.md](LICENSE-DATA.md) (CC BY 4.0 para datos/documentación), [DATASHEET.md](DATASHEET.md) (descripción FAIR), [CITATION.cff](CITATION.cff) y [REVIEW-CROSS.md](REVIEW-CROSS.md). Las licencias de Ubuntu y demás componentes de terceros no cambian.

**DOI de Zenodo:** pendiente de crear y publicar el depósito. No cites un DOI hasta que Zenodo confirme la publicación. Cuando exista, añade el DOI y el enlace del registro a este README, a `CITATION.cff` y al README principal del repositorio GitHub.

### Reproducción independiente para el evaluador

1. Descarga la versión etiquetada/archivada o clona el repositorio y cambia a la carpeta `mm/`.
2. Instala Docker Desktop o Docker Engine con soporte para contenedores Linux; en Windows puedes usar WSL2. Instala Python 3.12.
3. Construye manualmente las cuatro imágenes desde esta carpeta con los cuatro comandos de la sección anterior. No descargues imágenes `matmul-*` de ningún registry.
4. Instala las dependencias fijadas y ejecuta el workflow:

   ```bash
   python3 -m pip install -r requirements-lock.txt
   snakemake --cores 1
   ```

   En PowerShell, usa `py -3.12 -m pip install -r requirements-lock.txt` para instalar y luego `snakemake --cores 1`. Verifica primero `docker info` si Docker no responde.
5. Comprueba que se creen `results/cpp.csv`, `go.csv`, `python.csv`, `java.csv`, `benchmark.csv`, `analysis.md`, `anova.csv`, `tukey_hsd.csv`, `descriptive.csv` y `benchmark_by_language.png`. La validación de filas y checksums se ejecuta automáticamente; el workflow falla explícitamente si algo no coincide.
6. Compara los tiempos con los resultados publicados: los tiempos exactos pueden variar entre máquinas. Comprueba los tamaños, recuentos, estructura, checksums y patrones estadísticos; registra las diferencias, mensajes de error, dependencias faltantes y cambios en [REVIEW-CROSS.md](REVIEW-CROSS.md).

La primera reproducción puede tardar más de una hora, principalmente por Python con N=1500. `snakemake --cores 1 --forceall` fuerza una nueva medición completa. Para verificar únicamente la planificación, sin correr contenedores, usa `snakemake --cores 1 --dry-run`.

El depósito de Zenodo solo se considerará publicado tras completar la publicación en Zenodo y obtener su DOI. La revisión cruzada por un estudiante externo tampoco se considera terminada hasta completar y guardar el registro de [REVIEW-CROSS.md](REVIEW-CROSS.md).

## Resultados esperados

Los archivos CSV generados contienen columnas como:

```csv
language,N,repetition,time_seconds
cpp,500,1,0.078512345
```

Se generan además archivos de checksum y resúmenes de validación.

## Reproducibilidad

El proyecto está diseñado para ser reproducible y reutilizable bajo los principios de ACM Artifact Review and Badging. Se registran:

- entorno de ejecución
- versiones de software
- semilla y datos de entrada reproducibles; el orden aleatorio aplica al script nativo, no al flujo Docker de Snakemake
- validación de resultados por checksum
- estructura modular del experimento

## Observaciones relevantes

- Python es mucho más lento que C++ en la implementación naive, aun cuando el algoritmo es idéntico.
- El comportamiento cambia según `N`, y hay interacción entre lenguaje y tamaño de la matriz.
- La comparación debe hacerse dentro del mismo entorno experimental para evitar sesgos por hardware o sistema operativo.

## Licencia

El software se distribuye bajo [MIT](LICENSE). Los resultados y la documentación se ofrecen bajo [CC BY 4.0](LICENSE-DATA.md). Revisa esos archivos para conocer las condiciones de reutilización.

## Referencias

- Association for Computing Machinery. *Artifact Review and Badging.*
- Zenodo. *Research data repository*. https://zenodo.org/

---

Este repositorio permite comparar rendimiento, reproducibilidad y consistencia de resultados entre lenguajes distintos usando el mismo algoritmo y condiciones experimentales.
