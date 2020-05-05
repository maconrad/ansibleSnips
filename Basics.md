Basics
======

## Terms
* Module: Code, typically written in Python
* Task: single action
* Play: Set of tasks to host or host group
* Playbook: YAML File with one or more plays
* Role: A pre-built set of playbooks designed to perform some standard configuration in a repeatable fashion. A play could leverage a role rather than tasks. All playbooks from role will be executed against a host.

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

## CLI Commands
* ansible: On Demand/Ad-Hoc Module Execution without playbook
  * ansible all -m ping -i inventory 
  * ansible all -m setup 
  * ansible all -m docker_container -a "name=mycontainer image=ghost ports=8001:2368"
  * ansible -m setup all -a 'filter=ansible_distribution'
* ansible-playbook: Playbook
  * ansible-playbook --syntax-check xy.yaml
  * ansible-playbook -i inventory xy.yml -vvvv
* ansible-vault: Sensitive Data encrypt into YAML File
* ansible-pull: Lets client pull
* ansible-docs: Docstring parsing of ansible module -> see example
  * ansible-doc debug
* ansible-galaxy: Role download from community


## Ad-hoc Commands
* Setup: Query Facts
* -a for additional arguments
* -e for extra_vars (see below)

```bash
    ansible -m setup -u root servers
```

## Playbook Commands
* register & debug (Single line or with msg) -> Example:
* register creates new facts from output of other tasks
* Variables referenced with {{name}}
* vars_files to include variable files in playbook
* --extra-vars or -e for Command Line
* group_vars/all | group_vars/all.yaml | group_vars/groupname
* host_vars host_vars/xyz.boston.example.com
  
```bash  
  - name: Gather IOS Facts and save to output
    register: output
    ios_facts:
      gather_subset: all

  - debug: var=output

  - debug:
      msg:
        - " OS Version  is {{ ansible_net_image }} "
        - " OS Version  is {{ ansible_net_version }} "

```
```bash  
  ansible-playbook release.yml --extra-vars "version=1.23.45 other_variable=foo"
  ansible-playbook release.yml -e '{"version":"1.23.45","other_variable":"foo"}'
  ansible-playbook release.yml --extra-vars "@some_file.json"
  ansible-playbook arcade.yml --extra-vars "{\"name\":\"Conan O\'Brien\"}"
```  

## Loops
* With_items / with_dict (deprecated) -> loop
* Loop + flatten filter
* Until / Retries / Delay Statements


```bash  
    - name: with_items
      debug:
        msg: "{{ item }}"
      with_items: "{{ items }}"

    - name: with_items -> loop
      debug:
        msg: "{{ item }}"
      loop: "{{ items|flatten(levels=1) }}"
```

## Roles
* Search Role via Galaxy: ansible-galaxy search nginx
* Install with: ansible-galaxy install --roles-path . geerlingguy.nginx
* Example:

```bash  
  - hosts: servers
    roles:
      - { role: geerlingguy.nginx }
```

## Links
* See [CL2019](https://www.cisco.com/c/dam/m/sr_rs/events/2019/cisco-connect/pdf/using_ansible_in_dc_automation_radenko_citakovic.pdf)
* See [Ansible Documentation Variables](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)
* See [Ansible Documentation Roles](https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#installing-roles)