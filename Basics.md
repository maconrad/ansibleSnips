Basics
======

## Terms
* Module: Code, typically written in Python
* Task: single action
* Play: Set of tasks to host or host group
* Playbook: YAML File with one or more plays
* Role: A pre-built set of playbooks designed to perform some standard configuration in a repeatable fashion. A play could leverage a role rather than tasks. All playbooks from role will be executed against a host.
* FQCN: Fully Quallified Collection Name (e.g. ansible.builtin.template)
* Connection Plugins
  * Default: Ansible connects via SSH > Uploads Python Modules > Runs Module
  * For non-linux based systems: Connection=IOS

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
    
    #Add a host key so can run playbook on host (netconf & ssh)
    ssh-keyscan -p 830 host.domain.local >> /home/user/.ssh/known_hosts 
    ssh-keyscan host.domain.local >> /home/user/.ssh/known_hosts
```

```ini
[defaults] 
inventory = ./inventory 
remote_user = someuser 
host_key_checking = False
# ask_pass = false 

[privilege_escalation] 
# become = true
# become_method = sudo 
# become_user = root
```

***
## Inventory
* INI oder YAML Style
* INI Groups are in <span style="color:green">[]</span>
* See inventory.ini
* See inventory.yaml

```ini
[servers1]
demo1.example.com
demo2.example.com

[servers2]
demo3.example.com
demo4.example.com

[servers3]
demo5.example.com ansible_user=hans home=/home/joe

[servers:children]
servers1
servers2

[servers:vars]
user=joe

```

***

## CLI Commands
* ansible: On Demand/Ad-Hoc Module Execution without playbook
  * ansible all -m ping -i inventory 
  * ansible all -m setup 
  * ansible all -m docker_container -a "name=mycontainer image=ghost ports=8001:2368"
  * ansible ubuntu -m copy -a "src=/etc/hosts dest=/tmp/hosts"
  * ansible ubuntu -m shell -a 'echo $TERM'
  * ansible -m setup all -a 'filter=ansible_distribution'
  * ansible <host-pattern>|all -m <module> -a <module-args> -i <inventory>
* Help (list collections & modules)
  * ansible-doc -l | egrep aci
  * ansible-doc aci_bd
* ansible-playbook: Playbook
  * ansible-playbook --syntax-check xy.yaml
  * ansible-playbook -i inventory xy.yml -vvvv
* ansible-playbook nxos_fabric.yml --tags "bgp"
  * See samplePlaybooks: nxos_sample_bgp for settings Tags
* ansible-vault: Sensitive Data encrypt into YAML File
* ansible-pull: Lets client pull
* ansible-docs: Docstring parsing of ansible module -> see example
  * ansible-doc debug
* ansible-galaxy: Role download from community
* Ansible Facts (facts von Host aufrufen)
  * ansible all -m setup

## Connections
* See [Collection Index](https://docs.ansible.com/ansible/latest/collections/)
* List all connections > `ansible-doc -t connection -l`
* Details of a connection > `ansible-doc -t connection network_cli`
* Commonly used
  * ansible.netcommon.network_cli
  * ansible.netcommon.netconf
  * ansible.netcommon.httpapi
  * local
  * winrm
  * paramiko_ssh
  * ssh (default)

```yaml
---
- name: configure the router
  hosts: routers
  gather_facts: no
  connection: ansible.netcommon.network_cli
  network_os: cisco.ios.ios
  become: yes
  become_method: enable
  tasks:
    - name: set the motd banner
      ios_banner:
        banner: motd
        text: "Hello world!"

```

## Vars
* Ini-style (=) or YAML Style
* YAML Style preferred for vars (dict)
  * Example: 
    * "{{ varname }}" For accessing vars
    * bgp_neighbors uses YAML List (-) + dict {} do define subvars
      * access for example with "{{ item.neighbor }}" in with_items loop
* Definitionen
  * keine - oder spaces, $, zahlen am Anfang
* Global Scope 
  * extra_vars > ansible-playbook main.yml -e "packages=apache"
* Play Scope
  * vars innerhalb von play
  * vars_files referenced in playbook with vars_files: -vars/some_other_vars.yml
* Host Scope
  * Inventar variablen erfassen
* Runtime creation
  * register - Resultat des Tasks in einer Variable
  * set_fact - Neue Variable mit beliebigem Inhalt
    * run with -vvv to see output of task 
* Special Vars
  * ansible_version: Information zur genutzten Ansible Version
  * group_names: Liste der Gruppen des aktuellen Hosts
  * inventory_hostname: Inventar Hostname des aktuellen Hosts
  * role_name: Name der ausgeführten Ansible Rolle
  * ansible_facts: Facts des aktuellen Ansible Hosts
  * ansible_become_user: Ansible Superuser auf Zielhost
  * ansible_become_method: Methode, um Superuser zu werden (IOS: enable)
  * ansible_connection: Connection Plugin 
  * ansible_host: Alternative zu inventory_hostname
  * ansible_user: Ansible Nutzer auf Zielhost


```bash
playbook.yml
ansible.cfg
inventory
├── group_vars
│   ├── servers.yml
│   └── routers.yml
├── host_vars
│   ├── db.yml
│   └── rtr1.yml
|── vars
│   └── some_other_vars.yml
└── inventory

