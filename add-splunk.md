I apologize for that oversight. Here is the `Chart.yaml` file:

### `Chart.yaml`

```yaml
apiVersion: v2
name: logging-operator-config
description: A Helm chart for configuring logging operator with CloudWatch and Splunk outputs
type: application
version: 0.1.0
appVersion: "1.0"
```

### Updated Directory Structure

```
logging-operator-config/
├── helm-chart/
│   ├── Chart.yaml
│   ├── values.yaml
│   ├── values.schema.json
│   ├── templates/
│   │   ├── _helpers.tpl
│   │   ├── flow.yaml
│   │   ├── cloudwatch-output.yaml
│   │   ├── splunk-output.yaml
│   │   ├── secret-store.yaml
│   │   └── splunk-external-secret.yaml
└── README.md
```

### Full Updated Files

#### `Chart.yaml`

```yaml
apiVersion: v2
name: logging-operator-config
description: A Helm chart for configuring logging operator with CloudWatch and Splunk outputs
type: application
version: 0.1.0
appVersion: "1.0"
```

#### `values.yaml`

```yaml
namespace: project-1
nameOverride: ""
environmentName: production
region: us-east-1  # General region setting for all services

flow:
  baseName: flow
  labels:
    app.kubernetes.io/type: backend

cloudwatch:
  enabled: true
  output:
    name: cloudwatch-output
    log_group_name: /aws/eks/test-cluster/project-1
    log_stream_name: "${tag}"
    auto_create_stream: true
    buffer:
      timekey: 30s
      timekey_wait: 30s
      timekey_use_utc: true

splunk:
  enabled: false
  hecTokenSecretName: splunk-token
  secretStore: aws-secrets-manager
  secretName: splunk-hec-token
  output:
    name: splunk-output
    hec_host: splunk-single-standalone-headless
    insecure_ssl: true
    hec_port: 8088
    index: main
    format_type: json
```

#### `values.schema.json`

```json
{
  "$schema": "http://json-schema.org/schema#",
  "type": "object",
  "properties": {
    "namespace": {
      "type": "string",
      "description": "Kubernetes namespace"
    },
    "nameOverride": {
      "type": "string",
      "description": "Override the name of the chart"
    },
    "environmentName": {
      "type": "string",
      "description": "The environment name (e.g., production)"
    },
    "region": {
      "type": "string",
      "description": "The AWS region"
    },
    "flow": {
      "type": "object",
      "properties": {
        "baseName": {
          "type": "string",
          "description": "Base name for the flow"
        },
        "labels": {
          "type": "object",
          "properties": {
            "app.kubernetes.io/type": {
              "type": "string",
              "description": "Labels to select for the Flow"
            }
          }
        }
      }
    },
    "cloudwatch": {
      "type": "object",
      "properties": {
        "enabled": {
          "type": "boolean",
          "description": "Enable CloudWatch logging"
        },
        "output": {
          "type": "object",
          "properties": {
            "name": {
              "type": "string",
              "description": "Name of the CloudWatch Output resource"
            },
            "log_group_name": {
              "type": "string",
              "description": "CloudWatch log group name"
            },
            "log_stream_name": {
              "type": "string",
              "description": "CloudWatch log stream name"
            },
            "auto_create_stream": {
              "type": "boolean",
              "description": "Auto create CloudWatch log stream"
            },
            "buffer": {
              "type": "object",
              "properties": {
                "timekey": {
                  "type": "string",
                  "description": "Time key for CloudWatch buffer"
                },
                "timekey_wait": {
                  "type": "string",
                  "description": "Time key wait for CloudWatch buffer"
                },
                "timekey_use_utc": {
                  "type": "boolean",
                  "description": "Use UTC time for CloudWatch buffer"
                }
              }
            }
          }
        }
      }
    },
    "splunk": {
      "type": "object",
      "properties": {
        "enabled": {
          "type": "boolean",
          "description": "Enable Splunk logging"
        },
        "hecTokenSecretName": {
          "type": "string",
          "description": "Name of the Splunk HEC token secret"
        },
        "secretStore": {
          "type": "string",
          "description": "Secret store for Splunk HEC token"
        },
        "secretName": {
          "type": "string",
          "description": "Name of the secret in the secret store"
        },
        "output": {
          "type": "object",
          "properties": {
            "name": {
              "type": "string",
              "description": "Name of the Splunk Output resource"
            },
            "hec_host": {
              "type": "string",
              "description": "Splunk HEC host"
            },
            "insecure_ssl": {
              "type": "boolean",
              "description": "Allow insecure SSL"
            },
            "hec_port": {
              "type": "integer",
              "description": "Splunk HEC port"
            },
            "index": {
              "type": "string",
              "description": "Splunk index"
            },
            "format_type": {
              "type": "string",
              "description": "Format type for Splunk logs"
            }
          }
        }
      }
    }
  },
  "required": ["namespace", "environmentName", "region", "flow", "cloudwatch", "splunk"]
}
```

