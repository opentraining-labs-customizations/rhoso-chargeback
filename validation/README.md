# Validate CloudKitty Rating

This playbook verifies that the CloudKitty hashmap rating configuration was applied correctly on a RHOSO environment. It checks each resource created by the `create-ck-rating` playbook and fails with a clear message if anything is missing or misconfigured.

## Prerequisites

- Access to the OpenShift cluster with `oc` configured (`KUBECONFIG` set or `~/.kube/config` present)
- An openstackclient pod running in the `openstack` namespace
- Step by step for creating a rating in the training has been completed (or, the `create-ck-rating` playbook has been run previously)

## Running the playbook

```bash
ansible-playbook validation/validate-ck-rating.yml
```

To validate against custom values (must match what was used during creation):

```bash
ansible-playbook validation/validate-ck-rating.yml \
  -e flavor_name=m1.small \
  -e rating_cost=0.5
```

## What it checks

| Check | Fails if |
|-------|----------|
| Hashmap module enabled | Module is missing or disabled |
| Group exists | Group `instance_uptime_flavor_id` not found |
| Service exists | Service `instance` not found |
| Field exists | Field `flavor_id` not found under the service |
| Mapping exists | No mapping with the expected type and group |
| Rating data flowing | Soft check — warns but does not fail if no data yet |

## Rating reports

As part of the "rating data flowing" soft check, the playbook also prints the
rating data calculated for the current UTC day. These steps are informational
and never fail the run:

- **Dataframes** — `openstack rating dataframes get`
- **Summary** — `openstack rating summary get`
- **Summary by type** — `openstack rating summary get -g type`

Each query is scoped with `--begin`/`--end` to the start of the current UTC day
through the current time, so the reports reflect recently rated usage.
