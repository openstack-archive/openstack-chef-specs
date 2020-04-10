==========================================
Enable Bare Metal Service by Chef
==========================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/openstack-chef/+spec/bare-metal-enablement

OpenStack Bare Metal Service known as Ironic OpenStack project name provisions
Bare Metal machines by leveraging common technologies such as [PXE]_ boot and
[IPMI]_ to cover a wide range of hardware.


Problem description
===================

* OpenStack Bare Metal Service was available as an incubated project in the
  "Icehouse" release, with many stability and feature improvements in the
  "Juno", it is officially integrated with OpenStack beginning with the "Kilo"
  release.

* Currently, there is no Chef cookbook support to install and configure
  OpenStack Bare Metal Service.


Proposed change
===============

Add a cookbook included recipes and related attributes/unit tests to install
and configure OpenStack Bare Metal Service.

The packages which need to be installed include:

* Fedora/RHEL: openstack-ironic-api, openstack-ironic-conductor,
               openstack-ironic-common and python-ironicclient.
               For PXE: tftp-server and syslinux-tftpboot.
               For IMPI: ipmitool.

* Ubuntu: ironic-api, ironic-conductor and python-ironicclient.
          For PXE: tftpd-hpa, syslinux and syslinux-common.
          For IMPI: ipmitool.

Alternatives
------------

None

Data model impact
-----------------

None

REST API impact
---------------

Refer to [REST_API_V1]_ for details.

Security impact
---------------

* Authentication method to use when connecting to OpenStack other components is
  keystone with PKI as well as others.

* Allow ironicclient to preform "secure" SSL (https) requests, the SSL/TLS
  versions TLSv1.2. TLSv1.0 and TLSv1.1 are only recommended, but the
  certificates and key exchange are not in the scope of enabling Bare Metal
  Service.

* Hash algorithms to use for hashing PKI tokens include md5, etc.

* The commands which require the use of sudo are:
    qemu-img
    iscsiadm
    blkid
    blockdev
    mkswap
    mkfs
    mount
    umount
    dd
    fuser
    parted

* About resource exhaustion attack, there is no security impact of this Spec.

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
  - wenchma@cn.ibm.com

Other contributors:
  - None

Work Items
----------

* Add a basic cookbook with the default files for getting through first gate
  .gitignore
  .gitreview
  .rubocop.yml
  Berksfile
  CONTRIBUTING.md
  Gemfile
  metadata.md
  Rakefile
  README.md
  TESTING.md
  /spec/spec_helper.rb

* Add the default attribute file to determine Bare Metal Service configuration
  items.

* Add the recipe files to install the packages of Bare Metal Service,
  configure the [BARE_METAL_TEMPLATE]_ and start Bare Metal services.

* Add the template files to enable Bare Metal Service to be configurable.

* Add the unit tests.


Dependencies
============

* Ironic requires the supported Bare Metal hardwares, such as x68 servers,
  HP iLO, and Intel AMT platforms so far.

* This requires the supported platform operating systems, such as Fedora,
  RHEL and Ubuntu.

* This depends on the blueprint [BARE_METAL_ENABLEMENT]_

* This requires configuring the Bare Metal Service's driver for Compute Service.

* This requires the setup of supported boot mode, such as [PXE]_ and [IPMI]_.


Testing
=======

* Add unit tests for the recipes.

* For function and CI integration test, at least one node with OpenStack
  all-in-one deployment is recommended.


Documentation Impact
====================

* Add README.md.


References
==========

.. [BARE_METAL_TEMPLATE] `Bare Metal Template values
   <https://docs.openstack.org/trunk/config-reference/content/ch_configuring-openstack-bare-metal.html>`_

.. [BARE_METAL_ENABLEMENT] `Bare Metal enablement blueprint
   <https://blueprints.launchpad.net/openstack-chef/+spec/bare-metal-enablement>`_

.. [IPMI] `IPMI tool source code
   <http://ipmitool.sourceforge.net/>`_

.. [PXE] `PXE user guide
   <https://docs.openstack.org/developer/ironic/deploy/user-guide.html#pxe>`_

.. [REST_API_V1] `Bare Metal RESTful Web API v1
   <https://docs.openstack.org/developer/ironic/webapi/v1.html>`_

`Bare Metal Service Installation Guide
<https://docs.openstack.org/developer/ironic/deploy/install-guide.html>`_
