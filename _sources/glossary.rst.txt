Glossary
========

This document defines the common nomenclature used within mKTL. Some of the
terms are inherited from KTL; some are inherited from common design patterns
in the broader software world. Where there is a conflict between the two,
we prefer to use the common industry definition rather than traditional use
within WMKO.


External definitions
--------------------

 * **WMKO**:
	`W. M. Keck Observatory <https://keckobservatory.org/>`_, where mKTL
	was initially established, in collaboration with WMKO partner
	institutions at the
	`University of California <https://www.universityofcalifornia.edu>`_
	and `California Institute of Technology <https://www.caltech.edu>`_.

 * **KTL**:
	:ref:`Keck Task Library <heritage>`, the predecessor to mKTL at
	WMKO. KTL is a key-value application programming interface (API),
	written in C, and has been the dominant API for interprocess
	communication at both WMKO and Lick Observatory since the 1990's.

 * **EPICS**:
	`Experimental Physics and Industrial Control System
	<https://epics-controls.org>`_, another
	key-value API that WMKO adopted for its telescope and
	adaptive optics control systems. EPICS is one of the interprocess
	communication APIs for which KTL provides a common abstraction.


mKTL-specific terms
-------------------

 * **mKTL**:
	Modern Keck Task Library. It's just a name, but it's ours,
	and was the clear choice out of a field of a dozen options
	that were presented to the future power users of the protocol.

 * **Protocol**:
	Protocol, as used in the above statement, refers to
	a communication protocol, as opposed to being a synonym for an interface
	in an object-oriented context. Borrowing phrasing from
	`Wikipedia <https://en.wikipedia.org/wiki/Communication_protocol>`_,
	a communication protocol defines the rules, syntax,
	semantics, and synchronization of communication between
	two entities.

 * **Transport**:
	A `transport layer <https://en.wikipedia.org/wiki/Transport_layer>`_
	is the underlying layer of the network stack that handles application
	level messaging. Though it is built upon TCP and UDP,
	`ZeroMQ <https://zeromq.org/>`_ is effectively the transport layer
	for the mKTL protocol.

 * **Daemon**:
	A persistent process responsible for some or all of the key-value
	pairs in a given store. When client requests are initiated it is
	the daemon that will be contacted to satisfy the request. This is
	analogous to a KTL dispatcher or an EPICS IOC.

 * **Store**:
	A store is an aggregation of individual key-value pairs.
	Within a given deployment of mKTL the store will have a
	unique name. This is analogous to a KTL service, or an
	EPICS database, and is effectively an associative array,
	or a Python dictionary. "Database" is another term used
	in similar contexts, but that term is more commonly used
	in reference to a relational database. mKTL treats the
	store name as case-insensitive, any actual usage will
	render it as all lower-case.

 * **Key**:
	A unique name within a store, identifying a single key-value
	pair. This is analogous to a KTL keyword, or an EPICS channel.
	mKTL treats the key as case-insensitive, any actual usage will
	render it as all upper-case.

 * **Value**:
	The other half of the key-value pair. Like a KTL keyword,
	a mKTL value can be one of many native types (integer,
	floating point, string, etc.), and includes the possibility
	of compound values, similar to KTL arrays.

 * **Item**:
	The combination of the key/value pair. This term is borrowed
	from Python's dictionary, where it has a similar meaning. In
	the context of mKTL, the Store class in the reference Python
	module will return Item instances when referenced by the key.

 * **Get**:
	Retrieve a value corresponding to an individual key in a store.
	A typical client can issue a blocking or a non-blocking
	(synchronous or asynchronous) operation. This is analogous to
	a KTL read or an EPICS get.

 * **Set**:
	Establish a new value for an individual key in a store. A typical
	client can request a blocking or a non-blocking (synchronous or
	asynchronous) operation. This is analogous to a KTL modify or an
	EPICS put.

 * **Publish**:
	Broadcast a new value for an individual key in a store.
	The store name combined with the key makes up the bulk of
	the topic, which in turn has specific meaning in a typical
	`publish-subscribe design pattern <https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern>`_.

 * **Subscribe**:
	Request the receipt of any/all published broadcasts of an
	individual key in a store. This is analogous to a KTL monitor
	or EPICS monitor request.

 * **Callback**:
	A method to be called whenever a published broadcast arrives
	for an individual key in a store.

 * **Register**:
	The act of associating a callback with a specific key in a
	store. The callback will now be invoked whenever the value
	of that key changes. "Connect" is another term used in
	similar contexts, but that term is more commonly used with
	network sockets.
