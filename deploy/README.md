# deploy

This playbook updates the OpenStack Control Plane CR to enable Telemetry, Ceilometer, the MetricStorage and CloudKitty, and rolls out the telemetry service to the compute nodes (EDPM) so CloudKitty gets per-instance consumption data

Run this after `pre-deploy` and before `create-ck-rating`.

## What it does

1. Adds `CloudKittyPassword` to the `osp-secret` Secret
2. Enables the Telemetry service in the OpenStackControlPlane CR
3. Enables Ceilometer in the OpenStackControlPlane CR
4. Enables and configures MetricStorage with `pvcStorageClass: nfs-storage`
5. Enables CloudKitty in the OpenStackControlPlane CR
6. Waits for the OpenStackControlPlane to become ready (up to 30 minutes)
7. Adds the `telemetry` service to the OpenStackDataPlaneNodeSet (so `ceilometer-compute` runs on the compute nodes)
8. Creates an OpenStackDataPlaneDeployment to roll out the change and waits for it to complete (up to 60 minutes)

## Usage

```bash
ansible-playbook deploy/update-deployment-for-ck.yml
```
