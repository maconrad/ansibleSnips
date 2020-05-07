Netconf
=========

## Debug
* export ANSIBLE_DEBUG=True
* export ANSIBLE_LOG_PATH=/home/user/ansible.log
* -vvvv combined with ANSIBLE_DEBUG for lots of details (including py code lines that fail)

## Netconf Module
* ansible_network_os=iosxr for IOS devices
  * See Ncclient [Device_Params](https://pypi.org/project/ncclient/)
* Errors
  * nansible.module_utils.connection.ConnectionError: global name 'known_hosts_lookup' is not defined
    * => Solution: ssh-keyscan -p 830 hostname > /home/user/.ssh/known_hosts
  * ansible.module_utils.connection.ConnectionError: 'Connection' object has no attribute 'manager'
    * => Solution: set ansible_network_os to <span style="color:green">iosxr</span>
* Use Netconf-client
  * netconf-console --host host|ip --port 830 --user admin --password pw --db running --get-config --xpath /native/ntp (config data)
  * netconf-console --host host|ip --port 830 --user admin --password pw --get --xpath /cpu-usage (oper data)
  * netconf-console --host host|ip --port 830 --user admin --password pw --get --xpath /access-lists/access-list[access-control-list-name=100] (oper data)

## Netconf Flow
1. Download Switch capabilities with python Script
2. Go into Folder where models are stored (e.g. cd models)
3. Run pyang to get xpath
   1. Example: pyang -f tree --tree-depth 2 Cisco-IOS-XE-native.yang
      1. Example Output: native/tacacs, native/hw-module, native/call-home etc.
   2. Example: pyang -f tree Cisco-IOS-XE-ntp.yang | more
   3. Example: pyang -f tree --tree-path /native/process Cisco-IOS-XE-native.yang
   4. Example: pyang -f sample-xml-skeleton Cisco-IOS-XE-process-cpu-oper.yang
 

## Netconf Switch Configuration
* Verify via ssh -s user@ip -p 830 netconf


```bash
netconf-yang
netconf-yang feature candidate-datastore

show netconf-yang datastores
show netconf-yang sessions
show netconf-yang sessions detail
show netconf-yang statistics
show platform software yang-management process

ip access-list standard acl1_permit 
 permit 192.168.255.0 0.0.0.255

netconf-yang ssh ipv4 access-list name acl1_permit
restconf ipv4 access-list name ipv6-acl1_permit

```
***


## See
* Ansible Network Automation [Intro](https://blogs.cisco.com/developer/automating-network-operations-part1)
* Ansible Network Automation [Data Models](https://blogs.cisco.com/developer/network-operations-data-models)
* Ansible Network Automation [Netconf](https://blogs.cisco.com/developer/automating-network-operations-3)
* Ansible Network Automation [TextFSM](https://blogs.cisco.com/developer/automating-network-operations-5)
* XPath in Netconf / YANG [Tail-f](https://info.tail-f.com/xpath-netconf-yang)
* Candidate Data Store [CiscoCfgGuide](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/169/b_169_programmability_cg/configuring_yang_datamodel.html)
* Netconf and Restconf ACLs [ACLs](https://www.cisco.com/c/en/us/td/docs/ios-xml/ios/prog/configuration/1612/b_1612_programmability_cg/netconf_and_restconf_service_level_acls.html)