
.. _client:

Client interface
================

A client is any software component interacting with a daemon, in the form of
issuing requests, receiving responses to requests, and receiving asynchronous
updates of new item values. The Python interface described here makes liberal
use of background threads, callbacks in particular can and will arrive from
background threads as asynchronous updates arrive.

Unlike the :ref:`daemon`, a typical client application will not need subclasses
of the classes defined here, they are expected to be used directly, as-is.


Getting started
---------------

mKTL will cache information locally; there are no restrictions on where this
information gets cached, other than it needs to be cached somewhere. The
default location for this cache is::

    $HOME/.mKTL

You can override this location by setting the ``MKTL_HOME`` environment
variable to the absolute path of your choice; any directory specified in
this fashion will be created if it does not already exist. Alternatively,
the program may invoke the :func:`mktl.home` method to specify the location
at run time.

.. autofunction:: mktl.home

You may need to pre-populate your local cache with information from the
registries handling the catalog for the store(s) you wish to access.
The :ref:`mk` command line tool will handle this process; for a given
IP address or hostname, you would invoke::

    mk discover 192.168.5.34

This discovery process will query that address for all available catalogs,
but more importantly, will cache the address of that registry and use it again
in the future if/when discovering new services, without an explicit request to
do so. In other words, it should only be necessary to run this discovery process
a single time in order to gain access to a new set of mKTL stores.

The :func:`mktl.get` method is the universal entry point to retrieve a
:class:`mktl.Store` or :class:`mktl.Item` instance; cached metadata is
automatically refreshed if necessary, and the remainder of the connection logic
is handled by the :class:`mktl.Store`.

All other client operations, such as getting and setting item values, are
handled via the :class:`mktl.Item` instance.

.. autofunction:: mktl.get
