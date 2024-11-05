### Ansible & VMWare & AWX

This lab will give general guidelines but not give you anything to copy/paste.
Try to search on your own, don't hesitate to ask to your trainer for help.

1) Create a new inventory called vmware, no need to create an host.

2) Create all necessary variables in your group_vars/all.yml to link it to your own cluster.

3) Try to use a vmware module to get all available datastore for your ressource pool.

4) Try to use a vmware module to get all Machines with a snapshot older than 2 days.

5) Link it to AWX and try to schedule the snapshot playbook above to run every day at midnight.

6) Setup notifications in AWX so it notifies selected users by a mail when success or failure.