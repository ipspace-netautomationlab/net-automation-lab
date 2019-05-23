#############################################################
#															#
#	IP.SPACE - Building Network Automation Solutions		#
#															#
#					February-May 2019						#
#															#
#############################################################
#															#
#	Authors	:	Emanuele Ballarini and Valter Milanese		#
#															#
#	Date	:	09/03/2019									#
#															#
#	Ref.	:	virl-lab-scheme.pdf							#
#															#
#############################################################



- INTRODUCTION

This README file describes the laboratory architecture realized by the authors in order
to get the hands-on exercises done.

The first hands-on exercise requires the implementation of a virtual network environment,
by using a whatever preferred platform (in our case we used Cisco VIRL),
which shall be managed (configuration and/or monitored) by an external server running Ansible.
Please, refer to "virl-lab-scheme.pdf".



- LABORATORY ENVIRONMENT DESCRIPTION

The architecture has been realized by using a couple of bare metal servers (DELL R710) with the 
following features:

SERVER #1 (Ansible 2.7.8):

*	CPU:	2 x Intel Xeon (8 cores) at 2.4 GHz
*	RAM:	16 GB DDR3
*	O.S.:	Linux Centos 7.6

SERVER #2 (Cisco VIRL v.1.5.145):

*	CPU:	2 x Intel Xeon (8 cores) at 2.4 GHz
*	RAM:	64 GB DDR3
*	O.S.:	Linux Ubuntu 16.04 LTS

The ETH0 server NICs have been used for server management, while ETH1 ones for the communication
among the Ansible server and the virtual network devices; ETH1 on server #2 has been configured
in bridge mode.

The virtual environment management network is 172.16.1.0/24 where .1 has been assigned to Ansible
server; the virtual network devices come to be automatically deployed with an empty default 
configuration (just IP address for management) and the management IP address is assigned within
the IP address range 172.16.1.10-172.16.1.15.

The basic setup of the lab requires at least that the Ansible server is able to connect to
network devices; in our case, the implemented Ansible playbook, named "basic-show-commands.yml",
is able to run basic show commands on the devices, in particular the "show version" command.
The command output is then displayed in json format.

The virtual network is composed by six (6) Cisco Nexus family devices connected each other in a
spine-and-leaf topology (2 spines and 4 leafs).



- GITHUB REPOSITORY

All the project files have been uploaded to a GitHub repo where the "master" branch contains the
official release and the "development" branch is used as development environment.

Commits inside master branch will be referred to homework completion.

Within the master branch folder you can find the following folders:

*	Ansible: this folder contains the Ansible config files;
*	Virl-topology:	this folder contains the Cisco VIRL topology files.



- ANSIBLE FOLDER

The "inventory" folder contains the Ansible files for the devices inventory.

All the devices are gathered into a super-group named "nexus-switch", moreover they are also
gathered into two groups named "spine" and "leaf" based on their role.

In the "group_vars" folder you can find the specific variables defined for the groups; in our
case, we defined just one file for the group "nexus-switch" in which we defined the following
variables:

*	username and password for device access;
*	connection module = network_cli;
*	device OS = nxos.

- SECOND ASSIGNMENT: EASY WINS

The second assignment has been implemented by "info-collection.yml" playbook. It collects 
nxos system info by using nxos_facts module. The retrieved data is presented in HTML files
inside the "output/web" folder. The pages display ip addresses, net interfaces and system data.
Moreover all information has been stored into "output/json" folder as JSON files.

- THIRD ASSIGNMENT: DATA MODEL

The third assignment is focused on creating and validating the data model for an BGP EVPN VXLAN fabric.
The main part of data model has been implemented in "group_vars/all" file, moreover some device specific data
has been implemented in "host_vars/{{inventory_hostname" files.
Customer services (VLANs and VRFs) can be declared into "inventory/vars/services.yml" file.

To create and validate the data model a playbook named "config.yml" has been implemented with the use of "--tags=config" CLI switch.
You can still use the previous playbook "info-collection.yml" with the use of "--tags=info" CLI switch. The use of tags is a requirement for a future merge of the two playbook files.

The config file for each device is stored in "output/config" folder.

To create common config parts a role named "fabric" has been created with all the common config tasks. 
The specific tasks for leaves and spines are included in the "spine" and "leaf" roles.
Each task creates a single config file in the output config folder, the file name is composed by "inventory_hostname" and feature name.

A final task in the playbook assembles the single files in a "complete" config file for each device which is named {{inventory_hostname}}_complete.conf.

- FOURTH ASSIGNMENT: CHANGING NETWORK CONFIGURATIONS OR STATE

The fourth assignment concerns all the tasks for a complete configuration deployment from scratch or a part of it (i.e. services)
A new playbook has been released, named "netexec.yml", which import all of the rest playbooks, so you can call just this playbook
differentiating the tasks to be played by typing different tags. The specific playbook which handles the deployment tasks is named
"deploy.yml".

Hereby the defined tags up until now:
  * "info": the playbook gathers all the information related to the 2nd assignment;
  * "config": the playbook creates the config files as explained in the 3rd assignment;
  * "first_deploy": the playbook deploys the full-blown configuration file from scratch to a new device;
  * "basic": the playbook deploys the partial configuration file related to basic info (i.e. management, users, snmp, etc.);
  * "underlay": the playbook deploys the partial configuration file related to underlay fabric (i.e. P2P interfaces, OSPF, etc.);
  * "overlay": the playbook deploys the partial configuration file related to overlay fabric (i.e. BGP);
  * "services": the playbook deploys the partial configuration file related to services (i.e. VRFs, VLANs, etc.).

In case of non first deploy, the playbook copies the current device's configuration into a backup folder on the Ansible server.
The playbook handles also errors through the block/rescue paradigm. A rollback action to the latest config version will be taken if any error is found.

- FIFTH ASSIGNMENT: VALIDATION, ERROR HANDLING AND UNIT TESTS

The fifth assignment is focused on validating the deployment tasks, handling errors and making some unit tests.
A new playbook has been released, named "validate.yml", which handles the needed tasks. Furthermore, new tags
have been added in order to aggregate them into the unique playbook "netexec.yml".

Hereby the brand-new defined tags:

  * "validate": the playbook runs all the tasks except "predeploy";
  * "predeploy": the playbook checks if requested services, defined in "services.yml" file, already exist on current devices (before deployment);
  * "postdeploy": the playbook checks if requested services, defined in "services.yml" file, are missing on current devices (after deployment);
  * "rollback": the playbook rolls back to the latest saved configuration if any error has been found.

The tasks will be run even if you use "basic", "underlay", "overlay" and "services" tags by doing the related checks only.
We decided to split validation from rollback action in order to let the administrator do the best choice. Anyway, you can
merge the tasks by running the playbook with more tags.

Examples: 

* $> ansible-playbook playbooks/netexec.yml --tags=validate (validating only)

* $> ansible-playbook playbooks/netexec.yml --tags=validate,rollback (validating and rolling back in event of errors found)


NOTES: Some NXOS modules didn't work in our case, so we didn't manage to implement them (i.e. "nxos_file_copy")! We used alternative modules. 
We have embedded inside deployment process a rollback stage to revert syntax errors, 
this feature allows the system to make deterministic atomic changes and allows the developer to check any error inside templates. 
