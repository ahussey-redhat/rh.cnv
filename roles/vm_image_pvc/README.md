VM Image PVC
=========

Uploads a qcow2 image to a `PersistentVolumeClaim` in `openshift-virtualization-os-images project`

The `PersistentVolumeClaim` is then used to create the Virtual Machine.

Requirements
------------

Requires OpenShift Cluster Admin privileges.

Openshift command line tools:
* virtctl
* oc

Role Variables
--------------

Default values can be found in [defaults/main.yml](defaults/main.yml)
| Variable | Description |
| --- | --- |
| `images` | List of qcow2 image url to upload |
| `pvc_size` | Size of pvc in GiB |

Example Playbook
----------------
This playbook creates a Fedora 39 pvc. These pvc can then be attached to a virtual machine on openshift.

``` yaml
---
- name: Create pvc images
  hosts: localhost
  become: false
  gather_facts: true
  vars:
    images:
      - http://download.fedoraproject.org/pub/fedora/linux/releases/39/Cloud/x86_64/images/Fedora-Cloud-Base-39-1.5.x86_64.qcow2
    pvc_size: 20
  roles:
    - role: rh.cnv.vm_image_pvc
```
