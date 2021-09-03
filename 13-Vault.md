# Vault

Ansible vault is a way to store sensitive information on your playbooks, allowing to encrypt and decrypt variables as desired.

For example, lets suppose we should encrypt the value `ansible_become_pass` on our ubuntu playbook

```bash
# ansible-vault encrypt_string --ask-vault-pass --name <secret_name> <secret_value>

ansible-vault encrypt_string --ask-vault-pass --name 'ansible_become_pass' 'password'

# Result

ansible_become_pass: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          37633839303634646539313233326163303337616239373233613564616231373835333038366637
          6561323737336532373266623462653663363335303839350a623037666138353332656264376565
          31633063313835636539613161363063306166346161343131346630643830666636326234313735
          6664633331613138330a313538383861373938393036363232623530346139356438323138653639
          3966
```

Now it is possible to use its value anywhere like group_vars folder

```yaml
# ./group_vars/ubuntu
---
ansible_become: true
ansible_become_pass: !vault |
          $ANSIBLE_VAULT;1.1;AES256
          37633839303634646539313233326163303337616239373233613564616231373835333038366637
          6561323737336532373266623462653663363335303839350a623037666138353332656264376565
          31633063313835636539613161363063306166346161343131346630643830666636326234313735
          6664633331613138330a313538383861373938393036363232623530346139356438323138653639
          3966
...
```

However on every execution it will be necessary to inform the vault password

```bash
ansible --ask-vault-pass ubuntu -m ping -o
```

### `Encrypting Files`

Also it is possible to encrypt file using ansible vault

```bash
# ansible-vault encrypt <file-path>
ansible-vault encrypt external_vault_vars.yaml
```

For use the encrypted file, you do not need to perform any change in the playbook, just reference the encrypted file and it will be automatically detected

```yaml
# playbook
---
-
  hosts: linux
  vars_files:
     - external_vault_vars.yaml
  tasks:

    - name: Show external_vault_var
      debug:
        var: external_vault_var
...
```

```bash
ansible-playbook --ask-vault-pass playbook.yaml
```

### `Decrypting`

As it is possible to encrypt a string or a file, it is also possible to decrypt it.

```bash
# ansible-vault decrypt <file-path>
ansible-vault decrypt external_vault_vars.yaml
```

### `Viewing Content`

Also it is possible to just view the content using the command `view`

```bash
# ansible-vault view <file-path>
ansible-vault view external_vault_vars.yaml
```

### `Password as File`

Also, it is possible pass the password to ansible using a file
```bash
#ansible-vault view --vault-password-file <password-file-path> <encrypted-file-path>
echo "vaultpass" > password_file
ansible-vault view --vault-password-file ./password_file external_vault_vars.yaml
```

### `Using Multiple Vaults`

A very common approach at Ansible is use multiple vault, one to a specific variable, another to other. This is possible do archive using the flat `vault-id`

```bash
# ansible-vault encrypt_string --vaultd-id <vault-name>@[prompt|filename] --ask-vault-pass --name <secret-name> <secret-value>

ansible-vault encrypt_string --vaultd-id ssh@prompt --ask-vault-pass --name 'ansible_become_pass' 'password'

# Result
ansible_become_pass: !vault |
          $ANSIBLE_VAULT;1.1;AES256;ssh
          37633839303634646539313233326163303337616239373233613564616231373835333038366637
          6561323737336532373266623462653663363335303839350a623037666138353332656264376565
          31633063313835636539613161363063306166346161343131346630643830666636326234313735
          6664633331613138330a313538383861373938393036363232623530346139356438323138653639
          3966

ansible-vault encrypt --vault-id vars@password_file external_vault_vars.yaml
```

> Notice that on the first line of the result secret there is the name of the vault (after AES256;)

Now to execute the playbook will be necessary to specify the both vaults

```bash
ansible-playbook --vault-id vars@password_file --vault-id ssh@prompt playbook.yaml
```