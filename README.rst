========================
Team and repository tags
========================

.. image:: http://governance.openstack.org/badges/openstack-chef-specs.svg
    :target: http://governance.openstack.org/reference/tags/index.html

.. Change things from this point on

==================================
OpenStack-chef Specifications
==================================

This git repository is used to hold approved design specifications for additions
to the openstack-chef project.  Reviews of the specifications are done in gerrit,
using a similar workflow to how we review and merge changes to the code itself.

The layout of this repository is::

  specs/<release>/<cookbook>/

You can find an example spec in `specs/template.rst`.

Specifications are proposed for a given release by adding them to the
`specs/<release>` directory and posting it for review.  The implementation
status of a blueprint for a given release can be found by looking at the
blueprint in launchpad.  Not all approved blueprints will get fully implemented.
Use the Common cookbook directory for specifications that effect multiple
cookbooks.  Once the specification is approved and merged, the LaunchPad
blueprint will be updated accordingly.

Specifications have to be re-proposed for every release.  The review may be
quick, but even if something was previously approved, it should be re-reviewed
to make sure it still makes sense as written.

Prior to the Juno development cycle, this repository was not used for spec
reviews.  Reviews prior to Juno were completed entirely through Launchpad
blueprints::

  http://blueprints.launchpad.net/openstack-chef

Please note, Launchpad blueprints are still used for tracking the
current status of blueprints. For more information, see::

  https://wiki.openstack.org/wiki/Blueprints

For more information about working with gerrit, see::

  http://docs.openstack.org/infra/manual/developers.html#development-workflow

To validate that the specification is syntactically correct (i.e. get more
confidence in the Jenkins result), please execute the following command::

  $ tox

After running ``tox``, the documentation will be available for viewing in HTML
format in the ``doc/build/`` directory.
