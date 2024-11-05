# Testing with molecule

## Install Azure CLI

> Open a WSL terminal

Install Azure CLI:

```bash
curl -sL https://aka.ms/InstallAzureCLIDeb | sudo bash
```

Login to Azure:

```bash
az login
```

Change the active subscription to target the lab subscription:

```bash
# Change the active subscription using the subscription ID
az account set --subscription "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

## Install Molecule and Azure driver

```bash
pip install molecule 'molecule-plugins[azure]'
```

Check available drivers for molecule

```bash
molecule drivers
```

## Install/Upgrade `azure.azcollection`

Install/upgrade to the latest version of Azure collection:

```bash
ansible-galaxy collection install azure.azcollection --force
```

Install dependencies required by the collection:

```bash
pip install -r ~/.ansible/collections/ansible_collections/azure/azcollection/requirements.txt
```

## Create new role using Molecule

```bash
cd roles

molecule init scenario mycompany.lab06 --driver-name azure
```

## Update Molecule configuraiton

On the directory of your new role, go to the folder `molecule > default`

1) Replace the content of the file `molecule.yml`:

```yml
---
dependency:
  name: galaxy
driver:
  name: azure
platforms:
  - name: lab6win2022
provisioner:
  name: ansible
  connection_options:
    ansible_user: "{{ lookup('env', 'ADMIN_USERNAME') }}"
    ansible_password: "{{ lookup('env', 'ADMIN_PASSWORD') }}"
    ansible_connection: winrm
    ansible_winrm_transport: ntlm
    ansible_winrm_scheme: https
    ansible_winrm_port: 22
    ansible_winrm_server_cert_validation: ignore
    ansible_shell_type: powershell
  env:
    ANSIBLE_VERBOSITY: 1
    RESOURCE_GROUP_NAME: "changeme"
    VIRTUAL_MACHINE_NAME: lab6win2022
    VIRTUAL_NETWORK_RESOURCE_GROUP_NAME: "changeme"
    VIRTUAL_NETWORK_NAME: "changeme"
    SUBNET_NAME: "changeme"
    ADMIN_USERNAME: changeme
    ADMIN_PASSWORD: changeme
verifier:
  name: ansible

