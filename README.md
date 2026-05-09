# dataflow-example-app

Reference dataflow workload app for the Kafka/NiFi/Flink platform.

This repo contains only deployable workload manifests. Platform/runtime resources (Kafka brokers, Flink cluster, NiFi deployment, Keycloak, oauth2-proxy) remain in `k8s-kafka`.

## Layout

- `manifests/batch-processing-examples.yaml`
  - ConfigMap with pipeline topic config
  - ConfigMap with NiFi flow reference doc
  - `Job` to create example Kafka topics
  - `Job` to seed input topic data
  - `Job` to submit sample Flink SQL pipeline

## Deployment

This repo is deployed by the `dataflow-example-app` Argo CD Application defined in:

- `https://github.com/dotcomrow/dataflow-apps`

Target namespace: `kafka`
