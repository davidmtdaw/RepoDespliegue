# Registro de prompts

| Nº | Prompt (resumido) | Qué hizo el agente | Qué corregí yo |
|----|-------------------|--------------------|----------------|
| 1 | Explica qué hace esta app y qué necesita para arrancar; propón estrategia de contenerización | Resumen de rutas, Redis y variables; propuso web + cache | Comprobé en `app.py` y `requirements.txt`: Flask 3.0.3, redis 5.0.4, gunicorn 22.0.0; `CLAVE_ADMIN` no se usa en el código |
| 2 | Dockerfile con `python:3.12-slim`, deps antes del código, usuario no-root, `EXPOSE 8000`, `HEALTHCHECK` a `/salud`, gunicorn y `CMD` JSON; y `.dockerignore` | Creó `Dockerfile` y `.dockerignore`; la imagen construyó correctamente | Verifiqué con `docker image inspect` y `docker run`: `User=appuser`, `CMD` JSON con gunicorn y `/app` sin `.git` ni `.env` |
| 3 | docker-compose con web + Redis (sin puertos, volumen con nombre, healthcheck, `service_healthy`) | Generó el compose con ambas dependencias y volumen | Comprobé con `docker compose config` que `cache` no publica puertos y que `web` usa `condition: service_healthy` |
| 4 | Levanta, prueba portada/alta/`/salud`, verifica persistencia y borra volúmenes | Di los comandos y verifiqué la salida | Confirmé `notas:2` tras `restart`, volumen eliminado con `down -v` y `notas:0` tras redisplegar |
| 5 | Informe de problemas del Dockerfile y compose de `sin-revisar` | Listó los problemas y los verifiqué con `docker build`, `inspect`, `history` y `config` | Detecté un punto exagerado (el `REDIS_URL` sí está definido) y lo documenté en `auditoria.md` |
| 6 | Comprueba usuario, `.env`/`.git` fuera de la imagen, healthy y capas más pesadas | Di los comandos y verifiqué | Confirmé `appuser`, ausencia de `.env`/`.git`, estado `healthy` y capas base como las más pesadas |
| 7 | Escribe README.md (requisitos, variables, puesta en marcha, pruebas, datos, logs, limpieza) | Generó el README | Verifiqué el redespliegue reproducible tras `down -v` + `image rm` |
