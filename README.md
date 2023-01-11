[![Chart status](https://img.shields.io/badge/Chart%20status-WIP-yellow)](https://github.com/benthosdev/benthos-helm-chart)
[![benthos](https://img.shields.io/badge/benthos-v4.11.0-green)](https://github.com/Jeffail/benthos/releases/tag/v4.11.0)
[![Chart version](https://img.shields.io/badge/Chart%20version-v0.7.0-green)](https://github.com/benthosdev/benthos-helm-chart/releases/tag/0.7.0)

# benthos-helm-chart

This chart is relatively immature, and may not yet support all workloads or benthos features.  Please feel free to test and provide feedback, report bugs, and/or contribute!

## Repository

To add this repo:
```
helm repo add benthos https://benthosdev.github.io/benthos-helm-chart/
```
Then `helm search repo benthos` for all charts.

### Versions
Benthos is currently on v4.  If you need v3 for any reason, please set `image.tag` in your values.yaml.

## Configuration

### Benthos Parameters

For more information on configuring the HTTP component, refer to the [Benthos HTTP component documentation](https://www.benthos.dev/docs/components/http/about).
| Name                                          | Description                                      | Value              |
|-----------------------------------------------|--------------------------------------------------|--------------------|
| image.repository                              | Docker image repository                          | benthosdev/benthos |
| image.pullPolicy                              | Docker image pull policy                         | IfNotPresent       |
| image.tag                                     | Docker image tag override                        | ""                 |
| imagePullSecrets                              | Docker registry secrets array                    | []                 |
| nameOverride                                  | Sets name override                               | ""                 |
| fullnameOverride                              | Sets full name override                          | ""                 |
| serviceaccount.create                         | Enables creation of serviceaccount               | false              |
| serviceaccount.annotations                    | Sets serviceaccount annotations                  | {}                 |
| serviceaccount.name                           | Sets serviceaccount name                         | ""                 |
| podAnnotations                                | Sets pod annotations                             | {}                 |
| podLabels                                     | Sets pod labels                                  | {}                 |
| podSecurityContext                            | Sets pod security context                        | {}                 |
| securityContext                               | Sets security context                            | {}                 |
| service.type                                  | Kubernetes service type                          | ClusterIP          |
| service.ports                                 | Kubernetes service ports                         | []                 |
| ingress.enabled                               | Enables ingress                                  | false              |
| ingress.className                             | Sets ingress class name                          | ""                 |
| ingress.annotations                           | Sets ingress annotations                         | {}                 |
| ingress.tls                                   | Sets ingress TLS configuration                   | []                 |
| ingress.hosts                                 | Sets ingress hosts configuration                 | []                 |
| env                                           | Sets benthos environment variables               | ""                 |
| resources                                     | Set pod resource limits and/or requests          | {}                 |
| autoscaling.enabled                           | Enables the horizontal pod autoscaler            | false              |
| autoscaling.minReplicas                       | Sets min number of replicas                      | 1                  |
| autoscaling.maxReplicas                       | Sets max numbers of replicas                     | 100                |
| autoscaling.targetCPUUtilizationPercentage    | Sets desired CPU autoscaling threshold           | 80                 |
| autoscaling.targetMemoryUtilizationPercentage | Sets desired memory autoscaling threshold        | 80                 |
| nodeSelector                                  | Sets node selector configuration                 | {}                 |
| tolerations                                   | Sets tolerations configuration                   | []                 |
| affinity                                      | Sets affinity configuration                      | {}                 |
| extraVolumes                                  | Defines extra volumes configuration              | []                 |
| extraVolumeMounts                             | Sets additional mounts defined in extraVolumes   | []                 |
| streams.enabled                               | Enables 'Benthos streams' mode                   | false              |
| streams.streamsConfigMap                      | Name of K8s configMap containing streams configs | ""                 |
| streams.api.enable                            | Enables streams API                              | true               |
| http.enabled                                  | Enables the HTTP server component                | true               |
| http.address                                  | HTTP server component binding address            | 0.0.0.0:4195       |
| http.readTimeout                              | HTTP server component read timeout               | 5s                 |
| http.rootPath                                 | General Benthos HTTP endpoint prefix             | /benthos           |
| http.debugEndpoints                           | Enables debugging endpoints                      | false              |
| http.cors.enabled                             | Enables Cross-Origin Resource Sharing            | false              |
| http.cors.allowedOrigins                      | Allowed source domains for CORS                  | ""                 |
| http.tls.enabled                              | Enables TLS for all Benthos endpoints            | false              |
| http.tls.secretName                           | `kubernetes.io/tls` secret name                  | ""                 |
| watch                                         | Enables watch mode                               | false              |
| config                                        | Benthos component configuration                  | ""                 |

## Config

The config parameter should contain the configuration as it would be parsed by the Benthos binary.

For example, the default Helm chart config block looks like this:

```yaml
# /benthos.yaml configuration
config: |-
  input:
    label: "no_config_in"
    generate:
      mapping: root = "This Benthos instance is unconfigured!"
      interval: 1m
  output:
    label: "no_config_out"
    stdout:
      codec: lines
```

Adding an `http` block here is not recommended, please use the Helm directives described above.

## TLS

Benthos can be instructed to serve all endpoints exlusively over HTTPS.  This means that TLS configured in this way is not terminated at an ingress controller, but handled "end-to-end" at the container/binary. Prerequisites to enable TLS:
- Set `service.port` to 443 in values.yaml
- Create a Kubernetes secret in the targeted namespace of type `kubernetes.io/tls`

When TLS is enabled, the Kubernetes readiness and liveness probes will operate over HTTPS to the same container port (default 4195).

## Streams mode

When running Benthos in [streams mode](https://www.benthos.dev/docs/guides/streams_mode/about), all configuration files should be placed in a single Kubernetes configMap, like so:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: benthos-streams
data:
  hello.yaml: |
    input:
      generate:
        mapping: root = "hello"
        interval: 5s
        count: 0
    output:
      stdout:
        codec: lines
  aaaaa.yaml: |
    input:
      generate:
        mapping: root = "AAAAAAAAAA"
        interval: 2s
        count: 0
    output:
      stdout:
        codec: lines
```
Note: This exposes a [streams API](https://www.benthos.dev/docs/guides/streams_mode/streams_api) where configuration can be viewed and altered.

As of benthos version `3.61.0`, the streams API can be disabled to prevent changes to the configuration[s] of a running [Benthos in streams mode via config files](https://www.benthos.dev/docs/guides/streams_mode/using_config_files); use `.Values.streams.api.enabled = false` to toggle the API off.

For this chart, the stream fields configurations should go in the configMap while the shared fields go in the `config` block (with the exception of HTTP server configs, which are covered by the `http` block).

From the `benthos streams --help` context:

```
In streams mode the stream fields of a root target config (input, buffer,
      pipeline, output) will be ignored. Other fields will be shared across all
      loaded streams (resources, metrics, etc).
```
