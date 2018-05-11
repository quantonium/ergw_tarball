erGW Setup
==========

This repository contains the components to run erGW.  It interfaces with MME on
one side through S11 interface and with VPP on the other side through Sx
interface.  Together, erGW and VPP, creates a combined S-GW/PGW (sometimes also
called a SAE-GW) which support the S11, S1-U and SGi 3GPP reference points.

Requirements
------------

The setup requires at least three NICs available on the system running Ubuntu
16.04.  It will also require a network range that can be assigned to UEs.

In this document the following NIC topology is assumed:

ens3 - Used to login to the system  
ens4 - MME facing NIC used by erGW  

You will need super user privileges to follow the instructions below and run
erGW.


Prerequisites
-------------

ILA and VPP GW should be install first.

See: https://github.com/quantonium/ila/blob/master/README.quickstart


Quick Install
-------------
Download quick_install file from this repository on your system. Change the
permission to make it executable.  

	wget https://raw.githubusercontent.com/quantonium/ergw_tarball/master/quick_install -O /tmp/ergw_quick_install
	chmod +x /tmp/ergw_quick_install


Run the quick_install script:

	/tmp/ergw_quick_install


The script will install ergw along with other required packages (like Erlang)
on your system. Note that quick_install requires superuser priveleges.


Configuring erGW
----------------

make_test_conf script creates a test conifugration for erGW. This defines an
address pool assigment with SIR prefix of 2222:: which is compatible with the
VPP test configraion (see README.vpp_test). The script takes four IPv6 address
arguments:

     vfr_irx_addr - a local address for the vrf (ens4) interface
     enodeb_addr = eNodeB  address
     gw_addr - default gateway on en
     mme_addr - MME address

Run the following command:  

	cd  /usr/src/quantonium/ergw_tarball
	./make_test_conf <vrf_irx> \
	     <enodeb_addr> <gw_addr> <mme_addr> 

Note that QDIR must be set to the installation directory for ILA. This
is likely "export QDIR=~/quantonium/install"


Running erGW
------------

erGW is started by running:

	sudo /opt/ergw-gtp-c-node/bin/ergw-gtp-c-node foreground


Status Check
------------

Run the following command for checking if erGW has successfully opened the
ports for communication with MME and VPP

	sudo ss -anup \( sport = 2123 or sport = 8805 \)  

Port 2123 on vrf-irx device is used for communication with MME. Port 8805 is
used to coomunicate with VPP.


Further Reading
---------------

HOWTO file in this repository has the details of installing, configuring and
running of erGW.

