# benthos-helm-chart


This is in WIP status, very basic functionality tested at the moment:
- Ingress not tested

## Repository
---

To add this repo:
```
helm repo add benthos https://difabion.github.io/benthos-helm-chart/
```
Then `helm search repo benthos` for all charts.

## Configuration
---
### Common Parameters
| Name             | Description                   | Value           |
|------------------|-------------------------------|-----------------|
| image.repository | Docker image repository       | jeffail/benthos |
| image.pullPolicy | Docker image pull policy      | IfNotPresent    |
| image.tag        | Docker image tag override     | ""              |
| imagePullSecrets | Docker registry secrets array | []              |
| service.type     | Kubernetes service type       | ClusterIP       |
| service.port     | Kubernetes service port       | 80              |

### Benthos Parameters

For more information on configuring the HTTP component, refer to the [Benthos HTTP component documentation](https://www.benthos.dev/docs/components/http/about).  
| Name                     | Description                           | Value        |
|--------------------------|---------------------------------------|--------------|
| http.enabled             | Enables the HTTP server component     | true         |
| http.address             | HTTP server component binding address | 0.0.0.0:4195 |
| http.readTimeout         | HTTP server component read timeout    | 5s           |
| http.rootPath            | General Benthos HTTP endpoint prefix  | /benthos     |
| http.debugEndpoints      | Enables debugging endpoints           | false        |
| http.cors.enabled        | Enables Cross-Origin Resource Sharing | false        |
| http.cors.allowedOrigins | Allowed source domains for CORS       | ""           |
| http.tls.enabled         | Enables TLS for all Benthos endpoints | false        |
| http.tls.secretName      | `kubernetes.io/tls` secret name       | ""           |
| config                   | Benthos component configuration       | ""           |

## TLS

Benthos can be instructed to serve all endpoints exlusively over HTTPS.  This means that TLS configured in this way is not terminated at an ingress controller, but handled "end-to-end" at the container/binary. Prerequisites to enable TLS:
- Set `service.port` to 443 in values.yaml
- Create a Kubernetes secret in the targeted namespace of type `kubernetes.io/tls`

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