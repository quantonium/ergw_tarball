erGW Setup
==========

This repository contain the components to run erGW.  It interfaces with MME on one side 
through S11 interface and with VPP on the other side through Sx interface.
Together, erGW and VPP, creates a combined S-GW/PGW (sometimes also called a SAE-GW) 
which support the S11, S1-U and SGi 3GPP reference points.

Requirements
------------

The setup requires at least three NICs available on the 
system running Ubuntu 16.04.  It will also require
a network range that can be assigned to UEs.

You will need super user privileges to follow the instructions below and run erGW.

Setup
------

1. Go to /usr/src/quantonium (Create it if it doesn't already exist)

	cd /usr/src/quantonium
    
    This will likely need superuser privileges.  Make sure you have 
    permission to create files in this directory.


2. Clone this git repository.  

	git clone https://github.com/quantonium/ergw_tarball.git

    It will create ergw_tarball directory.  

	cd ergw_tarball

    It should download following files:

	rebar3- rebar3 binary to manage Erlang builds and packages  
	esl-erlang_20.1-1\~ubuntu\~xenial_amd64.deb - Erlang 20.1 Ubuntu 16.04 package  
	vrf.script - Script to configure VRFs   
	ergw-gtp-c-node.config - erGW config file
	README.md - This file containing instructions to configure and run erGW  
	ergw0424.tar - A tar archive of build and run time environment of erGW.


3. Copy rebar3 from current directory to a directory that is in your PATH (~/bin as an example)

	cp rebar3 ~/bin

4. Bring your Ubuntu system up to date with latest package updates

	sudo apt update  
	sudo apt upgrade  


5. Install Erlang v20.1 The Ubuntu  package file for Erlang 20.1 is included as part 
of this repository.  Originally it is 
from https://packages.erlang-solutions.com/erlang/ for 
Ubuntu 16.04.  These three steps below need to be executed in sequence, without any 
other package related operation on the system, so that apt can pull in all 
Erlang package dependencies. Ignore any error messages from the first step relating to 
package dependencies.

	sudo dpkg -i esl-erlang_20.1-1\~ubuntu\~xenial_amd64.deb  
	sudo apt -f install  
	sudo dpkg -i esl-erlang_20.1-1\~ubuntu\~xenial_amd64.deb 

6. Untar ergw.tar file in current directory

	tar xvf ergw0424.tar  
	cd gw  

7. This step is not required unless you are changing erGW source file(s)

	rebar3 release

8. This repository has pre-built erGW binaries.  Copy the relevent erGW files 
to system directory

	sudo cp -aL _build/default/rel/ergw-gtp-c-node /opt


9. Create VRF device to accept traffic from MME by executing the script vrf.script present in the parent directory.  Run this script with three parameters.  The first parameter 
should be the NIC name (like ens4) through which erGW will talk to MME.  This NIC will become
slave to VRF device that will be created as part of this step.  The 
second parameter is the IP address of the NIC (ens4 as an example). The third 
parameter is the network gateway through which traffic between MME and erGW will flow.

	../vrf_script < NIC identifier > < IP address of NIC > < Gateway IP Address >


Ensure that vrf-irx device is configured with the desired IP address by 
running the following command:
	
	ifconfig vrf-irx

Workaround VRF kernel bug by assigning the vrf-irx address to the NIC that you used to login

	sudo ip -6 addr add <IPv6 vrf-irx address like fd00:4888:2000:2062::121> dev <IP address that you used to login, e.g. IP addrss on ens3>


10. Ensure the routes to MME and eNodeB are correctly set on your system

	sudo ip route add  \<MME_ADDRESS like fd00:4888:2000:2051:524:23::1F48/128\> via \<GW ADDRESS like FD00:4888:2000:2062:524:23:0:2\> dev \<NIC facing MME like ens4\>

	sudo ip route add  \<eNodeB_ADDRESS like fd00:4888:2000:2051:524:23::1F48/128\> via \<GW ADDRESS like FD00:4888:2000:2062:524:23:0:2\> dev \<NIC facing eNodeB like ens4\>


11. Copy the erGW config file that came with this repository

	sudo mkdir /etc/ergw-gtp-c-node

	sudo cp ../ergw-gtp-c-node.config /etc/ergw-gtp-c-node/ergw-gtp-c-node.config

   This above file will need changes based on your network topology.  Following 
   are the places that will need to change to get erGW up and running:


	{sockets,
          [{epc, [{type, 'gtp-c'},
                  {ip, {16#FD00,16#4888,16#2000,16#2062,0,0,0,16#121}},
                  {netdev, "vrf-irx"}
                 ]}
          ]},

   The line that contains "ip" should have the ipv6 address that was assigned 
   to vrf-irx earlier.  The values are comma separated and 16# indicates the values are base 16.


      {vrfs,  
          [{sgi, [{pools,  [{{16#2222, 0, 0, 0, 16#1, 0, 0, 0}, {16#2222, 0, 0, 0, 16#1, 0, 0, 16#7}, 128}]},  
                  {'MS-Primary-DNS-Server', {8,8,8,8}},  
                  {'MS-Secondary-DNS-Server', {8,8,4,4}},  
                  {'MS-Primary-NBNS-Server', {127,0,0,1}},  
                  {'MS-Secondary-NBNS-Server', {127,0,0,1}}  
                 ]}   
          ]},  

    The line that contains "sgi" "pools" has the range of ip addresses that will be 
    assigned to UE.  It should be updated appropriately.


 	%% A/AAAA record alternatives  
              {"topon.s1u.saegw.$ORIGIN", [{16#FD00,16#4888,16#2000,16#2051,16#524, 16#23,0,16#1F47}], []},  
              {"topon.sx.saegw01.$ORIGIN", [{192,168,1,1}], []}  

    The line that contains "topon.s1u.saegw" should be updated to the IP address 
    of eNodeB.  This is the S1U interface.

    The line with "topon.sx.saegw01" should have the ip address of VPP (Sx interface).  This is
    internal communication between erGW and VPP and likely to not change.


12. Copy the two ergw system config files to erGW run time location
	sudo cp config/sys.config /opt/ergw-gtp-c-node/releases/1.16.0/
	sudo cp config/vm.args /opt/ergw-gtp-c-node/releases/1.16.0/

13. Start the erGW

	sudo /opt/ergw-gtp-c-node/bin/ergw-gtp-c-node foreground


14. Run the following command for checking if erGW has successfully opened the 
  ports for communication with MME and VPP

	sudo ss -anup \( sport = 2123 or sport = 8805 \)

    2123 port on vrf-irx device is used for communication with MME.  8805 is used to coomunicate with VPP.
