erGW (GTP-U) local build and run environment

- Copy rebar3 from current directory to a directory that is your PATH (~/bin as an example)

	cp rebar3 ~/bin

- Download and install Erlang v20.1 from https://packages.erlang-solutions.com/erlang/ for Ubuntu

- Create VRF to accept traffic from MME by executing the script vrf.script in current directory

	IP addresses in this script file might need to change. In the example below, ens4 NIC will be interfacing with MME.  It has address fd00:4888:2000:2062::121 The last line of the script has the gateway address through which VRF will talk to MME.


  	sudo ip link add vrf-irx type vrf table 10  

  	sudo ip link set dev vrf-irx up  

  	sudo ip link set dev ens4 master vrf-irx  
	
  	sudo ip link set dev ens4 up  
	
  	sudo ip -6 addr add fd00:4888:2000:2062::121 dev vrf-irx  

  	sudo ip -6 route add table 10 default via FD00:4888:2000:2062:524:23:0:2   


- Go to your working directory  

	mkdir work; cd work;

- Untar ergw.tar file in your work directory

	tar xvf ergw.tar

- In case you want to rebuild with additional changes:

	rebar3 release

- Copy the relevent files to system directories

	sudo cp -aL _build/default/rel/ergw-gtp-c-node /opt

	sudo mkdir /etc/ergw-gtp-c-node

- Copy the config file that came with the package

	sudo cp ergw-gtp-c-node.config /etc/ergw-gtp-c-node/ergw-gtp-c-node.config

  This above file might need changes based on your network topology.

- Start the erGW

	sudo /opt/ergw-gtp-c-node/bin/ergw-gtp-c-node foreground


