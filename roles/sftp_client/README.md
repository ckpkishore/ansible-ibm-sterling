# sftp_client

Deploys a persistent SFTP client pod on OpenShift using the **Red Hat UBI9 Toolbox** image.  
No image extension or custom Dockerfile is needed — `registry.access.redhat.com/ubi9/toolbox:latest`
ships **`openssh-clients` 9.9p1** out of the box, which provides `/usr/bin/sftp`.

## Image verification

The `openssh-clients` package was confirmed present via the Red Hat Container Catalog RPM manifest API
(`/v1/images/id/6ac5403ce778df3f5891b071/rpm-manifest`) for the `ubi9/toolbox:latest` image
(`sha256:5fe5f32837ae83cfbcca32c1244b737ec4702f136aa0338e0969b3d2642a4914`).

## Resources deployed

| Resource | Kind | Purpose |
|---|---|---|
| `sftp-client` | Deployment | Runs the toolbox pod, sleeps indefinitely |
| `sftp-known-hosts` | ConfigMap | Mounts `/root/.ssh/known_hosts` into the pod |

## Quick start

```bash
# 1. Create the project
oc new-project ibm-redhat-sftp

# 2. Run the playbook
ansible-playbook playbooks/tools/sftp_client.yml

# 3. Exec into the pod
oc rsh deployment/sftp-client -n ibm-redhat-sftp

# 4. Start an SFTP session (inside the pod)
sftp user@remote-host
```

## Variables

| Variable | Default | Description |
|---|---|---|
| `sftp_client_namespace` | `ibm-redhat-sftp` | OpenShift project/namespace |
| `sftp_client_image` | `registry.access.redhat.com/ubi9/toolbox:latest` | Container image |
| `sftp_remote_host` | `""` | Remote SFTP host (informational env var in pod) |
| `sftp_remote_port` | `22` | Remote SFTP port (informational env var in pod) |
| `sftp_remote_user` | `""` | Remote SFTP user (informational env var in pod) |
| `sftp_client_sleep_interval` | `infinity` | Pod keep-alive sleep interval |
| `sftp_client_resources_requests_cpu` | `50m` | CPU request |
| `sftp_client_resources_requests_memory` | `64Mi` | Memory request |
| `sftp_client_resources_limits_cpu` | `200m` | CPU limit |
| `sftp_client_resources_limits_memory` | `256Mi` | Memory limit |

## Known-hosts

Populate `roles/sftp_client/templates/sftp-client/configmap.yml.j2` with your remote
server's host key before deploying to enable strict host-key checking.  
If the `known_hosts` data is left empty, the pod's first SFTP connection will
auto-accept the host key (`StrictHostKeyChecking=accept-new`).

## Environment variable override

```bash
SFTP_CLIENT_NAMESPACE=my-project ansible-playbook playbooks/tools/sftp_client.yml
```

<!-- Made with Bob -->
