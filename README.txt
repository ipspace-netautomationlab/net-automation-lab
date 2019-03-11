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