# dataflow-example-app

Reference dataflow workload app for the Kafka/NiFi/Flink platform.

This repo contains only deployable workload manifests. Platform/runtime resources (Kafka brokers, Flink cluster, NiFi deployment, Keycloak, oauth2-proxy) remain in `k8s-kafka`.

## Layout

- `manifests/batch-processing-examples.yaml`
  - ConfigMap with staged pipeline topic + consumer group config
  - ConfigMap with NiFi flow reference doc (two NiFi chains)
  - `Job` to create staged Kafka topics
  - `Job` to seed stage-1 input topic
  - `Job` to submit Flink SQL stage-1 pipeline
  - `Job` to submit Flink SQL stage-2 pipeline
  - `Job` to verify final output topic is readable by the NiFi Kafka principal
- `manifests/nifi-declarative-flow-crs.yaml`
  - NiFiKop declarative CRs for NiFi workflow lifecycle:
    - `NifiCluster` (external mode)
    - `NifiRegistryClient`
    - `NifiParameterContext`
    - Registry bootstrap `Job` for `flowVersion: 1`
    - `NifiDataflow`
    - NiFi stage-chain bootstrap `Job` that declaratively creates/updates/starts
      processor chains for stage-1 and stage-2 Kafka bridging
  - This file is applied by Argo from `manifests/`.
  - Registry `flowVersion: 1` is created automatically by the bootstrap Job.
- `manifests/nifi-registry.yaml`
  - Internal NiFi Registry deployment + service + PVC
  - Stores versioned flow definitions consumed by `NifiDataflow` resources

## End-to-End Example

The deployed example models this path:

1. Flink stage 1 consumes `batch.example.stage1.input.v1` and writes `batch.example.stage1.flink-to-nifi.v1`.
2. NiFi stage 1 consumes `batch.example.stage1.flink-to-nifi.v1`, does simple transform, writes `batch.example.stage2.nifi-to-flink.v1`.
3. Flink stage 2 consumes `batch.example.stage2.nifi-to-flink.v1` and writes `batch.example.stage2.flink-to-nifi.v1`.
4. NiFi stage 2 consumes `batch.example.stage2.flink-to-nifi.v1`, does simple transform, writes `batch.example.stage3.nifi-final.v1`.

What is automated by manifests:

- Topic creation
- Seed stage-1 input data
- Flink stage-1 SQL submission
- Flink stage-2 SQL submission
- Final output verification using NiFi Kafka credentials
- NiFi stage-1 and stage-2 processor chain bootstrap and start-up

What remains operator-driven in NiFi UI:

- Applying any business routing/sink logic in NiFi

## Declarative NiFi Flow Path

Use `manifests/nifi-declarative-flow-crs.yaml` for GitOps-managed NiFi workflows.

1. Provide required secrets for the declarative CR file:
   - `secret/data/k8s-kafka-nifikop-client-cert-pem`
   - `secret/data/k8s-kafka-nifikop-client-key-pem`
   - `secret/data/k8s-kafka-nifi-ca-cert-pem`
   - `secret/data/k8s-kafka-nifi-registry-bucket-id`
   - `secret/data/k8s-kafka-nifi-registry-flow-id`
2. `flowVersion: 1` is bootstrapped automatically if missing.
3. NiFi processor topology is then reconciled by the stage-chain bootstrap Job
   (also declarative in manifests).

Notes:

- External-cluster reconciliation needs non-interactive NiFi API auth (`basic` or `tls`).
- `bucketId` and `flowId` come from NiFi Registry flow metadata.
- NiFiKop must watch `dataflow` namespace in addition to `kafka`.

## Deployment

This repo is deployed by the `dataflow-example-app` Argo CD Application defined in:

- `https://github.com/dotcomrow/dataflow-apps`

Target namespace: `dataflow`
