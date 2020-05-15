IOS
======

## Modules
* ios_facts
* ios_config

## Useful Facts from ios_facts
* ansible_net_serialnum: E.g. FXS1714Q3VV
* ansible_net_version: E.g. 03.06.04.E
* ansible_net_image: E.g. bootflash:cat4500e-universalk9.SPA.03.06.04.E.152-2.E4.bin
* ansible_net_neighbors: E.g. Gig2/9, host: sw2.domain.local, port: Gig1/3
* ansible_net_model: E.g. WS-C4506-E
* ansible_net_iostype: E.g. IOS
* ansible_net_all_ipv4_addresses: E.g. 1.1.1.1 (mgmt address)
* ansible_net_config: Full cfg
* ansible_net_filesystems_info: Free Space
* ansible_net_hostname: Hostname
* ansible_net_interfaces: All Interfaces (Type, MTU, MAC, Duplex, IP)
* ansible_net_stacked_serialnums: SN if stacked
* ansible_net_memfree_mb: Free memory

## See
* Network best practices [Ansible](https://docs.ansible.com/ansible/latest/network/user_guide/network_best_practices_2.5.html)
* DEVNET1002 Roles, Netconf, IOS[Github](https://github.com/kuhlskev/devnet1002)
* DEVNET1002 [CLEUR](https://www.ciscolive.com/c/dam/r/ciscolive/emea/docs/2019/pdf/DEVWKS-1002.pdf
)
