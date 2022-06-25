# Instructor Guide

This section seeks to help an instructor setup an Ansible Automation Platform environment to provide the workshop participants to use.

## Prerequisites
* Install an Ansible Automation Platform instance on an external VM (Red Hat employees can use RHPDS).

## Preperations for exercise 1 (Integration with Applications Lifecycle) -
1. Log into the Ansible Automation Platform web interface and create an Ansible Automation Platform application at _Administration_ -> _Applications_.
2. Create token for admin user for application at _Users_-> _admin_ -> _Tokens_ -> _Add_. Select the created application and the _Write_ scope. Copy the token to a local machine.
3. In order to host the participants log files, on the Ansible Automation Platform server, install an httpd server by running the next command - `yum install -y httpd`.
4. To allow the httpd server to serve on port 80, on the Ansible Automation Platform server, remove all `port 80` listeners at - `/etc/nginx/nginx.conf`.
5. Restart the Nginx server to apply the configurations, run the next command - `systemctl restart nginx`.
6. Create a log directory for the participants by running the next command - `mkdir /var/www/html/logs`.
7. Create an inventory with a reference to the local Ansible Automation Platform server in it.
8. Create a `Machine Credential` for the root user of the Automation Platform server.
9. In the Ansible Automation Platform web interface, create a project, and point it to the workshop's git repository (https://github.com/michaelkotelnikov/rhacm-workshop.git).
10. Create Ansible Automation Platform job template, name it Logger, make sure to allow `prompt vars` and `promt inventories` by ticking the boxes next to the instances. Associate the job template with the `07.Ansible-Tower-Integration/ansible-playbooks/logger-playbook.yml` playbook. Associate the job template with the created inventory and credentials.
11. Provide the participants with the token you created in `step 2` alongside the web URL for the Ansible Automation Platform web server. Also, provide participants with a user / password to login into the web portal in order to troubleshoot the exercise.

## Preperations for exercise 2 (Integration with Governance Policies) -
1. Create a job template named K8S-Namespace, associate it with the project, secret and inventory created in the previous exercise. Make sure to associate the job template with the `07.Ansible-Tower-Integration/ansible-playbooks/namespace-playbook.yml` playbook.
2. Provide the participants with the token you created in `step 2` in the previous exercise alongside the web URL for the Ansible Automation Platform web server. Also, provide participants with a user / password to login into the web portal in order to troubleshoot the exercise.