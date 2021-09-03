# Structuring Ansible

On Ansible there several forms of structuring the code

### `Including Tasks`

Tasks are dynamically included (Run Time)
`when` clause will be evaluated at run time

```yaml
# play1_task2.yaml
---
- name: Play 1 - Task 2
  debug: 
    msg: Play 1 - Task 2
...
```
```yaml
# playbook.yaml
---
-
  hosts: all
  tasks:
     - name: Play 1 - Task 1
       debug: 
         msg: Play 1 - Task 1

     # Including task from file
     - include_tasks: play1_task2.yaml
...
```

### `Importing Tasks`

Tasks are Static included (Compilation Time)
`when` clause will be evaluated at compilation time

```yaml
# play1_task2.yaml
---
- name: Play 1 - Task 2
  debug: 
    msg: Play 1 - Task 2
...
```
```yaml
# playbook.yaml
---
-
  hosts: all
  tasks:
     - name: Play 1 - Task 1
       debug: 
         msg: Play 1 - Task 1

     # Including task from file
     - import_tasks: play1_task2.yaml
...
```