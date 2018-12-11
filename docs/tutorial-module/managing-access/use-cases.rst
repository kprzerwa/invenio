..
    This file is part of Invenio.
    Copyright (C) 2015-2018 CERN.
    Copyright (C) 2018 Northwestern University, Feinberg School of Medicine, Galter Health Sciences Library.

    Invenio is free software; you can redistribute it and/or modify it
    under the terms of the MIT License; see LICENSE file for more details.

.. _use_cases:

Permissions use cases
=====================

To expand your permissions policy, follow the step by step tutorial. Below you will find short examples of solutions
for different use cases.

Group access
------------

In case you need create a permission for groups of users (community), your permission factory have to create a
``RoleNeed``:

.. code-block:: json

    "communities" : ["math-students"],

.. code-block:: python

    def record_read_permission_factory(record=None):
        """Example read permission factory for roles."""
        return Permission(*[RoleNeed(x) for x in record.get("communities", [])])

Access per action
-----------------

You can create specific permission factories to protect all of the CRUD operations one by one to suit your needs.
You can consider the example schema below to store the access data:

.. code-block:: json

    "_access": {
      "type": "object",
      "properties": {
        "read": {
          "items": {
            "type": "string",
            "title": "Read"
          },
          "type": "array"
        },
        "update": {
          "items": {
            "type": "string",
            "title": "Update"
          },
          "type": "array"
        },
        "delete": {
          "items": {
            "type": "string",
            "title": "Delete"
          },
          "type": "array"
        }
      }
    }

You have to also implement permission factory per every action and add them to your configuration file
(``records\config.py``)

.. code-block:: python

    create_permission_factory_imp=record_create_permission_factory,
    read_permission_factory_imp=record_read_permission_factory,
    update_permission_factory_imp=record_update_permission_factory,
    delete_permission_factory_imp=record_delete_permission_factory,
    list_permission_factory_imp=record_read_permission_factory

