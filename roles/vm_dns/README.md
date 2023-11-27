VM DNS
=========

This role adds DNS/DHCP entries in Infoblox for a VM. This ensures VMs can boot with the correct network configuration when created using the `rh.cnv.vm` role. The DNS/DHCP information added will be:

Hostname: `inventory_hostname` from the ansible inventory. Must be within the `dev.lab.local` domain.
IP: `vm_dns_primary_ip`
MAC: `vm_dns_mac` or a seeded random value

Requirements
------------
This role requires credentials for an Infoblox-L1 admin account.

Role Variables
--------------
Default values can be found in [defaults/main.yml](defaults/main.yml)
| Variable | Description |
| --- | --- |
| `vm_dns_configure` | If set to false, DNS configuration will be skipped for this host |
| `vm_dns_delegate_to` | Hostname to run the Infoblox API calls from. Useful if the Infoblox server is not accessible from the host the playbook is run on |
| `vm_dns_mac` | MAC address to use for DHCP. Defaults to a seeded random value. |
| `vm_dns_primary_ip` | IP address of the VM |
| `vm_dns_infoblox_hostname` | Hostname of the Infoblox server |
| `vm_dns_nios_password` | Password for the Infoblox-L1 admin account |
| `vm_dns_nios_username` | Username for the Infoblox-L1 admin account |
| `vm_dns_nios_provider` | Combined definition of the Infoblox hostname and credentials |

Dependencies
------------
```yaml
collections:
  - name: community.general
```

Example Playbook
----------------

```yaml
---
- name: Update DNS and DHCP for VMs
  hosts: all
  # This is required currently since you need to be root to run playbooks on bastion.dev.lab.local
  become: yes
  gather_facts: no
  vars_prompt:
    - name: vm_dns_nios_username
      prompt: "Infoblox username [leave blank if you are not updating DNS]"
      private: no
    - name: vm_dns_nios_password
      prompt: "Infoblox password [leave blank if you are not updating DNS]"
  pre_tasks:
    - name: End DNS playbook if infoblox username is blank
      meta: end_play
      when: not vm_dns_nios_username
  roles:
    - role: rh.cnv.vm_dns
```
