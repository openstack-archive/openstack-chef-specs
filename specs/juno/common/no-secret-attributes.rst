=================================================
Keep sensitive information out of node attributes
=================================================

https://blueprints.launchpad.net/openstack-chef/+spec/no-secret-attributes

Sensitive information such as passwords should not be stored as node
attributes, because they are persisted back to the server and can therefore
be easily retrieved.

Problem description
===================

Wrapped recipes (e.g. mysql::server) use node attributes for retrieving
sensitive information, such as passwords associated with privileged
accounts.  This type of information is specified via items in encrypted
data bags, but because node attributes are involved, the security provided
by the encryption mechanism can easily be defeated by simply retrieving the
node attributes from the server.

For example (from mysql-server):

::

  if node['openstack']['db']['root_user_use_databag']
      super_password = get_password 'user', node['openstack']['db']['root_user_key']
      node.set_unless['mysql']['server_root_password'] = super_password
  else
      super_password = node['mysql']['server_root_password']
  end

The password is retrieved from a (possibly encrypted) data bag and stored
into a node attribute to be consumed downstream by the :code:`server`
recipe in the :code:`mysql` cookbook.  After the node is converged, all
someone needs to do to retrieve the password as clear text is execute
:code:`knife node edit NODE_NAME`, thereby defeating a data bag's encryption
capabilities.

Proposed change
===============

The proposed solution is to operate on server resources directly or use
run_state to set passwords, depending on which is available for use in
a given recipe.

Again, using MySQL as an example (operating directly on resource):

::

  if node['openstack']['db']['root_user_use_databag']
      super_password = get_password 'user', node['openstack']['db']['root_user_key']
  else
      super_password = node['mysql']['server_root_password']
  end

  mysql_service node['mysql']['service_name'] do
    server_root_password super_password
    ...
  end

The password is stored only in a local variable and directly assigned to the
:code:`mysql_service` resource's :code:`server_root_password` attribute.
The :code:`mysql::server` recipe is not invoked and all of the other resource
attributes are set directly as well (indicated via :code:`...`). Since the
password is never stored as a node attribute, it cannot be retrieved via
:code:`knife node edit NODE_NAME`. Note that other resource attributes (for
instance :code:`port`) can still be set to node-attribute values and thus
their default values can still be specified in :code:`attributes/default.rb`.
Also note that the above example shows that a node attribute can still be
used to store password information, if that is what the user wants to do.
This is done by leaving the :code:`['openstack']['db']['root_user_use_databag']`
set to its default value of :code:`false`.

Alternatives
------------

One alternative would be to have the wrapped recipes fall back to default
attribute values and set the resource attributes directly, as described at
https://sethvargo.com/changing-chef-resources-at-runtime/.  However, this
is arguably a non-strategic workaround.

Data model impact
-----------------

none

REST API impact
---------------

none

Security impact
---------------

Security would be improved because passwords would no longer be accessible
as node attributes.

Notifications impact
--------------------

none

Other end user impact
---------------------

None. The recipes would be backward compatible because the mechanisms for
specifying the databag type (encrypted or standard) and name and related
node attributes needed to use data bags would remain unchanged.  The only
thing changing would be the mechanics of how the data contained in data
bags is populated into the resources created by the recipes.  Note that
end users would still have the option to not use data bags at all.

Performance Impact
------------------

none

Other deployer impact
---------------------

none

Developer impact
----------------

none

Implementation
==============

Assignee(s)
-----------

* jswarren

Work Items
----------

The known impacted recipes are:

* cookbook-openstack-ops-database::mysql-server
* cookbook-openstack-ops-database::postgresql-server
* cookbook-openstack-ops-messaging::rabbitmq-server

Dependencies
============

The cookbooks involved in setting up the resources in question need to
provide a mechanism for setting passwords either as a resource attribute
that can be set directly or as a run_state attribute.  Other mechanisms
that do not expose passwords as node attributes should also be acceptable.


Testing
=======

The cookbooks in question would need to be tested to ensure that when
data bags are used to specify passwords that the password values are not
reflected in the attributes (if any) used for setting passwords.  In other
words, a given referenced cookbook or recipe might still allow the use
of node attributes to set passwords in addition to the above-mentioned
possible alternatives.  However, when node attributes are not used to
specify passwords, the passwords must then not be stored as node
attributes by the referenced recipes.


Documentation Impact
====================

Documentation should not change, since the blackbox behavior of the recipes
should remain unchanged.  The documentation associated with the referenced
resource recipes may need to change when accommodations are made for
alternative means of setting passwords, but those changes are orthogonal to
the work described here.

References
==========

