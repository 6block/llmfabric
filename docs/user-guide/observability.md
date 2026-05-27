# Observability

This document describes how to monitor {{ brand.name }} Server/Worker/LLM serving runtime metrics using Prometheus and Grafana.

## Overview

{{ brand.name }} provides a comprehensive set of metrics for model serving and GPU resource management. By integrating Prometheus and Grafana, users can collect, store, and visualize these metrics in real time, enabling efficient monitoring and troubleshooting.

## Built-in Observability (Default)

By default, {{ brand.name }} starts with an embedded Prometheus and Grafana. You can access them via:

- **Prometheus**: `http://your_{{ brand.executable_name }}_server_host_ip/prometheus`
- **Grafana**: `http://your_{{ brand.executable_name }}_server_host_ip/grafana`

Built-in Grafana is configured for anonymous Viewer access and has the login form disabled. Admin credentials remain `admin` / `grafana` by default.

## External Observability (Optional)

If you want an external Prometheus/Grafana stack, we recommend using the provided Docker Compose files:

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

Before starting, set `{{ brand.env_prefix }}_GRAFANA_URL` to a browser-reachable Grafana URL (not a container-only hostname like `grafana`).

Start external Prometheus/Grafana (this disables the built-in stack):

```bash
sudo docker compose -f docker-compose.external-observability.yaml up -d
```

If you already have an external Prometheus/Grafana stack, you can configure it manually instead:

1. **Configure Prometheus to scrape {{ brand.name }} metrics**  
   Add targets for the {{ brand.name }} metrics endpoint (default `:10161`) and worker discovery endpoint. Example:

   ```yaml
   scrape_configs:
     - job_name: {{ brand.executable_name }}-worker-discovery
       scrape_interval: 5s
       http_sd_configs:
         - url: "http://<{{ brand.executable_name }}_server_host>:10161/metrics/targets"
           refresh_interval: 1m
     - job_name: {{ brand.executable_name }}-server
       scrape_interval: 10s
       static_configs:
         - targets: ["<{{ brand.executable_name }}_server_host>:10161"]
   ```
2. **Import {{ brand.name }} dashboards into Grafana**  
   Use the dashboards provided in the `docker-compose/grafana/grafana_dashboards/` directory as a starting point.
3. **Point {{ brand.name }} to your Grafana**  
   Set `{{ brand.env_prefix }}_GRAFANA_URL` to the externally reachable Grafana URL so dashboard redirects work. This must be a browser-reachable URL.

## Accessing Metrics

- **{{ brand.name }} Metrics Endpoint**:  
  Access metrics at `http://<{{ brand.executable_name }}_server_host>:10161/metrics`
- **{{ brand.name }} Worker Metrics Targets**:  
  Access metrics at `http://<{{ brand.executable_name }}_server_host>:10161/metrics/targets`
- **Prometheus UI**:  
  Access Prometheus at `http://<host>:19090` by default, or the port configured by `--builtin-prometheus-port` / `{{ brand.env_prefix }}_BUILTIN_PROMETHEUS_PORT`.
- **Grafana UI**:  
  Access Grafana at `http://<host>:13000` by default, or the port configured by `--builtin-grafana-port` / `{{ brand.env_prefix }}_BUILTIN_GRAFANA_PORT`. Built-in Grafana is configured for anonymous Viewer access with the login form disabled. The admin credentials remain `admin` / `grafana` by default.

## Migration from Older Compose Setups

If you previously used Docker Compose to run Prometheus/Grafana alongside {{ brand.name }}:

- **Keep external observability (recommended for continuity)**:  
  Leave your existing Prometheus/Grafana containers running. Update Prometheus scrape targets to the new {{ brand.name }} metrics endpoint and set `{{ brand.env_prefix }}_GRAFANA_URL` to your existing Grafana.

