# 🐧 Debian 13 Trixie — Setup Completo para Analista de Datos

> **Versión:** Mayo 2026  
> **Sistema:** Debian 13 "Trixie" (lanzado el 9 de agosto de 2025, versión estable actual: 13.4)  
> **Arquitectura:** amd64  
> **Soporte LTS:** hasta 2030  

---

## Índice

1. [Antes de empezar](#1-antes-de-empezar)
2. [Sistema base y utilidades esenciales](#2-sistema-base-y-utilidades-esenciales)
3. [Terminal y shell](#3-terminal-y-shell)
4. [Python con pyenv](#4-python-con-pyenv)
5. [Gestor de paquetes uv](#5-gestor-de-paquetes-uv)
6. [Stack de análisis de datos](#6-stack-de-análisis-de-datos)
7. [JupyterLab](#7-jupyterlab)
8. [VS Code](#8-vs-code)
9. [Bases de datos](#9-bases-de-datos)
10. [Git y GitHub](#10-git-y-github)
11. [Docker](#11-docker)
12. [Herramientas científicas y de productividad](#12-herramientas-científicas-y-de-productividad)
13. [Estructura de proyectos recomendada](#13-estructura-de-proyectos-recomendada)
14. [Verificación final del entorno](#14-verificación-final-del-entorno)
15. [Referencias y versiones](#15-referencias-y-versiones)

---

## 1. Antes de empezar

### ¿Por qué Debian 13 Trixie?

Debian 13 "Trixie" es la versión estable actual, liberada el **9 de agosto de 2025**, y soportada con actualizaciones de seguridad hasta **2030**. Viene con Python 3.13, PostgreSQL 17, Linux kernel 6.12 LTS, GNOME 48 y KDE Plasma 6.3 incluidos.

### Paso obligatorio: crear un snapshot de Timeshift

Antes de instalar cualquier cosa, crea un punto de restauración del sistema. Si algo sale mal, puedes revertir completamente.

```bash
# Instalar Timeshift
sudo apt install -y timeshift

# Crear snapshot inicial desde terminal
sudo timeshift --create --comments "sistema limpio antes de setup DS" --tags O
```

Para verificar que se creó:

```bash
sudo timeshift --list
```

> **Nota:** Si quieres guardar snapshots en un disco externo o USB, este debe estar formateado en `ext4`. Usa `sudo mkfs.ext4 /dev/sdX1` para formatear (reemplaza `sdX1` con tu dispositivo). Identifica el dispositivo con `lsblk` antes de formatear.

---

## 2. Sistema base y utilidades esenciales

### Actualización inicial del sistema

```bash
sudo apt update && sudo apt upgrade -y
```

### Paquetes base esenciales

```bash
sudo apt install -y \
  build-essential \
  curl \
  wget \
  git \
  gnupg \
  ca-certificates \
  apt-transport-https \
  software-properties-common \
  unzip \
  zip \
  lsb-release
```

### Utilidades de terminal y monitoreo

```bash
sudo apt install -y \
  btop \
  tmux \
  ripgrep \
  fd-find \
  tree \
  ncdu \
  net-tools
```

**¿Para qué sirve cada una?**

| Herramienta | Función |
|---|---|
| `btop` | Monitor de CPU, RAM, disco y red en tiempo real |
| `tmux` | Multiplexor de terminal: múltiples sesiones y paneles |
| `ripgrep` | Búsqueda ultrarrápida de texto en archivos |
| `fd-find` | Alternativa moderna y rápida a `find` |
| `ncdu` | Análisis de uso de disco interactivo |

---

## 3. Terminal y shell

### Zsh + Oh My Zsh

Zsh con Oh My Zsh es el estándar de productividad para desarrolladores y analistas. Ofrece autocompletado inteligente, historial mejorado y plugins para git, python y más.

```bash
# Instalar Zsh
sudo apt install -y zsh

# Establecer Zsh como shell por defecto
chsh -s $(which zsh)
```

Cierra sesión y vuelve a entrar. Luego instala Oh My Zsh:

```bash
sh -c "$(curl -fsSL https://raw.github.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

#### Plugins recomendados

```bash
# Autosuggestions: sugiere comandos basados en el historial
git clone https://github.com/zsh-users/zsh-autosuggestions \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

# Syntax highlighting: colorea comandos válidos e inválidos
git clone https://github.com/zsh-users/zsh-syntax-highlighting \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

Edita `~/.zshrc` y actualiza la línea de plugins:

```bash
plugins=(git zsh-autosuggestions zsh-syntax-highlighting python pip docker)
```

Recarga la configuración:

```bash
source ~/.zshrc
```

### Kitty — terminal GPU-acelerada

Kitty es una de las terminales más rápidas disponibles, con soporte para splits, tabs y renderizado de imágenes directamente en la terminal (útil para previsualizar gráficos).

```bash
sudo apt install -y kitty
```

---

## 4. Python con pyenv

**Nunca uses el Python del sistema** para proyectos de análisis de datos. Puede romper dependencias del sistema operativo. `pyenv` te permite instalar y cambiar entre versiones de Python sin afectar el sistema.

### Versiones actuales (mayo 2026)

- **Python 3.13.13** — versión estable recomendada para producción (lanzada el 7 de abril de 2026)
- **Python 3.14.5** — versión feature más reciente (10 mayo 2026), aún en adopción temprana
- Debian 13 Trixie incluye Python 3.13 en sus repositorios oficiales

### Instalar dependencias de pyenv

```bash
sudo apt install -y \
  libssl-dev \
  libffi-dev \
  libbz2-dev \
  libreadline-dev \
  libsqlite3-dev \
  liblzma-dev \
  libncurses-dev \
  libncursesw5-dev \
  tk-dev \
  xz-utils \
  zlib1g-dev
```

### Instalar pyenv

```bash
curl https://pyenv.run | bash
```

### Configurar pyenv en ~/.zshrc

Agrega estas líneas al final de `~/.zshrc`:

```bash
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

Recarga el shell:

```bash
source ~/.zshrc
```

### Instalar Python 3.13.13

```bash
pyenv install 3.13.13
pyenv global 3.13.13

# Verificar
python --version
# → Python 3.13.13
```

---

## 5. Gestor de paquetes uv

`uv` es el gestor de paquetes moderno para Python. Es entre 10 y 100 veces más rápido que `pip`, reemplaza a `pip`, `venv`, `pip-tools` y parcialmente a `poetry`. Es el nuevo estándar en 2026.

### Instalar uv

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
source ~/.zshrc
uv --version
```

### Crear el entorno principal de análisis de datos

```bash
mkdir -p ~/envs
cd ~/envs
uv venv ds-env --python 3.13.13
source ds-env/bin/activate
```

Para activar el entorno en el futuro:

```bash
source ~/envs/ds-env/bin/activate
```

---

## 6. Stack de análisis de datos

Con el entorno activado, instala todas las librerías esenciales:

### Librerías core

```bash
uv pip install \
  numpy \
  scipy \
  pandas \
  polars \
  pyarrow \
  fastparquet
```

**¿Cuándo usar qué?**

- **Pandas** — ecosistema amplio, integración con scikit-learn y la mayoría de librerías. Usa `dtype_backend='pyarrow'` para mejor rendimiento.
- **Polars** — 5-10x más rápido que Pandas en transformaciones, pipelines grandes, lazy evaluation. El estándar emergente en 2026.
- **DuckDB** — SQL analítico directamente sobre CSV, Parquet y DataFrames. Hasta 5x más rápido que Pandas en agregaciones. No necesita servidor.
- **PyArrow** — formato columnar en memoria, conecta los tres anteriores.

### Machine Learning

```bash
uv pip install \
  scikit-learn \
  xgboost \
  lightgbm \
  imbalanced-learn \
  optuna
```

### Visualización

```bash
uv pip install \
  matplotlib \
  seaborn \
  plotly \
  altair
```

| Librería | Cuándo usarla |
|---|---|
| `matplotlib` | Gráficos publicables, control total, papers científicos |
| `seaborn` | Visualización estadística de alto nivel rápida |
| `plotly` | Gráficos interactivos en notebooks y dashboards |
| `altair` | Visualización declarativa, ideal para EDA rápido |

### Análisis y utilidades

```bash
uv pip install \
  duckdb \
  sqlalchemy \
  psycopg2-binary \
  openpyxl \
  xlrd \
  requests \
  httpx \
  python-dotenv \
  pydantic \
  rich
```

### MLOps y seguimiento de experimentos

```bash
uv pip install \
  mlflow \
  dvc \
  great-expectations
```

### Apps de datos

```bash
uv pip install streamlit
```

---

## 7. JupyterLab

JupyterLab es el entorno de trabajo principal para exploración de datos, prototipado y reportes.

### Instalar JupyterLab y extensiones

```bash
uv pip install \
  jupyterlab \
  ipykernel \
  jupyterlab-git \
  jupyter-ai \
  nbformat \
  nbconvert
```

### Registrar el entorno como kernel

```bash
python -m ipykernel install \
  --user \
  --name=ds-env \
  --display-name "Data Science (Python 3.13)"
```

### Lanzar JupyterLab

```bash
jupyter lab
```

### Configurar JupyterLab (opcional, para servidores)

```bash
jupyter lab --generate-config
# Editar ~/.jupyter/jupyter_lab_config.py y agregar:
# c.ServerApp.open_browser = False
# c.ServerApp.ip = '0.0.0.0'
```

### Instalar Marimo (alternativa moderna)

Marimo es un notebook reactivo: cuando cambias una celda, todo lo que depende de ella se actualiza automáticamente. Es el futuro de los notebooks interactivos.

```bash
uv pip install marimo
marimo tutorial intro
```

---

## 8. VS Code

VS Code complementa JupyterLab para proyectos más grandes, scripts Python, control de versiones integrado y trabajo con múltiples archivos.

### Instalar VS Code

```bash
wget -qO- https://packages.microsoft.com/keys/microsoft.asc \
  | gpg --dearmor > packages.microsoft.gpg

sudo install -D -o root -g root -m 644 \
  packages.microsoft.gpg /etc/apt/keyrings/packages.microsoft.gpg

sudo sh -c 'echo "deb [arch=amd64 signed-by=/etc/apt/keyrings/packages.microsoft.gpg] \
  https://packages.microsoft.com/repos/code stable main" \
  > /etc/apt/sources.list.d/vscode.list'

sudo apt update && sudo apt install -y code
```

### Extensiones esenciales

```bash
# Python y Jupyter
code --install-extension ms-python.python
code --install-extension ms-toolsai.jupyter
code --install-extension ms-python.vscode-pylance

# Git
code --install-extension eamodio.gitlens
code --install-extension mhutchie.git-graph

# Datos
code --install-extension mechatroner.rainbow-csv
code --install-extension GrapeCity.gc-excelviewer

# IA
code --install-extension github.copilot
code --install-extension github.copilot-chat

# Utilidades
code --install-extension ms-python.black-formatter
code --install-extension ms-python.isort
code --install-extension tamasfe.even-better-toml
```

### Configuración recomendada de VS Code

Crea o edita `~/.config/Code/User/settings.json`:

```json
{
  "python.defaultInterpreterPath": "~/envs/ds-env/bin/python",
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "ms-python.black-formatter",
  "jupyter.notebookFileRoot": "${workspaceFolder}",
  "files.autoSave": "afterDelay",
  "terminal.integrated.defaultProfile.linux": "zsh"
}
```

---

## 9. Bases de datos

### PostgreSQL 17

Debian 13 Trixie incluye PostgreSQL 17 en sus repositorios oficiales.

```bash
sudo apt install -y postgresql postgresql-contrib
sudo systemctl enable postgresql
sudo systemctl start postgresql
```

Configuración inicial:

```bash
# Crear usuario para análisis de datos
sudo -u postgres psql -c "CREATE USER analista WITH PASSWORD 'tu_password';"
sudo -u postgres psql -c "CREATE DATABASE ds_db OWNER analista;"
sudo -u postgres psql -c "GRANT ALL PRIVILEGES ON DATABASE ds_db TO analista;"
```

Verificar conexión desde Python:

```python
import sqlalchemy as sa
engine = sa.create_engine("postgresql://analista:tu_password@localhost/ds_db")
print(engine.connect())
```

### DuckDB 1.2+

DuckDB es la herramienta más importante para análisis local de datos grandes en 2026. Versión actual: **1.2.x**. No necesita servidor, funciona embebido en Python.

```bash
uv pip install duckdb
```

Ejemplo de uso:

```python
import duckdb

# Consultar CSV directamente (sin importar)
duckdb.sql("SELECT * FROM 'datos.csv' LIMIT 10").show()

# Consultar Parquet
duckdb.sql("SELECT year, AVG(valor) FROM 'datos.parquet' GROUP BY year").df()

# Interoperar con Pandas
import pandas as pd
df = pd.read_csv("datos.csv")
resultado = duckdb.sql("SELECT * FROM df WHERE valor > 100").pl()  # → Polars DataFrame
```

### SQLite

```bash
sudo apt install -y sqlite3
uv pip install sqlite-utils
```

### DBeaver — cliente SQL universal

DBeaver conecta con PostgreSQL, MySQL, SQLite, DuckDB, MongoDB y más desde una interfaz gráfica.

```bash
wget https://dbeaver.io/files/dbeaver-ce_latest_amd64.deb
sudo dpkg -i dbeaver-ce_latest_amd64.deb
sudo apt install -f
rm dbeaver-ce_latest_amd64.deb
```

---

## 10. Git y GitHub

### Configuración global de Git

```bash
git config --global user.name "Tu Nombre"
git config --global user.email "tu@email.com"
git config --global init.defaultBranch main
git config --global core.editor "code --wait"
git config --global pull.rebase false
```

### Generar llave SSH para GitHub/GitLab

```bash
ssh-keygen -t ed25519 -C "tu@email.com"
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519

# Copiar la llave pública
cat ~/.ssh/id_ed25519.pub
```

Agrega la llave pública en: **GitHub → Settings → SSH and GPG keys → New SSH key**

### Verificar conexión

```bash
ssh -T git@github.com
# → Hi username! You've successfully authenticated.
```

### GitHub CLI

```bash
sudo apt install -y gh
gh auth login
```

Comandos útiles:

```bash
gh repo create mi-proyecto-ds --private --clone
gh repo list
gh issue list
gh pr list
```

### .gitignore recomendado para proyectos DS

Crea `~/.gitignore_global`:

```
# Python
__pycache__/
*.py[cod]
*.pyo
.env
.venv
venv/
*.egg-info/

# Jupyter
.ipynb_checkpoints/
*.ipynb_checkpoints

# Datos (no versionar datos sensibles o pesados)
*.csv
*.parquet
*.pkl
*.h5
data/raw/
data/processed/

# VS Code
.vscode/

# Sistema
.DS_Store
*.log
```

Activar globalmente:

```bash
git config --global core.excludesfile ~/.gitignore_global
```

---

## 11. Docker

Docker garantiza reproducibilidad: tu entorno de análisis funciona igual en tu máquina, en un servidor o en la nube.

### Instalar Docker Engine

```bash
sudo apt install -y docker.io docker-compose-v2
sudo systemctl enable docker
sudo systemctl start docker

# Agregar tu usuario al grupo docker (evita usar sudo)
sudo usermod -aG docker $USER
newgrp docker
```

### Verificar instalación

```bash
docker run hello-world
docker --version
docker compose version
```

### Lanzar JupyterLab con Docker

Para un entorno completamente aislado con todas las librerías de DS preinstaladas:

```bash
docker run -it --rm \
  -p 8888:8888 \
  -v $(pwd):/home/jovyan/work \
  jupyter/datascience-notebook
```

### Dockerfile recomendado para proyectos DS

```dockerfile
FROM python:3.13-slim

WORKDIR /app

COPY requirements.txt .
RUN pip install uv && uv pip install --system -r requirements.txt

COPY . .

CMD ["jupyter", "lab", "--ip=0.0.0.0", "--no-browser", "--allow-root"]
```

---

## 12. Herramientas científicas y de productividad

### LaTeX — reportes y papers científicos

Debian 13 incluye TeX Live. Para instalación completa:

```bash
sudo apt install -y texlive-full pandoc
```

Para instalación ligera (solo lo esencial):

```bash
sudo apt install -y texlive-base texlive-latex-recommended pandoc
```

Exportar notebook a PDF con LaTeX:

```bash
jupyter nbconvert --to pdf mi_analisis.ipynb
```

### Zotero — gestor de referencias bibliográficas

Zotero gestiona papers, artículos y libros. Integra con Word, LibreOffice y exporta a BibTeX para LaTeX.

```bash
wget -qO- https://raw.githubusercontent.com/retorquere/zotero-deb/master/install.sh \
  | sudo bash
sudo apt update && sudo apt install -y zotero
```

### Obsidian — notas científicas en Markdown

Para llevar un segundo cerebro: notas de investigación, journaling de proyectos, gestión de conocimiento.

```bash
# Descargar la última versión desde obsidian.md/download
# o instalar via Flatpak:
sudo apt install -y flatpak
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak install flathub md.obsidian.Obsidian
```

### Timeshift — snapshots del sistema

Ya instalado en el paso 1. Para crear snapshots adicionales en momentos clave:

```bash
# Antes de instalar software nuevo
sudo timeshift --create --comments "antes de instalar X"

# Listar snapshots disponibles
sudo timeshift --list

# Restaurar (en caso de emergencia)
sudo timeshift --restore
```

---

## 13. Estructura de proyectos recomendada

Esta estructura sigue las mejores prácticas de ciencia de datos reproducible en 2026:

```
mi-proyecto-ds/
├── README.md
├── .gitignore
├── .env.example           ← variables de entorno (nunca versionar .env)
├── pyproject.toml         ← dependencias del proyecto (con uv)
├── data/
│   ├── raw/               ← datos originales sin modificar (no versionar)
│   ├── processed/         ← datos procesados
│   └── external/          ← datos de fuentes externas
├── notebooks/
│   ├── 01-exploracion.ipynb
│   ├── 02-limpieza.ipynb
│   └── 03-modelado.ipynb
├── src/
│   ├── __init__.py
│   ├── data/              ← scripts de ingestión y limpieza
│   ├── features/          ← ingeniería de variables
│   ├── models/            ← entrenamiento y evaluación
│   └── visualization/     ← funciones de graficación
├── reports/
│   ├── figures/           ← gráficas generadas
│   └── final_report.pdf
├── tests/
└── Dockerfile
```

Para crear la estructura automáticamente:

```bash
mkdir -p mi-proyecto-ds/{data/{raw,processed,external},notebooks,src/{data,features,models,visualization},reports/figures,tests}
cd mi-proyecto-ds
touch README.md .gitignore .env.example pyproject.toml
git init && git add . && git commit -m "estructura inicial del proyecto"
```

---

## 14. Verificación final del entorno

Ejecuta este script para verificar que todo está instalado correctamente:

```bash
#!/bin/bash
echo "=== Verificación del entorno de análisis de datos ==="

echo -n "Python: "; python --version
echo -n "uv: "; uv --version
echo -n "Git: "; git --version
echo -n "Docker: "; docker --version
echo -n "PostgreSQL: "; psql --version
echo -n "DBeaver: "; dbeaver --version 2>/dev/null || echo "instalado"

echo ""
echo "=== Librerías Python ==="
python -c "
import numpy; print(f'numpy: {numpy.__version__}')
import pandas; print(f'pandas: {pandas.__version__}')
import polars; print(f'polars: {polars.__version__}')
import duckdb; print(f'duckdb: {duckdb.__version__}')
import sklearn; print(f'scikit-learn: {sklearn.__version__}')
import matplotlib; print(f'matplotlib: {matplotlib.__version__}')
import plotly; print(f'plotly: {plotly.__version__}')
import xgboost; print(f'xgboost: {xgboost.__version__}')
import mlflow; print(f'mlflow: {mlflow.__version__}')
print('✅ Todo instalado correctamente')
"
```

Guarda como `verificar_entorno.sh`, dale permisos y ejecuta:

```bash
chmod +x verificar_entorno.sh
./verificar_entorno.sh
```

---

## 15. Referencias y versiones

| Software | Versión (mayo 2026) | Fuente |
|---|---|---|
| Debian | 13.4 "Trixie" (LTS hasta 2030) | debian.org |
| Linux Kernel | 6.12 LTS | Incluido en Trixie |
| Python | 3.13.13 (estable recomendada) | python.org |
| Python | 3.14.5 (feature más reciente) | python.org |
| uv | última (auto-update) | astral.sh/uv |
| JupyterLab | 4.x (febrero 2026) | jupyter.org |
| Pandas | 2.x | pypi.org |
| Polars | 1.x | pola.rs |
| DuckDB | 1.2+ | duckdb.org |
| scikit-learn | 1.6+ | scikit-learn.org |
| PostgreSQL | 17 | postgresql.org |
| VS Code | última estable | code.visualstudio.com |
| Docker | 27+ | docker.com |
| GNOME | 48 | Incluido en Trixie |
| KDE Plasma | 6.3 | Incluido en Trixie |

---

## Flujo de trabajo diario recomendado

```bash
# 1. Activar el entorno
source ~/envs/ds-env/bin/activate

# 2. Navegar al proyecto
cd ~/proyectos/mi-proyecto-ds

# 3. Lanzar JupyterLab
jupyter lab

# 4. Al terminar, crear snapshot si hiciste cambios importantes
sudo timeshift --create --comments "checkpoint $(date +%Y-%m-%d)"
```

---

*Documento mantenido por: tu-usuario | Última actualización: Mayo 2026*  
*Sistema: Debian 13 "Trixie" para Analista de Datos*
