# OpenTelemetry Demo Helm Chart

The helm chart installs [OpenTelemetry Demo](https://github.com/open-telemetry/opentelemetry-demo)
in kubernetes cluster.

## Prerequisites

- Kubernetes 1.24+
- Helm 3.14+

## Installing the Chart

Add OpenTelemetry Helm repository:

```console
helm dependency update .
```

Lint Check

```console
helm lint .
```

To install the chart with the release name my-otel-demo, run the following command:

```console
helm install my-otel-demo .
```

## Upgrading

See [UPGRADING.md](UPGRADING.md).


## Chart Parameters

Chart parameters are separated in 4 general sections:
* Default - Used to specify defaults applied to all demo components
* Components - Used to configure the individual components (microservices) for
the demo
* Observability - Used to enable/disable dependencies
* Sub-charts - Configuration for all sub-charts

### Sub-charts

The OpenTelemetry Demo Helm chart depends on 5 sub-charts:
* OpenTelemetry Collector
* Jaeger
* Prometheus
* Grafana
* OpenSearch

Parameters for each sub-chart can be specified within that sub-chart's
respective top level. This chart will override some of the dependent sub-chart
parameters by default. The overriden parameters are specified below.

#### OpenTelemetry Collector

> **Note**
> The following parameters have a `opentelemetry-collector.` prefix.

| Parameter        | Description                                        | Default                                                  |
|------------------|----------------------------------------------------|----------------------------------------------------------|
| `enabled`        | Install the OpenTelemetry collector                | `true`                                                   |
| `nameOverride`   | Name that will be used by the sub-chart release    | `otel-collector`                                                |
| `mode`           | The Deployment or Daemonset mode                   | `deployment`                                             |
| `resources`      | CPU/Memory resource requests/limits                | 100Mi memory limit                                       |
| `service.type`   | Service Type to use                                | `ClusterIP`                                              |
| `ports`          | Ports to enabled for the collector pod and service | `metrics` is enabled and `prometheus` is defined/enabled |
| `podAnnotations` | Pod annotations                                    | Annotations leveraged by Prometheus scrape               |
| `config`         | OpenTelemetry Collector configuration              | Configuration required for demo                          |

#### Jaeger

> **Note**
> The following parameters have a `jaeger.` prefix.

| Parameter                      | Description                                        | Default                                                               |
|--------------------------------|----------------------------------------------------|-----------------------------------------------------------------------|
| `enabled`                      | Install the Jaeger sub-chart                       | `true`                                                                |
| `provisionDataStore.cassandra` | Provision a cassandra data store                   | `false` (required for AllInOne mode)                                  |
| `allInOne.enabled`             | Enable All in One In-Memory Configuration          | `true`                                                                |
| `allInOne.args`                | Command arguments to pass to All in One deployment | `["--memory.max-traces", "10000", "--query.base-path", "/jaeger/ui"]` |
| `allInOne.resources`           | CPU/Memory resource requests/limits for All in One | 275Mi memory limit                                                    |
| `storage.type`                 | Storage type to use                                | `none` (required for AllInOne mode)                                   |
| `agent.enabled`                | Enable Jaeger agent                                | `false` (required for AllInOne mode)                                  |
| `collector.enabled`            | Enable Jaeger Collector                            | `false` (required for AllInOne mode)                                  |
| `query.enabled`                | Enable Jaeger Query                                | `false` (required for AllInOne mode)                                  |

#### Prometheus

> **Note**
> The following parameters have a `prometheus.` prefix.

| Parameter                            | Description                                    | Default                                                   |
|--------------------------------------|------------------------------------------------|-----------------------------------------------------------|
| `enabled`                            | Install the Prometheus sub-chart               | `true`                                                    |
| `alertmanager.enabled`               | Install the alertmanager                       | `false`                                                   |
| `configmapReload.prometheus.enabled` | Install the configmap-reload container         | `false`                                                   |
| `kube-state-metrics.enabled`         | Install the kube-state-metrics sub-chart       | `false`                                                   |
| `prometheus-node-exporter.enabled`   | Install the Prometheus Node Exporter sub-chart | `false`                                                   |
| `prometheus-pushgateway.enabled`     | Install the Prometheus Push Gateway sub-chart  | `false`                                                   |
| `server.extraFlags`                  | Additional flags to add to Prometheus server   | `["enable-feature=exemplar-storage"]`                     |
| `server.persistentVolume.enabled`    | Enable persistent storage for Prometheus data  | `false`                                                   |
| `server.global.scrape_interval`      | How frequently to scrape targets by default    | `5s`                                                      |
| `server.global.scrap_timeout`        | How long until a scrape request times out      | `3s`                                                      |
| `server.global.evaluation_interval`  | How frequently to evaluate rules               | `30s`                                                     |
| `service.servicePort`                | Service port used                              | `9090`                                                    |
| `serverFiles.prometheus.yml`         | Prometheus configuration file                  | Scrape config to get metrics from OpenTelemetry collector |

#### Grafana

> **Note**
> The following parameters have a `grafana.` prefix.

| Parameter             | Description                                        | Default                                                              |
|-----------------------|----------------------------------------------------|----------------------------------------------------------------------|
| `enabled`             | Install the Grafana sub-chart                      | `true`                                                               |
| `grafana.ini`         | Grafana's primary configuration                    | Enables anonymous login, and proxy through the frontend-proxy service |
| `adminPassword`       | Password used by `admin` user                      | `admin`                                                              |
| `rbac.pspEnabled`     | Enable PodSecurityPolicy resources                 | `false`                                                              |
| `datasources`         | Configure grafana datasources (passed through tpl) | Prometheus and Jaeger data sources                                   |
| `dashboardProviders`  | Configure grafana dashboard providers              | Defines a `default` provider based on a file path                    |
| `dashboardConfigMaps` | ConfigMaps reference that contains dashboards      | Dashboard config map deployed with this Helm chart                   |


#### End Result
![image](https://github.com/user-attachments/assets/2f214d83-e8e1-4cde-a749-d9cc66c8eee9)

