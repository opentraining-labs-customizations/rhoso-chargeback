# Pre-deploy: Install Loki for CloudKitty

This Ansible playbook installs the Loki Operator and a MinIO-backed storage layer on an OpenShift cluster, preparing the environment for CloudKitty.

## What it does

The playbook (`cloudkitty-pre_deploy-install_loki.yml`) runs three tasks in sequence:

1. Waits for the OpenShift Route API to be available (retries up to 30 times, 10s apart).
2. Applies the Kubernetes manifests from `deploy-loki-for-ck.yaml` using `oc apply`.
3. Waits for the Loki Operator CSV to reach `Succeeded` status (up to 5 minutes).

The manifests (`deploy-loki-for-ck.yaml`) create the following resources:

- **Namespaces**: `openstack`, `openshift-operators-redhat`, `minio-dev`
- **Loki Operator**: OperatorGroup + Subscription (channel `stable-6.4`, source `gls-catalog-cs`)
- **MinIO** (S3-compatible object store for Loki): Pod, PVC (10Gi, `nfs-storage`), Service, console/API Routes
- **Secret** `cloudkitty-loki-s3` in namespace `openstack` with MinIO credentials

## Prerequisites

- `ansible` installed
- `oc` (OpenShift CLI) authenticated against your target cluster
- A valid `KUBECONFIG` pointing to the target cluster

## How to run

```bash
ansible-playbook /path/to/pre-deploy/cloudkitty-pre_deploy-install_loki.yml
```

By default it runs against `localhost`. To target a remote host or override variables:

```bash
ansible-playbook \
  -i <inventory_file> \
  -e "ansible_user_dir=/path/to/rhoso-chargeback" \
  /path/to/pre-deploy/cloudkitty-pre_deploy-install_loki.yml
```

The playbook references `{{ ansible_user_dir }}/pre-deploy/deploy-loki-for-ck.yaml`, so `ansible_user_dir` must point to the parent of `pre-deploy/`. When running locally, Ansible sets `ansible_user_dir` to the remote user's home directory automatically, so the repo (or at least the `pre-deploy/` directory) needs to be located at `~/pre-deploy/`, or you should override `ansible_user_dir` with `-e` as shown above.
