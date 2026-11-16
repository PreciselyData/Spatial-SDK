# Local Docker Desktop

Run the Precisely Spatial SDK services locally for development and testing.

## Prerequisites

- Docker Desktop with Docker Compose v2.
- A container registry containing the Spatial SDK images. The default is a local registry at `localhost:5000`.
- A host data folder accessible to Docker Desktop. The default is `C:/spatial`.

## Step 1: Download Reference Data and Required Docker Images

Download the Spatial SDK image archives and any licensed reference data using your Precisely entitlements. See the [Precisely spatial SDK downloads](https://pdt.data.precisely.com/software/families/Precisely%20Data%20Integrity%20Suite/delivery-groups/spatial-sdk) for available downloads. Extract the image archives so the `.tar` files are together in one directory.

The default `IMAGE_REGISTRY` in `.env` is `localhost:5000`. Start a local registry with persistent storage, or use a registry you already have:

```powershell
docker run -d --name spatial-sdk-registry --restart=always -p 5000:5000 -v spatial-sdk-registry-data:/var/lib/registry registry:2.7
```

The image-push scripts are Bash scripts. From the repository root, run one from Git Bash, WSL, or another Bash environment with Docker access. Use the same tag as `IMAGE_TAG` in `docker-desktop/.env`:

```bash
bash scripts/aks/push-images.sh --tag latest localhost:5000 /path/to/extracted-images
```

Bulk mode expects all nine image archives listed in the cloud quickstarts. This Docker Compose setup uses eight of them and does not run `private-sdk-mcp` or AI workflows. To push only the images used by Compose, run the script once for each available service image, using this form:

```bash
bash scripts/aks/push-images.sh --tag latest localhost:5000 feature-service /path/to/extracted-images
```

Compose image names are `resource-service`, `feature-service`, `tiling-service`, `mapping-service`, `composite-service`, `spatial-platform-ux-frontend`, `data-service`, and `samples-data`. For a remote registry, authenticate with `docker login` and replace `localhost:5000` with its registry/repository prefix. Set `IMAGE_REGISTRY` in `.env` to that same prefix.

## Step 2: Prepare Credentials

Before the first start, replace the three placeholders in `../deploy/realm-spatial.json` with strong, unique values:

| Placeholder | Use |
| --- | --- |
| `REPLACE_WITH_SPATIAL_CLIENT_SECRET` | Secret for the `spatial` client. |
| `REPLACE_WITH_SPATIAL_DATA_LOAD_CLIENT_SECRET` | Secret for the `spatial-data-load` client. |
| `REPLACE_WITH_SPATIAL_ADMIN_PASSWORD` | Password for the realm's `admin` user. |

Copy the two client-secret values exactly into the matching entries in `service-common.env`. The realm admin password is separate from the Keycloak bootstrap account, which is currently configured as `admin` / `admin` in `docker-compose.yml`. These are local-development credentials; do not use them in shared or production environments.

The realm admin password is not temporary. After importing the realm, the configured password remains active. If Keycloak has already imported the realm, editing the JSON file alone does not change existing credentials. Update them in the Keycloak Admin Console, or recreate the Keycloak container to reimport the realm. Recreating it resets local Keycloak state; MongoDB data is stored separately in named volumes.

## Step 3: Configure

The stack reads configuration from:

| File | Settings |
| --- | --- |
| `.env` | Compose profiles, image tag, host ports, data path, and MongoDB URI. |
| `service-common.env` | Shared Spatial and OAuth2 application settings. |
| `logging-vars.env` | Logging and Hibernate settings for services that load it. |
| `../deploy/realm-spatial.json` | Realm, clients, roles, and sample users imported by Keycloak. |

Review `.env` before starting. Set `SPATIAL_DATA_PATH` to an existing host folder if you do not want to use `C:/spatial`. `SPRING_MONGODB_URI` must use the Compose hostname `mongoDB` for connections from containers. Spatial images use `${IMAGE_REGISTRY}/<image-name>:${IMAGE_TAG}`; set `IMAGE_REGISTRY` to the registry/repository prefix where the images were pushed. The default, `localhost:5000`, requires a local registry to be running. The setup-resources entries in `.env` are not used by the current Compose file.

`COMPOSE_PROFILES` currently enables all defined services: Keycloak, MongoDB, sample data, Resource, Feature, Tiling, Mapping, Composite, Spatial Platform frontend, and Data Service. To run a subset, edit the comma-separated profile list and include the dependencies required by the selected services.

## Step 4: Start

After the images are pushed and the registry is running, run these commands from the `docker-desktop` directory:

```powershell
docker compose pull
docker compose up -d
docker compose ps
```

## Step 5: Verify Services and Logs

| Service | Local address / port |
| --- | --- |
| Keycloak | `http://localhost:8088` |
| MongoDB | `localhost:27017` |
| Resource Service | `http://localhost:8090` |
| Feature Service | `http://localhost:8040` |
| Tiling Service | `http://localhost:8060` |
| Mapping Service | `http://localhost:8032` |
| Composite Service | `http://localhost:8070` |
| Spatial Platform frontend | `http://localhost:3000` |
| Data Service | `http://localhost:8080` |

Sample data maps host port `8075` to its configured debug port. Other Java services also expose debug ports; see `docker-compose.yml` for details.

MongoDB has a health check. Resource Service waits for MongoDB to become healthy; Feature, Tiling, and Mapping wait for healthy MongoDB and for the Resource Service container to start. Resource Service has no Compose health check, so `service_started` does not guarantee application readiness. All services use `on-failure:5`; a successful one-shot sample-data run is not restarted.

Check status and follow service logs with:

```powershell
docker compose ps -a
docker compose logs -f resource-service keycloak
```

## Step 6: Stop and Reset

Stop and remove the containers and network:

```powershell
docker compose down
```

This keeps the MongoDB named volumes. To remove the local database data as well, run `docker compose down -v`.

## Related Guides

- [Helm chart documentation](../charts/precisely-spatial-sdk/README.md)
- [JDBC repository configuration](../docs/guides/JDBC-repository.md)
- [Identity provider integration](../docs/guides/IDP-integration.md)