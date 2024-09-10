# leapp
Ansible playbooks for performing RHEL In-Place Upgrades (IPU)

## Requirements

A number of collections from Ansible Galaxy are required to run the playbooks.  These can be found in [collections/requirements.yml](collections/requirements.yml).  This file will be processed by AWX or Ansible Automation Platform when using this repository as a project source which should automatically install the collections when required.

For local testing, the collections can be installed as follows:

`ansible-galaxy collection install -r collections/requirements.yml`

By default, this will install into `~/.ansible/collections/ansible_collections/`

## Using the playbooks with Ansible Automation Platform

The playbooks here can be used standalone or via Ansible Automation Platform.  The configuration for defining the Ansible Automation Platform setup can be found at [aap-configuration](https://github.com/rjo-uk/aap-configuration).

## Playbooks

All playbooks have are set to work on the `upgrade_group` set of hosts rather than `all`.  This is a precaution to ensure that a server is not accidentally upgraded.  A typical scenario is to use an inventory and then set the variable `upgrade_group` to match the host(s) or group(s) that you wish to work on.

### in-place-upgrade.yml

Performs an in-place upgrade using the [infra.leapp.upgrade](https://github.com/redhat-cop/infra.leapp/tree/main/roles/upgrade) role and uses the same variables as that role.

Additional, optional variables:

| Name | Required | Default Value | Possible Values | Description |
| ---- | -------- | ------------- | --------------- | ----------- |
| skip_latest_rhel7 | No | N/A | `yes`/`no` | When set to `yes` the playbook disables all repos except for `rhel-7-server-extras-rpms`.  Allows (for example) a server on RHEL 7.6 to be upgraded to RHEL 8.10 without applying the additional RHEL 7.9 RPMs from `rhel-7-server-rpms`.  This is NOT supported but can save time in a lab environment. |
| ipu_target | No | N/A | `7`, `8` or `9` | If the server is currently running the RHEL major version that matches this variable, the upgrade role will not be called. |

### remediate.yml

Playbook to perform known remediation activities on RHEL 7 and RHEL 8 hosts.  We apply a number of known safe activities to our servers prior to performing a leapp upgrade.  The remediation playbook is normally customised for your environment.

### bigboot.yml

Ensures that the /boot partition is configured for an appropriate size using the infra.lvm_snapshots.bigboot role.  The playbook uses the same variables as that role which.  A user can optionally define the variable `partition_size` which will override `bigboot_partition_size`.


| Name | Required | Default Value | Possible Values | Description |
| ---- | -------- | ------------- | --------------- | ----------- |
| partition_size | No | N/A | See https://github.com/redhat-cop/infra.lvm_snapshots/tree/main/roles/bigboot#bigboot_partition_size-string The variable specifies the minimum required size of the boot partition. If the boot partition is already equal to or greater than the given size, the role will end gracefully making no changes. The value can be either in bytes or with optional single letter suffix (1024 bases) using [human_to_bytes](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/human_to_bytes_filter.html) filter plugin.|

## References and Links

- https://github.com/redhat-partner-tech/leapp-project
- https://leapp.readthedocs.io/en/latest/
- https://github.com/redhat-cop/infra.leapp
- https://galaxy.ansible.com/ui/repo/published/infra/lvm_snapshots/content/role/bigboot/
