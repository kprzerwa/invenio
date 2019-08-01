Marshmallow migration to version 3
==================================

Marshmallow
-----------

Marhsmallow is a package for serialization and deserialization of complex data structures from and to Python types. You can discover it's usage within Invenio `here <https://invenio.readthedocs.io/en/latest/tutorials/understanding-data-models.html?highlight=marshmallow#define-a-marshmallow-schema>`_


v3.0.0
------

If you are using or you are about to use Marshmallow in your repository implementation, you should know that the package is about to be upgraded to version 3.x. There have been significant changes in the code which will influence already existing code, and you should be aware of them.

> The full guide is available in the Marshmallow `documentation <https://invenio.readthedocs.io/en/latest/tutorials/understanding-data-models.html?highlight=marshmallow#define-a-marshmallow-schema>`_.

We are upgrading the Invenio modules to be compatible with Marshmallow v3.x. **We will provide transition period in which the modules will support both v2.3.x and v3.x implementations of Marshmallow**. Deprecated methods will be marked with warnings to provide you with time to adjust your code to the latest changes.


Guide
-----

After upgrading Marshmallow 3 in your dependencies you might find some of your code failing. Mostly it will be connected with schema validation and `webargs <https://webargs.readthedocs.io/en/latest/quickstart.html>`_ package. Webargs in version 5.4.0 is compatible with Marshmallow 3.x.

The biggest change is the return type of ``Schema().dump()`` and ``Schema().load()`` (also ``dumps`` and ``loads``. They do not return the ``(data, errors)`` named tuple and the ``ValidationError`` is raised instead.
In this case if you were using ``.errors`` attribute, it will no longer work - you need to implement your own handlers for the type ``marshmallow.ValidationError``.
Alternatively by doing ``Schema.validate()`` you can still get a list of the errors, an example is
located `here <https://marshmallow.readthedocs.io/en/3.0/quickstart.html#schema-validate>`_.

Other significant changes and solutions

- ``ValidationError`` - usually result of the schema becoming strict.
    - ``Unknown field.`` indicates that your schema is trying to validate data containing a field which was not defined in the schema. You can either add this field to the schema or investigate the origin of the field in the data.

    - ``Not a valid type.`` indicates that the field has data of different type than defined in the schema

    - ``Missing field.`` indicates that field defined as required is mssing in the data.
- ``JSON object has no attribute data``
    Marshmallow schema provides methods ``.dump(..)``, ``.dumps(..)`` which no longer return namedtuple of ``(data, errors)``. They return json object only (formerly ``data``). All the errors are raised not returned.

- Your test data is not read properly with using webargs.
   You are probably using old parameters when initialising the arguments.

.. code-block:: python

       'part_number': fields.Int(
        load_from='partNumber',
        location='query',
        required=True,
    ),


change all ``load_from`` and ``dump_to`` to ``data_key``.






