# Tablón de notas (app + Redis en Docker)

Aplicación Flask que guarda hasta 50 notas en Redis, contenerizada con Docker Compose.

## Requisitos

- Docker Engine (`docker --version`)
- Docker Compose v2 (`docker compose version`)
- Puertos libres: **8000** (web). Redis no publica puertos.

## Variables de entorno

Se definen en `docker-compose.yml`, nunca dentro de la imagen:

| Variable | Servicio | Valor por defecto | Descripción |
|----------|----------|-------------------|-------------|
| `REDIS_URL` | web | `redis://cache:6379` | Conexión a Redis |
| `TITULO_APP` | web | `Tablón de notas` | Título mostrado en la portada |

## Puesta en marcha

```bash
docker compose up -d --build
```

Comprueba que ambos servicios quedan `healthy`:

```bash
docker compose ps
```

La aplicación queda en <http://localhost:8000>.

## Pruebas de comprobación

```bash
curl -s localhost:8000/salud            # {"estado":"ok","notas":N}
curl -s localhost:8000/                 # portada con las notas
curl -s -X POST -d "texto=Hola" localhost:8000/notas   # añade una nota
```

## Dónde quedan los datos

Las notas se guardan en Redis dentro del volumen con nombre **`cache-data`**, montado en `/data`
(persistencia AOF). Los datos sobreviven a `docker compose restart` y a `down` sin `-v`,
pero se pierden con `docker compose down -v`.

## Cómo ver los registros

```bash
docker compose logs -f web      # peticiones (gunicorn con --access-logfile -)
docker compose logs -f cache    # Redis
```

## Cómo parar y limpiar

```bash
docker compose stop             # para sin borrar nada
docker compose down             # para y elimina contenedores y red (conserva datos)
docker compose down -v          # además borra el volumen (se pierden las notas)
docker image rm notas-web:latest
```

## Redespliegue reproducible

Tras limpiar todo, basta con:

```bash
docker compose up -d --build
```
