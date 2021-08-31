# Ansible Architecture 

On this section we are going to explore several components of ansible architecture

## Ansible Configuration Files
---

There are several ways to provide configuration to ansible, and they are ordered in these priority:

1. `ANSIBLE_CONFIG` - Environment Variable
2. `./ansible.cfg` - In the current directory
3. `~/.ansible.cfg` - Hidden file on user home directory
4. `/etc/ansible/ansible.cfg` - Typically provided by a system installation

## Ansible Inventories
---

Ansible works against multiple systems in the infrastructure at the same time. And it is possible to select portions of systems listed in Ansible's inventory file.

```bash
# ansible.cfg file
[defaults]
inventory = hosts
host_key_checking = False

# hosts file
[all]
30.30.255.228
30.30.255.229
centos1
centos2
ubuntu3

# Executing Ansible
ansible all -m ping
```

Also it is possible to divide the host file in groups

```bash
# ansible.cfg file
[defaults]
inventory = hosts
host_key_checking = False

# hosts file
[vms]
30.30.255.228
30.30.255.229

[centos]
centos1
centos2

[ubuntu]
ubuntu3

# Executing Ansible
ansible '*' -m ping # executes command in all groups
ansible all -m ping # executes command in ALL groups
ansible vms -m ping # executes command in vms group
ansible centos -m ping # executes command in centos group
```

### `Output In one Line`
To display ansible output in one line pass the `-o` option
```bash
ansible all -m ping -o
```

### `Listing Hosts`

In order to list a group of hosts execute this command:

```bash
# ansible <group-name> --list-hosts
ansible centos --list-hosts
```

### `Targeting Hosts`

When executing ansible the first argument acts like a target, and ansible can target host, host group and regular expressions

```bash
ansible '*' -m ping      # ALL
ansible centos -m ping   # Host Group
ansible centos1 -m ping  # Host
ansible ~.*1 -m ping     # Ubuntu1 and Centos1
```

### `Defining Login User on Hosts`
To specify which user ansible should use to login in the hosts use the variable `ansible_user`

```bash
# ansible.cfg file
[defaults]
inventory = hosts
host_key_checking = False

# hosts file
[centos]
centos1 ansible_user=root
centos2 ansible_user=root
centos3 ansible_user=root

# Executing Ansible
ansible all -m command -a 'id' -o
```
### `Scaling User on Host`
To connect with a regular user on a host and elevate the privileged use the `ansible_become` and `ansible_become_pass`

```bash
# ansible.cfg file
[defaults]
inventory = hosts
host_key_checking = False

# hosts file
[ubuntu]
# <host-address> ansible_become=true ansible_become_pass=<password>
ubuntu1 ansible_become=true ansible_become_pass=password
ubuntu2 ansible_become=true ansible_become_pass=password
ubuntu3 ansible_become=true ansible_become_pass=password

# Executing Ansible
ansible all -m command -a 'id' -o
```

### `Using Custom Port`
In order to use a custom port to connect it is necessary to user the variable `ansible_port` or `:<port>`

```bash
# ansible.cfg file
[defaults]
inventory = hosts
host_key_checking = False

# hosts file
[centos]
# <host-address> ansible_port=<port>
centos1 ansible_user=root ansible_port=2222
# <host-address>:port
centos2:222 ansible_user=root
centos3 ansible_user=root

# Executing Ansible
ansible all -m command -a 'id' -o
```

### `Using Ranges`

Allows to shrink the host files, using ranges, like a for loop

```bash
# ansible.cfg file
[defaults]
inventory = hosts
host_key_checking = False

# hosts file
[control]
ubuntu-c ansible_connection=local

[centos]
centos1:2222 ansible_user=root
centos[2:3] ansible_user=root

[ubuntu]
ubuntu[1:3] ansible_become=true ansible_become_pass=password

# Executing Ansible
ansible all -m command -a 'id' -o
```

### `Using Group Vars`

Group Vars on Inventory file allows to setup up variables for specific groups avoiding code duplication. To do it just create section with the group's name and de sufix `:vars`

```bash
# ansible.cfg file
[defaults]
inventory = hosts
host_key_checking = False

# hosts file
[control]
ubuntu-c ansible_connection=local

[centos]
centos1:2222
centos[2:3]

[centos:vars] # centos group variables
ansible_user=root

[ubuntu]
ubuntu[1:3]

[ubuntu:vars] # ubuntu group variables
ansible_become=true 
ansible_become_pass=password

# Executing Ansible
ansible all -m command -a 'id' -o
```

### `Using Group of Groups`

On Ansible Inventory file also it is possible to declare a group of group, for example if we desire to create a group called `linux` and add the groups `ubuntu` and `centos` to it. In order to do that, just create a group with the suffix `:children` and add the groups under it

```bash
# ansible.cfg file
[defaults]
inventory = hosts
host_key_checking = False

# hosts file
[control]
ubuntu-c ansible_connection=local

[centos]
centos1:2222
centos[2:3]

[centos:vars]
ansible_user=root

[ubuntu]
ubuntu[1:3]

[ubuntu:vars]
ansible_become=true 
ansible_become_pass=password

[linux:children] # group of groups
ubuntu
centos

# Executing Ansible
ansible linux -m command -a 'id' -o
```

## Ansible Modules
---

Modules or Tasks Plugins/Library Plugins are discrete units of code that can be used from the command line or in a playbook task. Ansible executes each module, usually on the remote target node, and collects return values.

### `Setup Module`

Gather information (facts) about remote hosts
```bash
# ansible <host> -m setup
ansible centos1 -m setup
```

### `File Module`
Used for file, symlinks and directory manipulation
```bash
# ansible <host> -m file
# creates file
ansible all -m file -a 'path=/tmp/test state=touch'
# change file permission
ansible all -m file -a 'path=/tmp/test state=file mode=600'
```

### `Copy Module`
Used for copying files, from the local or remote, to a location on the remote

```bash
# ansible <host> -m file
# Copy file to destination
ansible all -m copy -a 'src=/tmp/x dest=/tmp/x'
```

### `Command Module`
Used for executing remote commands. It is not processed through the shell, so, variables like $HOME and operation (>,<,|,&) will not work

```bash
# ansible <host> -m file
# Execute on Host
ansible all -m command -a 'hostname'
```