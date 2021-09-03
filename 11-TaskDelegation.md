# Task Delegation

Task Delegation is a directive represented by `delegate_to` that allows to run a a specific task in our playbook in other host or machine.

This is the process of handing over the control of execution to another host.

A good case of use will be to exchange SSH keys between hosts

```yaml
---
-
  hosts: ubuntu-c
  gather_facts: False
  tasks:
    - name: Generate an OpenSSH keypair for ubuntu3
      openssh_keypair:
        path: ~/.ssh/ubuntu3_id_rsa

-
  hosts: linux
  gather_facts: False
  tasks:
    - name: Copy ubuntu3 OpenSSH keypair with permissions
      copy:
        owner: root
        src: "{{ item.0 }}"
        dest: "{{ item.0 }}"
        mode: "{{ item.1 }}"
      with_together:
        - [ ~/.ssh/ubuntu3_id_rsa, ~/.ssh/ubuntu3_id_rsa.pub ]
        - [ "0600", "0644" ]

-
  hosts: ubuntu3
  gather_facts: False
  tasks:
    - name: Add public key to the ubuntu3 authorized_keys file
      authorized_key:
        user: root
        state: present
        key: "{{ lookup('file', '~/.ssh/ubuntu3_id_rsa.pub') }}"

-
  hosts: all
  gather_facts: False
  tasks:
    - name: Check that ssh can connect to ubuntu3 using the ssh tool
      command: ssh -i ~/.ssh/ubuntu3_id_rsa -o BatchMode=yes -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@ubuntu3 date
      changed_when: False
      ignore_errors: True

-
  # This task is not the Ubuntu-c that executes, but Ubuntu 3
  hosts: ubuntu-c, centos1, ubuntu1
  serial: 1
  tasks:
    - name: Add host to /etc/hosts.allow for sshd
      lineinfile:
        path: /etc/hosts.allow
        line: "sshd: {{ ansible_hostname }}.diveinto.io"
        create: True
      delegate_to: ubuntu3

- 
  hosts: all
  gather_facts: False
  tasks:
    - name: Check that ssh can connect to ubuntu3 using the ssh tool
      command: ssh -i ~/.ssh/ubuntu3_id_rsa -o BatchMode=yes -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@ubuntu3 date
      changed_when: False
      ignore_errors: True

-
  hosts: ubuntu3
  gather_facts: False
  tasks:
    - name: Drop SSH connectivity from everywhere else
      lineinfile:
        path: /etc/hosts.deny
        line: "sshd: ALL"
        create: True

-
  # Now just ubuntu-c, ubuntu1 and centos1 can access Ubuntu3
  hosts: all
  gather_facts: False
  tasks:
    - name: Check that ssh can connect to ubuntu3 using the ssh tool
      command: ssh -i ~/.ssh/ubuntu3_id_rsa -o BatchMode=yes -o StrictHostKeyChecking=no -o UserKnownHostsFile=/dev/null root@ubuntu3 date
      changed_when: False
      ignore_errors: True
...
```