```

2) Replace the content of the file `create.yml`:

```yml
---
- name: Create
  hosts: localhost
  connection: local
  gather_facts: false
  no_log: "{{ molecule_no_log }}"
  vars:
    resource_group_name: "{{ lookup('env', 'RESOURCE_GROUP_NAME') }}"
    location: "westeurope"
    admin_username: "{{ lookup('env', 'ADMIN_USERNAME') }}"
    admin_password: "{{ lookup('env', 'ADMIN_PASSWORD') }}"
    virtual_network_resource_group_name: "{{ lookup('env', 'VIRTUAL_NETWORK_RESOURCE_GROUP_NAME') }}"
    virtual_network_name: "{{ lookup('env', 'VIRTUAL_NETWORK_NAME') }}"
    subnet_name: "{{ lookup('env', 'SUBNET_NAME') }}"
    virtual_machine_name: "{{ lookup('env', 'VIRTUAL_MACHINE_NAME') }}"
    network_interface_name: "{{ virtual_machine_name }}01"
    winrm_port: 22
  tasks:
    - name: Get facts for current logged in user
      azure.azcollection.azure_rm_account_info:

    - name: Create molecule instance(s)
      azure.azcollection.azure_rm_virtualmachine:
        resource_group: "{{ resource_group_name }}"
        location: "{{ location }}"
        virtual_network_name: "{{ virtual_network_name }}"
        virtual_network_resource_group: "{{ virtual_network_resource_group_name }}"
        subnet_name: "{{ subnet_name }}"
        name: "{{ virtual_machine_name }}"
        vm_size: Standard_B2s
        os_type: Windows
        admin_username: "{{ admin_username }}"
        admin_password: "{{ admin_password }}"
        public_ip_allocation_method: Disabled
        managed_disk_type: StandardSSD_LRS
        created_nsg: false
        image:
          offer: WindowsServer
          publisher: MicrosoftWindowsServer
          sku: '2022-Datacenter'
          version: latest
      register: server
      with_items: "{{ molecule_yml.platforms }}"
      async: 7200
      poll: 0

    - name: Wait for instance(s) creation to complete
      ansible.builtin.async_status:
        jid: "{{ item.ansible_job_id }}"
      register: azure_jobs
      until: azure_jobs.finished
      retries: 300
      with_items: "{{ server.results }}"

    - name: Create Azure vm extension to enable HTTPS WinRM listener
      azure.azcollection.azure_rm_virtualmachineextension:
        name: winrm-extension
        resource_group: "{{ resource_group_name }}"
        virtual_machine_name: "{{ virtual_machine_name }}"
        publisher: Microsoft.Compute
        virtual_machine_extension_type: CustomScriptExtension
        type_handler_version: '1.9'
        settings: '{"fileUris": ["https://raw.githubusercontent.com/mamouchan/ansible-windows/main/labs/tools/winrm/ConfigureWinRM.ps1", "https://raw.githubusercontent.com/mamouchan/ansible-windows/main/labs/tools/winrm/makecert.exe"],"commandToExecute": "powershell -ExecutionPolicy Unrestricted -File ConfigureWinRM.ps1"}'
        auto_upgrade_minor_version: true

    - name: Get facts for one private ip
      azure.azcollection.azure_rm_networkinterface_info:
        resource_group: "{{ resource_group_name }}"
        name: "{{ network_interface_name }}"
      register: privateipaddress

    - name: Set private ip address fact
      ansible.builtin.set_fact:
        privateipaddress: "{{ privateipaddress | json_query('networkinterfaces[0].ip_configurations[0].private_ip_address') }}"

    - name: Wait for the WinRM port to come online
      ansible.builtin.wait_for:
        port: '{{ winrm_port }}'
        host: '{{ privateipaddress }}'
        timeout: 600

    - name: Populate instance config dict
      ansible.builtin.set_fact:
        instance_conf_dict: {
          'instance': "{{ item.ansible_facts.azure_vm.name }}",
          'address': "{{ privateipaddress }}",
          'user': "{{ admin_username }}",
          'port': "{{ winrm_port }}",
          'identity_file': ""
        }
      with_items: "{{ azure_jobs.results }}"
      register: instance_config_dict
      when: server.changed | bool

    - name: Convert instance config dict to a list
      ansible.builtin.set_fact:
        instance_conf: "{{ instance_config_dict.results | map(attribute='ansible_facts.instance_conf_dict') | list }}"
      when: server.changed | bool

    - name: Dump instance config
      ansible.builtin.copy:
        content: "{{ instance_conf | to_json | from_json }}"
        dest: "{{ molecule_instance_config }}"
      when: server.changed | bool

```

This file is used by molecule to create the test instance.

3) Replace the content of the file `destroy.yml`:

```yml
---
- name: Destroy
  hosts: localhost
  connection: local
  gather_facts: false
  no_log: "{{ molecule_no_log }}"

  vars:
    resource_group_name: "{{ lookup('env', 'RESOURCE_GROUP_NAME') }}"
    virtual_machine_name: "{{ lookup('env', 'VIRTUAL_MACHINE_NAME') }}"

  tasks:
    - name: Destroy molecule instance(s)
      azure.azcollection.azure_rm_virtualmachine:
        resource_group: "{{ resource_group_name }}"
        name: "{{ virtual_machine_name }}"
        state: absent
        remove_on_absent:
          - all
      register: server
      with_items: "{{ molecule_yml.platforms }}"
      async: 7200
      poll: 0

    - name: Wait for instance(s) deletion to complete
      ansible.builtin.async_status:
        jid: "{{ item.ansible_job_id }}"
      register: azure_jobs
      until: azure_jobs.finished
      retries: 300
      with_items: "{{ server.results }}"
      failed_when: false

    - name: Populate instance config
      ansible.builtin.set_fact:
        instance_conf: {}

    - name: Dump instance config
      ansible.builtin.copy:
        content: "{{ instance_conf | to_json | from_json }}"
        dest: "{{ molecule_instance_config }}"
      when: server.changed | bool

