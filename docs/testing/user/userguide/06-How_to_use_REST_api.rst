.. This work is licensed under a creative commons attribution 4.0 international
.. license.
.. http://creativecommons.org/licenses/by/4.0
.. (c) opnfv, national center of scientific research "demokritos" and others.

========================================================
REST API - Readme
========================================================

Introduction
===============
As the internet industry progresses creating REST API becomes more concrete
with emerging best Practices. RESTful web services don’t follow a prescribed
standard except fpr the protocol that is used which is HTTP, its important
to build RESTful API in accordance with industry best practices to ease
development & increase client adoption.

In REST Architecture everything is a resource. RESTful web services are light
weight, highly scalable and maintainable and are very commonly used to
create APIs for web-based applications.

Here are important points to be considered:

 * GET operations are read only and are safe.
 * PUT and DELETE operations are idempotent means their result will
   always same no matter how many times these operations are invoked.
 * PUT and POST operation are nearly same with the difference lying
   only in the result where PUT operation is idempotent and POST
    operation can cause different result.


REST API in SampleVNF
=====================

In SampleVNF project VNF’s are run under different contexts like BareMetal,
SRIOV, OVS & Openstack etc. It becomes difficult to interact with the
VNF’s using the command line interface provided by the VNF’s currently.

Hence there is a need to provide a web interface to the VNF’s running in
different environments through the REST api’s. REST can be used to modify
or view resources on the server without performing any server-side
operations.

REST api on VNF’s will help adapting with the new automation techniques
being adapted in yardstick.

Web server integration with VNF’s
==================================

In order to implement REST api’s in VNF one of the first task is to
identify a simple web server that needs to be integrated with VNF’s.
For this purpose “civetweb” is identified as the web server That will
be integrated with the VNF application.

CivetWeb is an easy to use, powerful, C/C++ embeddable web server with
optional CGI, SSL and Lua support. CivetWeb can be used by developers
as a library, to add web server functionality to an existing application.

Civetweb is a project forked out of Mongoose. CivetWeb uses an [MITlicense].
It can also be used by end users as a stand-alone web server. It is available
as single executable, no installation is required.

In our project we will be integrating civetweb into each of our VNF’s.
Civetweb exposes a few functions which are used to resgister custom handlers
for different URI’s that are implemented.
Typical usage is shown below


VNF Application init()
=========================

Initialize the civetweb library
================================

mg_init_library(0);

Start the web server
=====================
ctx = mg_start(NULL, 0, options);


Once the civetweb server is started we can register our URI’s as show below
mg_set_request_handler(ctx, "/config", static_cfg_handler, 0);

In the above example “/config” is the URI & static_cfg_handler() is
the handler that gets called when a user invokes this URI through
the HTTP client. API's have been mostly implemented for existing VNF's
like vCGNAPT, vFW & vACL. you might want to implement custom handlers
for your VNF.

URI definition for different VNF’s
===================================

::

URI	   				REST Method	     Arguments			Description
===========================================================================================================================
/vnf          				GET  		       	None           		Displays top level methods available

/vnf/config   				GET         		None           		Displays the current config set
					POST        		pci_white_list: 	Command success/failure
								num_worker(o):
								vnf_type(o):
								pkt_type (o):
								num_lb(o):
								sw_lb(o):
								sock_in(o):
								hyperthread(o) :

/vnf/config/arp				GET			None			Displays ARP/ND info
					POST			action: <add/del/req>	Command success/failure
								ipv4/ipv6: <address>
								portid: <>
								macaddr: <> for add

/vnf/config/link			GET			None
					POST			link_id:<>		Command success/failure
								state: <1/0>

/vnf/config/link/<link id>		GET			None
					POST						Command success/failure
								ipv4/ipv6: <address>
								depth: <>

/vnf/config/route			GET			None			Displays gateway route entries
					POST			portid: <>		Adds route entries for default gateway
								nhipv4/nhipv6: <addr>
								depth: <>
								type:"net/host"

/vnf/config/rules(vFW/vACL only)	GET			None			Displays the methods /load/clear
/vnf/config/rules/load			GET			None			Displays if file was loaded
					PUT			<script file
								with cmds>		Executes each command from script file
/vnf/config/rules/clear			GET			None			Command success/failure clear the stat

/vnf/config/nat(vCGNAPT only)		GET			None			Displays the methods /load/clear
/vnf/config/nat/load			GET			None			Displays if file was loaded
					PUT			<script file
								with commands>		Executes each command from script file

/vnf/config/nat/clear			GET			None			Command success/failure clear the stats
/vnf/log				GET			None			This needs to be implemented for each VNF
											just keeping this as placeholder.

