..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
Bump Chef gem to 11.16
==========================================

Launchpad blueprint:

https://blueprints.launchpad.net/openstack-chef/+spec/bump-chef-to-11-16

Propose to update all the cookbooks at master branch to use Chef 11.16.
The current version is at 11.12.

Problem description
===================

A detailed description of the problem:

* During each Master branch cycle, we need to keep it up to date with the latest tooling.

Proposed change
===============

Change all the cookbook Gemfile's to use Chef 11.16

Alternatives
------------

none

Data model impact
-----------------

none

REST API impact
---------------

none

Security impact
---------------

none

Notifications impact
--------------------

none

Other end user impact
---------------------

none

Performance Impact
------------------

Should help improve performance as there have been many fixes, for example:

Other deployer impact
---------------------

none

Developer impact
----------------

The current bundle install command will take care of updating 
Chef to the required 11.16 version level. No additional steps 
should be needed by the developers.

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  Mark Vanderwiel (vanderwl)

Other contributors:
  none

Work Items
----------

* Each cookbooks Gemfile needs to specific Chef 11.16


Dependencies
============

none

Testing
=======

Basic testing (unit, lint, style, all-in-one) using Chef 11.16 has been done and no issues were found.


Documentation Impact
====================

none


References
==========

* `Chef versions <https://rubygems.org/gems/chef/versions>`_
* `Chef change log <https://github.com/opscode/chef/blob/master/CHANGELOG.md>`_