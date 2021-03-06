..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

==================================
IdP ID registration and validation
==================================

`bp idp-id-registration
<https://blueprints.launchpad.net/keystone/+spec/idp-id-registration>`_

This specification describes a more secure user mapping after authentication
using the ``OS-FEDERATION`` APIs. The Identity Provider releasing the
authentication assertion is identified to avoid wrong user mappings so its ID
has to be registered in Keystone and validated upon authentication request.

Problem Description
===================

With OS-FEDERATION it is possible to use external Identity Providers (IdPs)
for user authentication. A user, to be authenticated, needs to access a
specific path containing IdP name and protocol::

  /OS-FEDERATION/identity_providers/{identity_provider}/protocols/{protocol}/auth

Then, the authentication follows the steps required by the selected protocol,
e.g. in case of SAML the user is redirected to the IdP, and at the end of the
process the user will return to the above URL with the authentication
assertion. If it is accepted by Keystone, the assertion attributes are
translated, according to a mapping defined for the IdP, in order to identify
the roles for the new generated token to return to the user.

The mapping selected to convert the assertion attributes in roles depends on
the URL, which is strictly related to the IdP. If the user can provide the
assertion generated by IdP *A* to an URL identifying the IdP *B* then the
attributes will be translated according to the mapping defined for *B*. As a
result, users may get tokens with roles specified for other IdPs without any
control from Keystone.

Currently, the correspondence among IdP assertions and URLs is delegated to
a *httpd web server* and made use of some specific features of the Shibboleth
plug-in used for SAML authentication. Other SAML plugins or plugins for
other protocols could not support this configuration.

Proposed Change
===============

When an IdP is registered in Keystone, its ID (i.e. in SAML protocol this will
be the entity_id of the IdP) has to be stored alongside the other information
and used during the authentication to verify that the IdP releasing the
assertion matches the IdP specified in the URL. The ID is provided with a new
attribute, named ``remote_ids``, and this contains a list of IDs. The attribute
should be made optional, for backwards compatibility.

``remote_ids`` is defined as a list to allow the mapping of multiple external
IdPs in a single IdP in Keystone. This can simplify the sharing of a mapping
among IdPs, with benefit for the management. A possible scenario for this use
arises when Keystone has to be connected with a federation having multiple IdPs
and all share the same set of attributes and policies. In this case a single
map could be possible to manage all the IdPs, avoiding to register all of them
in Keystone.

As Keystone deployment should handle different protocols and different modules
handling protocols parameter where ``remote_id`` values is stored needs to be
dynamically configured in ``keystone.conf``. Each protocol handled by Keystone
must exist as a authentication method defined in ``[auth]`` section as well as
corresponding section must be defined. Inside such section a parameter called
``remote_id_attribute`` should be defined if ``remote_ids`` should be
validated.

Example of ``keystone.conf`` configuration:

::

    [auth]
    methods = saml2,oidc
    saml2 = auth.plugins.mapped.Mapped
    oidc = auth.plugins.mapped.Mapped

    [saml2]
    remote_id_attribute = "Shib-Identity-Provider"

    [oidc]
    remote_id_attribute = "Claim-Identity-Provider"


It's worth mentioning, that in this particular case both protocols ``saml2``
and ``oidc`` would also need to be registered and tied to identity providers.

Alternatives
------------

Maintain the current implementation where the map between IdPs and URLs is
performed by Apache and skip any control in Keystone. This approach has the
same level of security but requires a change in the configuration every time an
IdP is added/removed.


Security Impact
---------------

The proposed specification increment the security of the OS-Federation
authentication APIs by introducing a new check aimed at verifying that the IdP
performing the authentication is the same specified in the authentication URL.

Notifications Impact
--------------------

None

Other End User Impact
---------------------

The python-keystoneclient needs to be updated in order to reflect the new API
to register the IdP. In the current implementation there, the config attribute
``remote_ids`` does not exist. The attribute has to be included during the
creation/editing of the IdPs.

Performance Impact
------------------

The new code is invoked every time a user requires an authentication with an
IdP. This introduces a new check of an environmental variable but the impact is
quite limited because there are in place other checks of the information coming
from the IdP.

Other Deployer Impact
---------------------

A new configuration variable is introduced::

  ``remote_id_attribute``

The variable is a string and its value is the name of the attribute to look-up
to get the ID of the current IdP. Default value is an empty string and in this
case the new code does not take effect maintaining the previous behaviour of
the OS-FEDERATION APIs.

Developer Impact
----------------

None


Implementation
==============

Assignee(s)
-----------

Primary assignee:

* Marco Fargetta <marco-fargetta>
* Marek Denis <marek-denis>


Work Items
----------

* Modify the IdP APIs to contain the new attribute ``remote_ids``
* Modify the ``python-keystoneclient`` to interact with modified API


Dependencies
============

None


Documentation Impact
====================

The documentation of the APIs for the OS-Federation and the client needs to be
updated with the proposed change.


References
==========

A partial implementation of this specs under review: `review 142743
<https://review.openstack.org/#/c/142743>`_