```

This file is used by molecule to destroy the test instance.

4) Add a new file `prepare.yml` with content:

```yml
---
- name: Prepare
  hosts: all
  gather_facts: false
  tasks:
    - name: Ignore prepare senario for windows host
      ansible.builtin.debug:
        msg:
        - "No need to run the prepare senario for windows host"
```

The purpose of this file is to override the default senario of Molecule, which is specific for linux hosts.

## Play with molecule commands

> Molecule commands must be ran under the folder of your ansible role.

1) List instances

```bash
molecule list
```

2) Create your testing instance:

```bash
molecule create
```

⚠️ The first time you run molecule create, the command may fail due to a winrm error. Just re-run this command.

Run `molecule list` to see the state of your instance:

```bash
molecule list
```

2) Run your role against the test instance

```bash
molecule converge
```

Remarks:
- The tasks of your role are empty `molecule converge` will do nothing
- The converge senario will run the create and prepare senarios if they haven't been executed

Run `molecule list` to see the state of your instance:

```bash
molecule list
```

3) Run your tests

Tests are defined on the file `molecule/default/verify.yml`

```bash
molecule verify
```

## Create ansible tests

On the file `molecule/default/verify.yml`

1) Add a new test to assert that the target os distribution is Microsoft Windows Server 2022 Datacenter:

> Remark: We need to set gather_facts to true to get the facts of the target host and be able to use them in the test.

```yml
---
- name: Verify
  hosts: all
  gather_facts: true
  tasks:
    - name: "Assert that target os distribution is Microsoft Windows Server 2022 Datacenter"
      ansible.builtin.assert:
        that: ansible_distribution == "Microsoft Windows Server 2022 Datacenter"

```

2) Add a test to assert that the WinRM service is running:

```yml
# Get winrm service info
- name: "Get winrm service info"
  ansible.windows.win_service_info:
    name: WinRM
  register: winrm_service_info

# Assert that WinRM service is running
- name: "Assert that WinRM service is running"
  ansible.builtin.assert:
    that: winrm_service_info.services[0].state == "started"
```

3) Add a task to create a file with content and tests to assert is the file exists and its content matches:

On the file `tasks/main.yml`, add the task:

```yml
- name: "Create Hello World file"
  ansible.windows.win_copy:
    content: "Hello World"
    dest: C:\Temp\hello_world.txt
```

On the file `molecule/default/verify.yml`, add the tests:

```yml
# 1) Test that the file exists
- name: "Get stat on the file C:\\Temp\\hello_world.txt"
  ansible.windows.win_stat:
    path: C:\Temp\hello_world.txt
  register: hello_world_file_info

- name: "Assert that hello_world.txt exists"
  ansible.builtin.assert:
    that: hello_world_file_info.stat.exists == True

# 2) Test that the content of the file is Hello World
- name: "Get the content of C:\\Temp\\hello_world.txt"
  ansible.builtin.slurp:
    src: C:\Temp\hello_world.txt
  register: hello_world_file_content

- name: "Decode the content of C:\\Temp\\hello_world.txt"
  ansible.builtin.set_fact:
    hello_world_file_content: "{{ hello_world_file_content['content'] | b64decode }}"

- name: "Assert that the content of C:\\Temp\\hello_world.txt is Hello World"
  ansible.builtin.assert:
    that: hello_world_file_content == "Hello World"
```

## Exercice

Deploy a new IIS website

- The website should listen on port 8080
- The website should return `Hello World`

> Hint: You need to Install IIS Web-Server with sub features and management tools (See lab 05)

Exepcted result: The following test should pass:

```yml
- name: "Get IIS Website response"
  ansible.windows.win_uri:
    url: "http://localhost:8080"
    method: GET
    return_content: true
  register: iis_website_response

- name: "Assert IIS Website is up"
  ansible.builtin.assert:
    that:
      - iis_website_response.status_code == 200
      - iis_website_response.content == "Hello World"
```
