# Auditoría de `sin-revisar` (Ejercicio 5)

Contenerización "mala" entregada por un compañero, auditada tal cual (sin copiarla a la raíz).
Imagen construida con: `docker build -f sin-revisar/Dockerfile -t notas-malo .`

## Problemas del Dockerfile

| # | Problema | ¿Cierto? (evidencia) | Gravedad | Corrección |
|---|----------|----------------------|----------|------------|
| 1 | Base `python:latest` → versión impredecible e imagen enorme | Cierto. `docker run notas-malo python --version` → `Python 3.15.0`; imagen 1.84GB vs 191MB | Alta | Fijar `python:3.12-slim` |
| 2 | Instala `vim`, `git`, `curl`... innecesarios | Cierto. Capa `apt-get install` = 131MB (`docker history`) | Media | Quitar el `apt-get` |
| 3 | `COPY . .` antes de `pip install` → rompe la caché | Cierto. Cualquier cambio reinicia el `pip install` (24.7MB) | Media | Copiar `requirements.txt` primero |
| 4 | Se ejecuta como root (sin `USER`) | Cierto. `User=""` en `docker image inspect` | Alta | `useradd` + `USER appuser` |
| 5 | `CLAVE_ADMIN=admin1234` incrustada en la imagen | Cierto. Legible con `docker image inspect notas-malo --format '{{json .Config.Env}}'` sin arrancar; y `docker run notas-malo sh -c 'echo $CLAVE_ADMIN'` | Alta | Variables en compose/`.env`, nunca en la imagen |
| 6 | Arranca con servidor de desarrollo Flask (`python app.py`, `debug=True`) | Cierto. `Cmd=["/bin/sh","-c","python app.py"]`; `app.py:38` tiene `debug=True` | Alta | `CMD ["gunicorn", ...]` |
| 7 | Sin `EXPOSE` ni `HEALTHCHECK` | Cierto. `Exposed=null`, `Health=null` | Media | `EXPOSE 8000` + `HEALTHCHECK` a `/salud` |

## Problemas del docker-compose.yml

| # | Problema | ¿Cierto? (evidencia) | Gravedad | Corrección |
|---|----------|----------------------|----------|------------|
| 8 | Redis publica `6379:6379` al exterior | Cierto. `docker compose -f sin-revisar/docker-compose.yml config` muestra `published: "6379"`, sin auth | Alta | Quitar `ports` (solo red interna) |
| 9 | Redis sin volumen → pierde datos | Cierto. No hay `volumes` ni healthcheck | Alta | Volumen con nombre (`cache-data:/data`) |
| 10 | `depends_on` sin `condition` | Cierto. Config muestra `condition: service_started`; `web` arranca antes de que Redis esté listo | Media | `condition: service_healthy` + healthcheck |
| 11 | `restart: always` | Matizable, no es un bug. `unless-stopped` es más correcto | Baja | Opcional |

## Punto falso / irrelevante / exagerado (obligatorio)

- **Falso/exagerado:** el informe afirma que "`CMD python app.py` no puede leer `REDIS_URL` porque no hay variable en el Dockerfile".
  Es falso: el Dockerfile define `ENV REDIS_URL=redis://cache:6379` (verificado en `Config.Env`), y `app.py:7` usa
  `os.environ.get("REDIS_URL", default)`, así que tampoco rompería si faltara. El problema real es que la variable
  no debe ir dentro de la imagen, sino en el compose.
- **Irrelevante:** quejarse de que falta `templates/` en el `.dockerignore`: esa carpeta no tiene `.dockerignore` y con
  `COPY . .` sí se copia `templates/`, que la app necesita.

## Contraste obligatorio

| Pregunta | Buena (`notas-web`) | Mala (`notas-malo`) |
|----------|---------------------|---------------------|
| Tamaño | 191MB | 1.84GB |
| Usuario del contenedor | `appuser` (uid 10001) | root (`User=""`) |
| `CLAVE_ADMIN` | No existe | Incrustada en la imagen, legible con `inspect` |
| Servidor web | gunicorn (2 workers) | Flask dev (`python app.py`, debug) |
| Redis al exterior | No publica puertos | Publica 6379 |
| Persistencia | Volumen `cache-data` | Sin volumen |
