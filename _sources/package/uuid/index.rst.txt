****************
The UUID library
****************

.. current-library:: uuid

UUIDs are "Universally Unique IDentifiers". They are immutable objects
representing 128 bits of data.

In addition to constructing UUIDs from raw bytes or strings, it also
provides for creating version 3, 4 or 5 UUIDs as defined in `RFC 4122`_.

UUID creation functions:

- :class:`<uuid>`
- :gf:`make-uuid3`
- :gf:`make-uuid4`
- :gf:`make-uuid5`
- :meth:`as(<uuid>, <string>)`

The most common way of creating a UUID will likely be :gf:`make-uuid4`
to construct a random UUID.

The UUID module
===============

.. current-module:: uuid

.. class:: <uuid>

   :superclasses: :class:`<object>`

   :keyword required data: An instance of ``limited(<byte-vector>, size:, 16)``.

   :description:

     This class holds the data for a UUID. UUIDs are typically constructed with
     the functions :gf:`make-uuid3`, :gf:`make-uuid4`, or :gf:`make-uuid5`.

     UUIDs can also be created from a string using :meth:`as(<uuid>, <string>)`.

   :seealso:

     - :gf:`make-uuid3`
     - :gf:`make-uuid4`
     - :gf:`make-uuid5`
     - :meth:`as(<uuid>, <string>)`

.. constant:: $nil-uuid

   :type: :class:`<uuid>`
   :value: `make(<uuid>, data: make(<byte-vector>, size: 16, fill: 0))`

   :description:

     This is a UUID where all of the bits are ``0``.

Creation
--------

.. generic-function:: make-uuid3

   :signature: make-uuid3 (namespace name) => (uuid)

   :parameter namespace: An instance of :class:`<uuid>`.
   :parameter name: An instance of :class:`<string>`.
   :value uuid: An instance of :class:`<uuid>`.

   :description:

     Construct a version 3 UUID that is the MD5 hash of a *namespace*
     and *name*. Predefined namespaces are available as :const:`$namespace-dns`,
     :const:`$namespace-url`, :const:`$namespace-iso-oid`, and
     :const:`$namespace-x500`.

   :example:

     .. code-block:: dylan

        let uuid = make-uuid3($namespace-url,
                              "http://github.com/dylan-foundry/uuid");

   :seealso:

     - :gf:`make-uuid4`
     - :gf:`make-uuid5`
     - :const:`$namespace-dns`
     - :const:`$namespace-url`
     - :const:`$namespace-iso-oid`
     - :const:`$namespace-x500`

.. generic-function:: make-uuid4

   :signature: make-uuid4 () => (uuid)

   :value uuid: An instance of :class:`<uuid>`.

   :description:

     Construct a version 4 UUID that uses random data.

   :example:

     .. code-block:: dylan

        let uuid = make-uuid4();

   :seealso:

     - :gf:`make-uuid3`
     - :gf:`make-uuid5`

.. generic-function:: make-uuid5

   :signature: make-uuid5 (namespace name) => (uuid)

   :parameter namespace: An instance of :class:`<uuid>`.
   :parameter name: An instance of :class:`<string>`.
   :value uuid: An instance of :class:`<uuid>`.

   :description:

     Construct a version 5 UUID that is the SHA1 hash of a *namespace*
     and *name*. Predefined namespaces are available as :const:`$namespace-dns`,
     :const:`$namespace-url`, :const:`$namespace-iso-oid`, and
     :const:`$namespace-x500`.

   :example:

     .. code-block:: dylan

        let uuid = make-uuid5($namespace-dns, "opendylan.org");

   :seealso:

     - :gf:`make-uuid3`
     - :gf:`make-uuid4`
     - :const:`$namespace-dns`
     - :const:`$namespace-url`
     - :const:`$namespace-iso-oid`
     - :const:`$namespace-x500`

Conversion
----------

.. method:: as
   :specializer: <uuid>, <string>

   :signature: as(<uuid>, *string*) => *uuid*

   :parameter type: This must be :class:`<uuid>`.
   :parameter string: An instance of :class:`<string>`.
   :value uuid: An instance of :class:`<uuid>`.

   :description:

     Convert a string into a UUID. The string must only contain
     hexadecimal digits and, excluding dashes, must only 32
     characters long.

   :example:

     .. code-block:: dylan

        let uuid = as(<uuid>, "4699DE5F-1F40-41B8-AB8D-55CCA1A6C9E9");

