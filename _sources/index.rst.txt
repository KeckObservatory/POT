.. _introduction:

Introduction
------------

.. include:: ../README.md

The source code for mKTL is maintained in a GitHub repository:

https://github.com/KeckObservatory/mKTL


Documentation contents
----------------------

This documentation covers the use of mKTL from several different perspectives,
where awareness of every perspective is not strictly required in order to
effectively leverage mKTL. The :ref:`client` focuses on the needs of ephemeral
applications that wish to query a daemon; the :ref:`daemon` focuses on
the needs of a persistent application providing a structured interface to some
well-defined entity, whether it is logical, hardware, or otherwise; and lastly,
the maintainer view, focusing on mKTL internals such as the complete description
of the mKTL :ref:`protocol` and :ref:`metadata`. To put it another way,
the intent of mKTL is that users of the interface code should not need to
be expert mKTL maintainers in order to successfully develop an application.


.. toctree::
   :maxdepth: 2

   overview
   client
   daemon
   store
   item
   examples
   executables
   metadata
   built_ins
   protocol
   protocol_interface
   heritage
   code_of_conduct
   governance
   glossary

Indices and tables
------------------

* :ref:`genindex`
* :ref:`search`
