### Complex loops and filters

1) Create a variable called oscarng_version like below :

```yml
oscarng_versions:
  - version: 1.0.0.1
    metadata:
      version_major: 1
      version_minor: 0
      version_build: 0
      version_release: 1
      date: "01/01/2024"
      time: "12:00:00"
      author: 
        name: "DENNERY Maël"
        repo: "https://github.com/Mamouchan"
      setups:
        - name: OscarNG
          group: OscarNG 
          version: 7.1.6.24 
        - name: OscarNGReports 
          group: OscarNG 
          version: 7.0.1.0 
        - name: OscarNGReportUtil 
          group: OscarNG 
          version: 1.0.0.37 
      custom_tasks:
        before:
          - uninstall_telerik
          - cms_check
        after:
          - cms_check
  - version: 7.5.3.14
    metadata:
      version_major: 7
      version_minor: 5
      version_build: 3
      version_release: 14
      date: "18/05/2022"
      time: "13:49:00"
      author: 
        name: "DENNERY Maël"
        repo: "https://github.com/Mamouchan"
      setups:
        - name: OscarNG 
          group: OscarNG 
          version: 7.5.3.14 
        - name: OscarNGReports 
          group: OscarNG 
          version: 7.0.1.0 
        - name: OscarNGReportUtil 
          group: OscarNG 
          version: 1.0.0.37 
      custom_tasks:
        before:
          - uninstall_telerik
          - uninstall_library
          - cms_check
        after:
          - cms_check
  - version: 7.7.0.0
    metadata:
      version_major: 7
      version_minor: 7
      version_build: 0
      version_release: 0
      date: "18/05/2022"
      time: "13:49:00"
      author: 
        name: "DUGNE Tristan"
        repo: "none"
      setups:
        - name: OscarNG 
          group: OscarNG 
          version: 7.7.0.0
        - name: OscarNGReports 
          group: OscarNG 
          version: 7.1.1.1 
        - name: OscarNGReportUtil 
          group: OscarNG 
          version: 1.0.0.37 
      custom_tasks:
        before:
          - uninstall_telerik
          - configuration_item
          - cms_check
        after:
          - cms_check
```

For the next steps you can use this page as reference : 
- https://jinja.palletsprojects.com/en/3.0.x/templates/#list-of-builtin-filters
- https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_loops.html#complex-loops

This oscarng_versions variable is a complex list (basically a list containing multipe similar objects).
To display the variable inside the ansible controller you can use the builtin debug module as below :

```yml
- name: Debug the content of oscarng_versions variable
  ansible.builtin.debug:
    msg: "apply a filter to oscarng_versions {{ oscarng_versions }}"
```

To apply jinja filters inside the `{{ oscarng_versions }}` variable you will use the `|` character.

2) Find the number of entry in the oscarng_versions using the appropriate jinja filter ( length )
   
3) Sort the list by version_minor descending using the appropriate jinja filter ( sort with an attribute )
   
4) Find the item in the list with version 7.5.3.14 using the appropriate jinja filter ( select or selectattr )

5) Get only the custom_task for oscarng version 7.7.0.0 using the appropriate jinja filter ( select or selectattr then map )

### Custom module implementation

1) Create a library/module directory at your project root
   
2) Copy/Paste the following powershell script :

```ps1
#!powershell

# Copyright: (c) 2022, Maël Dennery <mael.dennery_ext@michelin.com>

#Requires -Module Ansible.ModuleUtils.Legacy

$params = Parse-Args $args -supports_check_mode $false
$diff_mode = Get-AnsibleParam -obj $params -name "_ansible_diff" -type "bool" -default $false

# Module control parameters
$path = Get-AnsibleParam -obj $params -name "path" -type "str" -failifempty ($null -ne $path)
$content = Get-AnsibleParam -obj $params -name "content" -type "str" -failifempty ($null -ne $content)
$state = Get-AnsibleParam -obj $params -name "state" -type "str" -default "present" -validateset 'present','absent'

# Return object 
$result = @{
    created = $false
    changed = $false
    deleted = $false
}

# Actual module actions
if("present" -eq $state)
{
    if(Test-Path -Path $path)
    {
        $currentContent = Get-Content -Path $path
        if($currentContent -ne $content)
        {
            Set-Content -Path $path -Value $content
            $result.changed = $true
        }
    }
    else
    {
        New-Item -Path $path -ItemType "file" -Value $content
        $result.created = $true
    }
}

if("absent" -eq $state)
{
    if(Test-Path -Path $path)
    {
        Remove-Item -Path $path
        $result.deleted = $true
    }
}

Exit-Json -obj $result
```

This script is custom made and is for creating, removing and modifying a file.

3) Update your ansible.cfg file and add `library = ./library/modules` under the `[default]` group if not already done

4) Call the script in a playbook or role as you would with any other built-in modules

```yml
---
- name: Custom Module Test
  hosts: "azure_vm"
  gather_facts: no
  tasks:
    - name: Call custom module
      win_custom_module:
        path: 'C:\Temp\MyApp\test.txt'
        content: 'This is a test'
        state: present
```

5) If you are confortable in coding with powershell, you may adapt the script and change it to see results on the distant host.
