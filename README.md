# Ansible Databricks

Galaxy role to manage Databricks resources and configurations. Helpful for easily
keeping mission-critical items under source control.

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

- As of version `0.7.2`, the Databricks CLI does not provide the ability to
  create new DBFS mounts
- However, we can check to see if expected mounts exist:

```
ansible-playbook databricks.yml -t dbfs
```