- **Switch to built-in observability**:  
  Stop the old Prometheus/Grafana containers, then use the latest `docker-compose.server.yaml` ({{ brand.name }} only). Built-in Grafana/Prometheus will take over. Historical metrics from the old Prometheus will not be migrated unless you keep the old stack read-only.

## Customizing Metrics Mapping

{{ brand.name }} supports dynamic customization of metrics mapping through its configuration API. This allows you to update how runtime engine metrics are mapped to {{ brand.name }} metrics without restarting the service. The configuration is managed centrally on the server and can be accessed or modified via HTTP API.

### API Endpoints

- **Get Current Metrics Config**

  - GET `http://<{{ brand.executable_name }}_server_host>:<{{ brand.executable_name }}_server_port>/v2/metrics/config`
  - Returns the current metrics mapping configuration in JSON format.

- **Update Metrics Config**

  - POST `http://<{{ brand.executable_name }}_server_host>:<{{ brand.executable_name }}_server_port>/v2/metrics/config`
  - Accepts a JSON payload to update the metrics mapping configuration. Changes take effect immediately for all workers.

- **Get Default Metrics Config**
  - GET `http://<{{ brand.executable_name }}_server_host>:<{{ brand.executable_name }}_server_port>/v2/metrics/default-config`
  - Returns the default metrics mapping configuration in JSON format, useful for reference or resetting.

### Example Usage

**Get current config:**

```bash
curl http://<{{ brand.executable_name }}_server_host>:<{{ brand.executable_name }}_server_port>/v2/metrics/config
```

**Update config:**

```bash
curl -X POST http://<{{ brand.executable_name }}_server_host>:<{{ brand.executable_name }}_server_port>/v2/metrics/config \
     -H "Content-Type: application/json" \
     -d @custom_metrics_config.json
```

_(where `custom_metrics_config.json` is your new config file)_

**Get default config:**

```bash
curl -X POST http://<{{ brand.executable_name }}_server_host>:<{{ brand.executable_name }}_server_port>/v2/metrics/default-config
```

> **Note**: The configuration should be provided in valid JSON format.

## Metrics Exposed by {{ brand.name }}

The following metrics are exposed by {{ brand.name }} and can be scraped by Prometheus. Each metric includes hierarchical labels for cluster, worker, model, and instance identification.

### LLM Serving Runtime Metrics

| Metric Name                            | Type      | Description                                                                 |
| -------------------------------------- | --------- | --------------------------------------------------------------------------- |
| {{ brand.executable_name }}:num_requests_running          | Gauge     | Number of requests currently being processed.                               |
| {{ brand.executable_name }}:num_requests_waiting          | Gauge     | Number of requests waiting in the queue.                                    |
| {{ brand.executable_name }}:num_requests_swapped          | Gauge     | Number of requests swapped out to CPU.                                      |
| {{ brand.executable_name }}:prefix_cache_hit_rate         | Gauge     | Prefix cache hit rate.                                                      |
| {{ brand.executable_name }}:kv_cache_usage_ratio          | Gauge     | KV-cache usage ratio. 1.0 means fully used.                                 |
| {{ brand.executable_name }}:prefix_cache_queries          | Counter   | Number of prefix cache queries (total tokens).                              |
| {{ brand.executable_name }}:prefix_cache_hits             | Counter   | Number of prefix cache hits (total tokens).                                 |
| {{ brand.executable_name }}:prompt_tokens                 | Counter   | Total number of prefill tokens processed.                                   |
| {{ brand.executable_name }}:generation_tokens             | Counter   | Total number of generated tokens.                                           |
| {{ brand.executable_name }}:request_prompt_tokens         | Histogram | Number of prefill tokens processed per request.                             |
| {{ brand.executable_name }}:request_generation_tokens     | Histogram | Number of generation tokens processed per request.                          |
| {{ brand.executable_name }}:time_to_first_token_seconds   | Histogram | Time to generate first token.                                               |
| {{ brand.executable_name }}:inter_token_latency_seconds   | Histogram | Time to generate the next token after the previous token has been produced. |
| {{ brand.executable_name }}:time_per_output_token_seconds | Histogram | Time per generated token.                                                   |
| {{ brand.executable_name }}:e2e_request_latency_seconds   | Histogram | End-to-end request latency.                                                 |
| {{ brand.executable_name }}:request_success               | Counter   | Total number of successful requests.                                        |

