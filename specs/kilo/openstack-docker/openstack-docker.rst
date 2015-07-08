=================================================
Configure openstack services with docker support
=================================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/openstack-chef/+spec/docker-driver-configuration

The Docker driver is a hypervisor driver for OpenStack Nova Compute. It was introduced with the Havana release.
Docker is an open-source engine which automates the deployment of applications as highly portable, self-sufficient
containers which are independent of hardware, language, framework, packaging system and hosting provider.

Refer [OPENSTACK_DOCKER_DOCUMENTATION]_.

This new change proposed will enable deployment and configuration of nova-docker driver, glance repo configuration
and any needed config to support seemless managment of docker nodes in openstack cloud.


Problem description
===================

* Currently, openstack-compute does not support nova-docker driver

* Currently, openstack-image does not support docker container formats
  for images which includes docker and dockerref

* [OPENSTACK_DOCKER_COOKBOOK_OLD]_ is an available
  option which is not maintained for last 2 years and it has embedded
  2 years old driver.


Proposed change
===============

Add support in openstack-compute cookbook to configure [NOVA_DOCKER_DRIVER]_.
Current support will be to download the nova docker driver from git repo and configure.

Also change openstack-image to support container formats for docker images
which includes docker and dockerref

Alternatives
------------

None

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

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  - skairali@cn.ibm.com

Other contributors:
  - ashish.billore1@in.ibm.com

Work Items
----------

* Add new attributes to openstack-compute

* Change openstack-compute / nova.conf.erb  template for including nova-docker driver

* Add new recipe for docker configuration in openstack-compute

* Change compute.rb  recipe in openstack-compute to include the new recipe based on configuration

* Add the unit tests.

* Change openstack-image and add new container formats in attributes


Dependencies
============

* In order to configure nova-docker driver - compute nodes should be pre installed with
  docker runtime. Users can opt to use cookbook [CHEF_DOCKER]_
  In case above cookbook does not support the OS where compute is getting configured
  use doucmentation which is available at [DOCKER_RUNTIME_INSTALLATION]_.

* This depends on Nova Docker driver [NOVA_DOCKER_DRIVER]_.
  Currently a git clone of above source in .zip format is required to complete nova configuration


Testing
=======

* Add unit tests for the recipes.

* For function and CI integration test, at least one node with OpenStack
  all-in-one deployment is recommended.

* In order to configure a compute node as docker compute(while testing using openstack-chef-repo)
  override the attribute to true using environment which indicates whether a node is docker type or not

* Prior to testing, install docker runtime to all compute nodes. Refer Dependencies for more details


Documentation Impact
====================

* Change README.md. in openstack-compute


References
==========


.. [CHEF_DOCKER] `Chef docker cookbook
   <https://github.com/bflad/chef-docker>`_

.. [DOCKER_RUNTIME_INSTALLATION] `Docker runtime installation
   <https://docs.docker.com/installation>`_

.. [NOVA_DOCKER_DRIVER] `Nova docker driver
   <https://github.com/stackforge/nova-docker/tree/master>`_

.. [OPENSTACK_DOCKER_COOKBOOK_OLD] `OpenStack Docker cookbook old
   <https://github.com/paulczar/cookbook-openstack-docker>`_

.. [OPENSTACK_DOCKER_DOCUMENTATION] `OpenStack Docker Documentation
   <https://wiki.openstack.org/wiki/Docker>`_


Possible Future Enhancements
============================

Change openstack-telemetry cookbook to support monitoring of docker computes
