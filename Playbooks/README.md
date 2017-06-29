# Ansible Playbooks for OS_Nagios

Disclaimer: All files that are copied over with these playbooks should not be edited manually afterwards, as re running the playbooks will destroy changes made. However additional .cfg files can be added manually that do not exist in this repo and the re running of playbooks will not affect them.

From Ansible Master:

1) Define "nagios-server" and "nagios-clients" groups in /etc/ansible/hosts 
