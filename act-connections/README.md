# ACT Topology Generation

This role generates ACT connections for already deployed devices.
The default groups for connecting is the WAN group

```yaml

    ACT_CUSTOM_CONNECTIONS:
      children:
        HQ_SPINES: null
        SITE1_SPINES: null
        SITE2_SPINES: null
```

## Setup

  - There should be an existing user with privleges on the device
    - Chnage ansible_user: to match user/pass
    - On cli run to match the user's password:
      ```bash
      LABPASSPHRASE=arista123abc!
      ```
  - 

## Example Playbook

Use the WAN group , modify the input variables to the role to make the topology you want.

```yaml
---
- name: Build ACT / Connections
  hosts: "{{ target_hosts | default('ACT_CUSTOM_CONNECTIONS') }}"
  gather_facts: false
  ignore_unreachable: false

  tasks:

    - name: Generate ACT / Connections
      vars:
        ansible_user: cvpadmin
        ansible_password: "{{ lookup('env', 'LABPASSPHRASE') }}"

      ansible.builtin.import_role:
        # For testing act-connections
        # name: act_connections/act-connections
        name: act-connections
```

## Role Defaults

```yaml
---
# defaults file for act-topgen

# Input/Output directories and AVD structured config file format
wan_folder: "{{ inventory_dir }}"
output_folder: "{{ wan_folder }}/output_configs"
# Default settings to build
act_build: 1
act_deploy: 0

# Device (veos/generic) service user name uses ansible password for the local account.
# and user's ssh rsa for conneciton
act_service_user: service_act


```
