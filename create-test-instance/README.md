# Create Test Instance

This playbook launches a test instance on a RHOSO environment so that CloudKitty has real consumption data to rate. It downloads a Cirros image, uploads it to the Image service (Glance), boots an instance from it, and waits for the instance to become `ACTIVE`.

Run this after `create-ck-rating` so the rating rule is already in place and charges the instance's usage as CloudKitty collects it.

## Prerequisites

- Access to the OpenShift cluster with `oc` configured (`KUBECONFIG` set or `~/.kube/config` present)
- An openstackclient pod running in the `openstack` namespace
- A `private` network the instance can attach to
- Ansible installed on the control node

## Running the playbook

```bash
ansible-playbook create-test-instance/create-test-instance.yml
```

To override default variables (e.g. flavor or image):

```bash
ansible-playbook create-test-instance/create-test-instance.yml \
  -e flavor_name=m1.medium \
  -e instance_name=demo-vm
```

### Available variables

| Variable              | Default                       | Description                            |
|-----------------------|-------------------------------|----------------------------------------|
| `openstack_namespace` | `openstack`                   | OpenShift namespace for the OS pods    |
| `image_name`          | `cirros`                      | Name of the image to create            |
| `image_url`           | Cirros 0.6.2 download URL      | Source image to upload to Glance       |
| `flavor_name`         | `m1.small`                    | Flavor to launch the instance with     |
| `instance_name`       | `test-vm`                     | Name of the test instance to launch    |
| `network_name`        | `private`                     | Network the instance attaches to       |

> **Note:** The flavor must be large enough to boot the image. `m1.tiny` is too
> small for the Cirros image, so `m1.small` is used by default. Keep this flavor
> in sync with the `flavor_name` used by `create-ck-rating` so the rating rule
> applies to the instance you launch.

## Cleaning up

To remove the test instance and the image:

```bash
ansible-playbook create-test-instance/cleanup-test-instance.yml
```

The cleanup playbook looks up the instance and image by name and deletes them, skipping gracefully if either does not exist, so it's safe to run repeatedly.

## What the playbook does

The steps below describe the individual OpenStack commands that the playbook automates.

```bash
# Download a Cirros image and upload it to the Image service
$ curl -fsSL https://download.cirros-cloud.net/0.6.2/cirros-0.6.2-x86_64-disk.img | \
  openstack image create --disk-format qcow2 --container-format bare --public cirros

# Launch a test instance using the rated flavor
$ openstack server create --flavor m1.small --image cirros --network private test-vm

# Confirm the instance reached ACTIVE
$ openstack server show test-vm
```
