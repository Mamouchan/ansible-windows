# Deploy a static web app on IIS

## Install IIS

1) Create a new role named `iis`

2) Add a task to install IIS using the module `win_feature`

```yml
---
- name: Install IIS Web-Server with sub features and management tools
  ansible.windows.win_feature:
    name: Web-Server
    state: present
    include_sub_features: true
    include_management_tools: true
  register: win_feature

- name: Reboot if installing Web-Server feature requires it
  ansible.windows.win_reboot:
  when: win_feature.reboot_required
```

3) Include this new role your playbook (you can comment/remove previous roles defined in this playbook)

```bash
ansible-playbook --vault-password-file=.vault_pass deploy.yml -i inventories/dev -vvv
```

When IIS is installed, open the ip of your VM: You will see the defaut page of IIS.

## Change index.html of the default IIS application

1) Create a new role named `lab05`

2) Add an html file on the folder files of the role `lab05`, named `default_index.html` with content:

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Ansible Lab 5</title>
    </head>
    <body>
        <p>Hello World</p>
    </body>
</html>
```

3) Add a task to copy the `default_index.html` to the remote directory `C:\\inetpub\\wwwroot\\index.html`

```yml
- name: "Change default website index.html"
  ansible.windows.win_copy:
    src: default_index.html
    dest: "C:\\inetpub\\wwwroot\\index.html"
```

4) Include this new role your playbook (you can comment/remove previous roles defined in this playbook)

5) Run your playbook

```bash
ansible-playbook --vault-password-file=.vault_pass deploy.yml -i inventories/dev -vvv
```

6) Open the url http://<VM_IP> on your browser: You will see the defaut page of IIS has been updated.

## Deploy an IIS application

1) Install the collection `community.windows` if it doesn't exist:

First run:

```bash
ansible-galaxy collection list | grep -i community.windows
```

If the collection `community.windows` is not installed, run:

```bash
ansible-galaxy collection install community.windows
```

2) Add task to create the folder `c:\\inetpub\\wwwroot\\lab05`

3) Add an html file on the folder files of the role `lab05`, named `lab05_index.html` with content:

```html
<!DOCTYPE html>
<html>
    <head>
        <title>Ansible Lab 5</title>
    </head>
    <body>
        <p>Hello World. This my new page</p>
    </body>
</html>
```

4) Add a task to copy the `lab05_index.html` to the remote directory `C:\\inetpub\\wwwroot\\lab05\\index.html`

5) Open the documentation of the collection [`community.windows`](https://docs.ansible.com/ansible/latest/collections/community/windows)

6) Create a new IIS pool named `lab05iispool` using the module `win_iis_webapppool`

7) Use the module `win_iis_website` to create a IIS website:

- Named `lab05iissite`
- Linked to the pool `lab05iispool`
- Listing to the port 8080
- physical_path must be `c:\\inetpub\\wwwroot\\lab05`

8) Add a new task that calls the module `community.windows.win_firewall_rule` to allow the port 8080

Example:

```yml
- name: Firewall rule to allow HTTP on TCP port 8080
  community.windows.win_firewall_rule:
    name: HTTP_8080
    localport: 8080
    action: allow
    direction: in
    protocol: tcp
    state: present
    enabled: true
```

9) Run your playbook

10) Open the url http://<VM_IP>:8080 on your browser: You will see your new site.

## Deploy dynamic html file using jinja

Same excercie as previously, but this time we will use a jinja template. Here we want to inject the value of a variable in the output index.html

1) Delete the file `lab05_index.html` from files folder.

2) Add a new variable named `welcome_message` with value `World` (in defaults folder)

3) Add a file named `lab05_index.html.j2` under the folder templates and with content:

```jinja
<!DOCTYPE html>
<html>
    <head>
        <title>Ansible Lab 5</title>
    </head>
    <body>
        <p>Hello {{ welcome_message }}. This my new page</p>
    </body>
</html>
```

4) Update the code of the task that copy the file `lab05_index.html` to use the module `win_template` to copy the template `lab05_index.html.j2` to the remote path `C:\\inetpub\\wwwroot\\lab05\\index.html`

5) Run your playbook

6) Open the url http://<VM_IP>:8080 on your browser: You will see the content you have added.

7) Let's update our html templates to handle condition on the value of the variable `welcome_message`

```jinja
<!DOCTYPE html>
<html>
    <head>
        <title>Ansible Lab 5</title>
    </head>
    <body>
        {% if welcome_message == 'World' %}
        <p>Hello {{ welcome_message }}. This my new page</p>
        {% else %}
        <h1>Goodbye {{ welcome_message }}</h1>
        {% endif %}
    </body>
