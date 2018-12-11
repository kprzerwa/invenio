..
    This file is part of Invenio.
    Copyright (C) 2015-2018 CERN.
    Copyright (C) 2018 Northwestern University, Feinberg School of Medicine, Galter Health Sciences Library.

    Invenio is free software; you can redistribute it and/or modify it
    under the terms of the MIT License; see LICENSE file for more details.

.. _records_access_overview:


Invenio records permissions
======================================


Prerequisites
-------------

You will need the ``invenio-access`` module for implementing the permissions. You can install it by using the command:

.. code-block:: bash

    pipenv install invenio-access


Steps in this tutorial
----------------------

1. Define access policy.
2. Expand the records structure to add access field(s).
3. Add the access field to elasticsearch mapping.
4. Implement permission factories.
5. Update your configuration.


Defining access policy
----------------------

Before starting the implementation you have to estabilish the access policy which will suit your needs. For the purpose
of this tutorial let's focus on the read access policy.

Only owners will have read access to their records.

.. note:: The **superuser-access** for admin users will already definec, which are allowed to perform any action
          (invenio-access module provides this feature by default). You need to assign this role to specific admin user.


Configuration
-------------

1. Config.py file

The first place you should take look into is ``records\config.py`` file. It contains the ``RECORDS_REST_ENDPOINT``
variable where we have defined the permission factories:

.. code-block:: python

    # TODO: Redefine these permissions to cover your auth needs
    read_permission_factory_imp=check_elasticsearch,

As you can see there are default permission factories assigned, which allow all users to perform CRUD operations. Let's
focus on the read permission factory.

2. Record schema

Once we have an existing record, we can assign access rights directly per record. It requires modification in the
records' schema - to be able to pass the validation when trying to save the record. We need to add the field containing
owners of the record.

``records\jsonschemas\record-v.1.0.0.json`` - example of the structure

.. code-block:: json

    "owners": {
      "type": "array",
      "items": {
        "type": "string"
      }
    }


3. Marshmallow validation

Marshmallow uses it's own validation model which you have to implement to pass the record validation:

.. code-block:: python

    class MyRecordSchemaV1(Schema):
        """My record schema."""

        pid = PersistentIdentifier()
        # add new field
        owners = fields.List()

    my_record_loader = marshmallow_loader(MyRecordSchemaV1)


4. Add your validator to the configuration in ``records/config.py`` in ``RECORDS_REST_ENDPOINT``

.. code-block:: python

        record_loaders={
            'application/json': ('test_access.records.loaders'
                                 ':my_record_loader'),
        },


5. Add the owners field to elasticsearch mapping ``records\mappings\v{ES_version}\records\record-v1.0.0.json``

.. code-block:: json

    "owners": {
      "type": "keyword"
    },


Implementation
--------------

1. Permission factory

The permission factory will describe your access policy. The implementation will use basic
functionality provided by Permission class of ``invenio-access`` module.

To have the access user has to fulfill certain conditions. In order to access his own records the user has to be
identified as it's owner.

.. code-block:: python

    def record_read_permission_factory(record=None):
        """Record read permission factory."""
        return Permission(*[UserNeed(x) for x in record.get("owners", [])])

2. Search results filter

In order to display only the records that user has permission to read while perform the search, we need to implement
search filter.

.. code-block:: python

    def search_permission_filter():
        """Filter list of search results."""
        from invenio_accounts.utils import get_identity
        provides = get_identity().provides

        # Filter for restricted records, that the user has access to
        read_restricted = Q('terms', **{'owners': provides})
        return Q('bool', filter=[read_restricted])


    class MyRecordSearch(RecordsSearch):
        """My Record search class."""

        class Meta:
            """Configuration for search."""

            index = 'records'
            doc_types = None
            fields = ('*',)
            default_filter = DefaultFilter(search_permission_filter)



3. Update your config file with permission factories that you have just implemented.

.. code-block:: python

    search_class=MyRecordSearch,
    read_permission_factory_imp=record_read_permission_factory,

Granting access per record
--------------------------

Example of record containing the owners field:

.. code-block:: json

    "owners" : [2, 3],

Record with the owners field defined like above means that the read access has been granted for ``Users`` who provide
ID 2 or 3.


Granting access per role
------------------------

You can also create custom roles by using:

.. code-block:: python

    custom_role = RoleNeed('custom-role')


In your application you can restrict access not only for records but also for some parts of the application,
like whole pages.

You can assing roles to the users by using CLI commands or by using the admin interface provided by
`invenio-admin <https://invenio-admin.readthedocs.io/en/latest/>`_.
