### README.md

## Overview

This Helm chart deploys the Logging configuration for CloudWatch and Splunk logging.

## Prerequisites

- Kubernetes 1.26+
- Helm 3.0+

## Installation

To install the chart with the release name `my-release`:

```sh
helm lint ./helm-chart
```

```sh
helm install my-release ./helm-chart
```
## Values

The following table lists the configurable parameters of the Logging Config chart and their default values.

| Parameter                                      | Description                                           | Default                                       |
|------------------------------------------------|-------------------------------------------------------|-----------------------------------------------|
| `namespace`                                    | Kubernetes namespace                                  | `project-1`                                   |
| `nameOverride`                                 | Override the name of the chart                        | `""`                                          |
| `environmentName`                              | The environment name (e.g., production)               | `dev`                                         |
| `region`                                       | The AWS region                                        | `us-east-1`                                   |
| `targetClusterName`                            | The name of the target cluster                        | `dev-cluster`                                 |
| `flow.baseName`                                | Base name for the flow                                | `flow`                                        |
| `flow.labels.app.kubernetes.io/type`           | Labels to select for the Flow                         | `backend`                                     |
| `cloudwatch.enabled`                           | Enable CloudWatch logging                             | `true`                                        |
| `cloudwatch.output.name`                       | Suffix Name of the CloudWatch Output resource         | `cloudwatch-output`                           |
| `cloudwatch.output.log_group_name`             | CloudWatch log group name  (Please SKIP)              | `/aws/eks/test-cluster/project-1`             |
| `cloudwatch.output.log_stream_name`            | CloudWatch log stream name                            | `"${tag}"`                                    |
| `cloudwatch.output.auto_create_stream`         | Auto create CloudWatch log stream                     | `true`                                        |
| `cloudwatch.output.buffer.timekey`             | Time key for CloudWatch buffer                        | `30s`                                         |
| `cloudwatch.output.buffer.timekey_wait`        | Time key wait for CloudWatch buffer                   | `30s`                                         |
| `cloudwatch.output.buffer.timekey_use_utc`     | Use UTC time for CloudWatch buffer                    | `true`                                        |
| `splunk.enabled`                               | Enable Splunk logging                                 | `true`                                        |
| `splunk.hecTokenSecretName`                    | Suffix Name of the Splunk HEC token secret            | `splunk-hec-token`                            |
| `splunk.secretStore`                           | Suffix Secret store for Splunk HEC token              | `aws-secrets-manager`                         |
| `splunk.refreshInterval`                       | Refresh interval for the Splunk external secret       | `30m`                                         |
| `splunk.secretName`                            | Suffix Name of the secret in the secret store         | `splunk-hec-token`                            |
| `splunk.output.name`                           | Suffix Name of the Splunk Output resource             | `splunk-output`                               |
| `splunk.output.hec_host`                       | Splunk HEC host                                       | `splunk-single-standalone-headless`           |
| `splunk.output.insecure_ssl`                   | Allow insecure SSL                                    | `true`                                        |
| `splunk.output.hec_port`                       | Splunk HEC port                                       | `8088`                                        |
| `splunk.output.index`                          | Splunk index                                          | `main`                                        |
| `splunk.output.format_type`                    | Format type for Splunk logs                           | `json`                                        |

This table lists all configurable parameters for the Helm chart, providing descriptions and default values for each parameter.
