# logging-operator-config

### Chart Structure Overview

```
logging-operator-config/
├── Chart.yaml
├── values.yaml
├── values.schema.json
├── templates/
│   ├── _helpers.tpl
│   ├── flow.yaml
│   └── output.yaml
└── README.md
```

### Chart.yaml

This file contains metadata about your Helm chart.

```yaml
apiVersion: v2
name: logging-operator-config
description: A Helm chart for Kubernetes to configure the Logging Operator for CloudWatch
type: application
version: 0.1.0
appVersion: "1.0"
```

### values.yaml

Here, we define the customizable values for the chart.

```yaml
namespace: project-1

flow:
  name: cloudwatch
  labels:
    app.kubernetes.io/type: backend

output:
  name: cloudwatch
  cloudwatch:
    log_group_name: /aws/eks/test-cluster/project-1
    region: us-west-2
    auto_create_stream: true
    log_stream_name: "${tag}"
    buffer:
      timekey: 30s
      timekey_wait: 30s
      timekey_use_utc: true
```

### values.schema.json

This JSON schema validates the `values.yaml` file structure.

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "type": "object",
  "properties": {
    "namespace": { "type": "string" },
    "flow": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "labels": {
          "type": "object",
          "additionalProperties": { "type": "string" }
        }
      },
      "required": ["name", "labels"]
    },
    "output": {
      "type": "object",
      "properties": {
        "name": { "type": "string" },
        "cloudwatch": {
          "type": "object",
          "properties": {
            "log_group_name": { "type": "string" },
            "region": { "type": "string" },
            "auto_create_stream": { "type": "boolean" },
            "log_stream_name": { "type": "string" },
            "buffer": {
              "type": "object",
              "properties": {
                "timekey": { "type": "string" },
                "timekey_wait": { "type": "string" },
                "timekey_use_utc": { "type": "boolean" }
              },
              "required": ["timekey", "timekey_wait", "timekey_use_utc"]
            }
          },
          "required": ["log_group_name", "region", "auto_create_stream", "log_stream_name", "buffer"]
        }
      },
      "required": ["name", "cloudwatch"]
    }
  },
  "required": ["namespace", "flow", "output"]
}
```

### templates/_helpers.tpl

Define reusable template snippets here.

```yaml
{{/* Generate basic labels */}}
{{- define "logging-operator-config.labels" -}}
app.kubernetes.io/name: {{ include "logging-operator-config.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
{{- end -}}
```

### templates/flow.yaml

This template defines the Flow resource.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: {{ .Values.flow.name }}
  namespace: {{ .Values.namespace }}
  labels: {{ include "logging-operator-config.labels" . | nindent 4 }}
spec:
  filters:
    - tag_normaliser: {}
  match:
    - select:
        labels:
          {{- range $key, $value := .Values.flow.labels }}
          {{ $key }}: {{ $value }}
          {{- end }}
  localOutputRefs:
    - {{ .Values.output.name }}
```

### templates/output.yaml

This template defines the Output resource.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Output
metadata:
  name: {{ .Values.output.name }}
  namespace: {{ .Values.namespace }}
  labels: {{ include "logging-operator-config.labels" . | nindent 4 }}
spec:
  cloudwatch:
    log_group_name: {{ .Values.output.cloudwatch.log_group_name }}
    log_stream_name: {{ .Values.output.cloudwatch.log_stream_name }}
    region: {{ .Values.output.cloudwatch.region }}
    auto_create_stream: {{ .Values.output.cloudwatch.auto_create_stream }}
    buffer:
      timekey: {{ .Values.output.cloudwatch.buffer.timekey }}
      timekey_wait: {{ .Values.output.cloudwatch.buffer.timekey_wait }}
      timekey_use_utc: {{ .Values.output.cloudwatch.buffer.timekey_use_utc }}
```

### README.md

This file should document how to use the chart, the parameters it accepts, and how to configure it.

```markdown
# Logging Operator Config Chart

## Overview

This Helm chart configures the Logging Operator to forward logs to CloudWatch, specifically designed for Kubernetes environments. It allows for detailed customization of logging flows and outputs, ensuring that logs can be efficiently managed and analyzed.

## Prerequisites

- Kubernetes 1.25+
- Helm 3.0+
- Logging Operator installed in your cluster

## Installing the Chart

To install the chart with the release name `my-logging-config` into the namespace `project-1`:

```sh
helm install my-logging-config ./logging-operator-config -n project-1
```

## Values

The following table lists the configurable parameters of the Logging Operator Config chart and their default values.

| Parameter                          | Description                                       | Default                                         |
|------------------------------------|---------------------------------------------------|-------------------------------------------------|
| `namespace`                        | Kubernetes namespace                              | `project-1`                                     |
| `flow.name`                        | Name of the Flow resource                         | `cloudwatch`                                    |
| `flow.labels.app.kubernetes.io/type` | Labels to select for the Flow                   | `backend`                                       |
| `output.name`                      | Name of the Output resource                       | `cloudwatch`                                    |
| `output.cloudwatch.log_group_name` | CloudWatch log group name                         | `/aws/eks/test-cluster/project-1`               |
| `output.cloudwatch.region`         | AWS region for CloudWatch                         | `us-west-2`                                     |
| `output.cloudwatch.auto_create_stream` | Auto create CloudWatch log stream             | `true`                                          |
| `output.cloudwatch.log_stream_name`| CloudWatch log stream name                        | `"${tag}"`                                      |
| `output.cloudwatch.buffer.timekey` | Time key for CloudWatch buffer                    | `30s`                                           |
| `output.cloudwatch.buffer.timekey_wait` | Time key wait for CloudWatch buffer          | `30s`                                           |
| `output.cloudwatch.buffer.timekey_use_utc` | Use UTC time for CloudWatch buffer        | `true`                                          |

## Customizing the Chart Before Installing

To edit the default configuration, fork this chart and modify the `values.yaml` file according to your needs. You can also override these default values with your own via the command line when installing the chart:

```sh
helm install my-logging-config ./logging-operator-config -n project-1 --set output.cloudwatch.region=eu-west-1
```

## Upgrading the Chart

To upgrade the chart to a newer version with the release name `my-logging-config`:

```sh
helm upgrade my-logging-config ./logging-operator-config -n project-1
```

## Additional Information

For more details on configuring logging flows and outputs, refer to the official documentation of the Logging Operator.

## Troubleshooting

- Ensure that the Logging Operator is correctly installed and operational within your Kubernetes cluster.
- Verify that the specified namespace exists and your Kubernetes context is set correctly.
- For issues related to log delivery to CloudWatch, ensure that your Kubernetes nodes or service accounts have the necessary IAM permissions.

```


To further parameterize the `log_group_name` in your Helm chart to incorporate dynamic values for `target_eks_clustername` and `namespace`, you'll need to adjust your `values.yaml` file to include these new parameters and then modify the template to construct the `log_group_name` dynamically using these parameters.

### Step 1: Update `values.yaml`

Add the `target_eks_clustername` parameter to your `values.yaml`. You already have `namespace`, so ensure it's also used correctly in this context.

```yaml
# Additions to values.yaml
namespace: project-1
target_eks_clustername: test-cluster

output:
  name: cloudwatch
  cloudwatch:
    # Update the log_group_name to just be a placeholder or remove it since we'll construct it dynamically
    region: us-west-2
    auto_create_stream: true
    log_stream_name: "${tag}"
    buffer:
      timekey: 30s
      timekey_wait: 30s
      timekey_use_utc: true
```

### Step 2: Update the Output Template

Modify the `templates/output.yaml` to dynamically construct the `log_group_name` using the `target_eks_clustername` and `namespace`.

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Output
metadata:
  name: {{ .Values.output.name }}
  namespace: {{ .Values.namespace }}
spec:
  cloudwatch:
    log_group_name: "/aws/eks/{{ .Values.target_eks_clustername }}/{{ .Values.namespace }}"
    log_stream_name: {{ .Values.output.cloudwatch.log_stream_name }}
    region: {{ .Values.output.cloudwatch.region }}
    auto_create_stream: {{ .Values.output.cloudwatch.auto_create_stream }}
    buffer:
      timekey: {{ .Values.output.cloudwatch.buffer.timekey }}
      timekey_wait: {{ .Values.output.cloudwatch.buffer.timekey_wait }}
      timekey_use_utc: {{ .Values.output.cloudwatch.buffer.timekey_use_utc }}
```

### Step 3: Update `values.schema.json` (If Used)

If you're using a `values.schema.json` to validate your `values.yaml`, remember to add or update the schema to include `target_eks_clustername`.

```json
{
  "properties": {
    "namespace": { "type": "string" },
    "target_eks_clustername": { "type": "string" },
    "output": {
      "type": "object",
      "properties": {
        "cloudwatch": {
          "properties": {
            "region": { "type": "string" },
            "auto_create_stream": { "type": "boolean" },
            "log_stream_name": { "type": "string" },
            "buffer": {
              "type": "object"
              // Include buffer properties here as before
            }
          },
          "required": ["region", "auto_create_stream", "log_stream_name", "buffer"]
        }
      }
    }
  },
  "required": ["namespace", "target_eks_clustername", "output"]
}
```

By making these adjustments, you're enabling the dynamic construction of the `log_group_name` based on the `target_eks_clustername` and `namespace` parameters, providing more flexibility and customization capabilities in your Helm chart. This approach ensures that your chart can be easily adapted to different environments or configurations without hardcoding specific values.
