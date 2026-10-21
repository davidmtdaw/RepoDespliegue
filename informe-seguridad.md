# Seguridad y tamaño (Ejercicio 6)

Comprobaciones sobre la versión buena (`notas-web:latest`).

| Comprobación | Comando | Resultado |
|--------------|---------|-----------|
| Usuario del contenedor | `docker compose exec web id` | `uid=10001(appuser) gid=999(appuser)` — no ejecuta como root |
| `.env` y `.git` fuera de la imagen | `docker compose exec web ls -a /app` y `find / -name .git -o -name .env` | `/app` solo contiene `app.py`, `requirements.txt`, `templates`; `.git` y `.env` ausentes (no encontrados en toda la imagen) |
| Estado healthy | `docker compose ps` y `docker inspect ... .State.Health` | `web` y `cache` → `healthy`, `FailingStreak: 0`, healthcheck con exit 0 |
| Tamaño | `docker image ls` | `notas-web:latest` = 191MB; base `python:3.12-slim` = 179MB |

## Capas más pesadas (`docker history notas-web:latest`)

| Tamaño | Capa |
|--------|------|
| 87.7MB | Heredada de `python:3.12-slim` (Python) |
| 41.4MB | Heredada de `python:3.12-slim` (Debian slim) |
| 10.3MB | `pip install --no-cache-dir -r requirements.txt` (Flask + redis + gunicorn) |
| 61.4kB | `useradd` del usuario `appuser` |
| KB | `COPY app.py`, `COPY templates` |

Conclusión: lo que pesa es la base Debian/Python, no la aplicación (~12MB sobre la base).
El `.dockerignore` evita que `.git`, `.env`, `sin-revisar/` y demás entren en la imagen.
