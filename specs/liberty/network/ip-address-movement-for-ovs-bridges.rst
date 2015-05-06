===================================
IP address movement for ovs bridges
===================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/openstack-chef/+spec/ip-movement

Problem description
===================

Neutron openvswitch plugin uses ovs bridges to manage network stream. Recipe
openvswitch, l3_agent, and vpn_agent use ovs commands to create those bridges
and will plug the corresponding NICs to each of them. But those recipes do not
move the NICs' IP addresses to the bridges, which will lead to the NIC IP address
can not be accessed once finish the bridges creation.


Proposed change
===============

Add recipes to move IP address from NIC to bridge. Make sure those IP addresses
can also be accessed after finish the bridge creation.

Three attributes will be added, including:

* ['openstack']['network']['ip_movement']['enable']: Boolean attribute to decide
  whether IP movement should be enabled or not. Default to true.

* ['openstack']['network']['ip-movement']['timeout']: Integer attribute to decide
  the timeout of IP movement. IP movement uses 'service network restart' (for fedora
  platform) and 'ifdown', 'ifup' (for ubuntu platform)  commands to let network
  configurations take effect. This attribute decides how long the node should wait
  for the network to become operational again after executing the previous commands.
  And it default to 180.

* ['openstack']['network']['ip-movement']['validate-ip']: String of IP address pinged
  by ip movement in the of its operations to check whether the ip movement is succeed.
  Default to nil. Means this ip movement will use default gateway to check its status.

Two libraries will be added, including:

* ip-movement-fedora: This library includes the fedora platform IP movement configurations.

* ip-movement-ubuntu: This library includes the ubuntu platform IP movement configurations.

One resource will be added, including:

* ovs_bridge: This resource will create the ovs bridge and move the ip address to the
  corresponding bridge depend on the resource attributes.

openvswitch and l3-agent recipes will changed to use this new resource create ovs bridge.

Alternatives
------------

Like devstack, using ovs commands and ip commands to flush the ip address in NIC,
and add ip address to ovs bridge can also move the ip address to the bridges. But
this is a temporary move. After network restart or reboot the OS, the ip address
will go back to NIC. We need make sure this movement is persistent by write those
configurations to network configuration files.

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

None

Notifications impact
--------------------

None

Other end user impact
---------------------

None

Performance Impact
------------------

None

Other deployer impact
---------------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  - hwhu@cn.ibm.com

Other contributors:
  - jckasper@us.ibm.com
  - lzklibj@cn.ibm.com

Work Items
----------

* Add new attributes to openstack-network.

* Add ovs_bridge resource and provider.

* Add ip-movement-fedora library to handle fedora platform IP movement configurations.

* Add ip-movement-ubuntu library to configure ubuntu platform IP movement configurations.

* Add the unit tests.

* Change openvswitch and l3-agent recipes use ovs_bridge resource to create ovs bridge.

Dependencies
============

* IP movement use 'service network restart' , 'ifdown' and 'ifup' to let network
  configurations take effect. Those scripts are contained in initscripts packege.
  This cookbook will install it.


Testing
=======

* Add unit tests for the new recipes.

* For function tests and CI integration tests, at least an all-in-one openstack
  configured with neutron openvswitch plugin and enabled l3 service is needed.


Documentation Impact
====================

* Configure attribute ['openstack']['network']['ip-movement']['enable'] = 'True'
  to enable IP movement.

* Using ['openstack']['network']['ip-movement']['timeout'] to configure the timeout
  of IP movement. This attribute decides how long the node should wait for the network
  to become operational again after executing the previous commands.

* IP movement default to use default gateway to check its network connections in the
  end of its operations. Assign an ip address to ['openstack']['network']['ip-movement']
  ['validate-ip'] attribute to use another ip address instead.


References
==========

* `Red Hat network scripts integration <https://github.com/osrg/openvswitch/blob/mpls-nx/rhel/README.RHEL>`_

* `README.Debian for openvswitch-switch <https://github.com/horms/openvswitch/blob/devel/add-group-help-fix/debian/openvswitch-switch.README.Debian>`_
