Basics
======

## Setup and Authentication
* /etc/ansible/ansible.cfg
* ANSIBLE_CONFIG ENV
* ansible.cfg in current directory
* .ansible.cfg in home directory
* Authentication:

```bash
    ssh-keygen
    ssh-copy-id root@1.2.3.4
    ssh root@1.2.3.4
```
***
## Inventory
* INI oder YAML Style
* INI Groups are in <span style="color:green">[]</span>
* See inventory.ini
* See inventory.yaml

***

## Commands
* ansible: On Demand Module Execution without playbook
* ansible-playbook: Playbook
* ansible-vault: Sensitive Data encrypt into YAML File
* ansible-pull: Lets client pull
* ansible-docs: Docstring parsing of ansible module -> see example
* ansible-galaxy: Role download from community

```bash
    ssh-keygen
    ssh-copy-id root@1.2.3.4
    ssh root@1.2.3.4
```


## Links
* See [CL2019](https://www.cisco.com/c/dam/m/sr_rs/events/2019/cisco-connect/pdf/using_ansible_in_dc_automation_radenko_citakovic.pdf)
* 