# Deploy Kafka on OpenShift using Ansible Scripts

Using Strimzi or Redhat operator

## Important Notes

### Strimzi Operator Version
This role now supports **Strimzi Operator 0.50.0** which includes:
- Support for Kafka versions: **4.0.0, 4.0.1, 4.1.0, 4.1.1** (default: 4.1.1)
- Java 21 runtime requirement
- Removal of deprecated `log.message.format.version` configuration (not needed for Kafka 4.x)
- Enhanced KRaft mode support (enabled by default)

### Upgrading from Previous Versions
If upgrading from Strimzi operator 0.40.0 or earlier:
1. The default Kafka version has been updated from 3.7.0 to 4.1.1
2. The deprecated `log.message.format.version` configuration has been removed from cluster templates
3. Ensure your OpenShift cluster meets the minimum Kubernetes version requirement (1.27+)

## Preparation

#### 1. Login on OpenShift

Do a login in Openshift console and run the command:

```bash 
oc login --token=sha256~P...k --server=https://c....containers.cloud.xxx.com:31234
```

#### 2. Cloning ansible-ibm-sterling from git

```bash 
git clone https://github.com/ibm-sterling-devops/ansible-ibm-sterling.git
```

#### 3. Set roles path

To run playbook the playbook

```bash 
cd ansible-ibm-sterling

export ANSIBLE_CONFIG=./ansible.cfg 
```

## Deploy Kafka

#### 1. Run the Playbook

To run the playbook:

```bash
ansible-playbook playbooks/tools/kafka.yml
```

#### 2. Check Installation Status

While the playbook is running, you can check the Strimzi operator installation status in another terminal:

**Check if the operator subscription is created:**
```bash
oc get subscription -n sterling-kafka-strimzi
```

**Check if the operator pod is running:**
```bash
oc get pods -n sterling-kafka-strimzi
```

**Check if the CRDs are created:**
```bash
oc get crd | grep kafka.strimzi.io
```

**Check the operator logs (if pod is running):**
```bash
oc logs -n sterling-kafka-strimzi -l name=strimzi-cluster-operator -f
```

**Check the installed CSV (ClusterServiceVersion):**
```bash
oc get csv -n sterling-kafka-strimzi
```

Expected output should show `strimzi-cluster-operator.v0.50.0` in Succeeded phase.

#### 3. Troubleshooting

If the playbook is stuck on "Wait until the kafkas.kafka.strimzi.io CRD is available":

1. **This is normal** - The operator can take 5-10 minutes to fully deploy, especially on first installation
2. Check operator pod status: `oc get pods -n sterling-kafka-strimzi`
3. If pod is in ImagePullBackOff or CrashLoopBackOff, check the logs
4. Verify the operator subscription: `oc describe subscription strimzi-kafka-operator -n sterling-kafka-strimzi`
5. The playbook will retry for up to 10 minutes before timing out

## Environment Variables

For all environment variables, see:

* Role [kafka](../../roles/kafka)