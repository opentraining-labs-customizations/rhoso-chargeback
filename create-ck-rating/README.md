# Create CloudKitty Rating

This playbook configures CloudKitty's hashmap rating module on a RHOSO environment. It enables the hashmap module, creates a group, service, and field matching rule, and maps a flavor to a flat cost rate.

## Prerequisites

- Access to the OpenShift cluster with `oc` configured (`KUBECONFIG` set or `~/.kube/config` present)
- An openstackclient pod running in the `openstack` namespace
- Ansible installed on the control node

## Running the playbook

```bash
ansible-playbook create-ck-rating/create-ck-rating.yml
```

To override default variables (e.g. flavor name or cost):

```bash
ansible-playbook create-ck-rating/create-ck-rating.yml \
  -e flavor_name=m1.small \
  -e rating_cost=0.5
```

### Available variables

| Variable              | Default                       | Description                          |
|-----------------------|-------------------------------|--------------------------------------|
| `openstack_namespace` | `openstack`                   | OpenShift namespace for the OS pods  |
| `flavor_name`         | `tiny`                     | Flavor to rate                       |
| `rating_cost`         | `0.3`                         | Cost assigned to the flavor          |
| `group_name`          | `instance_uptime_flavor_id`   | Rating group name                    |
| `service_name`        | `instance`                    | Service matching rule name           |
| `field_name`          | `flavor_id`                   | Field matching rule name             |
| `mapping_type`        | `flat`                        | Mapping type (`flat` or `rate`)      |

## Cleaning up

To remove all rating resources created by this playbook (mappings, field, service, group) and disable the hashmap module:

```bash
ansible-playbook create-ck-rating/cleanup-ck-rating.yml
```

The cleanup playbook looks up each resource by name and deletes them in reverse dependency order. Steps are skipped gracefully if a resource doesn't exist, so it's safe to run after a partial setup or repeated runs.

## What the playbook does

The steps below describe the individual OpenStack rating commands that the playbook automates.

# Enable Hashmap module
$ openstack rating module enable hashmap

# Check the enabled modules
$ openstack rating module list
+-----------+---------+----------+
| Module    | Enabled | Priority |
+-----------+---------+----------+
| hashmap   | True    |        1 |
| noop      | True    |        1 |
| pyscripts | False   |        1 |
+-----------+---------+----------+

# Create a group
$ openstack rating hashmap group create instance_uptime_flavor_id
+---------------------------+--------------------------------------+
| Name                      | Group ID                             |
+---------------------------+--------------------------------------+
| instance_uptime_flavor_id | a338809f-0191-4a16-a963-42354b533203 |
+---------------------------+--------------------------------------+

# Create a service matching rule
$ openstack rating hashmap service create instance
+----------+--------------------------------------+
| Name     | Service ID                           |
+----------+--------------------------------------+
| instance | 6868b43d-0791-4789-a86e-45bab8dea4ab |
+----------+--------------------------------------+

# Create a field matching rule
$ openstack rating hashmap field create 6868b43d-0791-4789-a86e-45bab8dea4ab flavor_id
+-----------+--------------------------------------+--------------------------------------+
| Name      | Field ID                             | Service ID                           |
+-----------+--------------------------------------+--------------------------------------+
| flavor_id | 0819eef5-3b54-49f4-935b-c286de843b8f | 6868b43d-0791-4789-a86e-45bab8dea4ab |
+-----------+--------------------------------------+--------------------------------------+

# Obtain the id of the flavor we want to rate
$ openstack flavor show tiny
+----------------------------+-----------------------+
| Field                      | Value                 |
+----------------------------+-----------------------+
| OS-FLV-DISABLED:disabled   | False                 |
| OS-FLV-EXT-DATA:ephemeral  | 0                     |
| access_project_ids         | None                  |
| description                | None                  |
| disk                       | 1                     |
| id                         | 1                     |
| name                       | tiny               |
| os-flavor-access:is_public | True                  |
| properties                 | hw_rng:allowed='True' |
| ram                        | 512                   |
| rxtx_factor                | 1.0                   |
| swap                       | 0                     |
| vcpus                      | 1                     |
+----------------------------+-----------------------+

# Create a mapping in the instance_uptime_flavor group that will map tiny instance to a cost of 0.3
$ openstack rating hashmap mapping create 0.3 \
 --field-id 0819eef5-3b54-49f4-935b-c286de843b8f \
 --value 1 \
 -g a338809f-0191-4a16-a963-42354b533203 \
 -t flat
+--------------------------------------+-------+--------------------------------+------+--------------------------------------+------------+--------------------------------------+------------+
| Mapping ID                           | Value | Cost                           | Type | Field ID                             | Service ID | Group ID                             | Project ID |
+--------------------------------------+-------+--------------------------------+------+--------------------------------------+------------+--------------------------------------+------------+
| f3d01510-34b6-47d9-aa58-f2b636eed479 | 1     | 0.2999999999999999888977697537 | flat | 0819eef5-3b54-49f4-935b-c286de843b8f | None       | a338809f-0191-4a16-a963-42354b533203 | None       |
+--------------------------------------+-------+--------------------------------+------+--------------------------------------+------------+--------------------------------------+------------+


# Debug the calculated ratings
$ openstack rating dataframes get

# This will not be needed in the final version
export OS_RATING_API_VERSION=2

# Get a rating report
$ openstack rating summary get

# Filter by type
$ openstack rating summary get -g type
