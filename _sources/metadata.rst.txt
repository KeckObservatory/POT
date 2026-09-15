.. _metadata:

JSON metadata
=============

The catalog syntax describes what it means to be a mKTL store,
enumerating the available items and all their intrinsic metadata. This
document lays out the configuration syntax, as might be returned from
a _catalog GET request or loaded from a local cache on disk, and additional
conventions around the handling of a catalog.


.. _catalog:

Catalog syntax
--------------

Each daemon will have a single catalog block for which it is authoritative.
This block will define any/all metadata associated with the daemon, including
the full description of each item belonging to that daemon.

A store may be composed of multiple daemons. Each daemon will contribute its
own block to the store's full catalog; the full catalog is represented as
a JSON dictionary, with each catalog block keyed by the unique identifier
(UUID) associated with its respective daemon.

For example, the 'kpfguide' store may contain multiple daemons, and therefore
multiple catalog blocks::

	{'uuid1': {'store': 'kpfguide', 'time': 1724892333.924, ...},
	 'uuid2': {'store': 'kpfguide', 'time': 1725892343.567, ...}}

Each catalog block will contain the following fields:

=============== ===============================================================
*Field*         *Description*
=============== ===============================================================
**store**	The name of the store. This is perhaps redundant,
		being implied by the structure of the catalog
		block, but the extra assertion is inexpensive and
		convenient.

**uuid**	The unique identifier associated with this block.
		The UUID is generated internally and does not need
		to be manipulated directly; it is used to uniquely
		associate a specific daemon with its catalog
		block, as might be necessary when a client needs
		to apply continuity (such as clearing local cache)
		when a remotely-served catalog block changes.

**alias**	A recognizable identifier for this daemon. This
		identifier will be unique to the daemon, and is
		one of two components, along with the UUID, used
		to determine uniqueness within this store. The
		alias is also used to build keys for built-in
		items associated with all daemons.

**provenance**	The chain of handling for this catalog
		block. The provenance is a sequence, listing every
		daemon between the client and the original source
		of authority for the block. Each element in the
		sequence contains a stratum, hostname, and req port
		number; the 'pub' port is optional, and will only
		be present for daemons that can handle subscribe
		requests. Clients will connect to the stratum zero
		entry to handle any requests; this may change in
		the future to allow identification of full proxies
		for all REQ/REP and PUB/SUB traffic for a daemon.

**time**	Daemon-provided timestamp for the contents of this
		block. The timestamp may change even if the contents
		(and hash) do not.

**hash**	A hash value for the 'items' component of this block.
		The hash is arbitrary and clients should not concern
		themselves validating the contents of the block with
		the hash; a change in the hash is used to signify the
		contents have changed, and any/all clients relying on
		the contents should update their cached data.

**items**	A dictionary of dictionaries, organized by item name (key),
		with one dictionary containing the full description
		of a single item. The description of an item is discussed
		in its :ref:`own section <items>`.
=============== ===============================================================


.. _items:

Item description
----------------

Similar to the catalog block for a store, the dictionary describing
an item contains a set of fields that provide a complete description of the
item.

=============== ===============================================================
*Field*         *Description*
=============== ===============================================================
**key**		The unique name for this item. The uniqueness
		constraint is applied across the entire store, not
		just within this catalog block.

**type**	The data type for the value associated with this
		item. The type is one of: boolean, bulk, enumerated,
		mask, numeric, numeric array, or string. The type is
		not strictly required, but it is a useful hint for
		what the value is expected to be. A more complete
		description of item types is below.

**description**	A human-readable description of what this item
		represents. Could be one sentence, could be several;
		ideally the reader can use this information to fully
		understand the intent and function of the key/value
		pair.

**units**	Terse description of the units for a numeric value;
		a typical item with units will only have one units
		string, assumed to be appropriate for all formats.
		If the value has multiple representations, there
		could be a units value for each representation, for
		example an angular value transmitted as radians but
		with optional formatting as sexagesimal; in this case
		the units field will be a dictionary, with the empty
		string and 'formatted' as keys for the different units.

**format**	How to format the value for printing. This is expected
		to be a printf-like expression, for example %d for
		integers, or %.3f for three decimal places of a floating
		point value.

**initial**	Initial value for this item. This is only relevant
		for the authoritative daemon, and is only used if
		there is no persistent value to restore for this item.

**persist**	A boolean to indicate whether this item's value
		should be persistent on the daemon side, such that
		values persist across a restart of the daemon. This
		is not common for hardware-facing daemons, where the
		controller is the authoritative source for most item
		values.

**gettable**	Generally not specified unless set to 'false',
		which indicates this item will reject any attempts
		to get its value. The use of this property is highly
		discouraged, any item should have a meaningful value;
		this property only exists for backwards compatibility.

**settable**	Generally not specified unless set to 'false',
		which indicates this item will reject any attempts
		to set a new value. Read-only items are fairly common,
		for example it may not make sense for a temperature
		probe to be settable.

**concurrency** Select the concurrency used for performing SET requests
		for this item. The two valid options are 'serial' and
		'parallel'; serialized SET requests are performed in order
		of receipt, where the next request does not begin processing
		until the previous request is completed; parallelized SET
		requests are performed immediately upon receipt, regardless
		of whether a previous request is still underway. The default
		concurrency option is 'serial'.

