# rhoso-chargeback

This repository provides a set of Ansible playbooks to set up the dependencies required to use the CloudKitty service in an RHOSO 18 FR5 environment.

The was mainly created for the [RHOSO Chargeback and Observability Fundamentals training](https://redhatquickcourses.github.io/rhoso-chargeback-v2/rhoso-chargeback-v2/1.0.0/index.html).

There are also playbooks for automatically executing the step by step in the training and to verify that the training was correctly finished.

## Repository structure

### pre-deploy

Ansible playbook that installs Loki and an S3-compatible storage backend (MinIO) required by CloudKitty. Run this before starting the training exercise.

See [pre-deploy/README.md](pre-deploy/README.md) for details.

### deploy

Ansible playbook that updates the OpenStack deployment to enable CloudKitty and its dependencies (telemetry, Ceilometer, MetricStorage). Run this after `pre-deploy` and before `create-ck-rating`.

See [deploy/README.md](deploy/README.md) for details.

### create-ck-rating

Ansible playbook that automates the execution of the step-by-step exercise available in the training.

See [create-ck-rating/README.md](create-ck-rating/README.md) for details.

### validation

Ansible playbook that validates that the step-by-step exercise has been successfully executed.

See [validation/README.md](validation/README.md) for details.