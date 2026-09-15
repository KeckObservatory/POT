.. _example_daemon:

Daemon: basics
==============

This example will establish a 'metal' store and items intended to portray
market prices for various precious metals. Refer to the :ref:`unabridged_daemon`
example to see these different aspects of a daemon presented in a single
block of code.


Getting started
---------------

.. py:currentmodule:: mktl

The canonical approach to establish an mKTL daemon involves creating a Python
module to represent all of your custom logic. Each of the steps described here
is required, though there is more than one way to satisfy each step.

In order to successfully run the daemon the Python interpreter being used to
start the daemon must be able to import the mKTL module as well as any custom
module(s) implementing the subclass of :class:`Daemon` that encapsulates
the remainder of the functionality.


:class:`Daemon` subclass
------------------------

The structure of the Python module containing the :class:`Daemon` subclass can
be completely arbitrary; for the sake of this example, the code is contained
in a ``metal`` Python module, and the components described here are in a
``precious`` submodule.

Within the ``precious`` submodule we define our subclass. For no reason other
than convenience it is defined with the name ``Daemon``. The structure of the
``precious.py`` file will be as follows::

    import mktl

    class Daemon(mktl.Daemon):

        def setup(self):
            pass

        def setup_final(self):
            pass


:class:`Item` subclasses
------------------------

The items represent the interface the daemon presents to its clients; the
application-specific functionality that motivates the use of mKTL is exposed
via items, and for most applications this means one or more custom :class:`Item`
subclasses.

An example subclass would have a structure like the following::

    class MarketPriced(mktl.Item):

        def __init__(self, *args, **kwargs):
            mktl.Item.__init__(self, *args, **kwargs)
            # Additional initialization steps would generally follow the
            # regular initialization from the base class. In this case,
            # our market-prices should update once per day:
            self.poll(86400)

        def perform_get(self):
            # Determine the current value for this item and return it.
            pass

        def perform_set(self, new_value):
            # Receive a request to set a new value for this item; return
            # once the request is complete.
            pass

Note in particular the documentation for :func:`Item.perform_get` and
:func:`Item.perform_set`, as it covers the expected behavior of each method.
For our example, the various items are intended to represent the market
spot price of different precious metals. In this case, the
:func:`Item.perform_get` method may request the current value from a website,
and :func:`Item.perform_set` would not be defined, since we don't get to change
the actual market value. To pick one example::


    class Gold(MarketPriced):

        def perform_get(self):
            spot = get_spot_value('gold', 'usd', 'grams')
            spot = float(spot)

            return self.to_payload(spot)



:func:`Daemon.setup` method
---------------------------

:func:`Daemon.setup` is the first pass of application-level setup for the
daemon. This is where instantiation of any custom subclasses of :class:`Item`
needs to
occur, otherwise the various items defined for this daemon within this store
will be populated with default, caching-only instances; this occurs immediately
after the :class:`Daemon` invokes :func:`Daemon.setup`.

Any application-specific initialization is reasonable to include in
:func:`Daemon.setup`, such as initializing a connection to a controller,
especially if it is a pre-requisite for any of the custom :class:`Item`
subclasses. For the example defined here, we only need to instantiate our
custom subclasses for each of the different precious metals for which we are
publishing prices::

    def setup(self):

        self.add_item(Gold, 'GOLD')
        self.add_item(Silver, 'SILVER')
        self.add_item(Platinum, 'PLATINUM')

Note the use of :func:`Daemon.add_item` here when establishing authoritative
:class:`Item` instances. The :func:`Daemon.add_item` method tweaks the
instantiation process such that the :class:`Item` is properly configured as
an authoritative instance, in addition to other tracking local to the
:class:`Daemon` instance that gets leveraged when handling requests.


:func:`Daemon.setup_final` method
---------------------------------

If this store contained any logic that must be executed only after all the items
have been fully instantiated, and/or populated with any previously cached
persistent values, that logic should be invoked in the:func:`Daemon.setup_final`
method. Most daemons will not take advantage of this method; this example daemon
is likewise too simple to require it.

A more complex example, such as a proxy for some other key/value protocol,
might establish all of the :class:`Item` instances, and the in the
:func:`Daemon.setup_final` method, it would take whatever actions are necessary
in the foreign protocol to subscribe to event broadcasts.


:func:`Daemon.cleanup` method
---------------------------------

The :func:`Daemon.cleanup` method will be invoked just prior to shutting down
the daemon; this provides an opportunity for any application-specific cleanup
actions to occur.


