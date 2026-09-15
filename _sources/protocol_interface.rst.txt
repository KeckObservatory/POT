
.. py:module:: mktl.protocol

Protocol interface
==================

Each aspect of the mKTL protocol is split into two different components: the
client-facing half, and a server-facing half. The interfaces described here
are generally not exposed to direct usage, they support the end-user
functionality exposed in the :class:`mktl.Item` class.

.. py:module:: mktl.protocol.message

Payload class
-------------

.. autoclass:: Payload
   :members:


Message classes
----------------

.. autoclass:: Message
   :members:

   .. automethod:: _finalize

.. autoclass:: Broadcast
   :members:

.. autoclass:: Request
   :members:


.. py:module:: mktl.protocol.request

Request client
--------------

.. autofunction:: client

.. autofunction:: send

.. autoclass:: Client
   :members: send

Request server
--------------

.. autoclass:: Server
   :members:


.. py:module:: mktl.protocol.publish

Publish client
--------------

.. autofunction:: client

.. autoclass:: Client
   :members:

Publish server
--------------

.. autoclass:: Server
   :members: publish

