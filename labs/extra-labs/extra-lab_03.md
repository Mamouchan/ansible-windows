### Delegation

We will change the handlers created in lab05 and adapt it to run it locally instead of using the distant host.

1) Add `delegate_to: localhost` and custom vars under the handler as shown here

```yml
- name: "Send Teams notification"
  ansible.builtin.uri:
    url: "{{ teams_webhook_url }}"
    method: POST
    body_format: json
    body:
      type: AdaptiveCard
      text: "{{ ansible_date_time.date }}: index.html has been updated on {{ ansible_host }}"
  delegate_to: localhost
  vars:
    ansible_connection: local
    ansible_shell_type: /bin/sh
```

This changes the module and some parameters to adapt to a Linux environment

2) run again the playbook and see the result

3) In lab04 adapt the `win_get_url` task to also delegate to localhost (adapt destination to your current directory)

```yml
- name: 'Download the image and store it to the controller role path'
  ansible.builtin.get_url:
    url: 'https://upload.wikimedia.org/wikipedia/commons/a/a8/NASA-Apollo8-Dec24-Earthrise.jpg'
    dest: "/home/mde/ansible-lab/roles/lab04/files/earthrise.jpg"
  delegate_to: localhost
  vars:
    ansible_connection: local

- name: copy 
  ansible.windows.win_copy:
    src: "./files/earthrise.jpg"
    dest: 'C:\Temp\MyApp\earthrise.jpg'
```