# dataflow-example-app

Reference dataflow workload app for the Kafka/NiFi/Flink platform.

This repo contains only deployable workload manifests. Platform/runtime resources (Kafka brokers, Flink cluster, NiFi deployment, Keycloak, oauth2-proxy) remain in `k8s-kafka`.

## Layout

- `manifests/batch-processing-examples.yaml`
  - ConfigMap with pipeline topic config
  - ConfigMap with NiFi flow reference doc (producer + consumer chains)
  - `Job` to create example Kafka topics
  - `Job` to seed input topic data
  - `Job` to submit sample Flink SQL pipeline
  - `Job` to verify Flink output is readable by the NiFi Kafka principal
- `templates/nifi-declarative-flow-crs.yaml`
  - NiFiKop CR template for declarative NiFi flow lifecycle:
    - `NifiCluster` (external mode)
    - `NifiRegistryClient`
    - `NifiParameterContext`
    - `NifiDataflow`

## End-to-End Example

The deployed example models this path:

1. NiFi publishes records to `batch.example.nifi.raw.v1`.
2. Flink SQL job consumes that topic and writes enriched records to `batch.example.flink.enriched.v1`.
3. NiFi consumes the Flink output topic for downstream routing/sinks.

What is automated by manifests:

- Topic creation
- Seed input data
- Flink SQL submission
- Output verification using NiFi Kafka credentials

What remains operator-driven in NiFi UI:

- Creating/running the NiFi producer and consumer processor chains
- Applying any business routing/sink logic in NiFi

## Declarative NiFi Flow Path

Use `templates/nifi-declarative-flow-crs.yaml` to move flow management to Kubernetes manifests.

1. Replace placeholder values:
   - NiFi API automation credentials and CA cert
   - root/parent process group IDs
   - versioned flow `bucketId`, `flowId`, and `flowVersion`
2. Copy the updated resources into `manifests/` when ready to have Argo deploy them.
3. Keep `syncMode: always` on `NifiDataflow` once you want Git to be source of truth.

Notes:

- External-cluster reconciliation needs non-interactive NiFi API auth (`basic` or `tls`).
- `bucketId` and `flowId` come from NiFi Registry flow metadata (`bucket.yml` / versioned flow metadata).
- If NiFiKop is watching only `kafka` namespace, extend its watch scope before applying these CRs in `dataflow`.

## Deployment

This repo is deployed by the `dataflow-example-app` Argo CD Application defined in:

- `https://github.com/dotcomrow/dataflow-apps`

Target namespace: `dataflow`
