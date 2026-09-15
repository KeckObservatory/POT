Client: register a callback
===========================

This example will use the 'oven' store and the item 'TEMP'. The objective is
to handle all broadcasts for 'oven.TEMP'; a callback method will be defined
that performs any/all desired operations whenever the value changes. This
callback method will be invoked asynchronously whenever a new broadcast
arrives. Callbacks arriving via these mechanisms will not be serialized
across items, though within an item they will be invoked in the order they
were originally registered. If you have the same callback method registered
with multiple items there is no inherent mKTL-based guarantee to prevent
that method from being called multiple times simultaneously.

The toy example here will calculate and print an exponentially weighted
average.


Getting started
---------------

.. py:currentmodule:: mktl

See the :ref:`getting_started` section of the :ref:`example_get` example for
more details. We'll get right to it::

    import mktl
    temp = mktl.get('oven.TEMP')


Defining a callback
-------------------

The expected signature of a callback method is::

    def my_callback(item, new_value, new_timestamp):

These arguments are always passed to the callback; that doesn't necessarily
mean the callback has to use them, a common pattern is to define a callback
that ignores the arguments provided, an approach made easier by the use of
:func:`get`. First, a callback that uses the arguments::

    def average(item, temp, time):

        factor = 0.01
        new_weight = factor
        old_weight = 1 - factor

        if average.computed is None:
            average.computed = temp
        else:
            new = new_weight * temp
            old = old_weight * average.computed
            average.computed = new + old

        print("%.3f %s average: %.1f" % (time, item.key, average.computed)

    average.computed = None


...and an alternate version, ignoring the arguments::

    def average(*args, **kwargs):

        temp = mktl.get('oven', 'TEMP')

        factor = 0.01
        new_weight = factor
        old_weight = 1 - factor

        if average.computed is None:
            average.computed = float(temp)
        else:
            new = new_weight * temp
            old = old_weight * average.computed
            average.computed = new + old

        timestamp = temp.timestamp
        print("%.3f %s average: %.1f" % (timestamp, temp.key, average.computed)

    average.computed = None


These two approaches are functionally identical for this simple example.
The second approach, relying on :func:`get`, becomes appealing
when multiple items need to be inspected in a given callback; for example,
if the current temperature were being compared to the current setpoint.
There is no provision for calling a callback with arguments from multiple
items, a single invocation of a callback is only ever triggered by a
broadcast event associated with a single item.

It is possible to block up a queue of events if the events arrive more
rapidly than their callbacks can be processed. Each different :class:`Item`
has its own processing queue on the client side, and events are processed
sequentially on a per-item basis. Callbacks should avoid delays in order
to prevent the local queue from backing up.


Calling :func:`Item.register`
-----------------------------

Once the callback method is defined it needs to be associated with the
:class:`Item` instance, so that the callback is invoked every time the value
of that item changes. This is accomplished via :func:`Item.register`::

    temp.register(average)

:func:`Item.register` will invoke :func:`Item.subscribe` if necessary,
though all client-facing :class:`Item` instances generally invoke
:func:`Item.subscribe` when they are first instantiated.


Full example
------------

Putting it all together::

    import mktl
    import time
    temp = mktl.get('oven.TEMP')

    def just_print(*args, **kwargs):

        temp = mktl.get('oven.TEMP')
        value = float(temp)
        time = temp.timestamp
        print ("%.3f oven.TEMP: %.1f" % (time, value))

    def average(*args, **kwargs):

        temp = mktl.get('oven', 'TEMP')

        factor = 0.01
        new_weight = factor
        old_weight = 1 - factor

        if average.computed is None:
            average.computed = float(temp)
        else:
            new = new_weight * temp
            old = old_weight * average.computed
            average.computed = new + old

        timestamp = temp.timestamp
        print("%.3f %s average: %.1f" % (timestamp, temp.full_key, average.computed)

    average.computed = None


    temp.register(just_print)
    temp.register(average)
    time.sleep(30)

