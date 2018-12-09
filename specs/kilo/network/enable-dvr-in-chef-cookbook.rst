=======================================================
Enable Distributed Virtual Router(DVR) in chef cookbook
=======================================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/openstack-chef/+spec/enable-dvr-chef-cookbook

Problem description
===================

Currently DVR is disabled by default in Neutron and not allowed to be
configured in Network cookbook. After deployed, user has to manually modify
the Neutron configuration files to enable DVR.


Proposed change
===============

The following attribute file in cookbook-openstack-network will be modified:
* default.rb
We will add attribute ['openstack']['network']['router_distributed'] in it.
User can set this attribute to 'auto', true and false. When this attribute is
set to 'auto', chef cookbook will do enough check, like checking whether
network type ML2 extensions support DVR, checking whether OVS is enabled,
after that chef cookbook will enable DVR or output warning messages and logs
to tell user what happened. And considering only GRE and VXLAN network types
support DVR, router_distributed's true and false setting will only work in
the two network types. To VLAN network type, DVR will be disabled by default
even router_distributed is set to true, warning messages will be given to
user to notify why DVR config doesn't work.

The following template files in cookbook-openstack-network will be modified:
* neutron.conf.erb
* l3_agent.ini.erb
* ovs_neutron_plugin.ini.erb
* ml2_conf.ini.erb
Modify attribute 'router_distributed' in neutron.conf.erb, 'agent_mode' in
l3_agent.ini.erb, 'enable_distributed_routing' and 'l2_population' in
ovs_neutron_plugin.ini.erb, 'mechanism_drivers' in ml2_conf.ini.erb. These
attributes can be found in the howto link in the following References section.

The following recipe files in cookbook-openstack-network may be modified:
* l3_agent.rb
DVR gives a new data path for vms, like East-West communication, give
compute nodes external IPs to make vms can get floating IPs not only from
network nodes. And DVR will only work on nodes which has L3 agent and OVS
agent, and these will installed for network node role and compute node role.
l3_agent.ini.erb will need query the current node is compute node or network
node when DVR is enabled. We will use existing network node role and compute
node role to deal with that. If a node have both the two roles, we will
consider this node as network node.

If necessary we also need methods to make sure necessary packages
like iproute are installed on compute node.

DVR is supported by network type GRE and VXLAN, but not VLAN yet, so
we also need a method to make sure the current network type is either GRE
or VXLAN, the network type need maps to key name tunnel_types in
ovs_neutron_plugin.ini with values of gre or vxlan. If current network type
is VLAN, we should stop the configuration of DVR. And we also need methods
to make sure necessary
network resource like tunnel network bridge are created on compute node.

If necessary we will change the role definition for compute node.

We did test and enabled DVR on Redhat and Ubuntu, but not all versions have
been tested. So in cookbook, we will deal with details from different
platforms and releases affected and output warning messages and logs to OS
we will not support.

Alternatives
------------

Another option to case that DVR is enabled while tunnel_types is vlan,
is that we can cover that value by gre or vxlan for tunnel_type in
ovs_neutron_plugin.ini. Consider that if user decides to enable DVR,
user can accept changing in openvswitch agent config file.

Data model impact
-----------------

REST API impact
---------------


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <lzklibj@cn.ibm.com>

Other contributors:

Work Items
----------


Dependencies
============


Testing
=======

Add attribute 'router_distributed' => 'true' in environment file,
then deploy a 1+N environment or a multiple network nodes environment.
(All-in-one case is unnecessary, we can consider it similar to 1+0 case)
Check if config files are modified according to the list in the wiki Neutron
DVR HowTo page.

Build network N1 and N2, router R1, add subnets of N1 and N2 to R1 as
interfaces, before booting any instances, we should see nothing listed in the
output when running "ip netns" on the compute nodes. Boot instances on N1 and
N2 on different compute nodes, we should see network namespace on those compute
nodes by running "ip netns".

ip-netns is process network namesapce management command. You can run
"ip netns help" to get more usage. And "ip netns" is short for
"ip netns list", it will show all of the named network namespaces, which
are under /var/run/netns.

Also, we can ping from vm to vm while checking output by running tcpdump
from compute nodes. If we boot vm1 on N1 on CN1(compute node 1), and vm2
on N2 on CN2, after we logon vm1(it doesn't matter we logon from network
node or CN1), we can ping vm2 and run 'tcpdump | grep -i "X"' on CN1 or CN2,
while "X" is your network type, we will find ICMP packages data path is
directly from CN1 to CN2, without passing network nodes (in a 1+N case, ICMP
packages will need centralized network node to transmit when DVR is disabled).


Documentation Impact
====================

* User can set ['openstack']['network']['router_distributed'] to 'auto' to
  let chef cookbook configure for DVR aumotically, enable DVR or give warning
  mesaages.
* DVR will be enabled by default when network type is GRE or VXLAN,
  user can set ['openstack']['network']['router_distributed]' to 'false'
  in override_attributes to disable it.
* When set ['openstack']['network']['router_distributed'] to 'true', user
  should check follow attributes to enable DVR: check ['openstack']['network']
  ['core_plugin'] has value 'neutron.plugins.ml2.plugin.ML2Plugin', check
  ['openstack']['network']['ml2']['mechanism_drivers'] has value 'openvswitch'
  and check ['openstack']['compute']['network']['plugins'] has value
  'openvswitch'.

References
==========

<https://wiki.openstack.org/wiki/Neutron/DVR>
<https://wiki.openstack.org/wiki/Neutron/DVR/HowTo>
