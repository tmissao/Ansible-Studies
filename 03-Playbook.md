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
  tasks:
  - name: Configure a MOTD
    copy:
      content: "{{ motd }}"
      dest: /etc/motd
...
```

Also, it is possible to pass variables through command line using the `-e` option:
```bash
ansible-playbook motd_playbook.yaml -e 'motd="Testing the motd playbook\n"'
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