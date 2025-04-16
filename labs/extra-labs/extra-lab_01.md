### Loop and conditionnal in ansible

1) Create a variable called users like the following
```yml
users:
  - name: mytempuser01
    password: Mytemppassword01+
  - name: mytempuser02
    password: Mytemppassword02+
  - name: mytempuser03
    password: Mytemppassword03+
```
2) Create all the users in the list with only one tasks, refer to this documentation : 
   https://docs.ansible.com/ansible/latest/playbook_guide/playbooks_loops.html#iterating-over-a-simple-list
   
3) Delete all the users in the list with only one tasks, put it before the creation task
   
4) Adapt the list and add group to each one
```yml
users:
  - name: mytempuser01
    password: Mytemppassword01+
    groups:
      - users
      - administrators
  - name: mytempuser02
    password: Mytemppassword02+
    groups:
      - users
      - administrators
  - name: mytempuser03
    password: Mytemppassword03+
    groups:
      - users
      - administrators
```
   
5) Rename the iteration loop variable (by default it is named item) to a custom one (named user for example) and refactor the necessary tasks after it
   
6) Adapt the loop to call a new set of tasks (using include_tasks to call a new file with a loop on it), and do both user and group creation into this new file.

### Tags management

1) Add tags to all of your tasks and separate them : one one side a user_creation tag and on the other a user_deletion tag
   
2) Run the playbook but only select the user_deletion using --tags option in your command

3) Run the playbook but only select the user_creation using --tags option in your command

4) Run the playbook but only select the user_creation using --skip-tags option in your command