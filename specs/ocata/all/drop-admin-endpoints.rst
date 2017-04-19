..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=======================================================
Stop generating admin endpoints in the keystone catalog
=======================================================

The admin endpoints offer no special functionality, users may talk to the
public endpoints instead. The only historic use case has been the keystone
v2 admin endpoint, but with keystone v3 API, even that is no longer needed.

Problem description
===================

Currently we generate admin endpoints for almost all services. As the service
catalog is sent to the API user for every transaction, this generates some
amount of overhead, as these endpoints aren't really needed anymore. Dropping
them will also reduce the time needed for chef runs.

Proposed change
===============

Drop the admin endpoints from all identity_registration recipes in our
cookbooks. This will affect:

- cookbook-openstack-block-storage
- cookbook-openstack-compute
- cookbook-openstack-identity
- cookbook-openstack-image
- cookbook-openstack-networking
- cookbook-openstack-orchestration
- cookbook-openstack-telemetry

Alternatives
------------

Stick to the status quo.

Data model impact
-----------------

None

REST API impact
---------------

None

Security impact
---------------

Deployments that have been using a different admin endpoint with restricted
access may need to switch to using the internal endpoint instead.

Notifications impact
--------------------

None

Other end user impact
---------------------

None

Performance Impact
------------------

The size of the service catalog will be reduced, as well as the duration of
chef runs, both with positively impact performance.

Other deployer impact
---------------------

Deployments that in some way make unexpected use of the admin endpoints will
need to be adapted.

Developer impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <j-rosenboom-j>

Other contributors:
  <jklare>

Work Items
----------

- Update identity_registration recipes
- Check for unknown dependencies


Dependencies
============

None


Testing
=======

Our integration tests should have sufficient coverage in order to make sure
that this change doesn't have any negative impact.


Documentation Impact
====================

None


References
==========

None
