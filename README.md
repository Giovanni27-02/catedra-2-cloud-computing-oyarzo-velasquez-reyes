# ACI606 - Docker Rescue Challenge (solución reparada)

Aplicación web multi-servicio con Docker Compose:

```
Navegador ──► frontend (Nginx :80, publicado en host :8080)
                 │  /api/*  → proxy a http://backend:8000
                 ▼
              backend (FastAPI :8000, solo red interna)
                 │  postgresql://…@db:5432/appdb
                 ▼
              db (PostgreSQL 16, solo red interna) ──► volumen nombrado pgdata
```

Diagrama: [`docs/arquitectura.md`](docs/arquitectura.md) · Evidencias: [`evidencias/`](evidencias/)

## Requisitos
Docker Engine + Docker Compose v2 (en GitHub Codespaces ya vienen instalados).

## 1. Configurar (opcional)
Todo tiene valores por defecto. Para cambiarlos:
```bash
cp .env.example .env
```
| Variable | Defecto | Uso |
|---|---|---|
| `POSTGRES_USER` | `appuser` | Usuario de PostgreSQL |
| `POSTGRES_PASSWORD` | `rescue123` | Contraseña de PostgreSQL |
| `POSTGRES_DB` | `appdb` | Base de datos |
| `FRONTEND_PORT` | `8080` | Puerto del host para el frontend |

> PostgreSQL aplica usuario y contraseña solo al **crear** el volumen. Si los cambian después: `docker compose down -v` (borra los datos).

## 2. Construir e iniciar
```bash
docker compose up --build -d
docker compose ps
```
Para esperar a que todo esté healthy: `docker compose up --build -d --wait`

## 3. Verificar
```bash
docker compose ps                                   # 3 servicios "Up (healthy)"
curl http://127.0.0.1:8080/api/health               # {"status":"ok","database":"reachable"}
curl -X POST -H "Content-Type: application/json" -d '{"name":"prueba"}' http://127.0.0.1:8080/api/items
curl http://127.0.0.1:8080/api/items
docker compose exec db psql -U appuser -d appdb -c "SELECT * FROM items;"
docker compose logs backend
```
En Codespaces: pestaña **PORTS**, abrir el puerto **8080**.

## 4. Detener
```bash
docker compose stop      # detiene, conserva todo
docker compose down      # borra contenedores y red, CONSERVA el volumen pgdata
docker compose down -v   # borra también el volumen (se pierden los datos)
```

## 5. Recrear
```bash
docker compose down
docker compose up --build -d --wait
curl http://127.0.0.1:8080/api/items   # los datos siguen ahí
```

## Reparaciones realizadas
| ID | Problema | Archivo |
|---|---|---|
| R-01 | `CMD` apuntaba a `app:app` (módulo inexistente) | `backend/Dockerfile` |
| R-02 | Faltaba el driver `psycopg2` | `backend/requirements.txt` |
| R-03 | `DATABASE_URL` usaba `localhost` en vez de `db` | `compose.yaml` |
| R-04 | Contraseña incorrecta en `DATABASE_URL` | `compose.yaml` |
| R-05 | Puerto `8080:8080`, cuando Nginx escucha en 80 | `compose.yaml` |
| R-06 | Nginx hacía proxy a `localhost:8000` | `frontend/nginx.conf` |
| R-07 | Datos en volumen anónimo | `compose.yaml` |
| R-08 | Backend arrancaba antes que la BD | `compose.yaml` |
| R-09 | Nginx guardaba la IP antigua del backend | `frontend/nginx.conf` |

Cada reparación tiene su commit: `git log --oneline`.
````