---
marp: true
title: Ansible for Windows
theme: uncover
_class: lead
paginate: true
_paginate: false
backgroundImage: url('https://marp.app/assets/hero-background.svg')
---

# <!--fit--> Ansible for Windows

Created by Hocine Hacherouf ([@hocinehacherouf](https://github.com/hocinehacherouf))
Presented by Maël Dennery ([@maeldennery](https://github.com/Mamouchan))
And Nicolas Mahé.

---

## Summary

- Ansible Concepts
- Development with Ansible
- Best Practices
- Testing with Molecule 
- Automation with AWX and CI

---

## Ansible Concepts

![bg right 40%](https://upload.wikimedia.org/wikipedia/commons/2/24/Ansible_logo.svg)

---

### What is Ansible ?

- Open source
- Based on Python
- Created in 2012 and acquired by Red Hat in 2015
- Tool for machine provisioning, configuration and deployment
- Agentless
- Idempotence, stateless

---

![bg 80%](https://static.wixstatic.com/media/559395_ca2e6a060f6d438d824cb974f8d43a27~mv2.png)

---

### Ansible Communication Protocoles

- Ansible + Linux (or Mac) 👉 SSH
- Ansible + Windows 👉
  - WinRM: **Win**dows **R**emote **M**anagement
  - SSH (:warning: Experimental)

---

### WinRM

- **Win**dows **R**emote **M**anagement: Used by Ansible to to communicate with Windows machines.
- WinRM default ports
  - HTTP : 5985
  - HTTPS : 5986 (SSL certificats required)
- SSH default port is 22
- 🛈 The python module **`pywinrm`** is required by ansible for Windows support.

---

#### WinRM and Authentification

- Basic
- Certificate
- NTLM
- Kerberos
- CredSSP

---

#### WinRM and Authentification: Basic

- The simplest authentication options to use,
- Can only be used for local accounts (not domain accounts).

---

#### WinRM and Authentification: Certificate

- Authentication uses certificates as key, similar to SSH key pairs
- Not enabled by default on a Windows host

---

#### WinRM and Authentification: NTLM

- Enabled by default on the WinRM service
- Support local and domain users
- More secure than Basic

🛈 We will use **NTLM** on our labs

---

#### WinRM and Authentification: Kerberos

- Recommended when running on a domain environment
- Support message encryption over HTTP and credential delegation
- The most secure available on WinRM service
- Ansible controller require configuration
- The wrapper **pywinrm[kerberos]** is required

---

#### WinRM and Authentification: CredSSP

- New authentication protocol that allows credential delegation
- Support message encryption over HTTP
- The wrapper **pywinrm[credssp]** is required

---

### Ansible :heart: PowerShell

- Ansible can execute PowerShell commands/files
  - Ansible required powershell 3.0+ and .NET 4.0+ on target hosts
- Ansible modules for Windows are based on PowerShell 👉 `ansible.windows`

---

![bg right 60%](https://upload.wikimedia.org/wikipedia/commons/thumb/8/87/Jinja_software_logo.svg/langfr-1920px-Jinja_software_logo.svg.png)

### Templating with Jinja2

---

## Ansible Installation

🧪 Lab 01

---

## Configure testing environement

🧪 Lab 02

---

## Ansible Project

---

### Ansible Configuration

- Ansible comes with a default configuration `/etc/ansible/ansible.cfg`.
- This configuration can be overriden with the following precedence:
  - The env variable: `ANSIBLE_CONFIG`
  - The file `ansible.cfg` on your current directory where ansible is executed
  - User home directory: `~/.ansible/ansible.cfg`
  - Default `/etc/ansible/ansible.cfg` file.

---

### Inventories

- File that describes your infrastructure: Contant machines and their variables
- Machines can be grouped by type: db, frontend, backend...
- Multiple format to define an inventory:
  - ini
  - yaml
  - json

---

### Inventory structure

```plain
├── inventories
│   ├── dev
│   │   ├── group_vars
│   │   │   └── all.yml
│   │   │   └── backend.yml
│   │   ├── host_vars
│   │   │   └── db01.yml
│   │   │   └── backend01.yml
│   │   │   └── backend02.yml
│   │   │   └── frontend01.yml
│   │   └── hosts
```

---

### Inventory hosts file

```ini
[db]           # group name
db01           # machine name

[backend]
backend0[1:2]  # use range to simplify backend01 and backend02

[frontend]
frontend01
```

---

### Variables

- We can declare variables in an Ansible project
- No need to specify the variable type (int, string, etc ...)
- You can call a variable inside another one (put it between {{ }})

```yml
my_variable_name: "my_variable_value"
my_second_variable_name: "my_second_variable_value {{ my_variable_name }}"
# ansible will interpret anything inside {{ }}
```

---

### Secret Management with Ansible Vault

- To deal with sensitive data such as passwords, tokens, certificats..., we have to use the cli `Ansible-vault` to encrypt them.
- The encrypted data can be distributed or placed in `source control` (gitlab, tfs, ...).

---

## Let's create an Ansible inventory with secrets

🧪 Lab 03

---

### Playbook

- An ansible playbook is a yaml file that triggers the actions to be performed by Ansible on an inventory 👉 Orchestrate

```yml
---
- name: My awesome playbook
  hosts: all
  gather_facts: yes
  roles:
    - create-vm
    - install-database
```

```bash
ansible-playbook deploy.yml -i inventories/dev
```

---

### Roles

Roles let you automatically load related vars, files, tasks, handlers, and other Ansible artifacts based on a known file structure. After you group your content in roles, you can easily reuse them and share them with other users.

---

#### Role's structure

- meta: Metadata of the role
- defaults: Default variables of the role
- vars: Other variables used by the roles and can be overridden by a user
- tasks: Set of tasks used by the role
- handlers: Handlers triggered used by the tasks of the role
- files: Static files used by the role
- templates: Templates based on jinja and used by the role

---

#### Create a role

- You can create the skeleton of an ansible role using the cli `ansible-galaxy`:

```bash
ansible-galaxy init roles/create-vm
```

---

```plain
├── roles
│   └── create-vm
│       ├── README.md
│       ├── defaults
│       │   └── main.yml
│       ├── files
│       ├── handlers
│       │   └── main.yml
│       ├── meta
│       │   └── main.yml
│       ├── tasks
│       │   └── main.yml
│       ├── templates
│       └── vars
│           └── main.yml
```

---

### Ansible variable precedence

See 👉 [Understanding variable precedence](https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_variables.html#variable-precedence-where-should-i-put-a-variable)

---

![bg right 60%](https://upload.wikimedia.org/wikipedia/commons/thumb/8/87/Jinja_software_logo.svg/langfr-1920px-Jinja_software_logo.svg.png)

### Jinja2

Live DEMO

---

## Let's play with Ansible

🧪 Lab 04

---

![bg right 90%](https://galaxy.ansible.com/assets/galaxy-logo-02.svg)

## Ansible Collections

---

- With Ansible Collections, you can use ansible playbooks/roles created and maintained by the community.
- Example: Microsoft provides an ansible collection [`azure.azcollection`](https://github.com/ansible-collections/azure) that allows to build and configure Azure ressources using Ansible

---

## [Ansible Galaxy](https://galaxy.ansible.com/)

Ansible Galaxy or Galaxy is a hub for finding and sharing Ansible content

![bg right 90%](https://galaxy.ansible.com/assets/galaxy-logo-02.svg)

---

### How to install ansible collections ?

Install a role

```shell
ansible-galaxy install geerlingguy.java
```

Install a collection

```shell
ansible-galaxy collection install azure.azcollection
```

Or install multiple collections/roles with a requirements file

```shell
ansible-galaxy install -r requirements.yml
```

---

```yml
---
roles:
  # Install a role from Ansible Galaxy.
  - name: geerlingguy.java
    version: "1.9.6" # note that ranges are not supported for roles

collections:
  # Install a role from Ansible Galaxy.
  - name: install azure.azcollection
    version: ">=1.14.0" # usage of ranges
  # Install a role from Ansible Galaxy.
  - name: awx.awx
    version: 21.11.0
    source: https://galaxy.ansible.com
  # Install a role from a git branch/tag/commit
  - name: https://github.com/organization/repo_name.git
    type: git
    version: develop
```

---

## Let's play with Ansible Collections

🧪 Lab 05

---

## Best Practices

---

### Ansible Lint

- Ansible Lint is a command-line tool for linting playbooks, roles and collections
- Promotes best pratices
- Helps you to upgrade your code to work with newer versions of Ansible
- Automated lint on pull requests checks (ci pipelines)

:information_source: VSCode extension for Ansible (for windows users 👉 `WSL is required`)

---

### Use Ansible FQCN

- FQCN 👉 Fully Qualified Collection Name
- Recommended since Ansible 2.10

```yml
- name: Task not using FQCN
  win_command: whoami
```

```yml
- name: Task using FQCN
  ansible.windows.win_command: whoami
```

---

### Usage of tags

If you have a large playbook, it may be useful to run/skip only specific parts of it instead of running the entire playbook. You can do this with Ansible tags:

```yml
---
- name: Install all critical and security updates
  ansible.windows.win_updates:
    category_names:
    - SecurityUpdates
    state: installed
  tags: patching
```

```bash
Ansible-playbook deploy.yml --tags "patching"
Ansible-playbook deploy.yml --skip-tags "patching"
```

---

### Smoke tests

- After a deployment on a target node, it's a good practice to run checks on new/updated compnents to verify that they are running and healty.
- The goal is to get information as soon as possible.
- Theses tests are called `smoke tests`
- An example of smoke tests : After deploying an api with ansible, the playbook should
  - Check if the authentication endpoint works
  - Call the api to query data
  - ...

---

![bg right 60%](https://repository-images.githubusercontent.com/216770803/aa9e2d00-f4bd-11e9-9149-3765c7f2d2a2)

### Testing roles with Molecule

---

- [Molecule](https://molecule.readthedocs.io/) is a wrapper for ansible that allows to tests and lint ansible roles.
- Tests are done by Molecule on disposable environements using drivers (supported by ansible): docker, vagrant...
- The usage of molecule is not only for local testing, it can/must be used on continuous integration pipelines.

---

### Molecule Live Demo

![bg right 60%](https://repository-images.githubusercontent.com/216770803/aa9e2d00-f4bd-11e9-9149-3765c7f2d2a2)

---

## Let's play with Molecule

🧪 Lab 06

---

![bg right 90%](https://raw.githubusercontent.com/ansible/awx-logos/master/awx/ui/client/assets/logo-login.svg)

## Automation with Ansible Tower/AWX

---

### Ansible Tower/AWX

- Ansible Tower : UI web solution for ansible projects management and automation.
- AWX : An open source version of Ansible Tower

---

![bg right 90%](https://raw.githubusercontent.com/ansible/awx-logos/master/awx/ui/client/assets/logo-login.svg)

### AWX

Live Demo

---

## 👌 Thanks