.. _executables:

Executables
===========

.. _mk:

mk
--

`mk` is intended to be the primary command line interface for client
interactions with mKTL. The command line arguments describe its usage:

.. literalinclude:: ./mk.txt
   :language: none


.. _mkd:

mkd
---

The `mkd` executable provides a command-line interface to invoke a
persistent daemon executing a :class:`mktl.Daemon` subclass to implement
application-specific functionality. The `mkd` executable is intended
to be the common point of entry for any Python-based mKTL daemon.

The command line arguments describe its usage:

.. literalinclude:: ./mkd.txt
   :language: none


.. _mkregistryd:

mkregistryd
-----------

The `mkregistryd` persistent daemon is a discovery aid, listening for UDP
broadcasts on a well-known port number so that clients can be directed to
a specific mKTL daemon handling requests for a specific store, or fraction
of a store. While having a `mkregistryd` daemon running is not a strict
requirement it is a key component of automated discovery of mKTL stores on
a local network.
