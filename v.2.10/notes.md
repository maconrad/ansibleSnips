## Differences
* More stuff via Ansible Galaxy

## Commands
* Galaxy
* NCC
* Netconf-Console (no get-oper)
* Pyang etc.

| Galaxy Commands                                    | Description                    |
|----------------------------------------------------|--------------------------------|
|ansible-galaxy role init <name>                     | Create own Role                |
|ansible-galaxy collection install ansible.netcommon | Native Netconf                 |
|ansible-galaxy collection install community.yang    | YANG modules + Netconf         |

| NCC,PYANG,json2xml,netconf-console Commands                                                | Description                    |
|--------------------------------------------------------------------------------------------|--------------------------------|
|ncc --host=1.1.1.1 --get-oper -x '/licensing' -u <user> -p <pw>                             | Create own Role                |
|ncc... --get-oper -x '(/licensing/state/state-info/registration \| /licensing/state/state-info/authorization )'  | XPathA Or XPathB |
|pyang -f tree --tree-depth=2 --tree-path=/licensing/config cisco-smart-license.yang         | Inspect (part of) YANG Model   |
|pyang -f sample-xml-skeleton --sample-xml-skeleton-doctype=config cisco-smart-license.yang  | XML Skeleton of type config (or data) |
|pyang -f sample-json-skeleton --plugindir .. cisco-smart-license.yang                       | JSON Skeleton                  |
|pyang -f jtox --lax-quote-checks -o openconfig-vlan.jtox openconfig-vlan.yang               | Generate JTOX (JSON Schema)    |
|json2xml -t config -o data.xml ietf-interfaces.jtox iface.json                              | Generate Data XML from JSON, JTOX + YANG |
|netconf-console --host 1.1.1.1 --port 830 --user <user> --password <pw> --db running --get --xpath /native/call-home | Get Operation |
|netconf-console  --host=example.com -i                                                      | Interactive shell |
|... lock                                                                                    | Netconf lock a datastore       |
|... edit-config fragment1.xml --db candidate                                                | Netconf edit config            |
|... rpc commit-confirmed.xml                                                                | Netconf send a specific rpc    |
|... unlock                                                                                  | Netconf unlock a datastore       |
|... get-config                                                                              | Netconf Get-Config               |
|... discard-changes                                                                         | Needs candidate Datastore enabled|

Sample Config Snippet
* Important: <config> Tag mus contain Namespace!!!! 
* Otherwise error on cisco box about bad Element (config).

```bash
<config xmlns="urn:ietf:params:xml:ns:netconf:base:1.0" xmlns:nc="urn:ietf:params:xml:ns:netconf:base:1.0">
  <native xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-native">
    <call-home>
      <contact-email-addr xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-call-home">sch-smart-licensing@cisco.com</contact-email-addr>
      <tac-profile xmlns="http://cisco.com/ns/yang/Cisco-IOS-XE-call-home">
        <profile>
          <CiscoTAC-1>
            <active>false</active>
            <destination>
              <transport-method>http</transport-method>
            </destination>
          </CiscoTAC-1>
        </profile>
      </tac-profile>
    </call-home>
  </native>
</config>
```
Sample RPC Snippet
* RPCs do not need config/data Tag or namespaces

```bash
- name: EXECUTE - Register to configured Smart License Server
  ansible.netcommon.netconf_rpc:
    xmlns: "http://cisco.com/ns/yang/cisco-smart-license"
    rpc: register-id-token
    content: |
      <id-token>
       "{{ smart_license_server.token }}"
      </id-token>
```

## Netconf Notes
* Oper State via ncc, filtering via regular XPath
  * ncc --host=10.64.101.10 --get-oper -x '/licensing' -u admin -p cisco12345
  * ncc --host=10.64.101.10 --get-oper -x '(/licensing/state/state-info/registration | /licensing/state/state-info/authorization )' -u admin -p cisco12345
* RPCs via pyang (look for rpcs)
  * pyang -f tree cisco-smart-license.yang 
