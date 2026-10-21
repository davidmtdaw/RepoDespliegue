# Registro de prompts

| Nº | Prompt (resumido) | Qué hizo el agente | Qué corregí yo |
|----|-------------------|--------------------|----------------|
| 1 | Explica qué hace esta app y qué necesita para arrancar; propón estrategia de contenerización | Resumen de rutas, Redis y variables; propuso web + cache | Comprobé en `app.py` y `requirements.txt`: Flask 3.0.3, redis 5.0.4, gunicorn 22.0.0; `CLAVE_ADMIN` no se usa en el código |
| 2 | Dockerfile con `python:3.12-slim`, deps antes del código, usuario no-root, `EXPOSE 8000`, `HEALTHCHECK` a `/salud`, gunicorn y `CMD` JSON; y `.dockerignore` | Creó `Dockerfile` y `.dockerignore`; la imagen construyó correctamente | Verifiqué con `docker image inspect` y `docker run`: `User=appuser`, `CMD` JSON con gunicorn y `/app` sin `.git` ni `.env` |
