.. _item:

The Item class
==============

.. autoclass:: mktl.Item

   The bulk of mKTL interactions involve the :class:`mktl.Item` class. In
   addition to the methods described here an :class:`mktl.Item` instance
   can be used with Python operators, such as addition/concatenation or
   multiplication.

   The behavior of an :class:`mktl.Item` when used in this fashion will be
   aligned with the native Python binary type for the item value; for example,
   if an item test.BAR has an integer value 12, ``test.BAR + 5`` will return
   the integer value 17. If test.BAR is instead the string value '12', the
   same operation would raise a TypeError exception; however, ``test.BAR + '5'``
   would return the string value '125', just like you would expect for string
   concatenation. In-place operators will set the current value of the item.

   Three properties allow unified access to getting and setting an item's
   value, regardless of whether the local application is authoritative for
   that specific item. The use of properties implies default behavior for
   both :func:`get` and :func:`set` in a client context, and for :func:`publish`
   in a daemon context. The use of these properties is encouraged as the
   preferred approach to getting and setting item values; any functionality
   beyond the default behavior requires using the methods directly.

   .. autoproperty:: value
   .. autoproperty:: formatted
   .. autoproperty:: quantity

   .. automethod:: get
   .. automethod:: set
   .. automethod:: register


   .. _item_daemon_methods:

   All of the client functionality remains unchanged when an :class:`Item`
   is used in a daemon context. The following methods have utility when
   a daemon is authoritative for an item.

   .. automethod:: add_performer
   .. automethod:: add_get_performer
   .. automethod:: add_set_performer
   .. automethod:: from_payload
   .. automethod:: poll
   .. automethod:: publish
   .. automethod:: to_payload
   .. automethod:: watch


   .. _item_subclassing:

   Some of the daemon-specific methods are intended to be overridden as
   part of implementing custom, application-specific logic. It may not
   be necessary to override all of these methods depending on the needs
   of the application.

   .. automethod:: perform_get
   .. automethod:: perform_poll
   .. automethod:: perform_set
   .. automethod:: validate

   The *req_\*()* set of methods can be overridden, but for nearly all
   applications their use is strictly internal and further customization
   should not be necessary.

   .. automethod:: req_get
   .. automethod:: req_set

