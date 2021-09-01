# Ansible Playbook

A playbook is a scripting functionality of Ansible, and it is written in YAML

An Ansible playbook is a collection of plays and each plays is composed by:

- `Hosts` - Where the play will run and option it will run with
- `Tasks` - The list of tasks that will be executed within the play
- `Handlers` - The list of handlers that are executed as a notify key from a task
- `Roles` - List of Roles to be imported into the play

```yml
# motd_playbook.yaml
---
- name: Update web servers
  hosts: webservers
  remote_user: root

  tasks:
  - name: Ensure apache is at the latest version
    ansible.builtin.yum:
      name: httpd
      state: latest
  - name: Write the apache config file
    ansible.builtin.template:
      src: /srv/httpd.j2
      dest: /etc/httpd.conf

- name: Update db servers
  hosts: databases
  remote_user: root

  tasks:
  - name: Ensure postgresql is at the latest version
    ansible.builtin.yum:
      name: postgresql
      state: latest
  - name: Ensure that postgresql is started
    ansible.builtin.service:
      name: postgresql
      state: started
```

### `Executing Playbook`
In order to execute the playbook run the following command

```bash
# ansible-playbook <playbook-name>
ansible-playbook motd_playbook.yaml
```

## `Using Variables`
In order to user variables in Ansible playbook first is necessary to declare the variable under `vars` section and also use `{{ variable-name }}`

```yaml
# motd_playbook.yaml
---
-
  hosts: centos
  user: root
  gather_facts: false
  vars:
    motd: "Welcome to CentOS Linux - Ansible Rocks\n"
    dict:
      dict_key: This is a dictionary value
    named_list:
      - item1
      - item2
      - item3
      - item4
  tasks:
  - name: Configure a MOTD
    copy:
      content: "{{ motd }}"
      dest: /etc/motd

  - name: Test named dictionary dictionary
      debug:
        msg: "{{ dict }}"
 
  - name: Test named dictionary dictionary key value with dictionary dot notation
      debug:
        msg: "{{ dict.dict_key }}"
 
  - name: Test named dictionary dictionary key value with python brackets notation
      debug:
        msg: "{{ dict['dict_key'] }}"
  
  - name: Test named list
      debug:
        msg: "{{ named_list }}"
 
  - name: Test named list first item dot notation
    debug:
      msg: "{{ named_list.0 }}"
 
  - name: Test named list first item brackets notation
    debug:
      msg: "{{ named_list[0] }}"
...
```

Also, it is possible to pass variables through command line using the `-e` option:
```bash
ansible-playbook motd_playbook.yaml -e 'motd="Testing the motd playbook\n"'
```

### `Using External Variables`

In Ansible is also possible to use external variables, in other words, reference variables from other YAML file using the keyword `vars_files` 

```yaml
# external_vars.yaml
---
external_example_key: example value

external_dict:
   dict_key: This is a dictionary value

external_inline_dict: 
   {inline_dict_key: This is an inline dictionary value}

external_named_list:
   - item1
   - item2
   - item3
   - item4

external_inline_named_list:
   [ item1, item2, item3, item4 ]
...

```
```yaml
# playbook.yaml
---
-
  hosts: centos1
  gather_facts: False
 
  vars_files:
    - external_vars.yaml

  tasks:
    - name: Test external dictionary key value
      debug:
        msg: "{{ external_example_key }}"

    - name: Test external named dictionary dictionary
      debug:
        msg: "{{ external_dict }}"

    - name: Test external named dictionary dictionary key value with dictionary dot notation
      debug:
        msg: "{{ external_dict.dict_key }}"

    - name: Test external named dictionary dictionary key value with brackets notation
      debug:
        msg: "{{ external_dict['dict_key'] }}"
 
    - name: Test external named inline dictionary dictionary
      debug:
        msg: "{{ external_inline_dict }}"
 
    - name: Test external named inline dictionary dictionary key value with dictionary dot notation
      debug:
        msg: "{{ external_inline_dict.inline_dict_key }}"
 
    - name: Test external named inline dictionary dictionary key value with brackets notation
      debug:
        msg: "{{ external_inline_dict['inline_dict_key'] }}"
 
    - name: Test external named list
      debug:
        msg: "{{ external_named_list }}"
 
    - name: Test external named list first item dot notation
      debug:
        msg: "{{ external_named_list.0 }}"
 
    - name: Test external named list first item brackets notation
      debug:
        msg: "{{ external_named_list[0] }}"
 
    - name: Test external inline named list
      debug:
        msg: "{{ external_inline_named_list }}"
 
    - name: Test external inline named list first item dot notation
      debug:
        msg: "{{ external_inline_named_list.0 }}"
 
    - name: Test external inline named list first item brackets notation
      debug:
        msg: "{{ external_inline_named_list[0] }}"
...
```

