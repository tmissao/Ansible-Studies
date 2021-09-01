# Ansible Facts

Ansible Facts are information of remote hosts which is gathered by the Ansible Controller.

```bash
ansible centos1 -m setup -a 'gather_subset=!all,!min,network
```
```json
centos1 | SUCCESS => {
    "ansible_facts": {
        "ansible_all_ipv4_addresses": [
            "172.18.0.9"
        ],
        "ansible_all_ipv6_addresses": [],
        "ansible_architecture": "x86_64",
        "ansible_default_ipv4": {
            "address": "172.18.0.9",
            "alias": "eth0",
            "broadcast": "172.18.255.255",
            "gateway": "172.18.0.1",
            "interface": "eth0",
            "macaddress": "02:42:ac:12:00:09",
            "mtu": 1500,
            "netmask": "255.255.0.0",
            "network": "172.18.0.0",
            "type": "ether"
        },
        "ansible_default_ipv6": {},
        "ansible_distribution": "CentOS",
    }
}
```

It is important to know that when executing `gather_facts` on a playbook everything under `ansible_facts` will be placed into the variable namespaces and are accessible for each hosts 

```yaml
---
  hosts: all
  tasks:
    - name: Show IP Address
      debug:
        msg: "{{ ansible_default_ipv4.address }}"
...
```

### `Custom Facts`

On Ansible it is possible to create Custom Facts, and by default custom facts are located at `/etc/ansible/facts.d`

A `custom fact` is a executable shell script that return a json like:

```bash
#!/bin/bash
echo {\""date\"" : \""`date`\""}
```

```bash
ansible ubuntu-c -m setup -a 'filter=ansible_local'
```
```json
ubuntu-c | SUCCESS => {
    "ansible_facts": {
        "ansible_local": {
            "getdate1": {
                "date": "Wed Sep  1 14:13:26 UTC 2021"
            }
        },
        "discovered_interpreter_python": "/usr/bin/python3"
    },
    "changed": false
}
```
```yaml
# playbook
---
-
  hosts: ubuntu-c

  tasks:
    - name: Show IP Address
      debug:
        msg: "{{ ansible_default_ipv4.address }}"

    - name: Show Custom Fact 1
      debug:
        msg: "{{ ansible_local.getdate1.date }}"

    - name: Show Custom Fact 1 in hostvars
      debug:
        msg: "{{ hostvars[ansible_hostname].ansible_local.getdate1.date }}"
...
```