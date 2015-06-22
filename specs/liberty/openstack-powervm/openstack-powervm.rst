..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================
Configure openstack services with PowerVM support
=================================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/openstack-chef/+spec/openstack-powervm

IBM PowerVM is a hypervisor that the POWER platform supports.
PowerVM admins can see benefits in their environments by making use of OpenStack.
The nova driver (along with a Neutron ML2 compatible agent and ceilometer
agent) will provide capability for admins of PowerVM to use OpenStack natively.
The PowerVM drivers are opensource and currently being worked in the StackForge
community.
For Nova, the PowerVM compute driver is on the openstack base incubation track.
For Neutron the driver will follow the BYOD model set forth in the Neutron extension
decomposition. There is a blueprint for supporting a PowerVM compute
inspector in Ceilometer at least for the L release.
In a later release it's been expressed that the Ceilometer compute notifications model may change.
For PowerVM systems this ML2 agent would replace the default openvswitch agent for compute nodes
with the PowerVM SEA ML2 agent.


Refer [OPENSTACK_NOVA_POWERVM]_.
Refer [OPENSTACK_NEUTRON_POWERVM]_.
Refer [OPENSTACK_CEILOMETER_POWERVM]_.
Refer [POWERVM_CEILOMETER_COMPUTE]_.

This new change proposed will enable deployment and configuration of the
PowerVM Nova compute driver, Neutron ML2 agent and Ceilometer compute inspector.
Similar to VMWare, this is an addition to support another type of hypervisor.

Problem description
===================

* Currently, cookbook-openstack-compute does not support the deployment and
  configuration of the PowerVM Nova compute driver.
* Currently, cookbook-openstack-network does not support the deployment and
  configuration of the PowerVM Neutron ML2 agent.
* Currently, cookbook-openstack-telemetry does not support the deployment and
  configuration of the PowerVM Ceilometer compute inspector.

Proposed change
===============

Add support in cookbook-openstack-* cookbooks to configure the PowerVM Nova
compute driver, Neutron ML2 agent and Ceilometer compute inspector.

* A new configuration attribute will be added in order to deploy PowerVM drivers
* If the new attribute is enabled, it will auto set other attributes and
  automatically include the PowerVM recipes
* By default, PowerVM drivers will be downloaded from source code on Stackforge
* A new configuration attribute will allow to download from either source code
  or public package repository
* A new configuration attribute will allow to override the package repository url


Alternatives
------------

* User manually downloads code from Stackforge and deploy/configure the PowerVM
  Nova compute driver, Neutron ML2 agent and Ceilometer compute inspector.
* User extends the existing OpenStack Puppet modules to deploy and configure
  the PowerVM Nova compute driver, Neutron ML2 agent and Ceilometer compute inspector.

Refer [OPENSTACK_PUPPET_NOVA]_.
Refer [OPENSTACK_PUPPET_NEUTRON]_.
Refer [OPENSTACK_PUPPET_CEILOMETER]_.

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

* Deployment of PowerVM drivers has to be explicitly enabled
* As of now, we're considering the use of configuration attributes rather than
  roles to deploy PowerVM drivers since its usage is not considered generic yet.
* The deployer will be able to deploy the PowerVM drivers from different sources
  (github, public or private package repositories) by overidding
  configuration attributes.

Developer impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  dclain

Other contributors:
  thorst

Work Items
----------

* Work with the PowerVM Driver team to fully understand all of the
  configuration options
* Add new attributes to openstack-compute, openstack-network,
  openstack-telemetry to support PowerVM
* Change openstack-compute / nova.conf.erb  template for including
  configuration specific to the PowerVM Nova compute driver
* Change openstack-compute / rootwrap.conf.erb template for including filters
  specific to the PowerVM Nova compute driver
* Add new recipe for PowerVM configuration in openstack-compute
* Change openstack-net work / ml2_conf.ini.erb in openstack-network for
  including configuration specific to the PowerVM Neutron ML2 agent
* Add new recipe for the PowerVM Neutron ML2 agent configuration in
  openstack-network
* Add new recipe for the PowerVM Ceilometer inspector configuration in
  openstack-telemetry
* Add Unit Tests for each new recipe
* Extend openstack-chef-repo to test all-in-one PowerVM nova-network


Dependencies
============

* TBD

Testing
=======

* Add unit tests for the recipes
* Add new test, environment to support all-in-one PowerVM nova compute using
  openstack-chef-repo
* We will report our function and CI integration test results (using
  openstack-chef-repo) back to the Chef team.


Documentation Impact
====================

* Update README.md in openstack-compute, openstack-network, openstack-telemetry
  cookbooks to expose the PowerVM configuration attributes and how to enable it
* Update README.md in openstack-chef-repo cookbook to explain
* Add documentation in openstack-chef-repo/doc to explain how to test a PowerVM
  specific all-in-one compute configuration


References
==========

.. [OPENSTACK_NOVA_POWERVM] `PowerVM driver for OpenStack Nova compute driver
   <https://github.com/stackforge/nova-powervm>`_

.. [OPENSTACK_NEUTRON_POWERVM] `PowerVM driver for OpenStack Neutron ML2 agent
   <https://github.com/stackforge/neutron-powervm>`_

.. [OPENSTACK_CEILOMETER_POWERVM] `PowerVM driver for OpenStack Ceilometer
  compute inspector <https://github.com/stackforge/ceilometer-powervm>`_

.. [OPENSTACK_PUPPET_NOVA] `OpenStack Nova Puppet Module
    <https://github.com/openstack/puppet-nova>`_

.. [OPENSTACK_PUPPET_NEUTRON] `OpenStack Neutron Puppet Module
    <https://github.com/openstack/puppet-neutron>`_

.. [OPENSTACK_PUPPET_CEILOMETER] `OpenStack Ceilometer Puppet Module
    <https://github.com/openstack/puppet-ceilometer>`_

.. [POWERVM_CEILOMETER_COMPUTE] `PowerVM Ceilometer Compute Launchpad
    <https://launchpad.net/ceilometer-powervm>`_