.. method:: as
   :specializer: <string>, <uuid>

   :signature: as(<string>, *uuid*) => *string*

   :parameter type: This must be :class:`<string>`.
   :parameter uuid: An instance of :class:`<uuid>`.
   :value string: An instance of :class:`<string>`.

   :description:

     Convert a UUID into a human readable string.

   :example:

     .. code-block:: dylan-console

        ? as(<string>, uuid)
        => "4699DE5F-1F40-41B8-AB8D-55CCA1A6C9E9"

Miscellaneous
-------------

.. constant:: $namespace-dns

   :type: :class:`<uuid>`
   :value: ``as(<uuid>, "6ba7b810-9dad-11d1-80b4-00c04fd430c8")``

   :description:

     A predefined namespace UUID that can be used with :gf:`make-uuid3` or
     :gf:`make-uuid5`. This namespace UUID is as it is specified in `RFC 4122`_.

     When used to create a UUID, the associated *name* should be a fully
     qualified domain name.

   :seealso:

     - :gf:`make-uuid3`
     - :gf:`make-uuid5`
     - :const:`$namespace-url`
     - :const:`$namespace-iso-oid`
     - :const:`$namespace-x500`

.. constant:: $namespace-iso-oid

   :type: :class:`<uuid>`
   :value: ``as(<uuid>, "6ba7b812-9dad-11d1-80b4-00c04fd430c8")``

   :description:

     A predefined namespace UUID that can be used with :gf:`make-uuid3` or
     :gf:`make-uuid5`. This namespace UUID is as it is specified in `RFC 4122`_.

     When used to create a UUID, the associated *name* should be an ISO OID.

   :seealso:

     - :gf:`make-uuid3`
     - :gf:`make-uuid5`
     - :const:`$namespace-dns`
     - :const:`$namespace-url`
     - :const:`$namespace-x500`

.. constant:: $namespace-url

   :type: :class:`<uuid>`
   :value: ``as(<uuid>, "6ba7b811-9dad-11d1-80b4-00c04fd430c8")``

   :description:

     A predefined namespace UUID that can be used with :gf:`make-uuid3` or
     :gf:`make-uuid5`. This namespace UUID is as it is specified in `RFC 4122`_.

     When used to create a UUID, the associated *name* should be a URL.

   :seealso:

     - :gf:`make-uuid3`
     - :gf:`make-uuid5`
     - :const:`$namespace-dns`
     - :const:`$namespace-iso-oid`
     - :const:`$namespace-x500`

.. constant:: $namespace-x500

   :type: :class:`<uuid>`
   :value: ``as(<uuid>, "6ba7b814-9dad-11d1-80b4-00c04fd430c8")``

   :description:

     A predefined namespace UUID that can be used with :gf:`make-uuid3` or
     :gf:`make-uuid5`. This namespace UUID is as it is specified in `RFC 4122`_.

     When used to create a UUID, the associated *name* should be an X.500 DN
     in DER or a text output format.

   :seealso:

     - :gf:`make-uuid3`
     - :gf:`make-uuid5`
     - :const:`$namespace-dns`
     - :const:`$namespace-url`
     - :const:`$namespace-iso-oid`

.. generic-function:: rfc4122-variant?

   :signature: rfc4122-variant? (uuid) => (res)

   :parameter uuid: An instance of :class:`<uuid>`.
   :value res: An instance of :class:`<boolean>`.

   :description:

     Identify whether or not the given *uuid* is one of the
     variants defined in `RFC 4122`_.

   :example:

     .. code-block:: dylan-console

        ? let uuid = make-uuid4();
        ? rfc4122-variant?(uuid)
        => #t

.. generic-function:: rfc4122-version

   :signature: rfc4122-version (uuid) => (res)

   :parameter uuid: An instance of :class:`<uuid>`.
   :value res: An instance of :class:`<integer>`.

   :description:

     If the given *uuid* is one of the variants defined in `RFC 4122`_,
     this will return which version it is.

     If the *uuid* is not a variant from `RFC 4122`_, the results of this
     function are undefined.

   :example:

     .. code-block:: dylan-console

        ? let uuid = make-uuid4();
        ? rfc4122-version(uuid)
        => 4

.. generic-function:: uuid-data

   :signature: uuid-data (uuid) => (data)

   :parameter uuid: An instance of :class:`<uuid>`.
   :value data: An instance of ``limited(<byte-vector>, size:, 16)``.

   :description:

     Return the byte vector containing the immutable data for the
     *uuid*.

.. _RFC 4122: https://tools.ietf.org/html/rfc4122.html
