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
- `manifests/nifi-declarative-flow-crs.yaml`
  - NiFiKop declarative CRs for NiFi workflow lifecycle:
    - `NifiCluster` (external mode)
    - `NifiRegistryClient`
    - `NifiParameterContext`
    - `NifiDataflow`
  - This file is applied by Argo from `manifests/`.
  - Replace placeholder values in this file before production rollout.
- `manifests/nifi-registry.yaml`
  - Internal NiFi Registry deployment + service + PVC
  - Stores versioned flow definitions consumed by `NifiDataflow` resources

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

Use `manifests/nifi-declarative-flow-crs.yaml` for GitOps-managed NiFi workflows.

1. Replace placeholder values:
   - TLS auth secret used by NiFiKop (`tls.crt`, `tls.key`, `ca.crt`)
   - root/parent process group IDs
   - versioned flow `bucketId`, `flowId`, `flowVersion`
2. Keep `syncMode: always` on `NifiDataflow` to make Git the source of truth.

Notes:

- External-cluster reconciliation needs non-interactive NiFi API auth (`basic` or `tls`).
- `bucketId` and `flowId` come from NiFi Registry flow metadata (`bucket.yml` / versioned flow metadata).
- NiFiKop must watch `dataflow` namespace in addition to `kafka`.

## Deployment

This repo is deployed by the `dataflow-example-app` Argo CD Application defined in:

- `https://github.com/dotcomrow/dataflow-apps`

Target namespace: `dataflow`
