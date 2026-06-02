# EventPass

Sistema de reserva de entradas con backend FastAPI, frontend estatico y persistencia SQLite.

## Ejecutar con Docker Compose

```bash
cp .env.example .env
docker compose up -d --build
```

Abrir:

```text
http://localhost
```

API:

```text
http://localhost/api/events
```

## Tests del backend

```bash
cd backend
pip install -r requirements.txt
pytest
```

## Deploy en EC2

La guia paso a paso esta en:

```text
docs/EC2_DEPLOY_GUIDE.md
```