```

```bash
---
  # vars file for leaf
  nxos_provider:
    username: "{{ user }}"
    password: "{{ pwd }}"
    transport: nxapi
    timeout: 30
    host: "{{ inventory_hostname }}"

  asn: 65000

  bgp_neighbors:
  - { remote_as: 65000, neighbor: 192.168.0.6, update_source: Loopback0 }
  - { remote_as: 65000, neighbor: 192.168.0.7, update_source: Loopback0 }
```

```yml
# Basic Var usage
---
- name: configure basic router settings
  hosts: routers
  gather_facts: no
  connection: local
  vars:
    company_name: XY Inc.
  tasks:
    - name: configure the motd banner
      ios_banner:
        banner: motd
        text: "This Device is managed by {{ company_name }}."
...

# Usage of Vars Files
- name: vars_files Block Example
  hosts: all
  vars_files: 
    - vars/users.yml
  tasks:
    # <SNIP>


```

```yml
- name: Install package and print result
  hosts: all
  tasks:
    - name: Install the package
      ansible.builtin.yum:
        name: httpd
        state: installed
      register: install_result
    - name: Display the result
      ansible.builtin.debug:
        msg: "{{ install_result }}"
    - name: Register new variable
      ansible.builtin.set_fact:
        httpd_packet: "{{ install_result.results[0].split(' ')[0] }}"
    - name: Display the httpd packet
      ansible.builtin.debug:
        msg: "{{ httpd_packet }}"

```

## Ansible Cfg
* Config
* See for Config ansible.cfg priority [Guide](https://docs.ansible.com/ansible/latest/reference_appendices/config.html)
* Very Basic Example

```ini
[defaults]
inventory = ./inventory
host_key_checking = False
```


## Ad-hoc Commands
* Setup: Query Facts
* -a for additional arguments
* -e for extra_vars (see below)

```bash
    ansible -m setup -u root servers
