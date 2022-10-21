# ANSIBLE

Ansible is a configuration and automation tool used for the following cases

1. Automation
    - linux, start service, stop server
2. change management
3. large scale automation

## Features
1. No agent on the clients. it uses ssh for authentication
2. No Databases
3. Simple setup
4. No residual left on the target machine
5. No programming uses yaml
6. it uses boto3

Ansible uses the following for connection
* Linux uses ssh
* Cloud uses API
* windows uses WINRM

The system where ansible is installed is called control machine
Ansible executes the script locally.


Adhoc commands

To ping all the host
```
ansible -i inventory -m ping all
```

To install a package
```
ansible -i inventory -m yum -a "name=httpd state=present" web01 --become
```

To start a service
```
ansible -i inventory -m yum -a "name=httpd state=started" web01 --become
```


To copy a service
```
ansible -i inventory -m copy-a "src=index.html src=/var/www/html/" web01 --become
```

To execute a playbook
```
ansible-playbook -i inventory inventory_name.yaml
```


To execute a playbook with argument
```
ansible-playbook -e USRNM=admin -e PSWD=123456 -i inventory inventory_name.yaml 
```


To check if a playbook will work before actual execution
```
ansible-playbook -e USRNM=admin -e PSWD=123456 -i inventory inventory_name.yaml -C
``