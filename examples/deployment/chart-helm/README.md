# Helm Charts

This directory contains a Helm chart for deploying the vllm application. The chart includes configurations for deployment, autoscaling, resource management, and more.

## Files

- Chart.yaml: Defines the chart metadata including name, version, and maintainers.
- ct.yaml: Configuration for chart testing.
- lintconf.yaml: Linting rules for YAML files.
- values.schema.json: JSON schema for validating values.yaml.
- values.yaml: Default values for the Helm chart.
- templates/_helpers.tpl: Helper templates for defining common configurations.
- templates/configmap.yaml: Template for creating ConfigMaps.
- templates/custom-objects.yaml: Template for custom Kubernetes objects.
- templates/deployment.yaml: Template for creating Deployments.
- templates/hpa.yaml: Template for Horizontal Pod Autoscaler.
- templates/job.yaml: Template for Kubernetes Jobs.
- templates/poddisruptionbudget.yaml: Template for Pod Disruption Budget.
- templates/pvc.yaml: Template for Persistent Volume Claims.
- templates/secrets.yaml: Template for Kubernetes Secrets.
- templates/service.yaml: Template for creating Services.

## Running Tests

This chart includes unit tests using [helm-unittest](https://github.com/helm-unittest/helm-unittest). Install the plugin and run tests:

```bash
# Install plugin
helm plugin install https://github.com/helm-unittest/helm-unittest

# Run tests
helm unittest .
```

## Configuring Slow Model Startup

The default startup probe checks `/health` every 10 seconds and allows 180
failures, giving model initialization up to 30 minutes. The deployment progress
deadline defaults to 35 minutes so that the startup budget can be exhausted
before Kubernetes reports `ProgressDeadlineExceeded`.

Disable the startup probe when another mechanism manages startup:

```yaml
startupProbe: null
```

The probe object is rendered directly and supports standard Kubernetes probe
handlers. Because Helm merges maps with the chart defaults, clear `httpGet`
when replacing the default handler with `exec` or `tcpSocket`:

```yaml
startupProbe:
  httpGet: null
  exec:
    command:
      - sh
      - -c
      - test -f /tmp/model-ready
  periodSeconds: 5
  failureThreshold: 360
```

When changing the startup budget, also set `progressDeadlineSeconds` longer
than the expected startup window.
