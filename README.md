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

# Input/Output directories
wan_folder: "{{ inventory_dir }}/act/custom-connections"
output_folder: "{{ wan_folder }}/output_configs"
act_connection_file: "{{wan_folder}}/connections.yml"
#
# Default settings to build
act_build: false
act_deploy: false
#
# Device (veos/generic) service user name uses ansible password for the local account.
# and user's ssh rsa for conneciton
act_service_user: service_act
act_gre_start_key: "60000"


```

## Install

```bash
ansible-galaxy install -g -f -r example/requirements.yml
```

## Example playbook run

```bash
# Build config for a group called WAN
ansible-playbook /playbooks/act-connections.yml -i "wan/merged_inventory.yml" \
-e "target_hosts=WAN" \
-e "act_build=true"
# The script will create a directory called "act/custom-connections"
# if the act/custom-connections/connecitons.yaml file is not there it will create a demo one commented out

# Deploy config for a group called WAN
ansible-playbook /playbooks/act-connections.yml -i "wan/merged_inventory.yml" \
-e "target_hosts=WAN" \
-e "act_deploy=true" \
-e "act_gre_start_key=\"70000\"" # start gre key tunnel at 70000 and increment

# Ping all devices in group called WAN
ansible-playbook /playbooks/act-connections.yml -i "wan/merged_inventory.yml" \
-e "target_hosts=WAN" \
-e "act_deploy=true" \
--tags ping

```

## Connections file

If you do not have the file in *act/custom-connecitons/connecitons.yml* the script will make it for you.
Your inventory group above needs to have login admin access

example connecitons: 
```yaml
---
links: 
  Spine1-Leaf1-2
  - connection:
    - s1-spine1:Ethernet31
    - s1-leaf1:Ethernet31
  - connection:
    - s1-spine1:Ethernet32
    - s1-leaf2:Ethernet31
```
## Full invenotry file:
```yaml
all:
  children:
    ACT:
      children:
        HQ_FABRIC:
          children:
            HQ_SPINES:
              hosts:
                hq-spine1:
                  ansible_host: 10.0.0.210
                  cloud_ip: 192.168.0.10
                hq-spine2:
                  ansible_host: 10.0.0.170
                  cloud_ip: 192.168.0.11
        SITE1_FABRIC:
          children:
            SITE1_LEAFS:
              hosts:
                s1-leaf1:
                  ansible_host: 10.0.0.53
                  cloud_ip: 192.168.1.20
                s1-leaf2:
                  ansible_host: 10.0.0.211
                  cloud_ip: 192.168.1.21
                s1-leaf3:
                  ansible_host: 10.0.0.57
                  cloud_ip: 192.168.1.22
                s1-leaf4:
                  ansible_host: 10.0.0.218
                  cloud_ip: 192.168.1.23
            SITE1_SPINES:
              hosts:
                s1-spine1:
                  ansible_host: 10.0.0.54
                  cloud_ip: 192.168.1.10
                s1-spine2:
                  ansible_host: 10.18.132.176
                  cloud_ip: 192.168.1.11
        SITE2_FABRIC:
          children:
            SITE2_LEAFS:
              hosts:
                s2-leaf1:
                  ansible_host: 10.18.142.48
                  cloud_ip: 192.168.2.20
                s2-leaf2:
                  ansible_host: 10.18.142.18
                  cloud_ip: 192.168.2.21
                s2-leaf3:
                  ansible_host: 10.18.142.9
                  cloud_ip: 192.168.2.22
                s2-leaf4:
                  ansible_host: 10.18.142.145
                  cloud_ip: 192.168.2.23
            SITE2_SPINES:
              hosts:
                s2-spine1:
                  ansible_host: 10.18.142.2
                  cloud_ip: 192.168.2.10
                s2-spine2:
                  ansible_host: 10.18.142.5
                  cloud_ip: 192.168.2.11
    CVP:
      hosts:
        cvp: null
    HQ_FABRIC_PORTS:
      children:
        HQ_SPINES: null
    HQ_FABRIC_SERVICES:
      children:
        HQ_SPINES: null
    SITE1_FABRIC_PORTS:
      children:
        SITE1_LEAFS: null
        SITE1_SPINES: null
    SITE1_FABRIC_SERVICES:
      children:
        SITE1_LEAFS: null
        SITE1_SPINES: null
    SITE2_FABRIC_PORTS:
      children:
        SITE2_LEAFS: null
        SITE2_SPINES: null
    SITE2_FABRIC_SERVICES:
      children:
        SITE2_LEAFS: null
        SITE2_SPINES: null
    TOOLS:
      children:
        TOOLSSERVER:
          hosts:
            tools-server:
              ansible_host: 10.0.0.209
    WAN:
      children:
        HQ_SPINES: null
        SITE1_SPINES: null
        SITE2_SPINES: null
```