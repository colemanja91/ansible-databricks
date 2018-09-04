# Ansible Databricks

Galaxy role to manage Databricks resources and configurations. Helpful for easily
keeping mission-critical items under source control. Uses the Databricks CLI, and attempts to
apply idempotency to most configurable components.

## Prerequisites

- Databricks organization account set up in AWS or Azure
- Databricks user account within your organization
- Ansible >= 2.6
- Token access to Databricks

## Using in your Ansible playbook

- Install in your Ansible repo: `ansible-galaxy install colemanja91.ansible-databricks`
- Example playbook:

```
---
- hosts:
    - heaven
  vars_files:
    - "my/secret/file.yml"
    - "my/ansible/variables.yml"
  roles:
    - { role: colemanja91.ansible-databricks }
```

## Tasks

### CLI installation and setup

- By default, attempts to install the CLI via pip
- Sets up configuration file
- Expects either an Ansible variable `databricks_token` or environment variable
  `DATABRICKS_TOKEN` to be defined
  + Recommended for each Ansible user to define the environment variable at their
    system-level, to ensure they are using their own account and have proper
    permissions
  + Ansible variable should be used only with a shared Databricks account (not recommended)
- Automatically run for any role execution

### DBFS mounts

- https://docs.databricks.com/user-guide/dbfs-databricks-file-system.html
- As of version `0.7.2`, the Databricks CLI does not provide the ability to
  create new DBFS mounts
- However, we can check to see if expected mounts exist:

```
ansible-playbook databricks.yml -t dbfs
```

- The variable `databricks_dbfs` is used to configure this task:

```
databricks_dbfs:
  - s3_path: "s3a://my-s3-bucket-name"
    dbfs_mount: "/mnt/my-dbfs-mount"
```

### Databricks Secrets

- https://docs.databricks.com/user-guide/secrets/index.html
- Each secret must have an associated scope
- Recommended to store secrets in repo using Ansible Value (**not** plain-text), then reference
  in the secrets config
- The variable `databricks_secrets` is used to configure this task:

```
databricks_secrets:
  - scope: "my_secret_scope"
    key: "my_secret_name"
    value: "{{ my_secret_variable }}"
```

### Libraries

- **NOTE:** Currently only libraries used on Databricks Jobs are supported
- Support for interactive cluster libraries is TBD
- Adds the target file from local file system to a given DBFS path
- The variable `databricks_libraries` is used to configure this task:

```
databricks_libraries:
  - src: "../path/to/my/jar.jar"
    dbfs: "dbfs:/target/path/to/my/jar.jar"
```

### Jobs

- https://docs.databricks.com/user-guide/jobs.html
- Configuring and managing jobs in Databricks
- The variable `databricks_jobs` is used to configure this task
- The content of `databricks_jobs` is translated to JSON and passed to the Databricks API,
  so it's structure should mimic what is expected in the documentation:
  + Job configuration: https://docs.databricks.com/api/latest/jobs.html#create
  + Cluster configuration (AWS): https://docs.databricks.com/api/latest/clusters.html#create
  + Cluster configuration (Azure): https://docs.azuredatabricks.net/api/latest/clusters.html#create
- Example `databricks_jobs` (for AWS):

```
databricks_jobs:
  - name: "my_job"
    notebook_task:
      notebook_path: "/User/Jeremy/my_notebook"
    new_cluster:
      autoscale:
        min_workers: 2
        max_workers: 4
      spark_version: "4.3.x-scala2.11"
      node_type_id: "r4.2xlarge"
      aws_attributes:
        first_on_demand: 0
        availability: ON_DEMAND
        zone_id: "{{ aws_zone }}"
        instance_profile_arn: "{{ aws_instance_profile_arn }}"
        ebs_volume_type: GENERAL_PURPOSE_SSD
        ebs_volume_count: 1
        ebs_volume_size: 100
      custom_tags:
        - key: environment
          value: "production"
      spark_env_vars:
        - key: "ENVIRONMENT"
          value: "production"
      enable_elastic_disk: true
    libraries:
      - jar: "dbfs:/target/path/to/my/jar.jar"
    email_notifications:
      on_start: []
      on_success: []
      on_failure:
        - example@example.com
    max_concurrent_runs: 1
```
