# deploy

This playbook updates the OpenStack Control Plane CR to enable Telemetry, Ceilometer, the MetricStorage and CloudKitty

Run this after `pre-deploy` and before `create-ck-rating`.

## What it does

1. Adds `CloudKittyPassword` to the `osp-secret` Secret
2. Enables the Telemetry service in the OpenStackControlPlane CR
3. Enables Ceilometer in the OpenStackControlPlane CR
4. Enables and configures MetricStorage with `pvcStorageClass: nfs-storage`
5. Enables CloudKitty in the OpenStackControlPlane CR
6. Waits for the OpenStackControlPlane to become ready (up to 30 minutes)

## Usage

```bash
ansible-playbook deploy/update-deployment-for-ck.yml
```