</html>
```

8) Run your playbook by overriding the value of the variable `welcome_message` using the parameter `extra-vars`:

```bash
ansible-playbook --vault-password-file=.vault_pass deploy.yml -i inventories/dev --extra-vars "welcome_message=Windows" -vvv
```

9) Open the url http://<VM_IP>:8080 on your browser: You will see the content you have added.


## Ansible filters and handlers

In this section, we will deploy an html file containning data from target node and we will a handler to trigger a Teams webhook when the html file is updated.

### Ansible filters

1) Replace the content of the file `lab05_index.html.j2` with:

```jinja
<!doctype html>
<html lang="en">
    <head>
        <meta charset="utf-8">
        <meta name="viewport" content="width=device-width, initial-scale=1">
        <title>Ansible Lab 5</title>
        <link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css" rel="stylesheet" integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN" crossorigin="anonymous">
    </head>
    <body>
        <table class="table">
            <thead>
              <tr>
                <th scope="col">Name</th>
                <th scope="col">State</th>
                <th scope="col">Start Mode</th>
                <th scope="col">Path</th>
              </tr>
            </thead>
            <tbody>
              {%- for service in my_local_services %}
              <tr>
                <td>{{ service.name }}</td>
                <td>{{ service.state }}</td>
                <td>{{ service.start_mode }}</td>
                <td>{{ service.path }}</td>
              </tr>
              {% endfor %}
            </tbody>
          </table>

        <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js" integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL" crossorigin="anonymous"></script>
    </body>
</html>
```

2) Before the code of the task that copy the template `lab05_index.html.j2`, we need to add tasks to:

- Get the list of windows services using the module `win_service_info`
- Filter the list of services to keep only the services that are running

Example:

```yml
- name: Get info for all installed services
  ansible.windows.win_service_info:
  register: service_info


- name: Set fact on services
  ansible.builtin.set_fact:
    my_local_services: "{{ service_info.services | list }}"
```

3) Run your playbook and open the url http://<VM_IP>:8080 on your browser: You will see the list of services on your VM.

4) Update the code of the task that sets fact on the variable `my_local_services` to keep only the services that are running, using the filter `selectattr`:

```yml
- name: Set fact on running services
  ansible.builtin.set_fact:
    my_local_services: "{{ service_info.services | selectattr('state', 'equalto', 'started') | list }}"
```

5) Run your playbook and open the url http://<VM_IP>:8080 on your browser: You will see the list of services on your VM.

### Handlers

1) Generate a Teams webhook url using the [Teams connector](https://docs.microsoft.com/en-us/microsoftteams/platform/webhooks-and-connectors/how-to/add-incoming-webhook)

2) On defaults/main.yml, add a variable named `teams_webhook_url` with the value of the webhook url generated in the previous step.

3) On the folder handlers in the role `lab05`, add a handler send an http event to a teams webhook to notify an update on the html file `lab05_index.html.j2`:

```yml
- name: "Send Teams notification"
  ansible.windows.win_uri:
    url: "{{ teams_webhook_url }}"
    method: POST
    body_format: json
    body:
      type: AdaptiveCard
      text: "{{ ansible_date_time.date }}: index.html has been updated on {{ ansible_host }}"
```

4) On the task that copy the template `lab05_index.html.j2`, add the `notify` instruction to trigger the handler `Send Teams notification` when a change is detected:

```yml
- name: Copy index lab05
  ansible.windows.win_template:
    src: lab05_index.html.j2
    dest: c:\\inetpub\\wwwroot\\lab05\\index.html
  notify: "Send Teams notification"
```

:warning: To detect a change the taks you are applying the handler on must actually perform a change (file modification, etc )

5) On the task `Set fact on running services`, update the filter to sort the list of windows services randomly, using the filter `shuffle`:

```yml
- name: Set fact on running services
  ansible.builtin.set_fact:
    my_local_services: "{{ service_info.services | shuffle | selectattr('state', 'equalto', 'started') | list }}"
```

5) Run your playbook and open the url http://<VM_IP>:8080 on your browser: You will see the list of services on your VM.

6) Visit your Teams channel: You will see a notification that the index.html file has been updated.
