Base64 library
**************

.. current-library:: base64
.. current-module:: base64

The base64 Module
=================

.. function:: base64-encode

   Return an encoded :drm:`<byte-string>` in base 64.

   :signature: base64-encode (string, #key encoding) => (string)
   :parameter string: An instance of :drm:`<byte-string>`
   :parameter #key encoding: Either ``#"standard"`` or ``#"http"``,
                             default ``#"standard"``.
   :value string: An instance of :drm:`<byte-string>`.
   :example:

      .. code-block:: dylan

	let original-string = "Many hands make light work";
        base64-encode(original-string);

	// Returns "TWFueSBoYW5kcyBtYWtlIGxpZ2h0IHdvcmsu"

.. function:: base64-decode

   Return an decoded :drm:`<byte-string>` from an encoded one.

   :signature: base64-encode (string, #key encoding) => (string)
   :parameter string: An instance of :drm:`<byte-string>`
   :parameter #key encoding: Either ``#"standard"`` or ``#"http"``,
                             default ``#"standard"``.
   :value string: An instance of :drm:`<byte-string>`.
   :example:

      .. code-block:: dylan

        base64-decode("TWFueSBoYW5kcyBtYWtlIGxpZ2h0IHdvcmsu");

	// Returns "Many hands make light work"
