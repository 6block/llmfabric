# Installation

## Prerequisites

**{{ brand.name }} server:**

- [Docker](https://docs.docker.com/engine/install/) must be installed. Docker Desktop (Windows and macOS) is also supported.

**{{ brand.name }} workers:**

- [Docker](https://docs.docker.com/engine/install/) must be installed. Docker Desktop is **not** supported.
- Only Linux is supported for {{ brand.name }} worker nodes. If you use Windows, consider using WSL2 and avoid using Docker Desktop. macOS is not supported for {{ brand.name }} worker nodes.
- Ensure the appropriate GPU drivers and container toolkits are installed for your hardware. See the [Installation Requirements](./requirements.md) for details.

## Install {{ brand.name }} Server

Run the following command to install and start the {{ brand.name }} server using Docker:

```bash
sudo docker run -d --name {{ brand.executable_name }} \
    --restart unless-stopped \
    -p 80:80 \
    --volume {{ brand.volume_name }}:{{ brand.data_dir }} \
    {{ brand.docker_image }}
```

!!! note

    {{ brand.name }} v2 uses a single unified container image for all GPU device types.

## Startup

Check the {{ brand.name }} container logs:

```bash
sudo docker logs -f {{ brand.executable_name }}
```

If everything is normal, open `http://your_host_ip` in a browser to access the {{ brand.name }} UI.

Log in with username `admin` and the default password. Retrieve the initial password with:

```bash
sudo docker exec -it {{ brand.executable_name }} \
    cat {{ brand.data_dir }}/initial_admin_password
```

## Add GPU Clusters and Worker Nodes

Please follow the UI instructions on the `Clusters` and `Workers` pages to add GPU clusters and worker nodes.

## Custom Configuration

The following sections describe examples of custom configuration options when starting the {{ brand.name }} server container. For a full list of available options, refer to the [CLI Reference](../cli-reference/start.md).

### Enable HTTPS with Custom Certificate


```diff
 sudo docker run -d --name {{ brand.executable_name }} \
     ...
     -p 80:80 \
+    -p 443:443 \
     --volume {{ brand.volume_name }}:{{ brand.data_dir }} \
+    --volume /path/to/cert_files:/path/to/cert_files:ro \
+    -e {{ brand.env_prefix }}_SSL_KEYFILE=/path/to/cert_files/your_domain.key \
+    -e {{ brand.env_prefix }}_SSL_CERTFILE=/path/to/cert_files/your_domain.crt \
     {{ brand.docker_image }}
     ...
```

### Using an External Database

By default, {{ brand.name }} uses an embedded PostgreSQL database. To use an external database such as PostgreSQL or MySQL, set the `{{ brand.env_prefix }}_DATABASE_URL` environment variable or use the `--database-url` argument when starting the {{ brand.name }} container:

```diff
 sudo docker run -d --name {{ brand.executable_name }} \
     ...
     --volume {{ brand.volume_name }}:{{ brand.data_dir }} \
+    -e {{ brand.env_prefix }}_DATABASE_URL="postgresql://username:password@host:port/dbname" \
     {{ brand.docker_image }}
     ...
```

### Configure External Server URL

If you use a cloud provider to provision workers, set the external server URL for worker registration to ensure that workers can connect to the server correctly.

```diff
sudo docker run -d --name {{ brand.executable_name }} \
    ...
+   -e {{ brand.env_prefix }}_SERVER_EXTERNAL_URL="https://your_external_server_url" \
    {{ brand.docker_image }}
    ...
```

### Additional Trusted CAs

If {{ brand.name }} needs to communicate with services that use certificates issued by a private or corporate CA (e.g., a self-hosted Identity Provider, a Hugging Face mirror, or an internal API endpoint), mount the CA certificate into the container under `/usr/local/share/ca-certificates/`. {{ brand.name }} will automatically import the mounted CA certificates during startup and add them to the system trust store.

```diff
 sudo docker run -d --name {{ brand.executable_name }} \
     ...
     --volume {{ brand.volume_name }}:{{ brand.data_dir }} \
+    --volume /path/to/custom-root-ca.crt:/usr/local/share/ca-certificates/custom-root-ca.crt:ro \
     {{ brand.docker_image }}
     ...
```

!!! note

    The certificate file must have a `.crt` extension. You can mount multiple CA certificates by adding additional `--volume` flags.

## Installation via Docker Compose

### Prerequisites

- [Docker Compose](https://docs.docker.com/compose/install/) must be installed.
- [Required ports](./requirements.md#port-requirements) must be available.

### Deployment

The Docker Compose files and configuration files are maintained in the [{{ brand.name }} repository](https://github.com/{{ brand.github_repo }}/tree/main/docker-compose).

Run the following commands to clone the latest stable release:

```bash
LATEST_TAG=$(
    curl -s "https://api.github.com/repos/{{ brand.docker_image }}/releases" \
    | grep '"tag_name"' \
    | sed -E 's/.*"tag_name": "([^"]+)".*/\1/' \
    | grep -Ev 'rc|beta|alpha|preview' \
    | head -1
)
echo "Latest stable release: $LATEST_TAG"
git clone -b "$LATEST_TAG" https://github.com/{{ brand.github_repo }}.git
cd {{ brand.executable_name }}/docker-compose
```

Start the {{ brand.name }} server:

```bash
sudo docker compose -f docker-compose.server.yaml up -d
```

If everything is normal, open `http://your_host_ip` in a browser to access the {{ brand.name }} UI.

Log in with username `admin` and the default password. Retrieve the initial password with:

```bash
sudo docker exec -it {{ brand.executable_name }}-server cat {{ brand.data_dir }}/initial_admin_password
```

For built-in and external observability options, see [Observability](../user-guide/observability.md).
