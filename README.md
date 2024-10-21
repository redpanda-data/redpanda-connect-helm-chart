# Redpanda Connect Helm Chart Specification
<p align="center" style="text-align: center">
    <img src="./assets/blob.png" width="30%"><br/>
    Redpanda Connect Helm Chart. <br/>
</p>

Redpanda Connect is a high performance and resilient stream processor, able to connect various sources and sinks in a range of brokering patterns and perform hydration, enrichments, transformations and filters on payloads.

This Helm Chart deploys a single Redpanda Connect instance in either streams mode or standalone.

## Installation
```bash
helm repo add redpanda https://benthosdev.github.io/charts/
helm repo update

helm install redpanda/connect
```

## Configuration

The config parameter should contain the configuration as it would be parsed by the Redpanda Connect binary.
```yaml
# values.yaml
config:
  input:
    label: "no_config_in"
    generate:
      mapping: root = "This Redpanda Connect instance is unconfigured!"
      interval: 1m
  output:
    label: "no_config_out"
    stdout:
      codec: lines
```

The full list of [available configuration for the Helm Chart can be found in the `values.yaml` file](./values.yaml). You should refer to the [upstream Redpanda Connect documentation](https://docs.redpanda.com/redpanda-connect/configuration/about/) for the configuration of your pipeline.

## Streams mode

When running Redpanda Connect in [streams mode](https://docs.redpanda.com/redpanda-connect/guides/streams_mode/about/), all individual stream configuration files should be combined and placed in a single Kubernetes `ConfigMap`, like so:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: connect-streams
data:
  hello.yaml: |
    input:
      generate:
        mapping: root = "woof"
        interval: 5s
        count: 0
    output:
      stdout:
        codec: lines
  aaaaa.yaml: |
    input:
      generate:
        mapping: root = "meow"
        interval: 2s
        count: 0
    output:
      stdout:
        codec: lines
```

Then you can simply reference your `ConfigMap` and enable streams mode in your `values.yaml` file.
```yaml
# values.yaml
streams:
  enabled: true
  streamsConfigMap: "connect-streams"
```

Currently the streams mode `ConfigMap` should be applied **separately from and before installation of** the helm chart; support for deploying additional `ConfigMap`'s within the chart may be implemented later.

### Global Configuration

When deploying Redpanda Connect in streams mode, you may want to configure global tracing, logging and http configuration which is shared across all of your pipelines.

This can be done by specifying configuration under the `metrics`, `logger` and `tracing` configuration sections in your `values.yaml` file. These all use their respective upstream Redpanda Connect configuration syntax.

```yaml
metrics:
  prometheus: {}

tracing:
  openTelemetry:
    http: []
    grpc: []
    tags: {}

logger:
  level: INFO
  static_fields:
    '@service': redpanda-connect
```
