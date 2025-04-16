# Let's play ansible and ansible windows modules

## Create your first ansible role

1) At the root of your ansible project, create a new role named `my-first-role`, using the cli `ansible-galaxy`

```bash
ansible-galaxy init roles/my-first-role
```

2) At the root of your ansible project, create a playbook file named `deploy.yml` that references your ansible role:

```yml
---
- name: My awesome playbook
  hosts: all
  gather_facts: yes
  roles:
    - my-first-role
```

3) Your role is empty, but we can run your playbook:

```bash
ansible-playbook --vault-password-file=.vault_pass deploy.yml -i inventories/dev
```
Or add `-vvv` to get more details

```bash
ansible-playbook --vault-password-file=.vault_pass deploy.yml -i inventories/dev -vvv
```

## Create your first task

1) Create a task in the role `my-first-role` that runs a basic powershell command using the module `win_powershell`:

```yml
- name: Run basic PowerShell script
  ansible.windows.win_powershell:
    script: |
      echo "Hello World"
```

2) Run your playbook

3) Update your task by adding the instruction register to store the output of the powershell command in a new variable `pwsh_output`. And add another task to print the value of the variable `pwsh_output`:

```yml
- name: Run basic PowerShell script
  ansible.windows.win_powershell:
    script: |
      echo "Hello World"
  register: pwsh_output

- name: Print pwsh_output variable
  ansible.builtin.debug:
    var: pwsh_output
```

4) Run your playbook

5) Add a task to print the first element the list output inluded in the variable `pwsh_output`

```yml
- name: Print pwsh_output.output[0]
  ansible.builtin.debug:
    var: pwsh_output.output[0]
```

6) Run your playbook

7) Add a task to print the first element the list output inluded in the variable `pwsh_output` using a filter

```yml
- name: Print pwsh_output.output | first
  ansible.builtin.debug:
    var: pwsh_output.output | first
```

## Excercises

1) At the root of your ansible project, create a new role named `my-second-role`, using the cli `ansible-galaxy`

2) Include the role `my-second-role` in the playbook file `deploy.yml`. Comment the reference to the role `my-first-role`

3) Go to the documentation of the modules of [ansible.windows](https://docs.ansible.com/ansible/latest/collections/ansible/windows) and for each exercise below, propose a task that meets the need:

- Create a directory with path `C:\Temp\MyApp`
- Create a file with path `C:\Temp\MyApp\Service1.config`
- Download the image from `https://upload.wikimedia.org/wikipedia/commons/a/a8/NASA-Apollo8-Dec24-Earthrise.jpg` and store it to the path `C:\Temp\MyApp\earthrise.jpg`
- Find files in the path `C:\Temp\MyApp` and where extension is `.config`
- Delete file with path `C:\Temp\MyApp\Service1.config`
- Delete the directory with path `C:\Temp\MyApp`
- Create a group `MyGroup`
- Create a user `MyUser` within the group `MyGroup`
- Delete the user `MyUser`
- Delete the group `MyGroup`
- Install security update without reboot
- Perform a windows reboot separately from the security updates