```

## Comments, Conditionals, Filters, Tags
* Comment

```bash
    # Single Line
    {#
      Multi Line
    #}
```

* When
  
```bash
    ...
    when: ansible_host == "1.1.1.1"
```

* Filters
  
```bash
    ...
    {{ myvar | ipaddr }}
```

* Tags
  * Only run tasks with a certain tag
  * ansible-playbook playbook.yml --tags "feature"
  
```bash
    ...
    - name: Enable BGP
      nxos_feature:
        feature: bgp
        provider: "{{nxos_provider}}"
        state: enabled
      tags: feature
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

## Conditionals 

### When
* When (if) statements
  * is
  * is defined
  * is undefined

```yml
---
- hosts: localhost
  tasks:
    - debug:
        msg: 'Test MSG'
      register: output

    - debug:
        msg: '{{ output }}'
      when: output.failed == true
...

```

### Loops
* With_items / with_dict (deprecated) -> loop
* Loop + flatten filter
* Until / Retries / Delay Statements
* Examples
  * Loop via item var (default)
  * Loop via user var (renamed item var via Loop control)
  * When (if) statements
    * is | is defined | is undefined

```yaml
---
- hosts: localhost
  tasks:
    - debug:
        msg: 'Name: {{ item.name }} - Alter: {{ item.age }}'
      loop:
        - {'name': 'testuser1', 'age': 32}
        - {'name': 'testuser2', 'age': 26}
...

# Loop Control steuert Variablename und Output
---
- hosts: localhost
  tasks:
    - debug:
        msg: 'Name: {{ user.name }} - Alter: {{ user.age }}'
      loop:
        - {'name': 'testuser1', 'age': 32}
        - {'name': 'testuser2', 'age': 26}
      loop_control:
          loop_var: user
          label: '{{ user.name }}'
...

# Combine loop and when with renamed loop var
---
- hosts: localhost
  tasks:
    - debug:
        msg: 'Name: {{ user.name }} - Alter: {{ user.age }}'
      loop:
        - {'name': 'testuser1', 'age': 32}
        - {'name': 'testuser2', 'age': 26}
      loop_control:
          loop_var: user
          label: '{{ user.name }}'
      when: user.age >= 30
...

## Combine loop and when with renamed loop var
---
- hosts: localhost
  tasks:
    - debug:
        msg: 'Name: {{ user.name }} - Alter: {{ user.age }}'
      loop:
        - {'name': 'testuser1', 'age': 32, 'special': True}
        - {'name': 'testuser2', 'age': 26}
      loop_control:
          loop_var: user
          label: '{{ user.name }}'
      when: user.special is defined
...


```

```yml  
    - name: with_items
      debug:
        msg: "{{ item }}"
      with_items: "{{ items }}"

    - name: with_items -> loop
      debug:
        msg: "{{ item }}"
      loop: "{{ items|flatten(levels=1) }}"
```

## Error Handling
* block, rescue, always = try/catch/finally
* ignore_errors: yes > Kein Abbruch playbook
* any_errors_fatal: true > All hosts marked as failed, default behaviour only the host where error occured
* failed_when: false > Define failed condition
* retries: If command needs to be run multiple times

```yml
# Define failed condition - string search
- name: Fail task when the command error output prints FAILED
  ansible.builtin.command: /usr/bin/example-command -x -y -z
  register: command_result
  failed_when: "'FAILED' in command_result.stderr"

# Define failed condition - return code value
- name: Fail task when both files are identical
  ansible.builtin.raw: diff foo/file1 bar/file2
  register: diff_cmd
  failed_when: diff_cmd.rc == 0 or diff_cmd.rc >= 2

```

```yml
# Failed when True => Always fails when return above true
---
- hosts: localhost
  tasks:
    - block:
      - debug:
          msg: 'Block'
        failed_when: True

      rescue:
      - debug:
          msg: 'Rescue'

      always:
      - debug:
          msg: 'Always'
...

## Rescue block not executed because of ignore errors
---
- hosts: localhost
  tasks:
    - block:
      - debug:
          msg: 'Block'
        failed_when: True
        ignore_errors: True

      rescue:
      - debug:
          msg: 'Rescue'

      always:
      - debug:
          msg: 'Always'
...


```


```yml
  tasks:
    - name: Gather nxos facts on the switches
      nc_nxos_facts:
        gather_subset: legacy
      register: result
      retries: 5
      delay: 5
      until: result is not failed

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

* Create Own Roles
  * Create own role with ansible-galaxy init 
  * Creates default folder structure with templates (j2), vars, tasks, etc.
    * Example: ansible-galaxy init myrole
    * Example 2.10: ansible-galaxy role init test-role-1
  * Structure:

```bash  
.
├── playbook.yml
├── inventory
├── roles
|   ├── rolename
|       └── tasks
|           └── main.yml
|       └── templates
|           └── sometempl.j2
|       └── vars
|           └── main.yml

```

## Handlers
* Helper Funktions, e.g. for restarting a service
* Only get executed if task is CHANGED
* Only executed once even if called multiple times
* Always exectued at the end of playbook

```yml
tasks:
  - name: template configuration file
    template:
      src: template.j2
      dest: /etc/foo.conf
    notify:
       - restart apache
handlers:
    - name: restart apache
      service:
        name: apache
        state: restarted

# Single Execution even on multiple calls
---
- hosts: localhost
  tasks:
    - debug:
        msg: 'Output 1'
      notify:
        - Handler 1
      changed_when: True

    - debug:
        msg: 'Output 2'
      notify:
        - Handler 1
      changed_when: True

  handlers:
    - name: Handler 1
      debug:
        msg: 'Handler 1'
...


```


## Vaults
* Sensitive Data encyption (Passwords/Keys)
  * File or single variable
* ansible-vault encrypt|decrypt|view|edit|rekey
* ansible-vault encrypt_string --stdin-name ansible_password
  * asks for vault pw which needs to be supplied when calling playbook
  * better than directly supplying
* ansible-playbook pod-router.yml --ask-vault-pass

```bash
  ---
    username: "admin"
    password: "supersecret"
  ...

  ansible-vault encrypt secretfile.yml
  ansible-vault encrypt_string "supersecret" --name "var_name_eg_password"
  ansible-vault encrypt_string --stdin-name ansible_password
  ansible-playbook --vault-password-file=vault-pw-file.txt site.yml
```

## Callbacks
* Modify default output behaviour of Ansible


```bash
  cat > ansible.cfg << _EOF
  [defaults]
  inventory = hosts
  host_key_checking = false
  record_host_key = true
  stdout_callback = debug

_EOF
```

## Links
* See Tutorial [Slideshare](https://de.slideshare.net/apnic/network-automation-netdevops-with-ansible-89230669)
* See [CL2019](https://www.cisco.com/c/dam/m/sr_rs/events/2019/cisco-connect/pdf/using_ansible_in_dc_automation_radenko_citakovic.pdf)
* See [Ansible Documentation Variables](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html)
* See [Ansible Documentation Roles](https://docs.ansible.com/ansible/latest/galaxy/user_guide.html#installing-roles)
* See [VXLAN EVPN with Ansible CL](https://fchaudhr.github.io/LTRDCN_1572/)
* See Callback Modules [List](https://docs.ansible.com/ansible/latest/plugins/callback.html)
* See [Preceden Rules](https://docs.ansible.com/ansible/latest/reference_appendices/general_precedence.html#general-precedence-rules)
* See [Understanding variable precedence](https://docs.ansible.com/ansible/latest/user_guide/playbooks_variables.html#:~:text=In%20general%2C%20Ansible%20gives%20precedence,that%20variable%20in%20the%20namespace.)
