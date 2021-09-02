# Ansible Execution Mode

When executing Ansible it is possible to choose different strategies to perform a task:

- `Linear`
- `Asynchronous`
- `Serial`
- `Parallel (Free)`


### `Linear`
By default ansible uses the `Linear Strategy`, meaning all hosts will execute the same task before moving on to the next task.

The task below will take `at least 30s` to be executed. So on each task ansible do 6 connections at same time (linux hosts = 6)

```yaml
---
-
  hosts: linux
  gather_facts: false
  tasks:
    - name: Task 1
      command: /bin/sleep 5

    - name: Task 2
      command: /bin/sleep 5

    - name: Task 3
      command: /bin/sleep 5

    - name: Task 4
      command: /bin/sleep 5

    - name: Task 5
      command: /bin/sleep 5

    - name: Task 6
      command: /bin/sleep 5

```

### `Async`
When using async strategy ansible does not keep the ssh connection active, it execute the command and wait for `poll` seconds to come back and check the command status.

`async` defines the timeout value
`poll` defines how long to wait to check the task status

```yaml
---
-
  hosts: linux
  tasks:
    - name: Task 1
      command: /bin/sleep 5
      async: 10
      poll: 1

    - name: Task 2
      command: /bin/sleep 5
      async: 10
      poll: 1

    - name: Task 3
      command: /bin/sleep 5
      async: 10
      poll: 1

    - name: Task 4
      command: /bin/sleep 5
      async: 10
      poll: 1

    - name: Task 5
      command: /bin/sleep 5
      async: 10
      poll: 1

    - name: Task 6
      command: /bin/sleep 5
      async: 10
      poll: 1
...
```

It is important to notice that if `poll = 0`, than Ansible will not check the task status, so it act like `run and leave to be`.

### `Serial`

Serial is another strategy where the number of hosts are split. For example, in the code below ansible will get 2 hosts and execute the each task in linear way, after ended the 6 task, it will get more 2 hosts and execute the task, and finally get the last 2 hosts and execute the 6 tasks.

```yaml
---
-
  hosts: linux
  gather_facts: false
  serial: 2
  tasks:
    - name: Task 1
      command: /bin/sleep 1

    - name: Task 2
      command: /bin/sleep 1

    - name: Task 3
      command: /bin/sleep 1

    - name: Task 4
      command: /bin/sleep 1

    - name: Task 5
      command: /bin/sleep 1

    - name: Task 6
      command: /bin/sleep 1
```

Also it is possible to use incremental and percentage strategy

```yaml
---
-
  hosts: linux
  gather_facts: false
  serial:
   - 16%
   - 34%
   - 50%
...
```

```yaml
---
-
  hosts: linux
  gather_facts: false
  serial:
   - 1
   - 2
   - 3
...
```

### `Free`
On this approach all tasks runs on all hosts at same time, in other words, one task does not wait for the other to start

```yaml
---
-
  hosts: linux
  gather_facts: false
  strategy: free
  tasks:
    - name: Task 1
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 2
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 3
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 4
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 5
      command: "/bin/sleep {{ 10 |random}}"

    - name: Task 6
      command: "/bin/sleep {{ 10 |random}}"
...
```