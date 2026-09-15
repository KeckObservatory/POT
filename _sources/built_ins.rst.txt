
.. _builtins:

Built-in items
==============

A small set of items are defined for metadata and other internal use.
There are two classes of built-in items: a handful that are only accessible
at the protocol level, and the remainder that are visible as fully functional
items in every way. The key for a built-in item has a leading underscore to
distinguish it from other items; regular items should refrain from using a
leading underscore.


Protocol only
-------------

The ``_hash`` and ``_catalog`` items are replacements for the HASH and CONFIG
request types that were defined in the initial prototype of the mKTL protocol.

.. list-table::

  * - *Key*
    - *Description*

  * - **_hash**
    - Request the current hash identifiers for any known catalog blocks
      of a single mKTL store. All available hash identifiers, for all known
      stores, will be returned for a query of the bare ``_hash`` item; with a
      store name prefix, such as `kpfguide._hash`, only the hashes for that
      specific store will be returned. An error will be returned if a store
      is requested and the responding daemon does not have a cached
      catalog for that store.

      To rephrase, the scope for a ``_hash`` item is at two levels:

        * ``_hash``: returning all locally known hash data
        * ``store._hash``: returning only the hash data for the requested store

      The hash is 32 hexadecimal integers. The actual hash format is not
      significant, as long as the source of authority is consistent about
      which hash format it uses, and the format can be transmitted as 32
      hexadecimal integers.

      All ``_hash`` items are read-only, and can only receive GET requests.

      To unify processing the response value is always a dictionary of
      dictionaries, even if only one hash is available.

      Example response values::

        {'kpfguide': {'uuid1': 0x84a30b35...,
                      'uuid2': 0x983ae10f...}}

        {'kpfguide': {'uuid1': 0x84a30b35...,
                      'uuid2': 0x983ae10f...},
         'kpfmet': {'uuid6': 0xe0377e7d...,
                    'uuid7': 0x7735a20a...,
                    'uuid8': 0x88645dab...,
                    'uuid9': 0x531c14fd...}}


  * - **_catalog**
    - .. _builtin_catalog:

      The full locally known catalog contents for a single mKTL store.
      There is no bare ``_catalog`` item available to request the catalog
      for all locally known stores; the key used for GET requests will
      always include the store name, such as `kpfguide._catalog`.

      The scope for a ``_catalog`` item is at two levels:

        * ``store._catalog``: represents the catalog for the requested store
        * ``store._aliascat``: represents the catalog for a single daemon

      A typical client interaction will request the catalog hash first,
      and if the hash for the cached local copy is not a match, request the
      full contents from the daemon to update the local cache.

      The catalog syntax is not fully described here, this is just
      a description of the request. See the
      :ref:`catalog syntax documentation <catalog>` for a full description
      of the data format.

      A SET operation on a ``_catalog`` item should only originate from an
      authoritative daemon; as such, it will be treated as an announcement,
      and any errors raised by the recipient have to be treated as a rejection
      of the announcement.


These two item definitions are critical for the exchange of their respective
data; both are integral to the initial handshake between a client and a daemon,
and between any intermediaries aiding in the discovery process.


Regular items
-------------

The remaining built-in items are visible in the catalog for the store,
and are available for each daemon in that store. They are intended to represent
metadata about an individual daemon, will all have the store name and alias
as the first elements of the key, including a leading underscore after the
store name; for example, ``kpfguide._disp1``. The built-in items append a
functional suffix to this common prefix, yielding full keys like
``kpfguide._disp1cfg`` and ``kpfguide_disp1clk``.

The built-in items defined in this fashion are typically read-only, though
there are some exceptions.


.. list-table::

  * - *Key suffix*
    - *Description*

  * - **cat**
    - See :ref:`the description <builtin_catalog>` of the ``_catalog`` item above.

  * - **clk**
    - The current uptime for this daemon, measured in seconds. The value will
      update and publish on a 1 Hz basis and can be used as a heartbeat
      indicator to assess whether the daemon is running normally.

  * - **cpu**
    - An instantaneous measurement of the processor consumption by this daemon.
      This value will udpate and publish on a 1 Hz basis.

  * - **dev**
    - A terse description of the function of this daemon. This item is settable,
      and its value will persist across restarts.

  * - **host**
    - The locally defined hostname for where this daemon is running.
      No guarantees are made about the external validity of this name.

  * - **mem**
    - An instantaneous measurement of the physical memory consumption by this
      daemon. This value will udpate and publish on a 1 Hz basis.

  * - **pid**
    - The proces identifier for this daemon.

  * - **uuid**
    - The universally unique identifier (UUID) for this daemon; this identifier
      will persist across restats.