Setting values of other items
-----------------------------

This scenario is not relevant to this simple example, but it occurs often
in actual usage. What if you have code that needs to set the value of an
:class:`Item` instance within the context of a single daemon? Not every
item will have a custom subclass defined for it; the default behavior of
a 'caching' item is adequate for a large fraction of authoritative items.
It's common for there to be an item that sets derived values elsewhere,
or sets a family of related item values as a result of an isolated change
elsewhere.

The :func:`Item.get` and :func:`Item.set` methods are inherently client-facing.
While they can be used in a daemon context they will invoke the full mKTL
request handling; in some cases this will be desired, but in the average
case it is not necessary, or desired-- and adds extra overhead.

Updating the value for an authoritative :class:`Item` is done via the
:func:`Item.publish` method. The :py:attr:`Item.value` property, for
authoritative items, will map to this method, as will using
:py:attr:`Item.formatted` or :py:attr:`Item.quantity`. The following
two calls are equivalent in a daemon context::

    self.value = 102.45
    self.publish(102.45)

The same calling semantics are available for other authoritative items within
a daemon. The following block shows two equivalent ways to retrieve a reference
to a locally authoriative item, and similar to the above, two equivalent calls
to publish a new value::

    other = self.store['OTHER_ITEM']
    other = mktl.get('metal.OTHER_ITEM')

    other.value = 33.67
    other.publish(33.67)


Reacting to external events
---------------------------

Event-driven logic involves responding to asynchronous updates. A common
pattern for custom Item subclasses involves watching other mKTL items for
new values, and update the local value (or otherwise respond) whenever one
of those other items publishes a new value.

The canonical way for an item to determine its current value is via the
:func:`Item.perform_get` method, as described above. ``The MarketPriced``
class in the example uses :func:`Item.poll` to drive its updates, but what
if we wanted to respond instantly to external changes? The :func:`Item.watch`
method will register an appropriate callback on a remote item, and effectively
enable asynchonous polling of the local value, invoking the exact same calling
sequence as would occur during a polling operation.

Here's what that would look like in practice::

    class MarketPriced(mktl.Item):

        def __init__(self, *args, **kwargs):
            mktl.Item.__init__(self, *args, **kwargs)

            # Always update at least once per day:
            self.poll(86400)

	    # Update immediately if other market values change.

	    dow = mktl.get('dow_jones.futures')
	    sp500 = mktl.get('sp500.futures')
	    nasdaq = mktl.get('nasdaq.futures')

	    self.watch(dow)
	    self.watch(sp500)
	    self.watch(nasdaq)


        def perform_get(self):
            # Determine the current value for this item and return it.
            pass

        def perform_set(self, new_value):
            # Receive a request to set a new value for this item; return
            # once the request is complete.
            pass


JSON description of items
-------------------------

The :ref:`catalog syntax <catalog>` describing a set of items is a
JSON associative array. When a daemon first starts it must have a complete JSON
description of every item; this forms the core of the catalog block managed by
that daemon, which is responsible for adding the metadata required for proper
client interactions.

The JSON contents can be generated at run time and registered for immediate use
by the daemon. This is the approach taken by the KTL protocol translation
daemon, where the JSON is a repackaging of the configuration metadata supplied
by KTL API calls; with a JSON-like dictionary in hand, the daemon would have
lines like the following in its initialization method::

    def __init__(self, *args, **kwargs):

        items = generate_catalog()
        mktl.meta.authoritative('metal', 'precious', items)
        mktl.Daemon.__init__(self, *args, **kwargs)

The catalog can also be written out as a JSON file, ready
to be used by the daemon. The file can be anywhere, so long as it is accessible
upon startup, after which the file is no longer referenced in any way. The
catalog file is expected to contain a single JSON-formatted dictionary,
with a :ref:`dictionary for each item <items>`. Whitespace is not important,
so long as JSON parsers understand the file contents.

The following is a catalog block appropriate for the items used in this
example:

.. literalinclude:: ./precious.json


Starting the daemon
-------------------

The :ref:`mkregistryd` executable is a persistent application that enables
clients to easily find authoritative mKTL daemons. Having one instance of
:ref:`mkregistryd` running on the local network is recommended.

The :ref:`mkd` executable provides a common entry point for a persistent
daemon. Assuming the default search path is set up correctly, for the example
outlined here the invocation would resemble::

    mkd metal precious --module metal.precious -c precious_metals.json

Both :ref:`mkregistryd` and :ref:`mkd` should be running before mKTL client
interactions are attempted.
