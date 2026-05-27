# Uninstallation

{{ brand.name }} is typically installed using containerization, 
so uninstallation mainly involves removing the container and any associated data volumes.

For example, if {{ brand.name }} is running in a Docker container named `{{ brand.executable_name }}`, run:

```bash
docker rm -f {{ brand.executable_name }}

```

To optionally remove associated data volumes, use:

```bash
docker volume rm <data_volume_name>

```
