erGW (GTP-U) local build and run environment. This environment is only tested for Ubuntu 16.04.  

- Go to /usr/src/quantonium (Create it if it doesn't already exist)

	cd /usr/src/quantonium


- Clone this git repository.  It should download following files

	rebar3- rebar3 binary to manage Erlang builds and packages  
	esl-erlang_20.1-1~ubuntu~xenial_amd64.deb - Erlang 20.1 Ubuntu 16.04 package
	vrf.script - Script to configure VRFs 
	README.md - This README file containing instructions on how to configure and run erGW 

- Copy rebar3 from current directory to a directory that is in your PATH (~/bin as an example)

	cp rebar3 ~/bin

- Bring the system upto date with latest package updates

	sudo apt update  
	sudo apt upgrade  


- Install Erlang v20.1 The package file is included as part of this 
repository.  Originally it is from https://packages.erlang-solutions.com/erlang/ for 
Ubuntu 16.04.  These three steps below need to be executed in sequence, without any 
other package related operation on the system, so that apt can pull in all 
Erlang package dependencies.

	sudo dpkg -i esl-erlang_20.1-1~ubuntu~xenial_amd64.deb  
	sudo apt -f install  
	sudo dpkg -i esl-erlang_20.1-1~ubuntu~xenial_amd64.deb 

- Untar ergw.tar file 

	tar xvf ergw.tar

- This step is not required unless you are changing erGW source file(s)

	rebar3 release

- This repository has pre-built erGW binaries.  Copy the relevent erGW files to system directories

	sudo cp -aL _build/default/rel/ergw-gtp-c-node /opt

- Copy the config file that came with the package

	sudo mkdir /etc/ergw-gtp-c-node

	sudo cp ergw-gtp-c-node.config /etc/ergw-gtp-c-node/ergw-gtp-c-node.config

  This above file might need changes based on your network topology.

- Create VRF to accept traffic from MME by executing the script vrf.script in 
current directory.  Run this script with three parameters.  The first parameter 
should be the NIC name (like ens4) through which erGW will talk to MME.  The 
second parameter is the IP address of the NIC (ens4 as an example). The third 
parameter is the network gateway through which traffic between MME and erGW will flow.

	./vrf_script <NIC identifier> <IP address of NIC> < Gateway IP Address>

- Start the erGW

	sudo /opt/ergw-gtp-c-node/bin/ergw-gtp-c-node foreground