**interval**	Specify the polling interval used for this item by its
		authoritative daemon. The interval is specified in seconds;
		fractional seconds (as a floating point number) are allowed.

**enumerators**	A dictionary mapping a human-readable string
		representation to numeric values. This is only
		meaningful for boolean, enumerated, and mask types,
		though it is also overloaded for use with the legacy
		numeric array type. An example set of enumerators for
		a boolean item might be ``{'0': 'False', '1': 'True'}``.
		In JSON a dictionary key must be a string, integer
		handling of these keys can and should be supported;
		this is particularly relevant for mask values.
=============== ===============================================================

Item types
----------

Each value type may be associated with one or more optional directives from
the set above. Note that any value could also be empty, as expressed with the
JSON null value.

=======================	=======================================================
*Item type*		*Description*
=======================	=======================================================
**boolean**		A two-state integer, either 0 or 1, with a string
			representation that is usually something like
			false/true, off/on, out/in, etc. The "truth" value
			should always map to 1, though there will be some
			backwards-compatible instances where a badly configured
			boolean value does not adhere to this standard. A
			boolean is effectively an enumerated value with only
			two enumerators.

**bulk**		A true data array, analogous to a Numpy array, unlike
			the legacy "numeric array" type, which is more like a
			dictionary or named sequence.

**numeric**		A numeric value, either a floating point number or
			an integer. A numeric value will generally have a
			'units' property defined.

**numeric array**	A sequence of numeric values, often with enumerators
			describing the individual values. This is a legacy type
			intended solely for backwards compatibility.

**enumerated**		An integer value with a string representation
			for each valid value. The valid enumerators are listed
			in the 'enumerators' description property.

**mask**		An integer value with a string representation for each
			of the possible bits in the integer. The enumerators
			reflect the status for each bit, counting from zero;
			the '0' enumerator represents the mask value if the
			zeroth bit is active, the '1' bit represents the value
			if the next bit is set, and so on. If a mask has
			multiple active bits the string representation is a
			concatenation of the relevant strings, joined by
			commas. The "none" enumerator reflects the string
			value if no bits are set.

**string**		A text string of arbitrary length.
=======================	=======================================================


Example
-------

Here is a complete two-item example for what a catalog block may look
like for a store named 'pie'::

      {
        "store": "pie",
        "hash": 236000907473448652729473003892320198915,
        "uuid": "8017ad5b-07a7-5135-a024-c46a0b79b74e",
        "time": 1738177027.4993615,
        "provenance": [
          {
            "stratum": 0,
            "hostname": "chonk",
            "rep": 10112,
            "pub": 10139
          }
        ]
        "items": {
          "ANGLE": {
            "type": "numeric",
            "units": {
              '': "rad"
              "formatted": "h",
            },
            "description": "Writable angle keyword.",
	    "initial": 0.0,
	    "persist": "true"
          },
          "DISPSTOP": {
            "type": "boolean",
            "description": "Dispatcher shutdown command. Tells dispatcher to execute a clean shutdown.",
            "enumerators": {
                "0": "no",
                "1": "yes"
            }
          }
	}
      }


Storage
-------

Catalog files are stored on-disk as part of a bootstrapping mechanism
to prevent transmission of catalog blocks for every new connection.
Two directory trees have been established; one, an automatic cache for any
received catalog blocks, and two, a tree for meta data used
by 'stratum 0' daemons providing authoritative access to a set of items.

The MKTL_HOME environment variable, if set, determines the top-level directory
used for these on-disk locations. Absent that variable being set, the default
location is '$HOME/.mKTL'.

The cache directory structure is as follows::

        $MKTL_HOME/
        $MKTL_HOME/client/cache/
        $MKTL_HOME/client/cache/some_store_name/
        $MKTL_HOME/client/cache/some_store_name/some_uuid.json
        $MKTL_HOME/client/cache/some_store_name/some_other_uuid.json

For each store name, each catalog block within a store is written to a
separate file, where each file is named for the UUID associated with that
catalog block.

The daemon directory structure is as follows::

        $MKTL_HOME/daemon/store/
        $MKTL_HOME/daemon/store/some_store_name/
        $MKTL_HOME/daemon/store/some_store_name/some_items.json
        $MKTL_HOME/daemon/store/some_store_name/some_items.uuid

The .json file located here is where a daemon is expected to establish the
items it provides. The adjacent .uuid file is auto-generated; the only content
of the file is a single UUID. If the .uuid file exists it will be used,
regardless of its origins, but there is no need for the developer to establish
it as part of the daemon's initial configuration. Unlike the cached client
side catalog file, the daemon catalog file only includes the
'items' component. This would be the daemon-side .json file for the above
two-item example::

	{
          "ANGLE": {
            "type": "numeric",
            "units": {
              '': "rad"
              "formatted": "h",
            },
            "description": "Writable angle keyword.",
	    "initial": 0.0,
	    "persist": "true"
          },
          "DISPSTOP": {
            "type": "boolean",
            "description": "Dispatcher shutdown command. Tells dispatcher to execute a clean shutdown.",
            "enumerators": {
                "0": "no",
                "1": "yes"
            }
          }
	}


