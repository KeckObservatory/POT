Examples
========

If you obscure most of the details, mKTL is fundamentally a distributed system
for event-driven applications. The :ref:`daemon` is built around the handling
and generation of events, the :ref:`client` is built to be the mirror of that
functionality. While mKTL can be used in a purely procedural fashion, usage
patterns that focus on an event-driven approach will be most closely aligned
with the design goals of mKTL.

The examples provided here are intended to highlight fundamental use cases for
mKTL in a production environment. All of the examples assume the mKTL module
is installed and the correct Python interpreter is being invoked; in other
words, that ``import mktl`` will succeed.


.. toctree::
   :maxdepth: 2

   example/get
   example/set
   example/callback
   example/daemon
   example/daemon_unabridged
   example/heater
