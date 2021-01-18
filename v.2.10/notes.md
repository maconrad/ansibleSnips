## Differences
* ansible-galaxy role init <name>


## Netconf Notes
* Oper State via ncc, filtering via regular XPath
  * ncc --host=10.64.101.10 --get-oper -x '/licensing' -u admin -p cisco12345
  * ncc --host=10.64.101.10 --get-oper -x '(/licensing/state/state-info/registration | /licensing/state/state-info/authorization )' -u admin -p cisco12345
* RPCs via pyang (look for rpcs)
  * pyang -f tree cisco-smart-license.yang 
* Config via Netconf Console
  * Replace data with config but KEEP namespaces (newer Versions need the namespace tag! Otherwise bad-element return value) 
  * netconf-console --host 10.64.101.10 --port 830 --user admin --password cisco12345 --db running --get --xpath /native/call-home

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