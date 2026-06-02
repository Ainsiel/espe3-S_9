# Plan de Despliegue (EventPass)

- **Entorno objetivo:** Amazon EC2 con Docker Compose.
- **Backend:** FastAPI + Uvicorn en contenedor Python.
- **Frontend:** HTML/React estatico servido por Nginx.
- **Persistencia:** SQLite en volumen Docker `eventpass_data`.
- **Automatizacion:** GitHub Actions sobre push a `main`.

## Flujo operativo

1. GitHub Actions ejecuta tests del backend.
2. Si los tests pasan, el workflow entra al EC2 por SSH.
3. En EC2 se actualiza la rama `main`.
4. Se ejecuta `docker compose up -d --build`.
5. Nginx sirve el frontend por el puerto `80` y reenvia `/api/*` al backend.

## Verificacion

```bash
docker compose ps
curl http://localhost/api/events
```

La guia completa esta en `docs/EC2_DEPLOY_GUIDE.md`.
