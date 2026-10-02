# Kafka L4 (Lightweight) Deployment Role

This Ansible role deploys a lightweight Kafka cluster with Zookeeper and Kowl UI on OpenShift/Kubernetes.

## Description

This role deploys:
- **Zookeeper**: Single instance for Kafka coordination
- **Kafka**: Single broker instance with persistent storage
- **Kowl**: Web UI for Kafka management and monitoring

## Requirements

- Ansible 2.9 or higher
- kubernetes.core collection
- OpenShift/Kubernetes cluster access
- Sufficient storage for PVCs

## Role Variables

### Required Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `kafka_l4_namespace` | Namespace for Kafka L4 deployment | `kafka-l4` |

### Optional Variables

| Variable | Description | Default |
|----------|-------------|---------|
| `zookeeper_client_port` | Zookeeper client port | `32181` |
| `zookeeper_tick_time` | Zookeeper tick time | `2000` |
| `kafka_broker_id` | Kafka broker ID | `1` |
| `kafka_port` | Kafka broker port | `29092` |
| `kafka_offsets_topic_replication_factor` | Replication factor | `1` |
| `kowl_port` | Kowl UI port | `8080` |
| `zk_data_storage` | Zookeeper data PVC size | `1Gi` |
| `zk_txn_logs_storage` | Zookeeper transaction logs PVC size | `1Gi` |
| `kafka_data_storage` | Kafka data PVC size | `2Gi` |
| `zookeeper_image` | Zookeeper container image | `confluentinc/cp-zookeeper:latest` |
| `kafka_image` | Kafka container image | `confluentinc/cp-kafka:7.3.0` |
| `kowl_image` | Kowl container image | `quay.io/cloudhut/kowl:master` |
| `kowl_route_tls_termination` | TLS termination for Kowl route | `edge` |

## Dependencies

None

## Example Playbook

```yaml
---
- name: Deploy Kafka L4
  hosts: localhost
  connection: local
  vars:
    kafka_l4_namespace: "my-kafka"
    kafka_data_storage: "5Gi"
  roles:
    - kafka_l4
```

## Example with Custom Namespace

```yaml
---
- name: Deploy Kafka L4 with custom namespace
  hosts: localhost
  connection: local
  vars:
    kafka_l4_namespace: "{{ lookup('env', 'KAFKA_NAMESPACE') | default('ibm-b2bi-dev01-app', true) }}"
  roles:
    - kafka_l4
```

## Deployment Components

### Persistent Volume Claims
- `zk-data-pvc`: Zookeeper data storage
- `zk-txn-logs-pvc`: Zookeeper transaction logs
- `kafka-data-pvc`: Kafka data storage

### Deployments
- `zookeeper`: Zookeeper deployment (1 replica)
- `kafka`: Kafka broker deployment (1 replica)
- `kowl`: Kowl UI deployment (1 replica)

### Services
- `zookeeper-service`: Internal service for Zookeeper
- `kafka-service`: Internal service for Kafka
- `kowl-service`: LoadBalancer service for Kowl UI

### Routes
- `kowl`: OpenShift route for Kowl UI access

## Accessing Kowl UI

After deployment, the Kowl UI will be accessible via the OpenShift route. The playbook will display the URL at the end of execution.

To get the route manually:
```bash
oc get route kowl -n <namespace>
```

## Notes

- This is a lightweight deployment suitable for development and testing
- Single replica configuration (not suitable for production)
- Persistent storage is required for data persistence
- The namespace can be passed as an environment variable

## License

Apache-2.0

## Author Information

IBM Sterling Team