VM
=========

This role deploys a development virtual machine to a specified CNV OpenShift cluster.

This role executes from localhost for each VM defined.

Requirements
------------
You must be logged in to the OpenShift cluster using the `oc` command prior to running the role.

Network Attachment Definitions must exist for each bridge interface used in `vm_networks`. The `rh.cnv.vm_network` role can be used for this.

The `rh.cnv.vm_dns` role can be used to enable DHCP and DNS for the VM.

The `rh.cnv.vm_deploy` playbook can be used to set up all of this in sequence.

Base PVC images referenced in `vm_source_pvc` must be created using the `virtctl` tool or vm_image_pvc role. The below example, is manually uploading the image using `virtctl`
```
oc project openshift-virtualization-os-images
virtctl image-upload dv rhel8u7 --size=20Gi --access-mode ReadWriteMany  --image-path=~/Downloads/rhel-8.7-x86_64-kvm.qcow2
```

Role Variables
--------------
Default values can be found in [defaults/main.yml](defaults/main.yml)
| Variable | Description |
| --- | --- |
| `vm_ssh_pubkey` | User's SSH public key to be injected into virtual machine image |
| `vm_domain` | Virtual Machine's domain |
| `vm_project` | OpenShift project/namespace to deploy the VM |
| `vm_root_password` | Initial root password |
| `vm_os` | Operating system for the VM |
| `vm_cpu` | Number of CPUs to allocate for the VM |
| `vm_memory` | Memory in MiB to allocate for the VM |
| `vm_source_pvc` | Persistent Volume Claim to use as a base for the VM (defaults for each OS can be found in [vars/](vars/)) |
| `vm_source_pvc_namespace` | Namespace containing the source PVC |
| `vm_disk_size` | Size for the VM's root disk (defaults for each OS can be found in [vars/](vars/)) |
| `vm_networks` | List of network interfaces to add to the VM. `bridge` is the only supported interface type. Does not include the `default` network interface. Additional values accepted by a cloudinit `cloudconfig` file can be specified under the `config` key (see example) |
| `vm_enable_default_network` | Enable the `default` interface that is connected to the OpenShift SDN |
| `vm_name` | Name of the VM |
| `vm_satellite_capsule` | Satellite capsule for the VM |
| `vm_activation_key` | Activation key to register VM |
| `vm_userdata_content` | Content of userdata file used by cloudinit |


Vars example
```yaml
vm_root_password: !vault |
  $ANSIBLE_VAULT;1.1;AES256
  ....
vm_project: testvms
vm_networks:
- type: bridge
  source_vlan: 10
  # specifying a random seeded MAC address like this will match up with the vm_dns role
  mac: "{{ '52:54:00' | community.general.random_mac(seed=inventory_hostname + '0') }}"
- type: bridge
  source_vlan: 101
  # ommiting the MAC address will cause cnv to assign a random one
- type: bridge
  source_vlan: 100
  # cloudconfig options can be specified like this:
  config:
    dhcp4: no
    mtu: 9000
    gateway4: 192.168.0.1
    addresses:
    - "192.168.0.10/24"
    nameservers:
      search:
      - local
      addresses:
      - 192.168.0.1
```

Dependencies
------------
```yaml
roles:
  - name: rhel-system-roles.network
collections:
  - name: kubernetes.core
  - name: community.general
```

Example Playbook
----------------

```yaml
---
# for this playbook to work, all VMs must be in a hostgroup called "virtual_machines"
- name: Deploy build/test VMS
  import_playbook: rh.cnv.vm_deploy

- name: Ensure VMs are available
  hosts: virtual_machines
  gather_facts: no
  remote_user: root
  vars:
    ansible_password: "{{ vm_root_password }}"
  tasks:
    - name: Ensure all VMs are online
      ansible.builtin.wait_for_connection:
        timeout: 600
        sleep: 10
```
