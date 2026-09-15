.. _heater_daemon:

Daemon: heater controller
=========================

This example is intended to demonstrate how a :class:`mktl.Daemon` might be
constructed to communicate with a hardware controller, in this case, a simple
temperature controller, which is capable of reporting a temperature value,
heater output, and allows a heater setpoint to be established.

A typical controller would have many more commands beyond what is presented
here, but the resulting content of the daemon could be an extension of the
structure shown here.


heater.py
---------

.. literalinclude:: ../../examples/heater/heater.py

