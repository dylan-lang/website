Usage
*****

.. current-library:: base64
.. current-module:: base64

See also :doc:`reference`


Quickstart
----------

Add ``use base64;`` to your library and module definitions.

This library exports two functions:

-  :func:`base64-encode`
-  :func:`base64-decode`

Both functions accept a :drm:`<byte-string>` and return a :drm:`<byte-string>`.

Types of encoding
-----------------

The functions have two types of encoding/decoding:

-  ``#"standard"`` (default) and

-  ``#"http"``

The main difference between them is the padding characters used. You
can choose the type with the ``encoding:`` parameter (see example
below).

Example
-------

Standard encoding/decoding
~~~~~~~~~~~~~~~~~~~~~~~~~~

Here is an example of usage of the standard encoding/decoding:

.. code:: dylan

   // Example string to encode
   let original-string = "Many hands make light work.";

   // Encoding the string to base64 standard
   let encoded-standard = base64-encode(original-string);
   format-out("Encoded string: %=\n", encoded-standard);

   // Shows in output
   // Encoded string: "TWFueSBoYW5kcyBtYWtlIGxpZ2h0IHdvcmsu"

   // Decoding the string in base64 standard
   let decoded-string = base64-decode(encoded-standard);
   format-out("Decoded string: %=\n", decoded-string);

   // Shows in output
   // Decoded string: "Many hands make light work."

HTTP encoding/decoding
~~~~~~~~~~~~~~~~~~~~~~

To show the HTTP encoding/decoding we will use a text that forces the
padding (base64 encoding uses padding to ensure that the length of the
encoded string is a multiple of 4 bytes).

.. code:: dylan

   // Example string to encode, note the missing dot at the end
   let original-string = "Many hands make light work";

   // Encoding the string to base64 http
   let encoded-http = base64-encode(original-string, encoding: #"http");
   format-out("Encoded string: %=\n", encoded-http);

   // Shows in output (note the padding character '@')
   // Encoded string: "TWFueSBoYW5kcyBtYWtlIGxpZ2h0IHdvcms@"

   // Decoding the string in base64 http
   let decoded-string = base64-decode(encoded-http, encoding: #"http");
   format-out("Decoded string: %=\n", decoded-string);

   // Shows in output
   // Decoded string: "Many hands make light work"
