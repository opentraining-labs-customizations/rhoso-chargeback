# Pre-deploy: Install CloudKitty dependencies

This Ansible playbook installs the Loki Operator with a MinIO-backed storage layer, and the Cluster Observability Operator (COO) on an OpenShift cluster, preparing the environment for CloudKitty.

## What it does

The playbook (`deploy-ck-dependencies.yaml`) runs the following tasks in sequence:

1. Waits for the OpenShift Route API to be available (retries up to 30 times, 10s apart).
2. Applies the Kubernetes manifests from `deploy-loki-for-ck.yaml` using `oc apply`.
3. Waits for the Loki Operator CSV to reach `Succeeded` status (up to 5 minutes).
4. Applies the Kubernetes manifests from `deploy-coo-for-ck.yaml` using `oc apply`.
5. Waits for the COO CSV to be created, then waits (up to 5 minutes) for it to reach `Succeeded` status.

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
ansible-playbook pre-deploy/deploy-ck-dependencies.yaml
```

By default it runs against `localhost`. To target a remote host or override variables:

```bash
ansible-playbook \
  -i <inventory_file> \
  -e "ansible_user_dir=/path/to/rhoso-chargeback" \
  pre-deploy/deploy-ck-dependencies.yaml
```

The playbook references `{{ ansible_user_dir }}/pre-deploy/deploy-loki-for-ck.yaml` and `{{ ansible_user_dir }}/pre-deploy/deploy-coo-for-ck.yaml`, so `ansible_user_dir` must point to the parent of `pre-deploy/`. When running locally, Ansible sets `ansible_user_dir` to the remote user's home directory automatically, so the repo (or at least the `pre-deploy/` directory) needs to be located at `~/pre-deploy/`, or you should override `ansible_user_dir` with `-e` as shown above.
