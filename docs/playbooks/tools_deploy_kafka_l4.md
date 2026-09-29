# Deploy Kafka L4 (Lightweight)

This playbook deploys a lightweight Kafka cluster with Zookeeper and Kowl UI on OpenShift/Kubernetes.

## Overview

The Kafka L4 deployment includes:
- **Zookeeper**: Single instance for Kafka coordination
- **Kafka**: Single broker with persistent storage
- **Kowl**: Web-based UI for Kafka management and monitoring

This is a lightweight deployment suitable for development, testing, and small-scale use cases.

## Prerequisites

- OpenShift/Kubernetes cluster access
- `oc` or `kubectl` CLI configured
- Ansible 2.9 or higher
- `kubernetes.core` Ansible collection

## Quick Start

### Using Default Namespace

```bash
ansible-playbook playbooks/tools/kafka_l4.yml
```

This will deploy Kafka L4 in the `kafka-l4` namespace.

### Using Custom Namespace (Environment Variable)

```bash
export KAFKA_NAMESPACE="my-kafka-namespace"
ansible-playbook playbooks/tools/kafka_l4.yml
```

### Using Custom Namespace (Command Line)

```bash
ansible-playbook playbooks/tools/kafka_l4.yml -e kafka_l4_namespace=my-kafka-namespace
```

## Configuration Options

### Default Variables

The following variables can be customized in the playbook or via command line:

| Variable | Description | Default |
|----------|-------------|---------|
| `kafka_l4_namespace` | Deployment namespace | `kafka-l4` |
| `zookeeper_client_port` | Zookeeper client port | `32181` |
| `kafka_port` | Kafka broker port | `29092` |
| `kowl_port` | Kowl UI port | `8080` |
| `zk_data_storage` | Zookeeper data PVC size | `1Gi` |
| `zk_txn_logs_storage` | Zookeeper transaction logs PVC size | `1Gi` |
| `kafka_data_storage` | Kafka data PVC size | `2Gi` |

### Example with Custom Storage

```bash
ansible-playbook playbooks/tools/kafka_l4.yml \
  -e kafka_l4_namespace=my-kafka \
  -e kafka_data_storage=10Gi \
  -e zk_data_storage=5Gi
```

## Deployment Architecture

```
┌─────────────────────────────────────────┐
│         Kafka L4 Namespace              │
│                                         │
│  ┌──────────────┐  ┌──────────────┐   │
│  │  Zookeeper   │  │    Kafka     │   │
│  │  (1 replica) │◄─┤  (1 broker)  │   │
│  └──────┬───────┘  └──────┬───────┘   │
│         │                  │            │
│         │                  │            │
│  ┌──────▼──────────────────▼───────┐   │
│  │         Kowl UI                 │   │
│  │       (Web Interface)           │   │
│  └─────────────┬───────────────────┘   │
│                │                        │
└────────────────┼────────────────────────┘
                 │
                 ▼
          OpenShift Route
         (External Access)
```

## Deployed Resources

### Persistent Volume Claims
- `zk-data-pvc`: Zookeeper data storage (1Gi)
- `zk-txn-logs-pvc`: Zookeeper transaction logs (1Gi)
- `kafka-data-pvc`: Kafka data storage (2Gi)

### Deployments
- `zookeeper`: Zookeeper deployment (1 replica)
- `kafka`: Kafka broker deployment (1 replica)
- `kowl`: Kowl UI deployment (1 replica)

### Services
- `zookeeper-service`: ClusterIP service (port 32181)
- `kafka-service`: ClusterIP service (port 29092)
- `kowl-service`: LoadBalancer service (port 8080)

### Routes
- `kowl`: OpenShift route for external access to Kowl UI

## Accessing Kowl UI

After deployment, access the Kowl UI:

```bash
# Get the route URL
oc get route kowl -n kafka-l4

# Or with custom namespace
oc get route kowl -n <your-namespace>
```