/vnf/dbg				GET			None			Will display methods supported like /pipelines/cmd
/vnf/dbg/pipelines			GET			None			Displays pipeline information(names)
											of each pipelines
/vnf/dbg/pipelines/<pipe id>		GET			None			Displays debug level for particular pipeline

/vnf/dbg/cmd				GET			None			Last executed command parameters
					POST			cmd:			Command success/failure
								dbg:
								d1:
								d2:

API Usage
===============

1. Initialization
================

In order to integrate to your VNF these are the steps required

In your VNF application init


#ifdef REST_API_SUPPORT
        Initialize the rest api
        struct mg_context *ctx = rest_api_init(&app);
#endif


#ifdef REST_API_SUPPORT
        rest api's for cgnapt
        rest_api_<vnf>_init(ctx, &app);
#endif


void rest_api_<vnf>_init(struct mg_context *ctx, struct app_params *app)
{
        myapp = app;

	VNF specific command registration
        mg_set_request_handler(,,,);

}


2. Run time Usage
====================

An application(say vFW) with REST API support is run as follows
with just PORT MASK as input. The following environment variables
need to be set before launching the application(To be run from
samplevnf directory).

export VNF_CORE=`pwd`
export RTE_SDK=`pwd`/dpdk-16.04
export RTE_TARGET=x86_64-native-linuxapp-gcc

./build/vFW -p 0x3 (Without the -f & -s option)

1. When VNF(vCGNAPT/vACL/vFW) is launched it waits for user to provide the
/vnf/config REST method. A typical curl command if used will look like below
shown. This with minimal parameter. For more options please refer to above REST
methods table.

e.g curl -X POST -H "Content-Type:application/json" -d '{"pci_white_list": "0000:08:00.0
 0000:08:00.1"}' http://<IP>/vnf/config

Note: the config is mostly implemented based on existing VNF's. if new parameters
are required in the config we need to add that as part of the vnf_template.

Once the config is provided the application gets launched.

Note for CGNAPT we can add public_ip_port_range as follows, the following e.g gives
a multiport configuration with 4 ports, 2 load balancers, worker threads 10, multiple
public_ip_port_range being added, please note the "/" being used to seperate multiple
inputs for public_ip_port_range.

e.g curl -X POST -H "Content-Type:application/json" -d '{"pci_white_list": "0000:05:00.0 0000:05:00.2 0000:07:00.0 0000:07:00.2",
		 "num_lb":"2", "num_worker":"10","public_ip_port_range_0": "04040000:(1, 65535)/04040001:(1, 65535)",
		 "public_ip_port_range_1": "05050000:(1, 65535)/05050001:(1, 65535)" }' http://10.223.197.179/vnf/config

2. Check the Link IP's using the REST API
e.g curl <IP>/vnf/config/link

This would indicate the number of links enabled. You should enable all the links
by using following curl command for links 0 & 1

e.g curl -X POST -H "Content-Type:application/json" -d '{"linkid": "0", "state": "1"}'
http://<IP>/vnf/config/link
curl -X POST -H "Content-Type:application/json" -d '{"linkid": "1", "state": "1"}'
http://<IP>/vnf/config/link

3. Now that links are enabled we can configure IP's using link method as follows

e.g  curl -X POST -H "Content-Type:application/json" -d '{"ipv4":"<IP to be configured>","depth":"24"}'
http://<IP>/vnf/config/link/0
curl -X POST -H "Content-Type:application/json" -d '{"ipv4":"IP to be configured","depth":"24"}'
http://<IP>/vnf/config/link/1

Once the IP's are set in place time to add NHIP for ARP Table. This is done using for all the ports
required.
/vnf/config/route

curl -X POST -H "Content-Type:application/json" -d '{"portid":"0", "nhipv4":"IPV4 address",
 "depth":"8", "type":"net"}' http://<IP>/vnf/config/route


4. For Firewall/ACL in order to load the rules a script file needs to be posted
using a script.
/vnf/config/rules/load

Typical example for loading a script file is shown below
curl -X PUT -F 'image=@<path to file>' http://<IP>/vnf/config/rules/load

vCGNAPT can use the following REST api's for runtime configuring through a script
/vnf/config/rules/clear
/vnf/config/nat(vCGNAPT only)
/vnf/config/nat/load

For debug purpose following REST API's could be used as described above.

/vnf/dbg
/vnf/dbg/pipelines
/vnf/dbg/pipelines/<pipe id>
/vnf/dbg/cmd

5. For stats we can use the following method

/vnf/stats
e.g curl <IP>/vnf/stats

6. For quittiong the application
/vnf/quit

e.g curl <IP>/vnf/quit