#### `templates/_helpers.tpl`

```yaml
{{/*
Expand the name of the chart.
*/}}
{{- define "logging-operator-config.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{/*
Create a default fully qualified app name.
*/}}
{{- define "logging-operator-config.fullname" -}}
{{- if .Values.nameOverride }}
{{- printf "%s-%s" (include "logging-operator-config.name" .) .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" (include "logging-operator-config.name" .) .Release.Name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}

{{/*
Common labels
*/}}
{{- define "logging-operator-config.labels" -}}
app.kubernetes.io/name: {{ include "logging-operator-config.name" . }}
app.kubernetes.io/instance: {{ .Release.Name }}
app.kubernetes.io/version: {{ .Chart.AppVersion }}
app.kubernetes.io/managed-by: {{ .Release.Service }}
app.kubernetes.io/type: {{ .Values.flow.labels["app.kubernetes.io/type"] }}
{{- end }}
```

#### `templates/flow.yaml`

```yaml
apiVersion: logging.banzaicloud.io/v1beta1
kind: Flow
metadata:
  name: {{ include "logging-operator-config.fullname" . }}
  namespace: {{ .Release.Namespace }}
  labels: 
    {{- include "logging-operator-config.labels" . | nindent 4 }}
spec:
  filters:
    - tag_normaliser: {}
  match:
    - select:
        labels:
          {{- range $key, $value := .Values.flow.labels }}
          {{ $key }}: {{ $value | quote }}
          {{- end }}
  localOutputRefs:
    {{- if .Values.cloudwatch.enabled }}
    - {{ .Values.cloudwatch.output.name }}
    {{- end }}
    {{- if .Values.splunk.enabled }}
    - {{ .Values.splunk.output.name }}
    {{- end }}
    {{- if not (or .Values.cloudwatch.enabled .Values.splunk.enabled) }}
    - null
    {{- end }}
```

#### `templates/cloudwatch-output.yaml`

```yaml
{{- if .Values.cloudwatch.enabled }}
apiVersion: logging.banzaicloud.io/v1beta1
kind: Output
metadata:
  name: {{ .Values.cloudwatch.output.name | trunc 63 }}
  namespace: {{ .Release.Namespace }}
spec:
  cloudwatch:
    log_group_name: {{ .Values.cloudwatch.output.log_group_name }}
    log_stream_name: {{ .Values.cloudwatch.output.log_stream_name }}
    region: {{ .Values.region }}
    auto_create_stream: {{ .Values.cloudwatch.output.auto_create_stream }}
    buffer:
      timekey: {{ .Values.cloudwatch.output.buffer.timekey }}
      timekey_wait: {{ .Values.cloudwatch.output.buffer.timekey_wait }}
      timekey_use_utc: {{ .Values.cloudwatch.output.buffer.timekey_use_utc }}
{{- end }}
```

#### `templates/splunk-output.yaml`

```yaml
{{- if .Values.splunk.enabled }}
apiVersion: logging.banzaicloud.io/v

1beta1
kind: Output
metadata:
  name: {{ .Values.splunk.output.name | trunc 63 }}
  namespace: {{ .Release.Namespace }}
spec:
  splunkHec:
    hec_host: {{ .Values.splunk.output.hec_host }}
    insecure_ssl: {{ .Values.splunk.output.insecure_ssl }}
    hec_port: {{ .Values.splunk.output.hec_port }}
    hec_token:
      valueFrom:
        secretKeyRef:
          name: {{ .Values.splunk.hecTokenSecretName }}
          key: SplunkHecToken
    index: {{ .Values.splunk.output.index }}
    format:
      type: {{ .Values.splunk.output.format_type }}
{{- end }}
```

#### `templates/secret-store.yaml`

