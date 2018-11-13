..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==========================================
OpenStack Cookbook Versioning scheme
==========================================

Include the URL of your launchpad blueprint:

https://blueprints.launchpad.net/openstack-chef/+spec/example

There has been multiple threads on the way that cookbook versioning is done.
This doc is to attempt to consolidate and organize and ideally agree upon some
guidelines on the way to bump/change work on the versioning for
`the cookbooks <https://github.com/stackforge/cookbook-openstack-common>`_ for
instance the common cookbook.

Problem description
===================

There is no standardization on the cookbook versioning we have. This doc is the
attempt to give some general guidelines to fix this.


Proposed change
===============

General Guidelines
------------------

When submitting cookbook patches, it is generally required that the version
number (within metadata.rb) is incremented in a manor reflective of the level
of the change.  Any patch should also update the CHANGELOG.md and if appropriate
the README.md should reflect the changes and any relevant how-to instructions. The
CHANGELOG.md is our executive summary of changes, it should inform what the change
was in a quick manner.

There are some differences between the development of patches on the Master and
Stable branches.  There is more restrictive and vigorous oversight given to changes
on the Stable branches.  The Master branch is bleeding edge and can relax the
version requirement in some simple cases to allow for increased productivity.

Semantic Versioning
-------------------

The cookbooks use the Semantic Versioning system (see http://semver.org)
The system uses a three part version number, Major.Minor.Patch.
For example: 9.2.33

The Major number shouldn't change within a development branch. It will reflect the
number that is the Alphabetized Letter of the base OpenStack release,
see: https://wiki.openstack.org/wiki/Releases.  An example would be Icehouse
being the 9th letter and the 9th release all the stable cookbooks would be 9.Y.Z.
When the Master branch becomes stabilized, a new Stable branch will be created from
it and the Major number in the Master will be incremented by the core team.

The Minor and Patch numbers will be incremented as described in the branch specific
sections below.

Stable Branch
-------------

The Stable branch cookbooks must leverage the Semantic Versioning system exclusively.
All patches must update the metadata.rb and the CHANGELOG.md at a minimum.
All patches should try to be backwards compatible.

+-------------------------------------------------------------------------+-----------------+
| Stable Branch Example Situations                                        | Level of Change |
+=========================================================================+=================+
| add a recipe                                                            | Increment Minor |
+-------------------------------------------------------------------------+-----------------+
| add a function or method                                                | Increment Minor |
+-------------------------------------------------------------------------+-----------------+
| change Gemfile or Berksfile                                             | Increment Minor |
+-------------------------------------------------------------------------+-----------------+
| backport a fix from Master branch                                       | Increment Minor |
+-------------------------------------------------------------------------+-----------------+
| add attribute for a value in a configuration file with the same default | Increment Patch |
+-------------------------------------------------------------------------+-----------------+
| changing a resource option                                              | Increment Patch |
+-------------------------------------------------------------------------+-----------------+
| add a test                                                              | Increment Patch |
+-------------------------------------------------------------------------+-----------------+
| fix a broken recipe                                                     | Increment Patch |
+-------------------------------------------------------------------------+-----------------+
| re-factoring recipe or test                                             | Increment Patch |
+-------------------------------------------------------------------------+-----------------+

The table above shows some examples of different levels of changes introduced by a patch and
what part of the version number to increment for Stable branches.

Version Locking
^^^^^^^^^^^^^^^

The Stable branches are also locked down by added Berksfile.lock and Gemfile.lock to each
cookbook.  If any changes are made to the Gemfile, the Gemfile.lock would also have to be
updated.  If any changes are dependent upon other cookbook changes, then the Berkfile.lock
and metadata.rb files would need to be updated accordingly.

Cookbook Dependencies
^^^^^^^^^^^^^^^^^^^^^

When a change requires hits to multiple cookbooks, like when adding attributes to Common, the
metadata.rb file would need to be updated to reflect that required version level.  The
Berksfile.lock would also need to be updated with the commit id of the dependent change.

Master Branch
-------------

The master cookbook should leverage the Semantic Versioning system lightly.
We consider the master branch a fast paced "Work in Progress" until we come to the Release
Candidate 1 date (RC-1). This means it will be under rapid and active development where
versioning isn't always required.
All patches must update the CHANGELOG.md at a minimum.

+-------------------------------------------------------------------------+------------------+
| Master Branch Example Situations                                        | Level of Change  |
+=========================================================================+==================+
| Branching for a new Stable branch                                       | Increment Major  |
+-------------------------------------------------------------------------+------------------+
| add a recipe                                                            | Increment Minor  |
+-------------------------------------------------------------------------+------------------+
| add a function or method                                                | Increment Minor  |
+-------------------------------------------------------------------------+------------------+
| change Gemfile or Berksfile                                             | Increment Minor  |
+-------------------------------------------------------------------------+------------------+
| add attribute for a value in a configuration file with the same default | Increment Patch**|
+-------------------------------------------------------------------------+------------------+
| changing a resource option                                              | Increment Patch**|
+-------------------------------------------------------------------------+------------------+
| add a test                                                              | Increment Patch**|
+-------------------------------------------------------------------------+------------------+
| fix a broken recipe                                                     | Increment Patch**|
+-------------------------------------------------------------------------+------------------+
| re-factoring recipe or test                                             | Increment Patch**|
+-------------------------------------------------------------------------+------------------+

The table above shows some examples of different levels of changes introduced by a patch and
what part of the version number to increment for Master branches.

** There are cases where incrementing the Patch number is not necessary and only updating the
CHANGELOG.md is required.  To avoid re-base collisions on Patch number changes and allow more
rapid development, if the change falls within these guidelines, incrementing the Patch version
would not be required.
- When the change only effects a single cookbook
- When the change is just a simple addition of a new attribute for a template and no other
logic change is required
- When a patch only effects the tests
- When a patch only effect the README or other comments

Version Locking
^^^^^^^^^^^^^^^

The Master branch is NOT locked down a Berksfile.lock or Gemfile.lock.  Changes to the Berksfile
and Gemfile can be made directly.

Cookbook Dependencies
^^^^^^^^^^^^^^^^^^^^^

When a change requires hits to multiple cookbooks, like when adding attributes to Common, the
metadata.rb file would need to be updated to reflect that required version level.


Implementation
==============

Assignee(s)
-----------

Everyone because this is an over arching process ;)

Work Items
----------

None.


Dependencies
============

None, apart from the community approving these guide lines.

Testing
=======

None.

Documentation Impact
====================

See above, this should also be put on the wiki too.

References
==========

This `youtube video <http://youtu.be/jA9L_g-d-X4>`_ is the major discussion
about this topic. There has also been multiple comments on the google group.
