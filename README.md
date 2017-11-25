This role installs and configures tools to allow for AD authentication using sssd and kerberos. The user needs to specify variables in order to connect and in addition create a few OUs in ldap. 

Setup
-----

All variables are mandatory in order to create a successfull connection with AD.

	ntp_servers: one or more NTP servers
	dns_servers: one or more DNS servers
	domain: your domain eg. acme.local
	workgroup: acme
	domain_ou: OU=Linux Servers,OU=Servers,OU=Infrastructure,OU=ACME,DC=acme,DC=local
	aduser: ad bind account (ensure user has bind permission in ldap)
	psw: "secret password"
	adserver: ip address of AD server
	sudo_ou: OU=sudoers,OU=Linux,OU=Global Security Groups,OU=ACME,DC=acme,DC=local
	allow_users_groups:
	  - root
	  - (wheel)
	  - system_admins (this could be your ldap group)

You can overwrite the variables in the group_vars or host_vars bases on your inventory structure.

Proxy
-----

Optionally set the Proxy and whitelist no_proxy if needed, otherwise disable with proxy_env: {null:"null"}
proxy_env: {'http_proxy': 'http://host.com:8080', 'https_proxy': 'http://host.com:8080', 'no_proxy': '.example.com'}

Examples
--------

example playbook: Assuming you have a inventory group called prodRODC that attempts to connect to Read Only Domain Controller i.e. DMZ
In this case you can run this Ansible role rodcKeytab(TODO) to create the keytab file and manually copy it to Linux.

	---
	- name: Group distributions by RHEL releases
	  hosts: "{{group}}"
	  remote_user: root

	  tasks:
	    - group_by: key=rhel{{ansible_distribution_major_version}}
	      tags: always

	- name: Running RHEL6 roles
	  hosts: rhel6
	  environment: "{{proxy_env}}"
	  roles:
	    - dns
	    - krb
	    - ntp
	    - packages
	    - timesync
	    - samba
	    - {role: adjoin, when: '"prodRODC" not in group_names' }
	    - {role: rodc, when: '"prodRODC" in group_names' }
	    - postinstall

	- name: Running RHEL7 roles    
	  hosts: rhel7
	  environment: "{{proxy_env}}"
	  roles:
	    - dns
	    - krb
	    - chrony
	    - packages
	    - timesync
	    - {role: installRealmd,  when: '"prodRODC" not in group_names' }
	    - {role: realmjoin,      when: '"prodRODC" not in group_names' }
	    - {role: rodc,           when: '"prodRODC" in group_names' }
	    - postinstall 
	    - login
