# CI/CD Tools and Practices - Proyecto final

Proyecto final del curso **CI/CD Tools and Practices** (IBM Skills Network / Coursera).

El servicio es una API REST en **Flask** que maneja contadores de visitas. El repositorio incluye su pipeline de integración continua en **GitHub Actions** y un pipeline de despliegue continuo en **Tekton** sobre **OpenShift**.

## Endpoints

| Método | Ruta | Descripción |
|--------|------|-------------|
| GET | `/health` | Estado del servicio |
| GET | `/` | Información del servicio |
| GET | `/counters` | Lista todos los contadores |
| POST | `/counters/<name>` | Crea un contador |
| GET | `/counters/<name>` | Lee un contador |
| PUT | `/counters/<name>` | Incrementa un contador |
| DELETE | `/counters/<name>` | Borra un contador |

## Correr en local

Requiere Python 3.10 o superior.

```bash
git clone https://github.com/facundogirardi/ci-cd-final-project.git
cd ci-cd-final-project
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Tests con cobertura
pytest

# Lint
flake8 service --count --max-complexity=10 --max-line-length=127 --statistics

# Levantar el servicio en http://localhost:8000
gunicorn --bind 0.0.0.0:8000 service:app
```

Con Docker:

```bash
docker build -t counter-service .
docker run --rm -p 8000:8000 counter-service
```

## Pipelines

### CI: GitHub Actions

[.github/workflows/workflow.yml](.github/workflows/workflow.yml) corre en cada push y pull request a `main`:

1. Instala las dependencias en un contenedor `python:3.12-slim`
2. Lint con **flake8**
3. Tests unitarios con **pytest** y cobertura

### CD: Tekton + OpenShift

[.tekton/tasks.yml](.tekton/tasks.yml) define el pipeline `cleanup → git-clone → flake8 + tests → buildah → deploy`, que construye la imagen y la despliega en OpenShift con `oc`. Las capturas `oc-pipelines-*.png` muestran la ejecución en el laboratorio del curso.

## Actualizacion 2026

* Flask 3.1, Werkzeug 3.1, gunicorn 27, python-dotenv 1.2, flake8 7, pylint 4, black 26 y coverage 7.17
* **nose** (sin mantenimiento e incompatible con Python 3.10+) se reemplazó por **pytest** + **pytest-cov**. La configuración está en `setup.cfg`
* El workflow de GitHub Actions usa `python:3.12-slim` y `actions/checkout@v4`, y mide la cobertura del paquete correcto (`service`)
* La imagen Docker pasó a `python:3.12-slim`

> **Nota:** el pipeline de Tekton referencia la tarea `nose` y `bin/setup.sh` instala Python 3.9: los dos dependen del laboratorio del curso y no se modificaron. Para volver a ejecutarlos en ese entorno hay que usar las versiones originales de `requirements.txt`, que siguen en el historial de git.

## Licencia

Apache License 2.0. Ver [LICENSE](LICENSE).

Plantilla original: Skills Network · © IBM Corporation 2023.