### `Using Prompt Variables`
Also is possible to prompt the user requiring variables using the keyword `vars_prompt`

```yaml
---
-
  hosts: centos1
  gather_facts: false

  vars_prompt:
    - name: username
      private: false

    - name: password
      private: true
 
  tasks:
    - name: Test vars_prompt
      debug:
        msg: "{{ username }}"
    - name: Test vars_prompt
      debug:
        msg: "{{ password }}"
...
```

### `Using Host Variables`
Furthermore it is possible to use variables defined in the inventory file using the magic variable `hostvars`

```ini
# hosts
[control]
ubuntu-c ansible_connection=local

[centos]
centos1 ansible_port=2222
centos[2:3]

[centos:vars]
ansible_user=root

[ubuntu]
ubuntu[1:3]

[ubuntu:vars]
ansible_become=true
ansible_become_pass=password

[linux:children]
centos
ubuntu
```
```yaml
# playbook
---
-
  hosts: centos
  gather_facts: True
 
  tasks:
    - name: Test hostvars with an ansible fact and collect ansible_port, dot notation, default if not found
      debug:
      # The '|' represents a default value if ansible_port is not defined, in the host file just centos1 has ansible_port defined
        msg: "{{ hostvars[ansible_hostname].ansible_port | default('22') }}"
        
    - name: Test groupvars with an ansible fact, show that the variable is also accessible from the hostvars section
      debug:
        msg: "{{ hostvars[ansible_hostname].ansible_user }}"
...

```

### `Handlers`
Handlers follow the same syntax of tasks, and are just executed at the end of task with the same `notify key` and if there is a change

```yaml
---
-
  hosts: centos
  user: root
  gather_facts: false
  vars:
    motd: "Welcome to CentOS Linux - Ansible Rocks\n"
  tasks:
  - name: Configure a MOTD
    copy:
      content: "{{ motd }}"
      dest: /etc/motd
    notify: MOTD changed # notify handler key
  handlers:
  - name: MOTD changed # handler key
    debug:
      msg: The MOTD was changed
...
```

### `Conditional Execution`

On Ansible it is possible to conditionally execute a task based on a condition using the `when` keyword

```yaml
---
-
  hosts: linux
  vars:
    motd_centos: "Welcome to CentOS Linux - Ansible Rocks!\n"
    motd_ubuntu: "Welcome to Ubuntu Linux - Ansible Rocks"
  tasks:
  - name: Configure a MOTD
    copy:
      content: "{{ motd_centos }}"
      dest: /etc/motd
    # Executes when ansible_distribution fact got by setup module is equals to CentOS (CentOS Hosts)
    when: ansible_distribution == "CentOS"
    notify: MOTD changed
  
  - name: Configure a MOTD
    copy:
      content: "{{ motd_ubuntu }}"
      dest: /etc/motd
     # Executes when ansible_distribution fact got by setup module is equals to Ubuntu (Ubuntu Hosts)
    when: ansible_distribution == "Ubuntu"
    notity: "MOTD changed"

  handlers:
  - name: MOTD changed
    debug:
      msg: The MOTD was changed
...
```

### `Role`
A Role in ansible is a way to associating variables, tasks and handlers based on a group in file structure

## References
---

- [`PlayBook Keywords`](https://docs.ansible.com/ansible/latest/reference_appendices/playbooks_keywords.html#playbook-keywords)