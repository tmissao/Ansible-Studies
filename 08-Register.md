# Ansible Register

Ansible Register is a way to capture the output from task execution and store it in a variable

```yaml
# playbook.yaml
---
-
  hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      register: hostname_output

    - name: Show hostname_output
      debug:
        var: hostname_output
    
    - name: Show hostname_output Stdout
      debug:
        var: hostname_output.stdout
...
```

Also is possible to combine `when` directive with `register`
```yaml
# playbook.yaml
---
-
  hosts: linux
  tasks:
    - name: Exploring register
      command: hostname -s
      when: 
        - ansible_distribution == "CentOS" 
        - ansible_distribution_major_version | int >= 8
      register: command_register
 
    # Alternative Way
    # - name: Install patch when changed
    #   yum:
    #     name: patch
    #     state: present
    #   when: command_register.changed

    - name: Install patch when changed
      yum:
        name: patch
        state: present
      when: command_register is changed

    - name: Install patch when skipped
      apt:
        name: patch
        state: present
      when: command_register is skipped
...
```