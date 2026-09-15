.. _example_get:

Client: get a value
===================

This example will use the 'population' store and the item 'CRAZY'. The
objective is to retrieve the current value for 'population.CRAZY'.


.. _getting_started:

Getting started
---------------

.. py:currentmodule:: mktl

An :class:`Item` instance is required to perform any client operations; the
:func:`get` method should be invoked to acquire a cached singleton for
any/all subsequent use; key names are case insensitive. The required
boilerplate is short::

    import mktl
    crazy = mktl.get('population.CRAZY')
    crazy = mktl.get('population', 'CRAZY')
    crazy = mktl.get('population', 'crazy')
    crazy = mktl.get('PoPuLaTiOn', 'crazy')

The ``crazy`` reference here will be identical for each different invocation
shown, always returning the same object-- not copies of the same mKTL item,
the exact same Python object. The important part is that we now have a
:class:`Item` that can be used for subsequent calls. A ``ValueError`` exception
will be raised if no
catalog is available for that store (i.e., the store does not exist);
a ``KeyError`` exception will be raised if the key does not exist.


The :py:attr:`Item.value` property
----------------------------------

Referencing the :py:attr:`Item.value` property is the preferred approach to
retrieve the current value of an item. The getter method behind the property
will retrieve the value if one is not already available, though this is not
expected to be the average case-- a client-side :class:`Item` instance will
automatically call :func:`Item.subscribe` when it is instantiated, and should
thus always have a value.

The property is used directly::

    current = crazy.value

There are two variants of the :py:attr:`Item.value` property:
:py:attr:`Item.formatted`, and :py:attr:`Item.quantity`. They behave in the same
fashion as :py:attr:`Item.value`, but work with different representations of
the value. The exact nature of what you get back will depend on the
description of the individual item, but for this example you might see::

    >>> print(crazy.value)
    True
    >>> print(crazy.formatted)
    'Yes'
    >>> print(crazy.quantity)
    Traceback (most recent call last):
    [...]
    TypeError: Invalid magnitude for Quantity: True


Calling :func:`Item.get`
------------------------

:func:`Item.get` offers additional options if the default behavior using the
:py:attr:`Item.value` property is not adequate. Most exceptions do not need
to be caught if you are calling :func:`Item.get`
directly, unless the caller expects to operate in a regime where the
authoritative daemon is offline and the application will retry at a later time.
This section shows exception handling for completeness's sake::

    try:
        current = crazy.get()
    except zmq.ZMQError:
        # The answering daemon could not be contacted. The timeout for the
        # initial request is short, just 0.05 seconds, so a failure of this
        # type should likewise be fast.
        raise

With no arguments the call to :func:`Item.get` will return the locally
cached value if the item is subscribed to broadcasts, or request the value
from the authoritative daemon if it is not-- the same behavior one would get
by using the :py:attr:`Item.value` property. The caller can bypass the local
cache if it is relevant for their application, explicitly requesting that
the daemon update (and broadcast) the current value while handling this call::

    current = crazy.get(refresh=True)

For nearly all applications the ``refresh`` argument does not need to be
specified.


Handling return values
----------------------

The reference at the :py:attr:`Item.value` property as well as the value
returned by :func:`Item.get`, is a Python native object, either a
primitive, such as an integer or floating point number, or something like a
numpy array. For example::

    >>> current
    True

If no value is availble the current value will be ``None``.


Full example
------------

Putting it all together::

    import mktl
    crazy = mktl.get('population.CRAZY')
    craziness = crazy.value

    if craziness is None:
        print('The population craziness is unknown.')
    elif craziness == True:
        print('The population is crazy.')
    else:
        print('The population is sane.')

The above comparison can be further simplified:
an :class:`Item` instance can be used directly in comparison operations,
and will behave as if :py:attr:`Item.value` is being used directly::

    import mktl
    crazy = mktl.get('population.CRAZY')

    if crazy == None:
        print('The population craziness is unknown.')
    elif crazy == True:
        print('The population is crazy.')
    else:
        print('The population is sane.')

Note that the first comparison did not use ``is None``, and instead used
``== None``, which is a departure from generally accepted practices in the
Python community. This is because the behavior of the ``is`` operator is
unchanged, and will compare the mktl.Item instance against None, whereas
the equality operator will trigger the rich comparison handling provided
by an mktl.Item, and instead compare the .value property against None.

Speaking of generally accepted practices in the Python community, the
first condition can be rearranged slightly to use a tighter set of
comparisons, omitting the explicit comparison against a boolean::

    import mktl
    crazy = mktl.get('population.CRAZY')
    craziness = crazy.value

    if craziness:
        print('The population is crazy.')
    elif craziness is None:
        print('The population craziness is unknown.')
    else:
        print('The population is sane.')

This also works for the second style above, which, depending on your choice
of variable names, may wind up being the most readable, in addition to being
the most concise::

    import mktl
    crazy = mktl.get('population.CRAZY')

    if crazy:
        print('The population is crazy.')
    elif crazy == None:
        print('The population craziness is unknown.')
    else:
        print('The population is sane.')

