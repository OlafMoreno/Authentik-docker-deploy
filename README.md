# Authentik con Docker Compose

Despliegue de [Authentik](https://goauthentik.io/) con Docker Compose. El stack incluye PostgreSQL, Redis, el servidor web de Authentik y su worker.

## Requisitos

- Docker Engine y Docker Compose v2 (`docker compose`).
- Puertos `9000` (HTTP) y `9443` (HTTPS) disponibles, o puertos alternativos configurados en `.env`.

## Puesta en marcha

1. Clona este repositorio y entra en su directorio.
2. Crea el archivo de configuración local:

   ```powershell
   Copy-Item .env.example .env
   ```

   En Linux o macOS:

   ```sh
   cp .env.example .env
   ```

3. Edita `.env` y sustituye **ambos valores de ejemplo** por secretos únicos. Puedes generar valores con:

   ```sh
   openssl rand -hex 32
   openssl rand -base64 36
   ```

   Usa el primer resultado para `PG_PASS` y el segundo para `AUTHENTIK_SECRET_KEY`. No compartas ni subas `.env` al repositorio.
4. Inicia los servicios:

   ```sh
   docker compose up -d
   ```

5. Abre `http://localhost:9000/if/flow/initial-setup/` para completar la configuración inicial. La URL debe terminar en `/`.

Para comprobar el estado y consultar los registros:

```sh
docker compose ps
docker compose logs -f server worker
```

Para detener los servicios sin borrar los datos:

```sh
docker compose down
```

## Servicios y datos

| Servicio | Función |
| --- | --- |
| `postgresql` | Base de datos PostgreSQL 16; espera a que la comprobación de salud esté lista. |
| `redis` | Redis para tareas y comunicación interna. |
| `server` | Interfaz y API de Authentik; escucha en los puertos HTTP y HTTPS configurados. |
| `worker` | Procesos en segundo plano; monta `media/`, `custom-templates/` y `certs/`. |

La base de datos y Redis usan volúmenes Docker con nombre (`database` y `redis`), que sobreviven a `docker compose down`. Los archivos de Authentik se montan desde `media/`, las plantillas personalizadas desde `custom-templates/` y los certificados desde `certs/`. Los contenidos locales de `media/` y `certs/` están excluidos de Git.

> **Importante:** el worker se ejecuta como `root` y monta `/var/run/docker.sock`. El acceso al socket de Docker puede dar control elevado sobre el host; protege este despliegue y no expongas el socket ni el panel de Authentik a redes no confiables.

## Configuración

Las variables requeridas están en `.env.example`. Los valores opcionales y sus valores predeterminados son:

| Variable | Predeterminado | Descripción |
| --- | --- | --- |
| `PG_USER` | `authentik` | Usuario de PostgreSQL. |
| `PG_DB` | `authentik` | Base de datos de PostgreSQL. |
| `AUTHENTIK_TAG` | `2024.8.3` | Etiqueta de la imagen de Authentik. |
| `AUTHENTIK_IMAGE` | `ghcr.io/goauthentik/server` | Imagen de Authentik. |
| `AUTHENTIK_PORT_HTTP` | `9000` | Puerto HTTP del host. |
| `AUTHENTIK_PORT_HTTPS` | `9443` | Puerto HTTPS del host. |
| `AUTHENTIK_ERROR_REPORTING__ENABLED` | `false` | Habilita el envío de informes de errores a Authentik. |

Para cambiar la versión, ajusta `AUTHENTIK_TAG` en `.env` y recrea los servicios. Antes de actualizar, consulta las instrucciones de actualización y respaldo de la [documentación oficial de Authentik](https://docs.goauthentik.io/).

No uses `docker compose down -v` salvo que quieras eliminar también los volúmenes y **borrar permanentemente la base de datos y los datos de Redis**.
