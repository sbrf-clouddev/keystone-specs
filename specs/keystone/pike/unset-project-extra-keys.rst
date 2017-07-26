..
 This work is licensed under a Creative Commons Attribution 3.0 Unported
 License.

 http://creativecommons.org/licenses/by/3.0/legalcode

========================
Unset project extra keys
========================

`bp unset-project-extra-keys <https://blueprints.launchpad.net/keystone/+spec/unset-project-extra-keys>`_

Add possibility to unset project extra keys.

Problem Description
===================

We have possibility to set key-value pairs (extra field in 'project' DB table)
to projects and then filter projects based on it. But in case we dynamically
update it, adding new pairs and removing old ones, we are not able to
remove old, not needed pairs anymore. It leads to garbaging and slowing down.

Proposed Change
===============

It is proposed to add small new API that will allow us to unset keys from
'extra' project column.

CLI view is expected to look as following:

.. code-block:: bash

    $ openstack project unset %project_id% key1 key2

New API is expected to look as following:

- URL: /v3/projects/%project_id%
- Method: PATCH
- Request body:

.. code-block:: json

    {
        "project": {"unset_extra_keys": ["key1", "key2"]}
    }

Alternatives
------------

Continue being allowed only to extend 'extra' column values, not unset/shrink.

Security Impact
---------------

None

Notifications Impact
--------------------

None

Other End User Impact
---------------------

End user will have possibility to use additional 'unset' API and CLI.

Performance Impact
------------------

None

Other Deployer Impact
---------------------

None

Developer Impact
----------------

None

Implementation
==============

Assignee(s)
-----------

Primary assignee:
 Dmitri Plakhov ("dmitri.plakhov" at lauchpad.net)

Other contributors:
  Artem Tiumentcev ("darland-maik" at lauchpad.net)

Work Items
----------

- Add 'unset' API to project object
- Add 'unset' support in python-keystone client
- Add 'unset' CLI support in python-openstackclient

Dependencies
============

None

Documentation Impact
====================

Update of user documentation will be required, where new API will be described.

References
==========

None