These metrics are mapped from various runtime engines (vLLM, SGLang, MindIE) as defined in metrics_config.yaml.

### Worker Metrics

| Metric Name                                      | Type  | Description                                      |
| ------------------------------------------------ | ----- | ------------------------------------------------ |
| {{ brand.executable_name }}:worker_status                           | Gauge | Worker status (with state label).                |
| {{ brand.executable_name }}:worker_node_os                          | Info  | Operating system information of the worker node. |
| {{ brand.executable_name }}:worker_node_kernel                      | Info  | Kernel information of the worker node.           |
| {{ brand.executable_name }}:worker_node_uptime_seconds              | Gauge | Uptime in seconds of the worker node.            |
| {{ brand.executable_name }}:worker_node_cpu_cores                   | Gauge | Total CPU cores of the worker node.              |
| {{ brand.executable_name }}:worker_node_cpu_utilization_rate        | Gauge | CPU utilization rate of the worker node.         |
| {{ brand.executable_name }}:worker_node_memory_total_bytes          | Gauge | Total memory in bytes of the worker node.        |
| {{ brand.executable_name }}:worker_node_memory_used_bytes           | Gauge | Memory used in bytes of the worker node.         |
| {{ brand.executable_name }}:worker_node_memory_utilization_rate     | Gauge | Memory utilization rate of the worker node.      |
| {{ brand.executable_name }}:worker_node_gpu                         | Info  | GPU information of the worker node.              |
| {{ brand.executable_name }}:worker_node_gpu_cores                   | Gauge | Total GPU cores of the worker node.              |
| {{ brand.executable_name }}:worker_node_gpu_utilization_rate        | Gauge | GPU utilization rate of the worker node.         |
| {{ brand.executable_name }}:worker_node_gpu_temperature_celsius     | Gauge | GPU temperature in Celsius.                      |
| {{ brand.executable_name }}:worker_node_gram_total_bytes            | Gauge | Total GPU RAM in bytes.                          |
| {{ brand.executable_name }}:worker_node_gram_allocated_bytes        | Gauge | Allocated GPU RAM in bytes.                      |
| {{ brand.executable_name }}:worker_node_gram_used_bytes             | Gauge | Used GPU RAM in bytes.                           |
| {{ brand.executable_name }}:worker_node_gram_utilization_rate       | Gauge | GPU RAM utilization rate.                        |
| {{ brand.executable_name }}:worker_node_filesystem_total_bytes      | Gauge | Total filesystem size in bytes.                  |
| {{ brand.executable_name }}:worker_node_filesystem_used_bytes       | Gauge | Used filesystem size in bytes.                   |
| {{ brand.executable_name }}:worker_node_filesystem_utilization_rate | Gauge | Filesystem utilization rate.                     |

### Server Metrics

| Metric Name                      | Type  | Description                                       |
| -------------------------------- | ----- | ------------------------------------------------- |
| {{ brand.executable_name }}:cluster                 | Info  | Cluster information (ID, name, provider).         |
| {{ brand.executable_name }}:cluster_status          | Gauge | Cluster status (with state label).                |
| {{ brand.executable_name }}:model                   | Info  | Model information (ID, name, runtime, source).    |
| {{ brand.executable_name }}:model_desired_instances | Gauge | Desired number of model instances.                |
| {{ brand.executable_name }}:model_running_instances | Gauge | Number of running model instances.                |
| {{ brand.executable_name }}:model_instance_status   | Gauge | Status of each model instance (with state label). |

> **Note**: All metrics are labeled with relevant identifiers (cluster, worker, model, instance, user) for fine-grained monitoring and filtering.