Open the displayed URL in your browser to access the Kowl web interface.

## Verifying Deployment

### Check Pod Status

```bash
oc get pods -n kafka-l4
```

Expected output:
```
NAME                         READY   STATUS    RESTARTS   AGE
kafka-xxxxxxxxxx-xxxxx       1/1     Running   0          2m
kowl-xxxxxxxxxx-xxxxx        1/1     Running   0          1m
zookeeper-xxxxxxxxxx-xxxxx   1/1     Running   0          3m
```

### Check PVC Status

```bash
oc get pvc -n kafka-l4
```

Expected output:
```
NAME               STATUS   VOLUME                                     CAPACITY   ACCESS MODES
kafka-data-pvc     Bound    pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   2Gi        RWO
zk-data-pvc        Bound    pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   1Gi        RWO
zk-txn-logs-pvc    Bound    pvc-xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx   1Gi        RWO
```

### Check Services

```bash
oc get svc -n kafka-l4
```

## Testing Kafka

### Create a Test Topic

```bash
# Get the Kafka pod name
KAFKA_POD=$(oc get pod -n kafka-l4 -l app=kafka -o jsonpath='{.items[0].metadata.name}')

# Create a test topic
oc exec -n kafka-l4 $KAFKA_POD -- kafka-topics \
  --create \
  --topic test-topic \
  --bootstrap-server localhost:29092 \
  --partitions 1 \
  --replication-factor 1
```

### List Topics

```bash
oc exec -n kafka-l4 $KAFKA_POD -- kafka-topics \
  --list \
  --bootstrap-server localhost:29092
```

### Produce Messages

```bash
oc exec -n kafka-l4 $KAFKA_POD -- kafka-console-producer \
  --topic test-topic \
  --bootstrap-server localhost:29092
```

### Consume Messages

```bash
oc exec -n kafka-l4 $KAFKA_POD -- kafka-console-consumer \
  --topic test-topic \
  --from-beginning \
  --bootstrap-server localhost:29092
```

## Troubleshooting

### Pods Not Starting

Check pod events:
```bash
oc describe pod <pod-name> -n kafka-l4
```

Check pod logs:
```bash
oc logs <pod-name> -n kafka-l4
```

### PVC Not Binding

Check PVC status:
```bash
oc describe pvc <pvc-name> -n kafka-l4
```

Ensure your cluster has a default storage class:
```bash
oc get storageclass
```

### Kafka Connection Issues

Verify Zookeeper is running:
```bash
oc logs deployment/zookeeper -n kafka-l4
```

Check Kafka logs:
```bash
oc logs deployment/kafka -n kafka-l4
```

### Kowl UI Not Accessible

Check route status:
```bash
oc get route kowl -n kafka-l4
```

Check Kowl logs:
```bash
oc logs deployment/kowl -n kafka-l4
```

## Cleanup

To remove the Kafka L4 deployment:

```bash
# Delete all resources in the namespace
oc delete all --all -n kafka-l4

# Delete PVCs
oc delete pvc --all -n kafka-l4

# Delete the namespace
oc delete namespace kafka-l4
```

Or with custom namespace:
```bash
oc delete namespace <your-namespace>
```

## Important Notes

- **Not for Production**: This is a single-replica deployment suitable for development/testing only
- **Data Persistence**: Data is stored in PVCs and will persist across pod restarts
- **Resource Requirements**: Ensure your cluster has sufficient resources for all components
- **Network Policies**: Adjust network policies if needed for your environment
- **Security**: Consider adding authentication and encryption for production use

## Related Documentation

- [Kafka Documentation](https://kafka.apache.org/documentation/)
- [Kowl Documentation](https://github.com/cloudhut/kowl)
- [Confluent Platform Documentation](https://docs.confluent.io/)

## Support

For issues or questions:
- Check the troubleshooting section above
- Review pod logs for error messages
- Consult the Kafka and Kowl documentation