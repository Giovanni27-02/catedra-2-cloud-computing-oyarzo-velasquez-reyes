# Arquitectura

## Inicial (paquete recibido)

```mermaid
flowchart LR
    U[Navegador] -- "host:8080 → contenedor:8080 ✗" --> F
    subgraph red["Red: catedra-2-cloud-computing-oyarzo-velasquez-reyes_default"]
      F["frontend<br/>Nginx listen 80"]
      B["backend<br/>CMD app:app ✗<br/>sin psycopg2 ✗"]
      D[("db<br/>postgres:16<br/>volumen anónimo ✗")]
    end
    F -- "proxy_pass localhost:8000 ✗" --> B
    B -- "wrongpass@localhost ✗" --> D
```

## Final (reparada)

```mermaid
flowchart LR
    U[Navegador] -- "host:8080 → :80" --> F
    subgraph red["Red bridge (DNS interno 127.0.0.11)"]
      F["frontend · Nginx :80<br/>healthcheck: wget"]
      B["backend · FastAPI :8000<br/>healthcheck: /health (SELECT 1)"]
      D[("db · PostgreSQL :5432<br/>healthcheck: pg_isready")]
    end
    V[["volumen pgdata"]]
    E[/".env<br/>POSTGRES_* · FRONTEND_PORT"/]
    F -- "/api/* → backend:8000" --> B
    B -- "db:5432" --> D
    D --- V
    E -.-> F & B & D
```

Arranque: `db (healthy) → backend (healthy) → frontend`. Solo se publica el puerto 8080.
````

> GitHub dibuja automáticamente los diagramas `mermaid`. Sirven para la sección "diagrama de arquitectura" que pide el repo.