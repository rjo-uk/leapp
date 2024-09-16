# leapp
Ansible playbooks for performing RHEL In-Place Upgrades (IPU)

## Requirements

A number of collections from Ansible Galaxy or Ansible Automation Hub are required to run the playbooks.  The [collections/requirements.yml](collections/requirements.yml) lists the requirements if using Ansible Galaxy. This file will be processed by AWX or Ansible Automation Platform when using this repository as a project source which should automatically install the collections when required.

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
| skip_latest_minor_updates | No | N/A | `yes`/`no` | When set to `yes` the playbook disables all repos except for `rhel-{6,7}-server-extras-rpms`.  Allows (for example) a server on RHEL 7.6 to be upgraded to RHEL 8.10 without applying the additional RHEL 7.9 RPMs from `rhel-7-server-rpms`.  This is NOT supported but can save time in a lab environment. |
| ipu_target | No | N/A | `7`, `8` or `9` | If the server is currently running the RHEL major version that matches this variable, the upgrade role will not be called. |

In theory, we can go from RHEL 7.6 to RHEL 8.10 or RHEL 6.6 to RHEL 7.9.  The leapp collection performs a [yum update](https://github.com/redhat-cop/infra.leapp/blob/main/roles/upgrade/tasks/update-and-reboot.yml) before the upgrade and needs to have at least one repository active. By toggling the `skip_latest_minor_updates` variable we can decide whether to leave enable/disable all repositories (except extras) and whether a server gets updated to the latest minor RHEL release before rolling forward to the next major RHEL release.

### remediate.yml

Playbook to perform known remediation activities on RHEL 7 and RHEL 8 hosts.  We apply a number of known safe activities to our servers prior to performing a leapp upgrade.  The remediation playbook is normally customised for your environment.

### bigboot.yml

Ensures that the /boot partition is configured for an appropriate size using the infra.lvm_snapshots.bigboot role.  The playbook uses the same variables as that role which.  A user can optionally define the variable `partition_size` which will override `bigboot_partition_size`.


| Name | Required | Default Value | Possible Values | Description |
| ---- | -------- | ------------- | --------------- | ----------- |
| partition_size | No | N/A | The value can be either in bytes or with optional single letter suffix (1024 bases) using [human_to_bytes](https://docs.ansible.com/ansible/latest/collections/ansible/builtin/human_to_bytes_filter.html) filter plugin.| See https://github.com/redhat-cop/infra.lvm_snapshots/tree/main/roles/bigboot#bigboot_partition_size-string The variable specifies the minimum required size of the boot partition. If the boot partition is already equal to or greater than the given size, the role will end gracefully making no changes. |
| use_bigboot | No | N/A |`true`/`false` | If set to `false` this causes the playbook to exit successfully early and not perform any changes to the system.  This is useful in AWX/Ansible Automation Platform workflows where the playbook can be called all of the time and 'toggled' to 'off' by setting this value to `false`. |

The [RHEL 6.10 QCOW image](https://access.redhat.com/downloads/content/69/ver=/rhel---6/6.10/x86_64/product-software) does not ship with a `/boot` partition so if we run the bigboot role on this type of server, it will fail.  Lack of `/boot` is not recommended, but it is valid.  As such we can set `use_bigboot` to `false` to let the `bigboot.yml` playbook acknowledge the fact that it's not meant to make any changes on such a server.

## References and Links

### Documentation

- https://leapp.readthedocs.io/en/latest/

### Ansible Galaxy Collections and Roles
- https://galaxy.ansible.com/ui/repo/published/infra/leapp/
- https://galaxy.ansible.com/ui/repo/published/infra/lvm_snapshots/
- https://galaxy.ansible.com/ui/repo/published/infra/lvm_snapshots/content/role/bigboot/


### Ansible Automation Platform Collections

- https://console.redhat.com/ansible/automation-hub/repo/validated/infra/leapp/
- https://console.redhat.com/ansible/automation-hub/repo/validated/infra/lvm_snapshots/


### Role Sources

- https://github.com/redhat-cop/infra.leapp/
- https://github.com/redhat-cop/infra.lvm_snapshots/

### Examples

- https://github.com/redhat-partner-tech/leapp-project/
