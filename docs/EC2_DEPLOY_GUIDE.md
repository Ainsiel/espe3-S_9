# Guia de despliegue en Amazon EC2 para EventPass

## Opciones para la pipeline

Antes de dejar el deploy final cerrado, hay tres formas razonables:

1. **SSH + git pull + docker compose en EC2**: GitHub Actions entra por SSH al servidor, actualiza la rama `main` y ejecuta `docker compose up -d --build`. Es la opcion mas simple y ya queda preparada en `.github/workflows/deploy-ec2.yml`.
2. **SSH + copia de archivos desde Actions**: GitHub Actions sube el codigo por `scp` o `rsync` y reinicia Docker Compose. Sirve si no quieres configurar acceso Git desde EC2 al repositorio.
3. **Imagenes Docker en registry**: GitHub Actions construye imagenes, las sube a Docker Hub o Amazon ECR, y EC2 solo descarga y reinicia. Es mejor para produccion, pero requiere mas configuracion.

La pipeline incluida usa la opcion 1. Cuando configuremos el EC2, confirma si quieres mantener esta opcion o cambiar a la 2 o 3.

## 1. Crear la instancia EC2

Usa una AMI Ubuntu Server reciente y abre estos puertos en el Security Group:

- `22`: SSH, idealmente limitado a tu IP.
- `80`: HTTP para entrar a EventPass.
- `443`: HTTPS, opcional si despues configuras dominio y certificado.

## 2. Entrar al servidor

Desde tu computador:

```bash
ssh -i tu-key.pem ubuntu@IP_PUBLICA_EC2
```

## 3. Instalar Docker y Git

En EC2:

```bash
sudo apt-get update
sudo apt-get install -y ca-certificates curl git
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo usermod -aG docker ubuntu
```

Cierra sesion y vuelve a entrar por SSH para que el grupo `docker` tome efecto.

## 4. Clonar el repositorio en EC2

```bash
sudo mkdir -p /opt/eventpass
sudo chown -R ubuntu:ubuntu /opt/eventpass
git clone URL_DE_TU_REPOSITORIO_ESPE3_S_9 /opt/eventpass
cd /opt/eventpass
```

Si el repositorio es privado, configura una deploy key o un token para que EC2 pueda hacer `git fetch`.

## 5. Configurar variables de entorno

```bash
cd /opt/eventpass
cp .env.example .env
nano .env
```

Cambia al menos:

```env
APP_PORT=80
DATABASE_URL=sqlite:////data/db.sqlite3
SECRET_KEY=pon-un-secreto-largo-y-unico
CORS_ORIGINS=*
```

La base de datos SQLite queda en un volumen Docker llamado `eventpass_data`, por eso no se sube `db.sqlite3` al repositorio.

## 6. Primer deploy manual

```bash
cd /opt/eventpass
docker compose up -d --build
docker compose ps
```

Prueba la API:

```bash
curl http://localhost/api/events
```

Luego abre en el navegador:

```text
http://IP_PUBLICA_EC2
```

## 7. Configurar GitHub Actions

En GitHub, entra al repositorio `espe3-S_9` y ve a:

`Settings > Secrets and variables > Actions > New repository secret`

Crea estos secretos:

- `EC2_HOST`: IP publica o dominio del EC2.
- `EC2_USER`: usuario SSH, por ejemplo `ubuntu`.
- `EC2_SSH_KEY`: contenido completo de la llave privada que entra al EC2.
- `EC2_APP_DIR`: ruta del proyecto en EC2, por ejemplo `/opt/eventpass`. Es opcional; si falta, usa `/opt/eventpass`.

Cuando hagas push a `main`, el workflow ejecuta tests y despues:

```bash
git fetch origin main
git reset --hard origin/main
docker compose up -d --build
```

## 8. Comandos utiles

Ver logs:

```bash
docker compose logs -f
```

Reiniciar:

```bash
docker compose restart
```

Actualizar manualmente:

```bash
cd /opt/eventpass
git pull origin main
docker compose up -d --build
```

Detener:

```bash
docker compose down
```
