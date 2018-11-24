=============================
Enable Neutron VPN as Service
=============================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/openstack-chef/+spec/neutron-vpnaas-enablement

Problem description
===================

VPN service is a key feature provided by Neutron to enable Secured Private
connection to OpenStack cloud. The reference VPN implementation in Neutron at
this time is [IPSec]_ VPN, and the IPSec driver is [OPENSWAN]_.

Currently, there is no Chef cookbook support to configure and start Neutron
VPN service.


Proposed change
===============

Add a recipe, and related attribute/unit tests to cookbook-openstack-network
to install, configure and start VPN service.

The packages need to be installed are:

* Ubuntu: neutron-plugin-vpn-agent

* RedHat: openstack-neutron

* Suse:  openstack-neutron-vpn-agent

The attributes need to be added includes:

* A new attribute to decide if VPN agent or L3 agent should be started::

       ['openstack']['network']['enable_vpn']

* New attributes for VPN configurations in /etc/neutron/vpn_agent.ini::

       ['openstack']['network']['vpn']['vpn_device_driver']
       ['openstack']['network']['vpn']['ipsec_status_check_interval']

The recipe will check if VPN service should be installed, configured and
started, then uses the attribute value to configure VPN agent configuration
file and start VPN service.

Alternatives
------------

No alternatives at this time.

Data model impact
-----------------

No Data model impact

REST API impact
---------------

No API change

Security impact
---------------

Right now only IKE with "PSK" (Pre-Shared Key) authentication mode is implemented
in Neutron VPNaaS for simplicity. And the psk is a input of IPsec site connection
establishment process.

For example, "secret" psk can be used in a new IPsec site connection::

      neutron ipsec-site-connection-create --name vpnconnection1 --vpnservice-id myvpn
      --ikepolicy-id ikepolicy1 --ipsecpolicy-id ipsecpolicy1 --peer-address 172.24.4.233
      --peer-id 172.24.4.233 --peer-cidr 10.2.0.0/24 --psk secret

Since the authentication and key exchange are not in the scope of starting and
configuring VPN service, there should be no security impact of this Spec.

Notifications impact
--------------------

No notification impact

Other end user impact
---------------------


Performance Impact
------------------


Other deployer impact
---------------------


Developer impact
----------------



Implementation
==============

Assignee(s)
-----------

Primary assignee:
  - gekun@cn.ibm.com


Work Items
----------

* Add a new attribute value to decide if VPN agent or L3 agent should be started,
  since these two services cannot be started at the same time.

* Add new attributes for VPN configurations in /etc/neutron/vpn_agent.ini and
  a new vpn template.

* Add a new recipe to install the VPN packages, configure the [VPN_TEMPLATE]_
  and start VPN service

* Enable VPN support in Horizon [VPN_HORIZON]_
  Configure /opt/stack/horizon/openstack_dashboard/local/local_settings.py::

       OPENSTACK_NEUTRON_NETWORK = {
            'enable_vpn': True,
       }

* Add validations to ensure VPN service is up and running correctly.

* Add unit tests

* ref to ask openstack on this topic [HOW_TO_SETUP_VPN]_


Dependencies
============

Testing
=======

* Add unit tests for the new recipe.

* For function tests and CI integration tests, at least one node with three NICs is
  recommanded. One NIC is used for external network connection, one NIC is used for
  data network and the other is used for management network.

Documentation Impact
====================

* Configure attribute ['openstack']['network']['enable_vpn'] = 'True'
  to enable VPN service.

* Configure VPN related attributes, for example::

      ['openstack']['network']['vpn']['vpn_device_driver'] =
      neutron.services.vpn.device_drivers.ipsec.OpenSwanDriver
      ['openstack']['network']['vpn']['ipsec_status_check_interval'] = 60

References
==========
.. [IPSec] `Security Architecture for the Internet Protocol RFC
   <http://tools.ietf.org/html/rfc4301>`_

.. [OPENSWAN] `OpenSwan website
   <https://www.openswan.org>`_

.. [VPN_TEMPLATE] `VPN template values
   <https://docs.openstack.org/trunk/config-reference/content/networking-options-vpn.html>`_

.. [VPN_HORIZON] `VPN support in Horizon <https://review.openstack.org/#/c/108493/1>`_

.. [HOW_TO_SETUP_VPN] `How to setup VPNaaS
   <https://ask.openstack.org/en/question/6243/how-to-set-up-neutron-vpn-service-vpnaas/>`_