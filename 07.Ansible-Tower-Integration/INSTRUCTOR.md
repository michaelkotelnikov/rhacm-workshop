1. Create application
2. Create token for admin user for application
3. yum install -y httpd
4. remove all port 80 reference in nginx.conf
5. systemctl restart nginx
6. mkdir /var/www/html/logs
7. Create AAP project
8. Create AAP job template, call it Logger, prompt vars and inventories, associate with logger-playbook.yml
9. passwd root
10. Create an inventory with localhost in it.
11. Associate the ansible_host variable with the inventory
12. Create a credential for root and associate with job template


13. Create a jobtemplate named K8S-Namespace for policy exercise
14. Associate the job template with the namespace-playbook.yml
15. ansible-galaxy collection install community.kubernetes
16. pip3 install openshift