* Config via Netconf Console
  * Replace data with config but KEEP namespaces (newer Versions need the namespace tag! Otherwise bad-element return value) 
  * netconf-console --host 10.64.101.10 --port 830 --user admin --password cisco12345 --db running --get --xpath /native/call-home
* Python Modules
  * Netconf-console requires specific version of ncclient
  * jxmlease, lxml, pyang, xmltodict & jsonschema not auto-installed
* Debug
  * ansible-playbook -i inventory  main.yml -vvvv
  * ansible-playbook -i inventory --syntax-check main.yml
  * ansible-playbook -i inventory play.yaml -e ansible_python_interpreter=python
  * source ~/virtualenvs/ansible/bin/activate
  * echo “set bell-style none” >> ~/.inputrc
  * echo “set background=dark” >> ~/.vimrc
  * sudo apt-get install python3.8-venv
  * python3 -m venv ansible

## Community YANG Module
* Uses a 3 Step process
  * 1: Generates JTOX (Json Driver File)
  * 2: Genartes Spec: JSON Specification/Schema (defines JSON with VARS) using json_skeleton_plugin
  * 2a: (Generate XML Spec using pyang) (instead of json)
  * 3: Generate XML with json2xml (from jtox + json)
* Steps
  * 1: pyang -f jtox --lax-quote-checks -o ietf-interfaces.jtox ietf-interfaces.yang
  * 2: json_skeleton_plugin.py (community.yang/plugins/pyang/plugins/)
    * pyang --plugindir ./xy -f sample-json-skeleton --sample-json-skeleton-doctype=config ietf-interfaces.yang
  * 2a: (pyang -f sample-xml-skeleton --sample-xml-skeleton-doctype=config ietf-interfaces.yang) (instead of json)
  * 3: json2xml -t config -o data.xml ietf-interfaces.jtox iface.json

## Good Resources
* https://github.com/CiscoDevNet/cvd-config-templates 
* https://developer.cisco.com/docs/ios-xe/#!operational-data/pre-requisites (ncc)
* https://github.com/CiscoDevNet/ncc (ncc + Snippets in snippets-xe folder)
* https://github.com/CiscoDevNet/ncc/blob/master/Examples-XE.md (Examples)
* https://docs.ansible.com/ansible/latest/collections/ansible/netcommon/netconf_get_module.html
* https://docs.ansible.com/ansible/latest/collections/ansible/netcommon/netconf_config_module.html
* https://docs.ansible.com/ansible/latest/collections/ansible/builtin/debug_module.html
* https://docs.ansible.com/ansible/latest/collections/community/general/xml_module.html
* https://docs.ansible.com/ansible/devel/collections/ansible/builtin/dict_lookup.html
* https://docs.ansible.com/ansible/latest/user_guide/complex_data_manipulation.html
* https://docs.ansible.com/ansible/latest/user_guide/playbooks_handlers.html
* https://ttl255.com/ansible-until-loop/ 
* https://medium.com/swlh/5-simple-ansible-tweaks-for-better-playbooks-fd9b789d5bca (include tasks for reusage)
* https://www.cisco.com/c/en/us/td/docs/switches/lan/catalyst9300/software/release/17-3/configuration_guide/sys_mgmt/b_173_sys_mgmt_9300_cg/cisco_smart_licensing_client.html#task_xqt_m5p_qgb (9300 register id token)
* https://www.cisco.com/c/en/us/td/docs/routers/sl_using_policy/b-sl-using-policy/info_about.html (Smart License policy model)
* https://www.cisco.com/c/en/us/td/docs/routers/sl_using_policy/b-sl-using-policy/how_to_configure_workflows.html (License smart transport)
* https://cscoblogs-prod-17bj.appspot.com/networking/new-deployment-method-for-cisco-smart-licensing-is-easier-faster-and-more-consistent 
* https://ansiblemaster.wordpress.com/2016/07/29/jinja2-lstrip_blocks-to-manage-indentation/