VM Network
=========

Creates `NodeNetworkConfigurationPolicy`s for Bridge and VLAN interfaces for VMs.

Creates `NetworkAttachmentDefinition`s for these bridge interfaces in specified OpenShift projects.

Requirements
------------
Requires OpenShift Cluster Admin privileges.

Role Variables
--------------
Default values can be found in [defaults/main.yml](defaults/main.yml)
| Variable | Description |
| --- | --- |
| `vm_network_vlans` | List of vlans to create bridge and vlan interfaces to create on each node |
| `vm_network_attachments` | List of dictionaries of project/vlan combinations (see example playbook) |

Dependencies
------------
```yaml
collections:
  - name: kubernetes.core
```

Example Playbook
----------------
This playbook creates multiple VLAN and bridge interfaces for XX08 vlans. Network Attachment Definitions are created in the `testvms` project for each of these vlans + vlans 60-63, which are already configured on the nodes.
```yaml
---
- name: Set up networking for VMs on the OpenShift cluster
  hosts: localhost
  become: no
  gather_facts: yes
  vars:
    vm_network_vlans:
    - 1108
    - 1208
    - 1308
    - 1408
    - 1508
    vm_network_attachments:
    - project: testvms
      vlans:
      - 60
      - 61
      - 62
      - 63
      - 1108
      - 1208
      - 1308
      - 1408
      - 1508
  roles:
  - role: rh.cnv.vm_network
```
