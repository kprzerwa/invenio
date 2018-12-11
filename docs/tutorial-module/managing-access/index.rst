..
    This file is part of Invenio.
    Copyright (C) 2015-2018 CERN.
    Copyright (C) 2018 Northwestern University, Feinberg School of Medicine, Galter Health Sciences Library.

    Invenio is free software; you can redistribute it and/or modify it
    under the terms of the MIT License; see LICENSE file for more details.

.. _access_index:

Overview
========

The Invenio instance that you have created when following :ref:`quickstart` comes with ``Custom`` data model.
The data model provides tools for CRUD operations and search. It leaves the flexibility
providing custom access policy for your instance.

An Invenio data model defines *a record type*. It is basically an entity for storing data.

In addition to storing records (aka documents or rows in a database) according
to a specific structure, a data model also deals with:

* Access to records via REST APIs and landing pages.
* Internal storage, representation and retrieval of records and
  persistent identifiers.
* Mapping external representations to/from the internal representation via
  loaders and serializers.


Records
-------

You can consider the record as a row in a database table. A record in Invenio consists of a structured collection of
fields and values (metadata), which provides information about other data. The format of these records is defined by
a schema, which is represented in Invenio as JSONSchema. Since the format of a record can change over time, it can be
associated to different JSONSchema versions. For more information on records and schemas visit
`InvenioRecords <http://invenio-records.readthedocs.io/>`_.

If you have selected custom datamodel, while scaffolding the Invenio instance through cookiecutter,
you have installed the records module under:

.. code-block:: shell

    |-- ...
    |-- docs
    |   |-- ...
    |-- records
    |   |-- config.py
    |   |-- jsonschemas/
    |   |-- loaders/
    |   |-- mappings/
    |   |-- marshmallow/
    |   |-- serializers/
    |   |-- static/
    |   |-- templates/
    |   `-- ...
    |-- setup.py
    `-- tests
        |-- ...


REST API
--------

The basic instance installation contains also a module which is taking care of REST requests.

The module uses Elasticsearch as the backend search engine and builds on top a REST API that supports
features such as:

- Search with sorting, filtering, aggregations and pagination, which allows the full capabilities of
Elasticsearch to be used, such as geo-based quering.
- Record serialization and transformation (e.g., JSON, XML, DataCite, DublinCore, JSON-LD, MARCXML,
Citation Style Language).
- Pluggable query parser.
- Tombstones and record redirection.
- Customizable access control.
- Configurable record namespaces for exposing different classes of records (e.g., authors, publications, grants, …).
- CRUD operations for records with support for persistent identifier minting.
- Super-fast completion suggesters for implementing Google-like instant autocomplete suggestions.
- The REST API follows best practices and supports, e.g.:
    - Content negotiation and links headers.
    - Cache control via ETags and Last-Modified headers.
    - Optimistic concurrency control via ETags.
    - Rate-limiting, Cross-Origin Resource Sharing, and various security headers.
    - The Search REST API works as the central entry point in Invenio for accessing records.

The REST API in combination with, e.g., Invenio-Search-UI/JS allows to easily display records anywhere in Invenio
and still only maintain one single search endpoint.

For further extending the REST API take a look at the guide in `
Invenio-REST: <http://invenio-rest.readthedocs.io/en/latest/>`_.

The configuration for REST API is located in ``records/config.py`` file.



Records UI
----------

The records UI has configurable views for display records in human readable way. It uses Invenio-PIDStore to resolve
an external persistent identifier into an internal record object. It also has support for displaying tombstones
for deleted records, as well as redirecting an external persistent identifier to another in case e.g. records are merged.

In simple terms, Records-UI works by creating one or more endpoints for displaing records.
You can e.g. have one endpoint (/records/) for displaying bibliographic records and another endpoint (/authors/)
for displaying author records.

One endpoint is bound to one and only one specific persistent identifier type. For instance, /records/ could be bound to integer record identifiers, while /authors/ could be bound to ORCID identifiers.

`Invenio-Records-ui <https://invenio-records-ui.readthedocs.io>`_.

You can pass your custom configuration as a config variables in the ``records/config.py`` file:

.. code-block:: python

    RECORDS_UI_ENDPOINTS = {
        'recid': {
            'pid_type': 'recid',
            'route': '/records/<pid_value>',
            'template': 'records/record.html',
        },
    }
    """Records UI for test-access."""

    SEARCH_UI_JSTEMPLATE_RESULTS = 'templates/records/results.html'
    """Result list template."""

    PIDSTORE_RECID_FIELD = 'id'

    TEST_ACCESS_ENDPOINTS_ENABLED = True
    """Enable/disable automatic endpoint registration."""

Users, roles and identity
-------------------------

First we have subjects which can be granted access to a protected resource.

**User**: Represents an authenticated account in the system.

**Role**: Represents a job function. Roles are created by e.g. system administrators and defined by a name. U
sers can be assigned zero or more roles.

**System role**: Represents special roles that are created and defined by the system and automatically assigned to
users (i.e. system roles cannot be created and defined by system administrators).

During request handling any user (authenticated or unauthenticated) is represented as an identity. The identity is
used to express the set of needs that the current user provides.

**Need**: A need represents the smallest level of access control. It is very generic and can express statements such as
 “has admin role” and “has read access to record 42”.
**Permission**: Represents a set of required needs, any of which should be fulfilled to access a resouce.
 E.g. a permission can combine the two statements above into “has admin role or has read access to record 42”.


Granting access
---------------
Subjects (users, roles and system roles) are assigned actions. E.g. a user or a role can be assigned the action
“read record”. If the action has parameters, then it can be assigned to the subject for any parameters or for a
 specific parameter (e.g. read any record vs read record 42).


.. toctree::
    :maxdepth: 2

    access-and-permissions/records-how-to.rst
    access-and-permissions/use-cases.rst
