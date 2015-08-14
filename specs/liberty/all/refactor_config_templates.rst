..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

=================================================
Refactor configuration templates in all cookbooks
=================================================

URL of launchpad blueprint:

https://blueprints.launchpad.net/openstack-chef/+spec/refactor-configuration-templates

Problem description
===================

Our templates for the configurations files for all services have grown through
the years and it is getting harder and harder to figure out where to set a
specific option that goes into the config files. The same is true for reading
these configuration files after they have been rendered, since they contain a
lot more options and code than actually needed or used. Additionally every time
one of the OpenStack service projects adds or removes an configuration option,
we manually need to add an attribute, some conditions and an entry in the
template for this. Same goes for every change in the defaults, that is either
not reflected at all in our config files, or set explicitly to the same thing,
which is just unneeded code after all.  Finally, the fact that all of our
templates are quite big blocks of code and even contain some hardcoded values
for specific configuration options, which came from the original configuration
files imported years ago, does not help with keeping up with the multiple changes
in the service projects.

Proposed change
===============

The templates for OpenStack service configurations files should implement or use
a logic of setting the needed key-value configuration options in the appropriate
section automatically from properly defined chef attributes.

* logic to render proper configuration files needs to be defined and added
  (example given below)
* attributes used for or in configurations templates need to be checked and
  refactored to fit this logic
* commonly used configuration options should be handled in one place
* only needed configuration options should be set in the configuration files
  after rendering the template
* configuration options that are specific to only a few setups should be removed
  from the default attributes and recipes (especially the switchcases for them)
  and moved to the documentation. Although this will mean to remove
  functionality in the sense of removing some switchcases and attributes in the
  original cookbooks, this should not mean to loose these completely, but rather
  move them to the documentation to allow an easy implementation in wrapper
  cookbooks.  

Scope
-----

This spec tries to define a refactoring process that implements a more elegant
handling of template files for OpenStack service configurations throughout all
cookbooks used to deploy OpenStack.


End user impact
---------------

* Users of the cookbooks will be able to wrap all OpenStack chef cookbook more
  easily and set configuration options if needed without patching the actual
  templates.
* Old wrapper cookbooks before this change will stop working partially and
  would need to be refactored accordingly. The same is true for all environment
  files that include configuration options for the services via attributes.
* Reading configuration files will become a lot easier, since only specifically
  wanted and needed options will be included in them.
* The user can add environment and company specific comments in the service
  configuration files via attributes.


Developer impact
----------------

Cookbook developers will not need to add attributes for configuration templates,
nor should there be any need to change existing attributes to fit the changing
defaults in OpenStack service projects. It should become easier to read the
overall cookbook, since a big part of the code in the templates and attribute
files will be cut or moved with this feature. Therefore it might be easier to
contribute as a newcomer without spending hours to understand how all the
attributes work with each other (given that the logic used to render the
templates is easier to understand of course).

Implementation
==============

Assignee(s)
-----------

Primary assignee:
  <j-klare>

Other contributors:

Work Items
----------

* define a generic structure for templates, that can be used for most of the
  configuration files
* main config files for services look like this for all configuration
  options:

    [section]
    # optional comment or description
    key = value (usually simple boolean, numeric or string)
    # optional comment or description
    key = [ array which could be large ]

* create a logic to render configuration files from properly set chef attributes
* attributes could look like this for main configuration files (e.g. nova
  or neutron.conf):

::

  default['openstack'][service]['conf'][section][key] =
    value

without a comment:

::

  default['openstack'][service]['conf'][section]['debug'] =
    true

with a comment:

::

  default['openstack'][service]['conf'][section]['debug'] =
    { set_to: true, comment: 'this option is the switch to enable/disable debug loggging' }

cookbook-openstack-common/templates/common.conf.erb :

::

  <% @service_config.each do |section, values| -%>
  [<%= section %>]
    <% values.each do |key, value| -%>
      <% if value.class == Hash -%>
  <%= "# {value['comment']}" -%>
  <%= key %> = <%= value['set_to'] %>
      <% else -%>
  <%= key %> = <%= value %>
      <% end -%>
    <% end -%>
  <% end -%>

cookbook-openstack-compute/recipes/common.rb :

::

  ...
  # activesupport currently needs to be upgraded, since the one used in
  # chef 12 is too old (3.2.19) and does not include the needed deep_merge
  chef_gem 'activesupport' do
    action :upgrade
    version '4.2.4'
  end
  require 'active_support'
  ...

  neutron_admin_password = get_password 'service', 'openstack-network'
  identity_admin_endpoint = admin_endpoint 'identity-admin'
  identity_uri = identity_uri_transform(identity_admin_endpoint)
  ...

  secrets = {
              neutron: {
                admin_password: neutron_admin_password,
                ...
              },
              keystone_authtoken: {
                identity_uri: identity_uri,
                ...
              },
              ...
            }

  # merge node attributes with secrets like passwords etc. for
  # usage in template['/etc/nova/nova.conf']
  nova_config_options =
    node['openstack']['compute']['conf'].deep_merge(secrets)

  template '/etc/nova/nova.conf' do
    source 'common.conf.erb'
    cookbook 'openstack-common'
    owner node['openstack']['compute']['user']
    group node['openstack']['compute']['group']
    mode 00640
    variables(
      service_config: nova_config_options
    )
  end
  ...

cookbook-openstack-compute/templates/nova.conf.erb -- not needed anymore

* add a link to documentation/config reference to all config files
* refactor currently used attributes to fit into that logic
* adapt specs
* define a set of minimal needed attributes to create a working stack and move
  the rest of the attributes into documentation
* remove attributes that would set configuration options that are equal to the
  defaults
* propagate not backward compatible change at a fitting point in time without
  making a lot of people angry


Testing
=======

* lint and style tests with rubocop (as is)
* unit tests with chefspec with special focus on testing the proper rendering of
  the configuration templates (including the sections)
* integration testing with chef provisioning


Documentation Impact
====================

These patches should move a good part of the currently defined attributes to the
documentation as examples for specific setups.


References
==========
