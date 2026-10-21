# Reto (ampliación, no puntúa)

## 1. Dockerfile multietapa

`Dockerfile.multistage` instala las dependencias en una etapa `builder` con `pip install --prefix=/install`
y en la etapa final (`runtime`) copia solo `/install` a `/usr/local`. Resultado:

| Imagen | Tamaño |
|--------|--------|
| `notas-web:latest` (una etapa) | 191MB |
| `notas-web-ms:latest` (multietapa) | 191MB |

Ambas pesan igual porque la app y las dependencias de Python ya son ligeras, y la etapa builder parte de la
misma base `python:3.12-slim`. El multietapa **no reduce** el tamaño aquí (no se compilan binarios con
herramientas de build), pero sí aislaría artefactos de compilación si los hubiera.

## 2. Proxy Nginx delante de gunicorn

Se añade `docker-compose.nginx.yml` con un servicio `proxy` (`nginx:1.27-alpine`) que escucha en `8080:80`
y redirige a `web:8000` mediante `nginx.conf` (upstream gunicorn).

Comprobación con `curl -I`:

| Acceso | Cabecera `Server` |
|--------|-------------------|
| Directo a gunicorn (`web:8000`, sin publicar puerto al exterior) | `gunicorn` |
| A través de Nginx (`localhost:8080`) | `nginx/1.27.5` |

La app funciona igual a través del proxy: `GET /salud` → `{"estado":"ok",...}` y `POST /notas` → 302,
mostrando las notas en la portada.
