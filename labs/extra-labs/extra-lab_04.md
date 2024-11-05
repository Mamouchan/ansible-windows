### Going further with Ansible

This lab will give general guidelines but not give you anything to copy/paste.
Try to search on your own, don't hesitate to ask to your trainer(s) for help.

Usefull inks for this extra-lab:
    - https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_async.html 

1) Prepare multiple task within a new role:
    - one to copy a file to `C:\Temp`
    - one to run a iisreset on powershell
    - one to create a schedule task which will run a simple `echo "Hello World"` written in bash
    
2) Find a way to run all of these task simultaneously and not one by one.

3) Using the `assert` module, try to validate the following conditions:
    - assert that the WinRM service is correctly started
    - assert that your website is responsive and add a custom success message
    - assert that you have more than 30 GO of available space in your main disk
    - change the value of the previous assert to make it fail and add a custom fail message

4) Try to create a custom registry key `version` under `Computer\HKEY_LOCAL_MACHINE\SOFTWARE\Michelin.ePC`.

5) Do the same as previously but using at least one different method.

6) Create two different task (win_copy to the distant host for example) and adapt the first task so it doesn't stop the playbook/roles even if it fails (it will ignore errors).

7) Create a new role containing only one standard debug task to check the content of the variable `user_input`.

8) Create a new playbook that ask for an input from the user, register it to `user_input` variable and run your new role.
