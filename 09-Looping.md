# Looping

Looping is a powerful concept in Ansible, allowing great flexibility on playbooks

### `With_Items`

It is possible to define a loop using the directive `with_items`, so each task will be executed N times, according with the number of items. Each item could be accessed using the `item` variable

```yaml
---
-
  hosts: linux
  tasks:
    - name: Creating user
      user:
        name: "{{ item }}" # creates a user for each Item in the list
      with_items: # List of items
        - james
        - hayley
        - lily
        - anwen
...
```

Also it is possible to loop dictionaries.

```yaml
---
-
  hosts: linux
  tasks:
    - name: Creating user
      user:
        name: "{{ item.key }}" # access dictionary's key
        comment: "{{ item.value.full_name }}" # access dictionary's values
      with_dict: 
        james: 
          full_name: James Spurin
        hayley: 
          full_name: Hayley Spurin
        lily: 
          full_name: Lily Spurin
        anwen:
          full_name: Anwen Spurin
...
```

### `With_subelements`

with_subelements is a way to traverse nested key from a list of dictionaries

```yaml
---
- hosts: ubuntu-c
  gather_facts: no
  vars:
    families:
      - surname: Smith
        children:
          - name: Mike
            age: 4
          - name: Kate
            age: 7
      - surname: Sanders
        children:
          - name: Pete
            age: 12
          - name: Sara
            age: 17

  tasks:
    - name: List children
      debug:
        msg: "Family={{ item.0.surname }} Child={{ item.1.name }} Age={{ item.1.age }}"
      with_subelements:
        - "{{ families }}"
        - children
...
```

### `With_nested`

composes a list with nested elements of other lists

```yaml
---
-
  hosts: linux
  tasks:
    # Creates photos, movies, documents directories for each of users (james, hayley ... sara)
    - name: Creating user directories
      file:
        dest: "/home/{{ item.0 }}/{{ item.1 }}" # .1 refers to directory
        owner: "{{ item.0 }}" # refers to the name
        group: "{{ item.0 }}"
        state: directory
      with_nested:
        - [ james, hayley, freya, lily, anwen, ana, abhishek, sara ]
        - [ photos, movies, documents ]
...

```

### `With_together`
merges lists into synchronized list. For example, [ ‘a’, ‘b’ ] and [ 1, 2 ] turn into [ (‘a’,1), (‘b’, 2) ]

```yaml
---
-
  hosts: linux
  tasks:
    - name: Creating user directories
      file:
        dest: "/home/{{ item.0 }}/{{ item.1 }}"
        owner: "{{ item.0 }}"
        group: "{{ item.0 }}"
        state: directory
      with_together:
        - [ james, hayley, freya, lily, anwen, ana, abhishek, sara ]
        - [ tech, psychology, acting, dancing, playing, japanese, coffee, music ]
 ...
``` 

### `With_file`
pass the content of a  list of files to item variable

```yaml
---
-
  hosts: linux
  tasks:
    - name: Create authorized key
      authorized_key:
        user: james
        key: "{{ item }}"
      with_file:
        - /home/ansible/.ssh/id_rsa.pub
        - /another/file
...
```

### `With_sequence`
Iterates within a sequence range, like a for loop

```yaml
---
-
  hosts: linux
  tasks:
    - name: Create sequence directories
      file:
        dest: "/home/james/sequence_{{ item }}"
        state: directory
        # starts on 0 up to 100, increments 10
        # 0 10 20 30 40 50 60 70 80 90 100
      with_sequence: start=0 end=100 stride=10
...
```

```yaml
---
-
  hosts: linux
  tasks:
    - name: Create sequence directories
      file:
        dest: "{{ item }}"
        state: directory
      with_sequence: start=0 end=100 stride=10 format=/home/james/sequence_%d
...
```

```yaml
---
-
  hosts: linux
  tasks:
    - name: Create hex sequence directories
      file:
        dest: "{{ item }}"
        state: directory
      # 1 to 5 using hex decimal
      with_sequence: count=5 format=/home/james/count_sequence_%x
...
```

### `With_random_choice`
Gets an item in random way from a list

```yaml
---
-
  hosts: linux
  tasks:
    - name: Create random directory
      file:
        dest: "/home/james/{{ item }}"
        state: directory
      with_random_choice:
        - "google"
        - "facebook"
        - "microsoft"
        - "apple"
...
```

### `until`

Runs a task until a condition is met

```bash
# random.sh

#!/bin/bash
echo $((1 + RANDOM % 10))
```

```yaml
---
-
  hosts: linux
  tasks:
    - name: Run a script until we hit 10
      script: random.sh
      register: result
      retries: 100
      until: result.stdout.find("10") != -1
      delay: 1
...
```