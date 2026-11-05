# Benchmark reproducible de multiplicación de matrices NxN en C++, Go, Java y Python

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23954259.svg)](https://doi.org/10.5281/zenodo.23954259)

> **Versión archivada (v1.0.0):** [https://zenodo.org/records/23954259](https://zenodo.org/records/23954259) · DOI [10.5281/zenodo.23954259](https://doi.org/10.5281/zenodo.23954259)

Artifact científico del **Módulo 2 – Documentación de experimentos (Reproducibilidad y
Replicabilidad)**, curso *Experimentación Computacional* (Doctorado, USACH), construido según
la política [ACM Artifact Review and Badging v1.1](https://www.acm.org/publications/policies/artifact-review-and-badging-current).

Este README es la **guía completa del experimento**, escrita para alguien que llega por primera
vez. Recorre la actividad en el orden en que se construyó (semana 1 a semana 5) y, en cada
semana, explica **qué se pide, qué se hizo, cómo ejecutarlo paso a paso en Linux y en Windows y
qué resultado se debe obtener**.

| Estado del artifact (autoevaluación ACM) | Semana | |
|---|---|---|
| Artifacts Evaluated – **Functional** | 3 | ✅ autoevaluado ([§5.6](#56-autoevaluación-acm-functional)) |
| Artifacts Evaluated – **Reusable** | 4 | ✅ autoevaluado ([§6.6](#66-autoevaluación-acm-reusable)) |
| Artifacts **Available** | 5 | ✅ Zenodo, DOI [10.5281/zenodo.23954259](https://doi.org/10.5281/zenodo.23954259) ([§7](#7-semana-5-publicación-en-zenodo-y-evaluación-cruzada)) |
| Results **Reproduced** | 5 | ⏳ evaluación cruzada por un compañero ([§7.4](#74-evaluación-cruzada-reproducir-el-artifact-de-otra-persona)) |

> La autoevaluación no equivale a un badge oficial: los badges ACM los otorga el comité de
> evaluación de artifacts de cada conferencia o revista.

## Inicio rápido

Requiere Docker (o Podman) y Python ≥ 3.11. Detalle de la instalación en [§2](#2-requisitos-e-instalación).

> **¿Descargaste el artifact desde Zenodo?** Descomprime el zip y entra a la carpeta
> `Modulo2-Inv-experimental-v1.0.0/` en lugar de hacer `git clone`; el resto de los comandos es
> idéntico. El zip archivado contiene exactamente la etiqueta `v1.0.0` de GitHub.

**Linux / macOS**

```bash
git clone https://github.com/Pmiqueles-usach/Modulo2-Inv-experimental.git
cd Modulo2-Inv-experimental
scripts/build_images.sh                                          # construye las 5 imágenes
python3 -m venv .venv && .venv/bin/pip install -r requirements-host.txt
.venv/bin/snakemake --config results_dir=results_smoke 'sizes=[50,100]' repetitions=3   # prueba (<1 min)
.venv/bin/snakemake --config results_dir=results_repro                                  # completo (~2,5 h)
```

**Windows (PowerShell + Docker Desktop)**

```powershell
git clone https://github.com/Pmiqueles-usach/Modulo2-Inv-experimental.git
cd Modulo2-Inv-experimental
powershell -ExecutionPolicy Bypass -File scripts\reproducir.ps1 -Rapido   # prueba
powershell -ExecutionPolicy Bypass -File scripts\reproducir.ps1           # completo (~2,5 h)
```

## Índice

1. [Cómo usar esta guía: reproducir o replicar](#1-cómo-usar-esta-guía-reproducir-o-replicar)
2. [Requisitos e instalación](#2-requisitos-e-instalación)
3. [Semana 1: diseño experimental y documentación](#3-semana-1-diseño-experimental-y-documentación)
4. [Semana 2: implementación y ejecución local](#4-semana-2-implementación-y-ejecución-local)
5. [Semana 3: entornos reproducibles con Docker](#5-semana-3-entornos-reproducibles-con-docker)
6. [Semana 4: automatización con Snakemake y análisis estadístico](#6-semana-4-automatización-con-snakemake-y-análisis-estadístico)
7. [Semana 5: publicación en Zenodo y evaluación cruzada](#7-semana-5-publicación-en-zenodo-y-evaluación-cruzada)
8. [Resultados oficiales](#8-resultados-oficiales)
9. [Cómo replicar el experimento desde cero](#9-cómo-replicar-el-experimento-desde-cero)
10. [Reutilización y extensión](#10-reutilización-y-extensión)
11. [Solución de problemas](#11-solución-de-problemas)
12. [Licencia, cita y referencias](#12-licencia-cita-y-referencias)

---

## 1. Cómo usar esta guía: reproducir o replicar

ACM (v1.1, terminología armonizada con NISO) distingue tres situaciones:

| Término | Quién ejecuta | Con qué | Qué hacer en esta guía |
|---|---|---|---|
| **Repetir** (*repeatability*) | el mismo autor, mismo equipo | los artifacts del autor | §6.4 con `--forceall` |
| **Reproducir** (*reproducibility*) | otra persona | **los artifacts de este repositorio** (código, Dockerfile, Snakefile) | §2 → §5 → §6 → comparar con §8 |
| **Replicar** (*replicability*) | otra persona | **su propia implementación**, siguiendo el protocolo | §3 y §4 (diseño y protocolo) → §9 |

Rutas sugeridas según el objetivo:

- **Solo quiero ver que funciona (≈10 min):** §2 → §5.4 (contenedores) → §6.4 paso 1 (prueba rápida).
- **Quiero reproducir el experimento completo:** §2 → §6.4 (pipeline completo) → §7.4 (comparación).
- **Quiero replicarlo con mi propio código:** §3 y §4.2 (protocolo) → §9.
- **Quiero entender la actividad semana a semana:** leer §3 a §7 en orden.

**Convenciones de los comandos.** Todos se ejecutan desde la **raíz del repositorio** (la carpeta
que contiene este README). Los bloques `bash` son para Linux/macOS y los bloques `powershell`
para Windows. Donde dice `docker` puede usarse `podman` sin cambios (ver §2.1).

---

## 2. Requisitos e instalación

No se instalan compiladores ni librerías en el equipo: **todo el experimento corre dentro de
contenedores**. En el equipo solo se necesita:

| Herramienta | Versión probada | Para qué |
|---|---|---|
| Docker **o** Podman | Docker 29.4 / Podman 5.8 | construir y ejecutar los contenedores |
| Git | cualquiera reciente | descargar el artifact |
| Python | ≥ 3.11 | solo para instalar Snakemake |
| Snakemake | 9.27.0 (fijado en `requirements-host.txt`) | pipeline de la semana 4 |

Hardware: CPU x86_64, ≥ 4 GB de RAM libre y ~5 GB de disco para las imágenes. El experimento
completo tarda ≈ 2,5 h (dominado por Python con N = 1500); la prueba rápida, menos de 1 minuto.

Probado en: Fedora 42 (Linux 6.18) con Docker 29.4 y con Podman 5.8. El soporte de Windows
(Docker Desktop + PowerShell) está implementado y documentado en esta guía.

### 2.1 Linux

```bash
# Fedora / RHEL
sudo dnf install -y git python3 docker        # o: podman
sudo systemctl enable --now docker
# Debian / Ubuntu
sudo apt install -y git python3 python3-venv docker.io

# Para usar docker sin sudo (cerrar sesión y volver a entrar después):
sudo usermod -aG docker "$USER"
```

Si el usuario no pertenece al grupo `docker`, hay tres opciones equivalentes:

| Opción | Construir imágenes | Pipeline |
|---|---|---|
| Docker con sudo | `ENGINE="sudo docker" scripts/build_images.sh` | `--config engine="sudo docker"` |
| Podman | `ENGINE=podman scripts/build_images.sh` | `--config engine=podman` |
| Podman sin sesión systemd (error `sd-bus call ... Input/output error`) | `ENGINE="podman --cgroup-manager=cgroupfs" scripts/build_images.sh` | `--config engine="podman --cgroup-manager=cgroupfs"` |

### 2.2 Windows 10/11

| Herramienta | Dónde | Nota |
|---|---|---|
| Docker Desktop | https://www.docker.com/products/docker-desktop/ | Dejar marcado **"Use WSL 2"**. Reiniciar si lo pide. |
| Git para Windows | https://git-scm.com/download/win | Opciones por defecto. |
| Python ≥ 3.11 | https://www.python.org/downloads/ | Marcar **"Add python.exe to PATH"**. |

Abra **Docker Desktop** y espere a que indique *Engine running*. Los scripts `.ps1` se ejecutan
con `powershell -ExecutionPolicy Bypass -File ...`, que no cambia la configuración del equipo.

### 2.3 Comprobar la instalación

```bash
docker --version && docker run --rm hello-world      # "Hello from Docker!"
git --version
python3 --version                                    # ≥ 3.11
```

```powershell
docker --version; docker run --rm hello-world        # "Hello from Docker!"
git --version
py --version                                         # o: python --version  (≥ 3.11)
```

---

## 3. Semana 1: diseño experimental y documentación

### 3.1 Lo que pide el módulo

- Crear un repositorio GitHub como espacio de desarrollo, documentación y control de versiones.
- Versión inicial del `README.md` con objetivo, pregunta de investigación, H₀/H₁, variables y
  diseño experimental.
- Presentar la propuesta (10 min) con comentarios sobre ACM Artifact Review & Badging
  ([docs/presentaciones/](docs/presentaciones/)).

**Objetivo del experimento.** Diseñar y ejecutar un experimento computacional **reproducible**
que compare el tiempo de ejecución del algoritmo **naive** de multiplicación de matrices
cuadradas NxN implementado de forma equivalente en **C++, Go, Java y Python**.

### 3.2 Pregunta de investigación

> ¿Existen diferencias estadísticamente significativas en el tiempo de ejecución del algoritmo
> naive de multiplicación de matrices NxN entre implementaciones equivalentes desarrolladas en
> C++, Go, Java y Python, y cómo varían estas diferencias según el tamaño de la matriz?

### 3.3 Hipótesis

- **H₀:** El lenguaje de implementación no produce diferencias estadísticamente significativas
  en el tiempo de ejecución del algoritmo de multiplicación de matrices.
- **H₁:** Al menos uno de los lenguajes presenta un tiempo de ejecución significativamente
  diferente de los demás.
- **Complementaria (interacción):** el efecto del lenguaje depende del tamaño N.

### 3.4 Variables

| Tipo | Variable | Niveles / unidad |
|---|---|---|
| Independiente (factor A) | Lenguaje de programación | C++, Go, Java, Python |
| Independiente (factor B) | Tamaño de la matriz N | 500, 1000, 1500 |
| Dependiente | Tiempo de ejecución de la multiplicación | segundos (`time_seconds`) |
| Controladas | Algoritmo naive i-j-k, tipo `double`, mismas matrices (SplitMix64, semilla 42), reloj monotónico, solo se mide la multiplicación, ejecución secuencial en contenedores | — |
| Registradas | CPU, RAM, SO, arquitectura, versiones de compiladores/VM | [results/environment/environment.md](results/environment/environment.md) |

### 3.5 Diseño experimental

Diseño **factorial completo 4 × 3** con **10 repeticiones** por celda → **120 mediciones**.

| Lenguaje | N = 500 | N = 1000 | N = 1500 | Total |
|---|---|---|---|---|
| C++ | 10 | 10 | 10 | 30 |
| Go | 10 | 10 | 10 | 30 |
| Java | 10 | 10 | 10 | 30 |
| Python | 10 | 10 | 10 | 30 |

Análisis previsto: **ANOVA de dos factores con interacción** (lenguaje, N, lenguaje × N) y
**Tukey HSD** post-hoc; verificación de supuestos (Shapiro-Wilk, Levene); como robustez, el
mismo modelo sobre log10(tiempo) y Kruskal-Wallis por N.

### 3.6 Estructura del repositorio

```
.
├── README.md                  ← esta guía
├── LEEME.md                   ← versión mínima: compilar y ejecutar cada programa a mano
├── LICENSE / LICENSE-DATA     ← MIT (código) / CC BY 4.0 (datos y resultados)
├── CITATION.cff, .zenodo.json ← metadatos de cita y de publicación en Zenodo
├── CHANGELOG.md               ← historia de versiones por semana
├── .gitattributes             ← finales de línea LF fijos (clonado correcto en Windows)
├── requirements-host.txt      ← Snakemake (única dependencia del equipo)
├── config/config.yaml         ← parámetros: N, repeticiones, semilla, motor, directorio
├── src/{cpp,go,java,python}/  ← las 4 implementaciones equivalentes            (semana 2)
├── docker/<lang>/Dockerfile   ← entornos reproducibles + docker/analysis        (semana 3)
├── workflow/Snakefile         ← pipeline de automatización                      (semana 4)
├── workflow/profiles/default/ ← perfil Snakemake (ejecución secuencial)
├── scripts/                   ← construcción, ejecución, integración, análisis, verificación
│                                 (*.sh Linux/macOS · *.ps1 Windows · *.py ambos)
├── results/                   ← resultados OFICIALES (versionados)
│   ├── raw/{cpp,go,java,python}.csv      ← un CSV por lenguaje
│   ├── benchmark.csv                     ← CSV integrado (120 filas)
│   ├── verification/                     ← checksums y reporte de equivalencia
│   ├── analysis/{tables,figures}/        ← ANOVA, Tukey, tablas y gráficos
│   ├── analysis/reporte_estadistico.md
│   ├── environment/environment.md        ← hardware y versiones
│   └── logs/
└── docs/
    ├── PROTOCOLO.md           ← protocolo y justificación de equivalencia      (semana 2)
    ├── EXPLICACION_CODIGO.md  ← el código explicado parte por parte
    ├── ACM_BADGES.md          ← autoevaluación ACM detallada
    ├── DATASHEET_FAIR.md      ← descripción FAIR de los datos                   (semana 5)
    ├── ZENODO.md              ← procedimiento de publicación                    (semana 5)
    ├── evaluacion-cruzada/    ← plantilla de evaluación cruzada                 (semana 5)
    ├── presentaciones/        ← presentaciones de las semanas 1 y 2
    └── semanas/               ← enlaces a cada semana de esta guía
```

Los directorios `results_*` (`results_smoke`, `results_demo`, `results_repro`, …) son de pruebas
y no se versionan: **nunca se sobrescribe `results/`** con una prueba.

### 3.7 Paso a paso

**A. Obtener el artifact (para reproducirlo).**

```bash
git clone https://github.com/Pmiqueles-usach/Modulo2-Inv-experimental.git
cd Modulo2-Inv-experimental
git tag -n1                                  # una etiqueta por semana: semana-1 … semana-4, v1.0.0
git log --oneline --decorate | tail -n 12    # historia del desarrollo
ls src docker workflow scripts results
```

```powershell
git clone https://github.com/Pmiqueles-usach/Modulo2-Inv-experimental.git
cd Modulo2-Inv-experimental
git tag -n1
git log --oneline --decorate
Get-ChildItem src, docker, workflow, scripts, results
```

También puede descargarse la versión archivada desde Zenodo (DOI al inicio de esta guía),
descomprimirla y abrir la terminal dentro de la carpeta. `git checkout semana-N` muestra el
estado del artifact al cierre de cada semana.

**B. Crear un artifact propio con la misma organización (para replicarlo).**

```bash
# 1) Crear en https://github.com/new un repositorio vacío y clonarlo
git clone https://github.com/<usuario>/<repositorio>.git && cd <repositorio>
# 2) Estructura inicial
mkdir -p src/{cpp,go,java,python} docker results docs
# 3) Escribir README.md con objetivo, pregunta, hipótesis, variables y diseño (§3.1–3.5)
git add . && git commit -m "docs(semana-1): README inicial con diseño experimental"
git tag -a semana-1 -m "Entregable Semana 1"
git push origin main --tags
```

### 3.8 Comentarios sobre ACM Artifact Review & Badging

1. **Badges independientes:** *Artifacts Evaluated* (Functional / Reusable), *Artifacts
   Available* y *Results Validated* (Reproduced / Replicated). El módulo los recorre en orden:
   Functional (semana 3) → Reusable (semana 4) → Available + Reproduced (semana 5).
2. **Functional** exige artifacts *documented, consistent, complete, exercisable* y evidencia de
   verificación. Por eso se diseñó desde el inicio un **checksum** de la matriz resultado que
   permite comprobar que las 4 implementaciones calculan exactamente lo mismo.
3. **Available** exige un repositorio de **archivo** con DOI: GitHub no basta, de ahí Zenodo.
4. **Results Reproduced** no exige igualdad exacta, sino concordancia que no cambie las
   conclusiones. En un benchmark de tiempos los segundos absolutos cambian con el hardware: lo
   que debe reproducirse son las **decisiones estadísticas** y el **orden de los lenguajes**.
5. **Crítica:** los badges evalúan el artifact, no la validez del diseño; un benchmark mal
   controlado podría ser "Reusable". Por eso [docs/PROTOCOLO.md](docs/PROTOCOLO.md) documenta
   además las amenazas a la validez.

El README original de la semana 1 se conserva en
[docs/semanas/README_semana1_original.md](docs/semanas/README_semana1_original.md).

---

## 4. Semana 2: implementación y ejecución local

### 4.1 Lo que pide el módulo

- Implementar el algoritmo naive NxN en C++, Go, Java y Python, funcionalmente equivalentes
  (mismo algoritmo, tipo de dato, matrices de entrada y criterio de medición), sin librerías
  optimizadas.
- Matrices generadas con semilla definida; N = 500, 1000, 1500; 10 repeticiones; medir solo la
  multiplicación (las matrices se crean antes de iniciar el temporizador).
- Resultados en `cpp.csv`, `go.csv`, `java.csv`, `python.csv` con las columnas
  `language,N,repetition,time_seconds`.
- Registrar el entorno de ejecución (CPU, RAM, SO, arquitectura, versiones).

### 4.2 Protocolo de equivalencia

Resumen del protocolo completo de [docs/PROTOCOLO.md](docs/PROTOCOLO.md). Estas reglas son las
que hay que respetar para **replicar** el experimento (§9).

| Aspecto | Decisión | Por qué |
|---|---|---|
| Algoritmo | Triple bucle **i-j-k** con acumulador local; sin BLAS/NumPy, sin reordenar bucles, sin paralelismo | Mismo trabajo en los 4 lenguajes |
| Datos | `double` IEEE-754 de 64 bits en un **arreglo plano row-major** de N×N | Mismo tipo y patrón de acceso a memoria |
| Generador | **SplitMix64** implementado igual en los 4 lenguajes; real en [0,1) = `(x >> 11) × 2⁻⁵³`; semilla 42 reiniciada para cada N; primero A y luego B | El RNG nativo de cada lenguaje daría matrices distintas con la misma semilla |
| Matriz C | Reservada **antes** del temporizador y reutilizada | La reserva de memoria no es parte de la multiplicación |
| Reloj | Monotónico: `steady_clock` (C++), `time.Since` (Go), `System.nanoTime` (Java), `perf_counter_ns` (Python) | Alta resolución, inmune a cambios de hora |
| Medición | El temporizador envuelve **solo** la llamada a `multiplicar` | Generación, checksum y escritura quedan fuera |
| Sin FMA | `-ffp-contract=off` (C++), `float64(a*b)` (Go); Java ≥ 17 y CPython no fusionan | Resultados **bit a bit idénticos** entre lenguajes |
| Verificación | Tras cada repetición (fuera del temporizador) se guarda la suma de C, `C[0,0]` y `C[N-1,N-1]` | Prueba objetiva de equivalencia; impide eliminar el cálculo como código muerto |
| Optimización | La habitual de cada ecosistema: `g++ -O2`, `go build`, HotSpot JIT, CPython | Comparar implementaciones típicas, no la mejor posible |
| Calentamiento JIT | La repetición 1 de Java **se conserva** y se analiza | No se descartan datos |

Pseudocódigo común:

```
para i en 0..N-1:
  para j en 0..N-1:
    suma = 0.0
    para k en 0..N-1:
      suma += A[i*N + k] * B[k*N + j]
    C[i*N + j] = suma
```

### 4.3 Interfaz común de los cuatro programas

```
<programa> <salida.csv> <tamaños> <repeticiones> <semilla> [checksum.csv]
<programa> results/raw/cpp.csv 500,1000,1500 10 42 results/verification/cpp_checksum.csv
```

Los tamaños van separados por coma y sin espacios. Para cambiar N no se edita el código.

### 4.4 Formato de los datos

`<lang>.csv` y `benchmark.csv`:

| columna | tipo | descripción |
|---|---|---|
| `language` | texto | `cpp`, `go`, `java`, `python` |
| `N` | entero | tamaño de la matriz |
| `repetition` | entero | número de ejecución (1–10) |
| `time_seconds` | real | tiempo de la multiplicación en segundos (9 decimales) |

`<lang>_checksum.csv`: `language,N,repetition,checksum,c_first,c_last` (17 dígitos
significativos). Descripción FAIR completa: [docs/DATASHEET_FAIR.md](docs/DATASHEET_FAIR.md).

### 4.5 Paso a paso (sin contenedores)

Esta etapa usa los compiladores del propio equipo. Si no los tiene instalados, pase
directamente a la semana 3, que hace lo mismo dentro de contenedores.

**Requisitos:** `g++` (MinGW en Windows), Go ≥ 1.20, JDK ≥ 11 y Python ≥ 3.9.

**Linux / macOS** — automático, con tiempos medios y comparación de checksums (≈30 s):

```bash
scripts/semana2.sh                         # N = 200,400 · 3 repeticiones → results_semana2/
scripts/semana2.sh 500,1000,1500 10 42     # experimento completo (~2,5 h)
```

**Linux / macOS** — manual, un lenguaje a la vez:

```bash
mkdir -p results_local
g++ -O2 -std=c++17 -ffp-contract=off -o matmul_cpp src/cpp/matmul.cpp
./matmul_cpp results_local/cpp.csv 200,400 3 42 results_local/cpp_checksum.csv

(cd src/go && go build -o ../../matmul_go .)
./matmul_go results_local/go.csv 200,400 3 42 results_local/go_checksum.csv

java -Xms1g -Xmx1g src/java/MatMul.java results_local/java.csv 200,400 3 42 results_local/java_checksum.csv

python3 src/python/matmul.py results_local/python.csv 200,400 3 42 results_local/python_checksum.csv

head -n 4 results_local/{cpp,go,java,python}.csv
awk -F, 'FNR > 1 && $3 == 1' results_local/*_checksum.csv | sort -t, -k2,2n
```

**Windows (PowerShell)** — manual:

```powershell
New-Item -ItemType Directory -Force results_local | Out-Null
g++ -O2 -std=c++17 -ffp-contract=off -o matmul_cpp.exe src/cpp/matmul.cpp
.\matmul_cpp.exe results_local/cpp.csv 200,400 3 42 results_local/cpp_checksum.csv

cd src/go; go build -o ..\..\matmul_go.exe .; cd ..\..
.\matmul_go.exe results_local/go.csv 200,400 3 42 results_local/go_checksum.csv

java -Xms1g -Xmx1g src/java/MatMul.java results_local/java.csv 200,400 3 42 results_local/java_checksum.csv

py src/python/matmul.py results_local/python.csv 200,400 3 42 results_local/python_checksum.csv

Get-ChildItem results_local\*.csv -Exclude *_checksum.csv | ForEach-Object { "== $($_.Name)"; Get-Content $_ -TotalCount 4 }
Import-Csv (Get-ChildItem results_local\*_checksum.csv) | Where-Object repetition -eq 1 | Sort-Object { [int]$_.N } | Format-Table
```

**Resultado esperado:**

- 4 CSV con la cabecera `language,N,repetition,time_seconds` y 6 filas cada uno (2 tamaños × 3
  repeticiones).
- La columna `checksum` es **idéntica** en los 4 lenguajes para cada N:

| N | checksum esperado |
|---|---|
| 200 | 1998326.3241066386 |
| 400 | 15974925.372778159 |
| 500 | 31283745.652596552 |
| 1000 | 250196210.25040233 |
| 1500 | 844048831.71274 |

Con N = 200 o 400 los tiempos son pequeños y solo sirven para comprobar que todo funciona; el
experimento real se ejecuta en contenedores (semanas 3 y 4).

**Registrar el entorno** (CPU, RAM, SO y versiones; funciona en Linux, macOS y Windows):

```bash
python3 scripts/collect_env.py results_local/environment.md docker
```

```powershell
py scripts\collect_env.py results_local\environment.md docker
```

### 4.6 Entregables

| Entregable | Ubicación |
|---|---|
| Implementaciones | [src/cpp/matmul.cpp](src/cpp/matmul.cpp), [src/go/matmul.go](src/go/matmul.go), [src/java/MatMul.java](src/java/MatMul.java), [src/python/matmul.py](src/python/matmul.py) |
| Protocolo y justificación de la equivalencia | [docs/PROTOCOLO.md](docs/PROTOCOLO.md), [docs/EXPLICACION_CODIGO.md](docs/EXPLICACION_CODIGO.md) |
| CSV por lenguaje (corrida oficial) | [results/raw/](results/raw/) |
| Verificación de equivalencia | [results/verification/equivalence_report.md](results/verification/equivalence_report.md) |
| Registro del entorno | [results/environment/environment.md](results/environment/environment.md) |
| Presentación | [docs/presentaciones/](docs/presentaciones/) |

---

## 5. Semana 3: entornos reproducibles con Docker

### 5.1 Lo que pide el módulo

- Cuatro imágenes Docker (C++, Go, Java, Python) con lo necesario para compilar o ejecutar
  cada implementación, **construidas manualmente**.
- README con instrucciones claras para que un tercero ejecute el experimento hasta este punto.
- Cada ejecución genera su CSV independiente (`cpp.csv`, `go.csv`, `java.csv`, `python.csv`)
  con la estructura común `language,N,repetition,time_seconds`.
- Revisar el artifact con los criterios ACM para alcanzar *Functional*.

### 5.2 Las imágenes

| Imagen | Dockerfile | Imagen base (versión exacta) | Qué hace en el build |
|---|---|---|---|
| `matmul-cpp:1.0` | [docker/cpp/Dockerfile](docker/cpp/Dockerfile) | `gcc:14.2.0` | `g++ -O2 -std=c++17 -ffp-contract=off` |
| `matmul-go:1.0` | [docker/go/Dockerfile](docker/go/Dockerfile) | `golang:1.23.4-bookworm` | `go build` (`CGO_ENABLED=0`) |
| `matmul-java:1.0` | [docker/java/Dockerfile](docker/java/Dockerfile) | `eclipse-temurin:21.0.5_11-jdk` | `javac`; se ejecuta con `java -Xms1g -Xmx1g` |
| `matmul-python:1.0` | [docker/python/Dockerfile](docker/python/Dockerfile) | `python:3.12.8-slim-bookworm` | copia `matmul.py` (sin dependencias) |
| `matmul-analysis:1.0` | [docker/analysis/Dockerfile](docker/analysis/Dockerfile) | `python:3.12.8-slim-bookworm` | numpy, pandas, scipy, statsmodels, matplotlib con versión fija (semana 4) |

### 5.3 Decisiones de diseño de los contenedores

- **Versiones exactas** de la imagen base, nunca `latest`.
- La imagen **compila el código durante el build**: el binario sale siempre del código versionado.
- **Construcción manual** desde los Dockerfile: nunca se descargan imágenes del benchmark ya
  publicadas en un registro; solo las imágenes base oficiales.
- `ENTRYPOINT` = el programa; `CMD` = los parámetros del experimento oficial. Así,
  `docker run matmul-cpp:1.0` sin argumentos ejecuta el experimento completo de ese lenguaje.
- Los resultados salen del contenedor por un **volumen** montado en `/results`.
- `--network none`: el experimento no necesita red.
- En Linux, `--user $(id -u):$(id -g)` (Docker) o `--userns=keep-id` (Podman) para que los CSV
  queden a nombre del usuario, y `:z` en el volumen para SELinux (Fedora/RHEL). En Windows no
  se necesitan.

### 5.4 Paso a paso

**Paso 1 · Ver un Dockerfile.**

```bash
cat docker/cpp/Dockerfile
```

```powershell
Get-Content docker/cpp/Dockerfile
```

**Paso 2 · Construir manualmente las imágenes.** La primera vez se descargan las imágenes base
(~1–2 GB) y tarda varios minutos; después, Docker reutiliza la caché y tarda segundos.

```bash
docker build -f docker/cpp/Dockerfile      -t matmul-cpp:1.0 .
docker build -f docker/go/Dockerfile       -t matmul-go:1.0 .
docker build -f docker/java/Dockerfile     -t matmul-java:1.0 .
docker build -f docker/python/Dockerfile   -t matmul-python:1.0 .
docker build -f docker/analysis/Dockerfile -t matmul-analysis:1.0 .   # se usa en la semana 4
docker images --filter 'reference=*matmul-*'
# Equivalente en un solo comando: scripts/build_images.sh   (ENGINE="sudo docker" o ENGINE=podman, ver §2.1)
```

```powershell
docker build -f docker/cpp/Dockerfile      -t matmul-cpp:1.0 .
docker build -f docker/go/Dockerfile       -t matmul-go:1.0 .
docker build -f docker/java/Dockerfile     -t matmul-java:1.0 .
docker build -f docker/python/Dockerfile   -t matmul-python:1.0 .
docker build -f docker/analysis/Dockerfile -t matmul-analysis:1.0 .
docker images --filter "reference=matmul-*"
# Equivalente: powershell -ExecutionPolicy Bypass -File scripts\build_images.ps1
```

**Paso 3 · Ejecutar los 4 contenedores (versión corta: N = 200, 400 y 3 repeticiones, ≈20 s).**

```bash
mkdir -p results_demo/raw results_demo/verification
for lang in cpp go java python; do
  docker run --rm --user "$(id -u):$(id -g)" --network none -v "$PWD/results_demo:/results:z" \
    matmul-$lang:1.0 /results/raw/$lang.csv 200,400 3 42 /results/verification/${lang}_checksum.csv
done
# Un solo lenguaje: scripts/run_container.sh java 200,400 3 42 results_demo
```

```powershell
New-Item -ItemType Directory -Force results_demo\raw, results_demo\verification | Out-Null
foreach ($lang in "cpp", "go", "java", "python") {
  docker run --rm --network none -v "${PWD}\results_demo:/results" matmul-${lang}:1.0 `
    /results/raw/$lang.csv 200,400 3 42 /results/verification/${lang}_checksum.csv
}
# Un solo lenguaje: powershell -ExecutionPolicy Bypass -File scripts\run_container.ps1 java 200,400 3 42 results_demo
```

Para el **experimento oficial de un lenguaje** (N = 500, 1000, 1500 · 10 repeticiones) basta
con no pasar argumentos: `docker run --rm --network none -v "$PWD/results_repro:/results:z" matmul-cpp:1.0`
(crear antes `results_repro/raw` y `results_repro/verification`).

**Paso 4 · Comprobar los 4 CSV independientes.**

```bash
ls -l results_demo/raw/
head -n 4 results_demo/raw/*.csv
```

```powershell
Get-ChildItem results_demo\raw
Get-ChildItem results_demo\raw\*.csv | ForEach-Object { "== $($_.Name)"; Get-Content $_ -TotalCount 4 }
```

**Paso 5 · Comprobar que los 4 lenguajes calcularon la misma matriz.**

```bash
awk -F, 'FNR > 1 && $3 == 1' results_demo/verification/*_checksum.csv | sort -t, -k2,2n
```

```powershell
Import-Csv (Get-ChildItem results_demo\verification\*_checksum.csv) | Where-Object repetition -eq 1 | Sort-Object { [int]$_.N } | Format-Table
```

**Resultado esperado:** 4 archivos `cpp.csv`, `go.csv`, `java.csv`, `python.csv` con la misma
cabecera, y el mismo checksum en los 4 lenguajes (1998326.3241066386 para N = 200 y
15974925.372778159 para N = 400).

En Linux, `scripts/demo.sh 3` hace los pasos 1 a 5 de forma guiada, con pausas, para una
demostración en clase ([docs/DEMO.md](docs/DEMO.md)).

### 5.5 Entregables

| Entregable | Ubicación |
|---|---|
| 4 Dockerfile + imagen de análisis | [docker/](docker/) |
| Construcción manual | [scripts/build_images.sh](scripts/build_images.sh), [scripts/build_images.ps1](scripts/build_images.ps1) |
| Ejecución manual por contenedor | [scripts/run_container.sh](scripts/run_container.sh), [scripts/run_container.ps1](scripts/run_container.ps1) |
| Instrucciones para terceros | esta sección |

### 5.6 Autoevaluación ACM: Functional

| Criterio ACM | Cumple | Evidencia |
|---|---|---|
| **Documented** | ✅ | Inventario (§3.6), protocolo (§4.2) e instrucciones paso a paso (§5.4) |
| **Consistent** | ✅ | Cada componente participa en la cadena código → contenedor → CSV → análisis |
| **Complete** | ✅ | Código, Dockerfile, scripts y configuración incluidos; solo se descargan imágenes base oficiales con versión fija |
| **Exercisable** | ✅ | Los 4 contenedores se ejecutan y generan los 4 CSV (§5.4) |
| **Evidencia de verificación** | ✅ | Checksums idénticos bit a bit entre lenguajes ([equivalence_report.md](results/verification/equivalence_report.md)) |

Detalle: [docs/ACM_BADGES.md §2.1](docs/ACM_BADGES.md).

---

## 6. Semana 4: automatización con Snakemake y análisis estadístico

### 6.1 Lo que pide el módulo

- Pipeline Snakemake que coordine la ejecución de los 4 contenedores (imágenes construidas
  manualmente; prohibido usar imágenes publicadas en registros).
- N = 500, 1000, 1500; 10 repeticiones; generar los 4 CSV e integrarlos en `benchmark.csv`.
- Ejecutar automáticamente el análisis estadístico (ANOVA de dos factores con interacción y
  Tukey HSD) y generar tablas y gráficos.
- Dependencias explícitas entre etapas; un único comando o una secuencia mínima documentada.
- Revisión ACM orientada a *Reusable*.

### 6.2 El pipeline

```
run_benchmark(cpp|go|java|python) ─► raw/<lang>.csv ─► merge ─► benchmark.csv ─► analysis ─► tablas, figuras, reporte
                                  └─► verification/<lang>_checksum.csv ─► verify_equivalence ─► equivalence_report.md
environment ─► environment/environment.md
```

| Regla | Entrada | Salida |
|---|---|---|
| `run_benchmark` | imagen `matmul-<lang>:1.0` | `raw/<lang>.csv`, `verification/<lang>_checksum.csv`, `logs/<lang>.log` |
| `merge` | los 4 CSV | `benchmark.csv` (valida columnas, duplicados y tiempos) |
| `verify_equivalence` | los 4 checksums | `verification/equivalence_report.md` |
| `analysis` | `benchmark.csv` | `analysis/tables/`, `analysis/figures/`, `analysis/reporte_estadistico.md` |
| `environment` | imágenes | `environment/environment.md` |

- Antes de ejecutarse, cada regla comprueba que su imagen exista localmente; si falta, se
  detiene e indica el comando `build` exacto. **Nunca descarga imágenes del benchmark.**
- Los benchmarks corren **de a uno** (recurso `benchmark_slot=1`, `cores: 1` en
  [workflow/profiles/default/config.yaml](workflow/profiles/default/config.yaml)), para que no
  compitan por la CPU.
- La integración y el análisis corren dentro de `matmul-analysis:1.0`, con las versiones de
  las librerías fijadas en [docker/analysis/requirements.txt](docker/analysis/requirements.txt).
- El Snakefile lanza los contenedores desde Python (sin bash), por lo que funciona igual en
  Linux, macOS y Windows.

Parámetros en [config/config.yaml](config/config.yaml); cualquiera se cambia con
`--config clave=valor`:

| Clave | Valor oficial | Descripción |
|---|---|---|
| `engine` | `docker` | motor de contenedores (`podman`, `"sudo docker"`, …) |
| `languages` | `[cpp, go, java, python]` | factor 1 |
| `sizes` | `[500, 1000, 1500]` | factor 2 |
| `repetitions` | `10` | repeticiones por celda |
| `seed` | `42` | semilla de SplitMix64 |
| `results_dir` | `results` | directorio de salida |
| `alpha` | `0.05` | nivel de significancia |

### 6.3 Análisis estadístico

- Modelo `time ~ C(language) * C(N)`, ANOVA tipo II, con η² parcial.
- Supuestos: Shapiro-Wilk sobre los residuos y Levene (mediana) entre las 12 celdas.
- Como los tiempos difieren en órdenes de magnitud (heterocedasticidad), se ajusta también el
  modelo sobre **log10(tiempo)** y se contrasta con **Kruskal-Wallis** por N.
- Post-hoc **Tukey HSD**: lenguaje global, lenguaje dentro de cada N y las 12 combinaciones.
- Figuras: boxplot, interacción lenguaje × N, razón respecto de C++, tiempo por repetición
  (calentamiento JIT) y gráficos Q-Q de los residuos.

### 6.4 Paso a paso

Requiere las 5 imágenes construidas (§5.4, paso 2).

**Paso 0 · Instalar Snakemake en un entorno virtual (una sola vez).**

```bash
python3 -m venv .venv
.venv/bin/pip install -r requirements-host.txt
.venv/bin/snakemake --version          # 9.27.0
```

```powershell
py -m venv .venv
.venv\Scripts\python.exe -m pip install -r requirements-host.txt
.venv\Scripts\snakemake.exe --version
```

**Paso 1 · Ver el plan y hacer una prueba rápida** (menos de 1 minuto, en `results_smoke/`).

```bash
.venv/bin/snakemake --config results_dir=results_smoke 'sizes=[50,100]' repetitions=3 -n   # plan, sin ejecutar
.venv/bin/snakemake --config results_dir=results_smoke 'sizes=[50,100]' repetitions=3
```

```powershell
.venv\Scripts\snakemake.exe --config results_dir=results_smoke "sizes=[50,100]" repetitions=3 -n
.venv\Scripts\snakemake.exe --config results_dir=results_smoke "sizes=[50,100]" repetitions=3
```

Debe terminar con `9 of 9 steps (100%) done`. Si no usa Docker sin sudo, agregue
`engine="sudo docker"` o `engine=podman` dentro de `--config` (§2.1).

**Paso 2 · Ejecutar el experimento completo con un solo comando** (≈2,5 h, en `results_repro/`).

```bash
.venv/bin/snakemake --config results_dir=results_repro
```

```powershell
.venv\Scripts\snakemake.exe --config results_dir=results_repro
```

> `results/` ya contiene los resultados publicados; por eso un `snakemake` sin parámetros
> responde *Nothing to be done*. Para **reproducir**, use un directorio nuevo
> (`results_repro`) y compare con `results/`. Durante la corrida conviene no usar el equipo
> para otras tareas pesadas.

**Paso 3 · Verificar el contenido generado.**

```bash
python3 scripts/check_artifact.py results_repro        # todo [ OK ] y "Artifact completo."
cat results_repro/verification/equivalence_report.md   # "Resultado global: EQUIVALENTES"
cat results_repro/analysis/tables/speedup.md
sed -n '/^## 8. Decisión/,/^## 9/p' results_repro/analysis/reporte_estadistico.md
```

```powershell
py scripts\check_artifact.py results_repro
Get-Content results_repro\verification\equivalence_report.md
Get-Content results_repro\analysis\tables\speedup.md
Get-Content results_repro\analysis\reporte_estadistico.md
start results_repro\analysis\figures\interaccion_lenguaje_N.png
```

**Paso 4 · Ver que el pipeline solo recalcula lo necesario.**

```bash
rm results_smoke/benchmark.csv
.venv/bin/snakemake --config results_dir=results_smoke 'sizes=[50,100]' repetitions=3   # rehace merge y analysis, no los benchmarks
```

```powershell
Remove-Item results_smoke\benchmark.csv
.venv\Scripts\snakemake.exe --config results_dir=results_smoke "sizes=[50,100]" repetitions=3
```

**Paso 5 (opcional) · Otras ejecuciones.**

```bash
.venv/bin/snakemake --forceall                                        # repetir la corrida oficial en results/
.venv/bin/snakemake --config results_dir=results_repro --forcerun analysis   # rehacer solo el análisis
.venv/bin/snakemake --dag | dot -Tpng > dag.png                       # grafo del pipeline (requiere graphviz)
```

En Windows, todo lo anterior (verificación de herramientas, construcción de imágenes,
instalación de Snakemake, pipeline y verificación) se hace con
`powershell -ExecutionPolicy Bypass -File scripts\reproducir.ps1` (agregue `-Rapido` para la
prueba y `-SinConstruir` si las imágenes ya existen).

### 6.5 Entregables

| Entregable | Ubicación |
|---|---|
| Snakefile y perfil | [workflow/Snakefile](workflow/Snakefile), [workflow/profiles/default/config.yaml](workflow/profiles/default/config.yaml) |
| Parámetros | [config/config.yaml](config/config.yaml) |
| Integración | [scripts/merge_results.py](scripts/merge_results.py) → [results/benchmark.csv](results/benchmark.csv) |
| Análisis | [scripts/analysis.py](scripts/analysis.py) → [results/analysis/](results/analysis/) |
| Reporte | [results/analysis/reporte_estadistico.md](results/analysis/reporte_estadistico.md) |

### 6.6 Autoevaluación ACM: Reusable

| Aspecto | Cumple | Evidencia |
|---|---|---|
| Automatización completa | ✅ | Un comando ejecuta contenedores → CSV → `benchmark.csv` → ANOVA/Tukey → tablas y figuras |
| Parametrización | ✅ | `config/config.yaml`, sobrescribible con `--config` |
| Entornos fijados | ✅ | Imágenes base y librerías de análisis con versión exacta |
| Interfaz común y extensible | ✅ | Mismo CLI en los 4 programas; guía para agregar lenguajes (§10) |
| Normas de la comunidad | ✅ | Estructura estándar de Snakemake (`workflow/`, `config/`, perfiles), etiquetas OCI, CITATION.cff, licencias SPDX, datasheet FAIR |
| Prueba rápida | ✅ | `results_smoke` con `sizes=[50,100]` |
| Portabilidad | ✅ | Docker o Podman; Linux, macOS y Windows |

Detalle: [docs/ACM_BADGES.md §2.2](docs/ACM_BADGES.md).

---

## 7. Semana 5: publicación en Zenodo y evaluación cruzada

### 7.1 Lo que pide el módulo

- Publicar una versión estable del artifact en **Zenodo** con DOI e incorporarlo al README.
- Contenido mínimo: código de los 4 lenguajes, README, Dockerfile, Snakefile, los 4 CSV,
  `benchmark.csv`, análisis estadístico, tablas y gráficos, licencia y mini datasheet FAIR.
- Evaluar el artifact de un compañero usando **solo su artifact publicado**, registrando
  dificultades, dependencias faltantes, ambigüedades, modificaciones y diferencias.
- Presentación final: resultados, artifact en Zenodo y experiencia de evaluación cruzada.

### 7.2 Paso a paso: verificar el contenido mínimo

```bash
python3 scripts/check_artifact.py          # sobre results/ (resultados oficiales)
```

```powershell
py scripts\check_artifact.py
```

Debe mostrar todo `[ OK ]` y "Artifact completo." Mientras no exista el DOI, también muestra
`[AVISO] README.md aún no contiene el DOI de Zenodo`.

### 7.3 Paso a paso: publicar en Zenodo

Se usó la **API REST de Zenodo** con [scripts/zenodo_publicar.py](scripts/zenodo_publicar.py)
(solo biblioteca estándar). Su ventaja es que el DOI se **reserva antes** de archivar, de modo que
la copia archivada ya trae el DOI en el README y en `CITATION.cff`. Procedimiento completo y
alternativas (integración GitHub → Zenodo o carga web) en [docs/ZENODO.md](docs/ZENODO.md).

1. En https://zenodo.org: *Settings → Applications → Personal access tokens → New token* con
   permisos `deposit:write` y `deposit:actions`; guardarlo en `~/.zenodo_token` (fuera del repo).
2. Crear el borrador y reservar el DOI (metadatos tomados de [.zenodo.json](.zenodo.json)):
   ```bash
   python3 scripts/zenodo_publicar.py reservar
   ```
3. Escribir el DOI reservado en este README (badge, §7.5 y §12), [CITATION.cff](CITATION.cff),
   [docs/DATASHEET_FAIR.md](docs/DATASHEET_FAIR.md) y [docs/ACM_BADGES.md](docs/ACM_BADGES.md);
   comprobar con `python3 scripts/check_artifact.py` (ya no debe mostrar el aviso del DOI).
4. Commit, etiqueta y zip de la versión (el zip contiene exactamente lo versionado en la etiqueta):
   ```bash
   git tag -a v1.0.0 -m "Artifact v1.0.0 - publicación Zenodo"
   git archive --format=zip --prefix=Modulo2-Inv-experimental-v1.0.0/ -o artifact-v1.0.0.zip v1.0.0
   ```
5. Subir, revisar el borrador en la web y publicar (publicar es **irreversible**):
   ```bash
   python3 scripts/zenodo_publicar.py subir artifact-v1.0.0.zip
   python3 scripts/zenodo_publicar.py estado
   python3 scripts/zenodo_publicar.py publicar
   ```
6. Subir a GitHub: `git push origin main --tags`.

### 7.4 Evaluación cruzada: reproducir el artifact de otra persona

1. Descargar el artifact del compañero **desde Zenodo** (no desde GitHub) en una carpeta nueva,
   fuera de este repositorio. El script verifica los checksums de Zenodo, descomprime y deja un
   resumen `DESCARGA.md` como evidencia, **sin ejecutar nada** del artifact descargado:
   ```bash
   python3 scripts/descargar_zenodo.py https://zenodo.org/records/<id> ../evaluacion-<compañero>
   ```
2. Copiar [docs/evaluacion-cruzada/PLANTILLA_EVALUACION.md](docs/evaluacion-cruzada/PLANTILLA_EVALUACION.md)
   como `docs/evaluacion-cruzada/evaluacion-<compañero>.md`.
3. Seguir **solo** su README; anotar cada comando, error, dependencia faltante, ambigüedad y
   modificación necesaria, con el tiempo de cada paso.
4. Verificar: 4 contenedores ejecutados, 4 CSV con las columnas comunes, `benchmark.csv` de
   120 filas, ANOVA + Tukey, tablas y gráficos.
5. Comparar con los resultados publicados según estos criterios (ACM *Results Reproduced*):

| # | Criterio | Tolerancia |
|---|---|---|
| 1 | Checksums | idénticos a los publicados (igualdad exacta) |
| 2 | Decisión sobre H₀ (efecto del lenguaje, α = 0,05) | la misma |
| 3 | Decisión sobre la interacción lenguaje × N | la misma |
| 4 | Orden de los lenguajes por tiempo medio en cada N | el mismo, o diferencias explicadas por pares no significativos en Tukey |
| 5 | Razón de tiempos respecto de C++ | del mismo orden de magnitud |

Los segundos absolutos **no** tienen que coincidir: dependen del hardware.

6. Emitir un dictamen por badge (Functional, Reusable, Available, Reproduced) con su
   justificación y entregarlo al autor.

### 7.5 Entregables y estado

| Entregable | Ubicación | Estado |
|---|---|---|
| Metadatos Zenodo y cita | [.zenodo.json](.zenodo.json), [CITATION.cff](CITATION.cff) | ✅ |
| Procedimiento de publicación | [docs/ZENODO.md](docs/ZENODO.md) | ✅ |
| Verificación automática del contenido | [scripts/check_artifact.py](scripts/check_artifact.py) | ✅ |
| Licencias | [LICENSE](LICENSE) (MIT), [LICENSE-DATA](LICENSE-DATA) (CC BY 4.0) | ✅ |
| Mini datasheet FAIR | [docs/DATASHEET_FAIR.md](docs/DATASHEET_FAIR.md) | ✅ |
| Reproducción en Windows con un comando | [scripts/reproducir.ps1](scripts/reproducir.ps1) | ✅ |
| Plantilla de evaluación cruzada | [docs/evaluacion-cruzada/PLANTILLA_EVALUACION.md](docs/evaluacion-cruzada/PLANTILLA_EVALUACION.md) | ✅ |
| Publicación por API y descarga para evaluar | [scripts/zenodo_publicar.py](scripts/zenodo_publicar.py), [scripts/descargar_zenodo.py](scripts/descargar_zenodo.py) | ✅ |
| Release `v1.0.0` y DOI | [https://zenodo.org/records/23954259](https://zenodo.org/records/23954259) · **DOI: [10.5281/zenodo.23954259](https://doi.org/10.5281/zenodo.23954259)** | ✅ |
| Evaluación del artifact de un compañero | `docs/evaluacion-cruzada/evaluacion-<compañero>.md` | ⏳ |
| Respuesta a la evaluación recibida | [docs/evaluacion-cruzada/respuesta-evaluacion.md](docs/evaluacion-cruzada/respuesta-evaluacion.md) | ⏳ |

### 7.6 Autoevaluación ACM: Available y Reproduced

| Badge | Requisito | Estado |
|---|---|---|
| **Available** | Depósito en un repositorio de archivo con DOI (Zenodo) | ✅ Cumple: `v1.0.0` archivada con DOI [10.5281/zenodo.23954259](https://doi.org/10.5281/zenodo.23954259) |
| **Reproduced** | Otra persona obtiene los resultados principales con este artifact (criterios de §7.4) | ⏳ pendiente de la evaluación cruzada |

---

## 8. Resultados oficiales

<!-- RESULTADOS:INICIO -->
Corrida oficial del 2026‑10‑17 (Intel Core Ultra 7 165H, 30 GB RAM, Fedora 42 / kernel 6.18,
Podman 5.8.2; entorno completo en [results/environment/environment.md](results/environment/environment.md)).
Duración total del pipeline: 2 h 33 min.

**Tiempo medio de la multiplicación [s] (10 repeticiones; entre paréntesis, razón respecto de C++)**

| N | C++ | Go | Java | Python |
|---|---|---|---|---|
| 500 | 0,080 | 0,118 (1,47×) | 0,163 (2,03×) | 20,0 (249×) |
| 1000 | 0,889 | 1,460 (1,64×) | 1,220 (1,37×) | 178,2 (200×) |
| 1500 | 5,519 | 9,415 (1,71×) | 8,724 (1,58×) | 688,5 (125×) |

**ANOVA de dos factores con interacción**

| Efecto | F (tiempo) | p (tiempo) | F (log10) | p (log10) | η² parcial (log10) |
|---|---|---|---|---|---|
| Lenguaje | 6091 | 2,8·10⁻¹²⁰ | 15 740 | 1,8·10⁻¹⁴² | 0,998 |
| N | 3109 | 3,5·10⁻⁹⁶ | 14 230 | 1,5·10⁻¹³¹ | 0,996 |
| Lenguaje × N | 2828 | 2,8·10⁻¹¹⁶ | 37,6 | 2,5·10⁻²⁴ | 0,676 |

**Conclusiones**

1. **Se rechaza H₀** (α = 0,05): el lenguaje produce diferencias significativas en el tiempo
   de ejecución. Tukey HSD encuentra diferencias significativas en **los 6 pares de lenguajes
   para cada uno de los 3 tamaños** (18/18 comparaciones); Kruskal‑Wallis lo confirma en cada N
   (p < 2·10⁻⁷).
2. **La interacción lenguaje × N es significativa**: la diferencia entre lenguajes depende del
   tamaño de la matriz. Se observa en dos hechos:
   - **Go y Java intercambian posiciones**: con N = 500 Go es más rápido que Java, mientras que
     con N = 1000 y 1500 Java supera a Go.
   - **La desventaja relativa de Python disminuye** al crecer N (249× → 125×). Una explicación posible es que, en los
     lenguajes compilados el costo de los accesos a memoria por columnas de B (fallos de caché)
     crecería más rápido que el costo del intérprete. Es una hipótesis plausible que este
     experimento no pone a prueba.
3. Orden de los lenguajes, de más rápido a más lento: C++ < Go < Java ≪ Python con N = 500;
   C++ < Java < Go ≪ Python con N = 1000 y 1500.
4. **Supuestos:** los residuos no son normales y las varianzas no son homogéneas, ni siquiera
   en log10 (Shapiro p = 0,002; Levene p < 10⁻⁵). Aun así, con diferencias de esta magnitud,
   el modelo log10 y la prueba no paramétrica llegan a la misma conclusión.
5. **Equivalencia funcional:** las 4 implementaciones producen la **misma matriz bit a bit**
   (checksums idénticos en los 3 tamaños,
   [equivalence_report.md](results/verification/equivalence_report.md)).
6. **Observación de estabilidad:** en Python con N = 1500, las repeticiones 7–10 fueron
   ~8 % más rápidas (634–667 s frente a 704–727 s). Es compatible con cambios de frecuencia o
   de carga del equipo durante una corrida de ~2 h; queda registrada como amenaza a la validez.

Reporte completo: [results/analysis/reporte_estadistico.md](results/analysis/reporte_estadistico.md)

![Interacción lenguaje × N](results/analysis/figures/interaccion_lenguaje_N.png)
![Razón respecto de C++](results/analysis/figures/speedup_vs_cpp.png)
<!-- RESULTADOS:FIN -->

---

## 9. Cómo replicar el experimento desde cero

**Replicar** significa obtener las mismas conclusiones con una **implementación propia**, sin
usar el código de este repositorio. El protocolo (§4.2) está pensado para que dos
implementaciones independientes calculen **exactamente** la misma matriz, lo que da una prueba
objetiva de que la réplica es correcta.

**Paso 1 · Implementar el benchmark en los 4 lenguajes** respetando cada fila de la tabla de
§4.2. Lista de control:

- [ ] SplitMix64 con semilla 42, reiniciado para cada N; se genera A y luego B; real en [0,1) = `(x >> 11) × 2⁻⁵³`.
- [ ] Arreglos planos row-major de `double`; C reservada antes del temporizador.
- [ ] Triple bucle i-j-k con acumulador local; sin librerías optimizadas; sin FMA.
- [ ] Temporizador monotónico alrededor de la multiplicación y de nada más.
- [ ] Salida `language,N,repetition,time_seconds` y checksum `language,N,repetition,checksum,c_first,c_last`.
- [ ] Interfaz `<programa> <salida.csv> <tamaños> <repeticiones> <semilla> [checksum.csv]` (opcional, pero permite reutilizar el pipeline).

**Paso 2 · Validar la réplica con los checksums publicados.** Con N = 200 la suma de todos los
elementos de C debe ser `1998326.3241066386`; con N = 500, 1000 y 1500, los valores de la
tabla de §4.5. Si coinciden, su implementación es **equivalente bit a bit** a la original.

**Paso 3 · Ejecutar el diseño completo** (4 lenguajes × N = 500, 1000, 1500 × 10 repeticiones),
de a un lenguaje por vez y con el equipo en reposo, y registrar el entorno.

**Paso 4 · Analizar sus CSV con el mismo análisis estadístico.** Copie sus 4 CSV en
`results_replica/raw/` y use la imagen de análisis de este artifact:

```bash
docker build -f docker/analysis/Dockerfile -t matmul-analysis:1.0 .
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work:z" -w /work matmul-analysis:1.0 \
  python scripts/merge_results.py results_replica/benchmark.csv results_replica/raw/{cpp,go,java,python}.csv
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work:z" -w /work matmul-analysis:1.0 \
  python scripts/analysis.py results_replica/benchmark.csv results_replica/analysis 0.05
```

```powershell
docker build -f docker/analysis/Dockerfile -t matmul-analysis:1.0 .
docker run --rm -v "${PWD}:/work" -w /work matmul-analysis:1.0 `
  python scripts/merge_results.py results_replica/benchmark.csv results_replica/raw/cpp.csv results_replica/raw/go.csv results_replica/raw/java.csv results_replica/raw/python.csv
docker run --rm -v "${PWD}:/work" -w /work matmul-analysis:1.0 `
  python scripts/analysis.py results_replica/benchmark.csv results_replica/analysis 0.05
```

Alternativa: si sus programas respetan la interfaz común, cree sus propios Dockerfile con las
mismas etiquetas `matmul-<lang>:1.0` y ejecute el pipeline completo de §6.4 con
`--config results_dir=results_replica`.

**Paso 5 · Comparar las conclusiones** con §8 usando los criterios de §7.4.

---

## 10. Reutilización y extensión

- **Otros tamaños, repeticiones o semilla:** editar [config/config.yaml](config/config.yaml) o
  usar `--config` (p. ej. `'sizes=[256,512,1024]' repetitions=20`).
- **Agregar un lenguaje (p. ej. Rust):** crear `src/rust/` y `docker/rust/Dockerfile` respetando
  la interfaz común y el protocolo de [docs/PROTOCOLO.md](docs/PROTOCOLO.md), y añadir `rust` a
  `languages`. `verify_equivalence` comprobará automáticamente que produce la misma matriz.
  El análisis usa C++ como referencia para las razones de tiempo.
- **Otro análisis:** [scripts/analysis.py](scripts/analysis.py) lee cualquier `benchmark.csv` con
  las 4 columnas comunes.
- **Fijar un núcleo de CPU** para reducir ruido en procesadores híbridos: agregar
  `--cpuset-cpus=0` a los `docker run`.

---

## 11. Solución de problemas

| Síntoma | Sistema | Solución |
|---|---|---|
| `permission denied ... /var/run/docker.sock` | Linux | El usuario no está en el grupo `docker`: usar `sudo docker`, Podman o `sudo usermod -aG docker $USER` (§2.1). |
| `sd-bus call: ... Input/output error` | Linux (Podman) | Usar `podman --cgroup-manager=cgroupfs` (§2.1). |
| `Permission denied` al escribir en `/results` | Linux (SELinux) | Falta `:z` en el volumen; el pipeline y los scripts ya lo agregan. |
| `... running scripts is disabled on this system` | Windows | Ejecutar con `powershell -ExecutionPolicy Bypass -File scripts\<script>.ps1`. |
| `error during connect` / `dockerDesktopLinuxEngine` | Windows | Docker Desktop no está abierto: iniciarlo y esperar *Engine running*. |
| `py` no se reconoce | Windows | Usar `python`, o reinstalar Python marcando *Add python.exe to PATH*. |
| `snakemake` / `snakemake.exe` no se reconoce | ambos | Falta el paso 0 de §6.4 (instalar Snakemake en `.venv`). |
| `Falta la imagen matmul-<x>:1.0` | ambos | Construirla (§5.4, paso 2). El pipeline nunca descarga imágenes del benchmark. |
| `Nothing to be done` | ambos | `results/` ya tiene los resultados publicados: usar `--config results_dir=results_repro`. |
| Los CSV no aparecen en la carpeta | Windows | Docker Desktop → *Settings → Resources → File sharing*; evitar carpetas sincronizadas con OneDrive. |
| Java termina con `OutOfMemoryError` | ambos | Asignar ≥ 4 GB de memoria a Docker (Docker Desktop → *Settings → Resources*). |
| El análisis falla con un solo tamaño N | ambos | El ANOVA necesita al menos 2 niveles de N y 2 repeticiones. |
| Tiempos muy distintos de los publicados | ambos | Esperado en otro hardware (y con la virtualización de WSL 2). Deben coincidir los checksums y las decisiones estadísticas (§7.4). |

---

## 12. Licencia, cita y referencias

- Código: [MIT](LICENSE). Datos, resultados, figuras y documentación: [CC BY 4.0](LICENSE-DATA).
- Cómo citar: [CITATION.cff](CITATION.cff) · DOI de Zenodo: [10.5281/zenodo.23954259](https://doi.org/10.5281/zenodo.23954259).

  > Miqueles, P. (2026). *Benchmark reproducible de multiplicación de matrices NxN en C++, Go,
  > Java y Python* (v1.0.0) [Software]. Zenodo. https://doi.org/10.5281/zenodo.23954259
- Historia de versiones: [CHANGELOG.md](CHANGELOG.md).

Referencias:

- ACM. *Artifact Review and Badging – Current (v1.1)*. https://www.acm.org/publications/policies/artifact-review-and-badging-current
- National Academies of Sciences, Engineering, and Medicine (2019). *Reproducibility and Replicability in Science*. https://doi.org/10.17226/25303
- Fehr, J. et al. (2016). *Best Practices for Replicability, Reproducibility and Reusability of Computer-Based Experiments Exemplified by Model Reduction Software*. https://arxiv.org/abs/1607.01191
