# Ansible - Studies

This project intends to summarize all the knowledge earned during my studies of istio.

It can be divided into the following sections:

-  [`Introduction`](./01-Introduction.md)
-  [`Ansible Architecture`](./02-AnsibleArchitecture.md)
-  [`Playbook`](./03-Playbook.md )
-  [`Facts`](./04-Facts.md)
-  [`Templates`](./05-Templates.md)
-  [`Playbook Modules`](./06-PlaybookModules.md)
-  [`Dynamic Inventories`](./07-DynamicInventories.md)
-  [`Register`](./08-Register.md)
-  [`Looping`](./09-Looping.md)
-  [`Executions`](./10-Executions.md)
-  [`Task Delegation`](./11-TaskDelegation.md)
-  [`Block`](./12-Block.md)
-  [`Vault`](./13-Vault.md)
-  [`Structuring Ansible`](./14-Structuring.md)
-  [`Roles`](./15-Roles.md)

## Troubleshoot
---

- `debug module` - Print command
```yaml
- 
  hosts: linux
  tasks:
  - name: Print
    debug:
      msg: "Hello World"
```

- `syntax check` - Checks Ansible Playbook Syntax
```bash
ansible-playbook playbook.yaml --syntax-check
```

- `step` - Runs playbook step-by-step
```bash
ansible-playbook playbook.yaml --step
```

- `start-at-task` - Skip all task prior the target task
```
ansible-playbook playbook.yaml --start-at-task='task-name'
```

- `log_path` - On ansible.cfg it is possible to define where ansible should persist the log execution
```ini
# ansible.cfg
[defaults]
inventory = hosts
log_path=my-log.txt
```

- `verbose` - Runs ansible with different levels of verbose
```bash
ansible-playbook -v playbook.yaml # 1 Level
ansible-playbook -vv playbook.yaml # 2 Level
ansible-playbook -vvv playbook.yaml # 3 Level
ansible-playbook -vvvv playbook.yaml # 4 Level
```


## References
---

- [Ansible Course](https://www.udemy.com/course/diveintoansible/)
- [Course Lab](https://github.com/spurin/diveintoansible-lab)
- [Course Repository](https://github.com/spurin/diveintoansible)
- [Best Practices](https://docs.ansible.com/ansible/2.3/playbooks_best_practices.html)
- [Tips and Tricks](https://docs.ansible.com/ansible/latest/user_guide/playbooks_best_practices.html)