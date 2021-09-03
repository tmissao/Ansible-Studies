# Block

Blocks create logical group of tasks, offering ways to handle task errors, similar to exception handling in many programming languages.

```yaml
---
-
  hosts: linux
  tasks:

    - name: Install patch and python-dns
      block: # Defines a Block
        - name: Install patch
          package:
            name: patch
        
        # This package just exists on Ubuntu, so on CentOS
        # It will fail and force rescue block to executed just 
        # on CentOS hosts
         
        - name: Install python-dnspython
          package:
            name: python-dnspython

      rescue: # Error Handler for Block
        - name: Rollback patch
          package:
            name: patch
            state: absent

        - name: Rollback python-dnspython
          package:
            name: python-dnspython
            state: absent

      always: # Always Execute on Block
        - debug:
            msg: This always runs, regardless
...
```