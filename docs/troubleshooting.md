# Troubleshooting

## View {{ brand.name }} Logs

You can view {{ brand.name }} logs with the following commands for the default setup:

```bash
docker logs -f {{ brand.executable_name }}
```

## Enable Debug Mode

You can enable the `DEBUG` mode by setting the `--debug` flag when running {{ brand.name }}:

```diff
sudo docker run -d --name {{ brand.executable_name }} \
    ...
    {{ brand.docker_image }} \
+    --debug
    ...
```

You can also enable {{ brand.name }}'s debug mode at runtime by running the following command inside the **server container**:

```bash
{{ brand.executable_name }} reload-config --set debug=true
```

## Configure Log Level

You can configure log level of the {{ brand.name }} server at runtime by running the following command inside the **server container**:

```bash
curl -X PUT http://localhost/debug/log_level -d "debug"
```

The same applies to {{ brand.name }} workers:

```bash
curl -X PUT http://localhost:10150/debug/log_level -d "debug"
```

The available log levels are: `trace`, `debug`, `info`, `warning`, `error`, `critical`.

## Reset Admin Password

In case you forgot the admin password, you can reset it by running the following command inside the **server container**:

```bash
{{ brand.executable_name }} reset-admin-password
```

If you changed the default port using `--port` when starting {{ brand.name }}, specify the {{ brand.name }} URL using the `--server-url` parameter. It must be run locally on the server and accessed via `localhost`:

```bash
{{ brand.executable_name }} reset-admin-password --server-url http://localhost:9090
```

## Assist in Accelerators Detection Diagnosis

After successfully deploying the {{ brand.name }} Worker as described in the [installation guide](./installation/requirements.md),  
if the Worker fails to detect any devices,  
please enter the corresponding Worker container, run the following command, and report the results to [{{ brand.name }}](https://github.com/{{ brand.github_repo }}/issues).

```bash
time {{ brand.env_prefix }}_RUNTIME_LOG_LEVEL=debug {{ brand.env_prefix }}_RUNTIME_LOG_EXCEPTION=1 {{ brand.runtime_name }} detect --format json
```

## Assist in Model Deployment Diagnosis

If you experience issues after deploying a model, 
please enter the corresponding Worker container, run the following command, and report the results to [{{ brand.name }}](https://github.com/{{ brand.github_repo }}/issues).

```bash
{{ brand.runtime_name }} inspect <model instance name>
```