```yaml
{{- if .Values.splunk.enabled }}
apiVersion: external-secrets.io/v1beta1
kind: SecretStore
metadata:
  name: {{ printf "%s-secret-manager" (include "logging-operator-config.name" .) | trunc 63 }}
  namespace: {{ .Release.Namespace }}
spec:
  provider:
    aws:
      service: SecretsManager
      region: {{ .Values.region }}
{{- end }}
```

#### `templates/splunk-external-secret.yaml`

```yaml
{{- if .Values.splunk.enabled }}
apiVersion: external-secrets.io/v1beta1
kind: ExternalSecret
metadata:
  name: {{ include "logging-operator-config.fullname" . }}-splunk-hec-token
  namespace: {{ .Release.Namespace }}
  labels: 
    {{- include "logging-operator-config.labels" . | nindent 4 }}
spec:
  refreshInterval: "1h"
  secretStoreRef:
    kind: SecretStore
    name: {{ printf "%s-secret-manager" (include "logging-operator-config.name" .) | trunc 63 }}
  target:
    name: {{ .Values.splunk.hecTokenSecretName }}
    creationPolicy: Merge
  data:
  - secretKey: SplunkHecToken
    remoteRef:
      key: {{ printf "/datajar/%s/%s" .Values.splunk.secretName .Values.environmentName }}
      property: splunk-hec
{{- end }}
```

#### `README.md`

```markdown
# Logging Operator Config

## Overview

This Helm chart deploys the Logging Operator configuration for CloudWatch and Splunk logging.

## Prerequisites

- Kubernetes 1.16+
- Helm 3.0+

## Installation

To install the chart with the release name `my-release`:

```sh
helm install my-release ./helm-chart
```

## Values

The following table lists the configurable parameters of the Logging Operator Config chart and their default values.

| Parameter                                          | Description                                 | Default                                         |
|----------------------------------------------------|---------------------------------------------|-------------------------------------------------|
| `namespace`                                        | Kubernetes namespace                        | `project-1`                                      |
| `nameOverride`                                     | Override the name of the chart              | `""`                                             |
| `environmentName`                                  | The environment name (e.g., production)     | `production`                                     |
| `region`                                           | The AWS region                              | `us-east-1`                                      |
| `flow.baseName`                                    | Base name for the flow                      | `flow`                                           |
| `flow.labels.app.kubernetes.io/type`               | Labels to select for the Flow               | `backend`                                        |
| `cloudwatch.enabled`                               | Enable CloudWatch logging                   | `true`                                           |
| `cloudwatch.output.name`                           | Name of the CloudWatch Output resource      | `cloudwatch-output`                              |
| `cloudwatch.output.log_group_name`                 | CloudWatch log group name                   | `/aws/eks/test-cluster/project-1`                |
| `cloudwatch.output.log_stream_name`                | CloudWatch log stream name                  | `"${tag}"`                                       |
| `cloudwatch.output.auto_create_stream`             | Auto create CloudWatch log stream           | `true`                                           |
| `cloudwatch.output.buffer.timekey`                 | Time key for CloudWatch buffer              | `30s`                                            |
| `cloudwatch.output.buffer.timekey_wait`            | Time key wait for CloudWatch buffer         | `30s`                                            |
| `cloudwatch.output.buffer.timekey_use_utc`         | Use UTC time for CloudWatch buffer          | `true`                                           |
| `splunk.enabled`                                   | Enable Splunk logging                       | `false`                                          |
| `splunk.hecTokenSecretName`                        | Name of the Splunk HEC token secret         | `splunk-token`                                   |
| `splunk.secretStore`                               | Secret store for Splunk HEC token           | `aws-secrets-manager`                            |
| `splunk.secretName`                                | Name of the secret in the secret store      | `splunk-hec-token`                               |
| `splunk.output.name`                               | Name of the Splunk Output resource          | `splunk-output`                                  |
| `splunk.output.hec_host`                           | Splunk HEC host                             | `splunk-single-standalone-headless`              |
| `splunk.output.insecure_ssl`                       | Allow insecure SSL                          | `true`                                           |
| `splunk.output.hec_port`                           | Splunk HEC port                             | `8088`                                           |
| `splunk.output.index`                              | Splunk index                                | `main`                                           |
| `splunk.output.format_type`                        | Format type for Splunk logs                 | `json`                                           |

## License

This Helm chart is licensed under the Apache 2.0 License. See the [LICENSE](LICENSE) file for details.
```

This setup should now reflect the proper structure and include all necessary files and configurations for your Helm chart.
