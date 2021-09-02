# Playbook Modules

On Ansible there are a multitude of modules of distinct areas and technical specialties to the everyday playbook creation.

### `Set Fact`

Dynamically add or change facts during execution.

```yaml
---
-
  hosts: ubuntu3,centos3
  tasks:
    - name: Set a fact
      set_fact:
        our_fact: Ansible Rocks!
        ansible_distribution: "{{ ansible_distribution | upper }}"

    - name: Show custom fact
      debug:
        msg: "{{ our_fact }}"

    - name: Show ansible_distribution
      debug:
        msg: "{{ ansible_distribution }}"
...
```

Also, it is possible to use facts with the `when` directive

```yaml
---
-
  hosts: ubuntu3,centos3
  tasks:
    - name: Set our installation variables for CentOS
      set_fact:
        webserver_application_port: 80
        webserver_application_path: /usr/share/nginx/html
        webserver_application_user: root
      when: ansible_distribution == 'CentOS'

    - name: Set our installation variables for Ubuntu
      set_fact:
        webserver_application_port: 8080
        webserver_application_path: /var/www/html
        webserver_application_user: nginx
      when: ansible_distribution == 'Ubuntu'

    - name: Show pre-set distribution based facts
      debug:
        msg: "webserver_application_port:{{ webserver_application_port }} webserver_application_path:{{ webserver_application_path }} webserver_application_user:{{ webserver_application_user }}"
...
```

### `Pause`

Pause a playbook for a set amount of time or until a prompt is acknowledged

```yaml
---
-
  hosts: ubuntu3,centos3
  tasks:
    - name: Pause our playbook for 10 seconds
      pause:
        seconds: 10

    - name: Prompt user to verify before continue
      pause:
        prompt: Please check that the webserver is running, press enter to continue
...
```

### `Wait For`

Waits for a condition before continuing

```yaml
---
-
  hosts: ubuntu3,centos3
  tasks:
    - name: Wait for the webserver to be running on port 80
      wait_for:
        port: 80
...
```
### `Assemble`
Allows configuration file to be broken into segments and concatenated to form a destination file. The folder concatenation are done in string sorting order

```txt
# conf.d/centos1

Host centos1
  User root
  Port 2222
```

```txt
# conf.d/default
Port 22
Protocol 2
ForwardX11 yes
GSSAPIAuthentication no
```

```yaml
---

-
  hosts: ubuntu-c
  tasks:
    - name: Assemble conf.d to sshd_config
      assemble:
        src: conf.d
        dest: sshd_config
...
```

```txt
# Result
# sshd_config

Host centos1
  User root
  Port 2222

Port 22
Protocol 2
ForwardX11 yes
GSSAPIAuthentication no
```

### `Add Host`
Allows to dynamically add targets to the running playbook

```yaml
---
-
  hosts: ubuntu-c
  tasks:
    - name: Add centos1 to adhoc_group
      add_host:
        name: centos1
        groups: adhoc_group1, adhoc_group2

-
  hosts: adhoc_group1
  tasks:
    - name: Ping all in adhoc_group1
      ping:
...
```

### `Group By`
Create groups, based on facts

```yaml
---
-
  hosts: all
  tasks:
    - name: Create group based on ansible_distribution
      group_by:
        key: "custom_{{ ansible_distribution | lower }}"
-
  hosts: custom_centos
  tasks:
    - name: Ping all in custom_centos
      ping:
...
```

### `Fetch`
Capture files from remote hosts and targets
```yaml
---
-
  hosts: centos
  tasks:
    - name: Fetch /etc/redhat-release
      fetch:
        src: /etc/redhat-release
        dest: /tmp/redhat-release

...
```