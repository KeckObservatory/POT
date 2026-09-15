
.. _protocol:

Protocol
========

The mKTL protocol is the primary interface layer for mKTL as a whole;
associated Python code can be considered a reference implementation,
but the intent is for the protocol to be language agnostic. mKTL clients
and daemons communicate using ZeroMQ sockets. This document describes the
socket types used, the formatting of the messages, and the types of requests
that can be made.


Principles of operation
-----------------------

Unique addressing of daemons within a store is accomplished by the use of
a unique port number to connect to that daemon; messages arriving on a
specific port are thus guaranteed, by construction, to only need decoding
exactly once; there is no envelope for the message contents, such that the
message contents may need to be rerouted to a new location. Each daemon
will use a unique :ref:`publish <publish>` and :ref:`request <request>`
listener, operating on a unique port on that host.

A given host providing connectivity for mKTL could have thousands of daemons
running locally, each listening on a unique port number. Each daemon deploys
a UDP listener on a predetermined port number (10111) to enable discovery of
daemons on that host; a :ref:`dedicated registry process <mkregistryd>` also
listens on a predetermined port number (10103) to streamline discovery from
the client side. This usage pattern expects there to be a single
:ref:`mkregistryd` process running on every host running one or more mKTL
daemons. The discovery exchange is :ref:`described below <discovery>`
in more detail.

An mKTL proxy, were such a thing to exist, would follow the same principle:
a unique port for each listener of each proxied daemon, which allows the use
of the ``zmq.proxy()`` method to cleanly bridge between endpoints at the
protocol level without any inspection of message contents, thus ensuring the
proxy has minimal processing overhead.


.. _request:

Request/response
----------------

The first socket type implements a request/response pattern. This represents
the majority of the interactive traffic between a mKTL client and daemon.
Because requests can be asynchronous, mKTL does not use the REQ/REP
implementation in ZeroMQ, which enforces a strict one request, one response
pattern; instead, we use DEALER/ROUTER, which allows any amount of messages
in any order, in any direction.

.. mermaid:: protocol_req_rep.mmd

Upon receipt of a request the daemon will immediately issue an ACK response.
The purpose of the ACK response is to verify that the appropriate endpoint
for the request is online and available to respond; the final response to a
request may be significantly delayed compared to its initial receipt, and
the ACK response allows a client to distinguish between delayed operations
and the complete absence of a proper handler for the request. The client
should immediately raise an error in the absence of a quick response.

The full REP response can be issued at any time. If it arrives prior to the
ACK response the client will consider this final response to be a combined
ACK and REP response. There is never more than one ACK response for a request;
likewise, there is never more than one REP response.

All requests are handled
fully asynchronously; a client could send a thousand requests in quick
succession, but the responses will not be serialized, and the response order
is not guaranteed. Synchronous behavior, if desired, is implemented by client
code and not in the protocol itself.
Here is an example of what the full exchange on the client side might look
like, in this case handling the exchange as a synchronous request::

        self.socket = zmq_context.socket(zmq.DEALER)
        self.socket.setsockopt(zmq.LINGER, 0)
        self.socket.identity = identity.encode()
        self.socket.connect(daemon)

	self.socket.send_multipart(request)
	result = self.socket.poll(100) # milliseconds
	if result == 0:
	    raise TimeoutError('no response received in 100 ms')

	ack = self.socket.recv_multipart()
	response = self.socket.recv_multipart()

The request/response interaction between the client and daemon is a multipart
message, where each component of the message has a specific meaning.
The message parts are identical for both ends of the request/response
exchange.

.. list-table::

  * - *Field*
    - *Data type*

  * - **version**
    - A single ASCII character indicating the mKTL protocol version number.
      The initial release of the mKTL protocol uses the version character 'a'.
      The version identifier is always present.

  * - **identifier**
    - A unique identifier for the request. The format of this identifier is
      not strict, it could be any byte string (such as a UUID), but the initial
      implementation uses a monotonically increasing eight-byte integer. The
      identifier is set by the client, and allows the client to tie a response
      to the original request. Note that this identifier does not necessarily
      have significance on the daemon side, daemons will use their own internal
      scheme to uniquely identify requests, but the response will always include
      this original identifier. The request identifier is always present.

  * - **type**
    - The message type. This is a short string of characters that identifies
      what type of request, or response, this message represents. It is one
      of the values described in the :ref:`message_types` section below. The
      message type is always present.

  * - **target**
    - The target for this request/response, if any. Not all requests have a
      target; responses don't need to specify it, since the identifier field
      associates a response with a request. If a target is specified it
      is a store or a key, depending on the request; this field will be an empty
      byte sequence if the target is not specified.

  * - **flags**
    - A big-endian integer representing boolean flags that modify how this
      message is handled. The default value is an integer zero; if this field
      is transmitted as an empty byte sequence it must be interpreted as the
      integer zero. Each bit in the integer has a specific meaning:

      .. list-table::

	 * - *Bit*
	   - *Name*
	   - *Meaning*

         * - 0b0001
           - NO_ACK
	   - Suppress the ACK response to this request.

         * - 0b0010
           - NO_REP
	   - Suppress the REP response to this request.

  * - **payload**
    - The message payload. This is the JSON representation of any additional
      data required as part of this exchange; if setting a new value, it would
      contain the value; if it is a response containing additional information
      it would go here. See the :ref:`message_payload` section
      for a more complete description of the payload contents. This field will
      be an empty byte sequence if there is no payload.

  * - **bulk**
    - A bulk byte sequence, typically a component of the payload. This is to
      allow the transmission of information like image data, where the bulk
      bytes represent the image buffer, and the JSON payload describes how
      to interpret the buffer. This field will be omitted entirely if there
      is no bulk component, which allows a recipient to distinguish between
      an empty byte sequence and the complete absence of data.

Here is a representation of what the on-the-wire messages might look like
for a simple GET request::

	b'a'
	b'00000023'
	b'GET'
	b'kpfguide.LASTFILENAME'
	b'\x00'
	b''

	b'a'
	b'00000023'
	b'ACK'
	b''
	b''
	b''

	b'a'
	b'00000023'
	b'REP'
	b''
	b''
	b'{"value": /sdata1701/kpf1/2025-06-23/image_672.fits', "time": 234.23}'

The reference implementation provides a :class:`mktl.protocol.message.Message`
class to minimize the amount of code that has to be aware about the on-the-wire
message structure.


.. _message_types:

Message types
-------------

This section describes the various requests a client can make of the mKTL
daemon via the request/response socket. An additional message type, the PUB,
also exists, but has its own message structure outside this scheme.

.. list-table::

  * - *Message type*
    - *Description*

  * - **GET**
    - Request the current value for a single item.
      The target is always the name of the store, and the key for the item,
      concatenated with a period. No additional payload is required for a
      basic GET request.

      The default behavior for a GET request is for a cached value to be
      returned by the handling daemon. A client can explicitly request an
      up-to-date value by setting the 'refresh' field in the payload to
      'True'; see the :ref:`message_payload` section for additional details.

      The payload of the response will contain 'value' and 'time' fields,
      corresponding to the item value and the last-changed timestamp. If
      the item has a bulk data component, the payload will instead describe
      the bulk data.

  * - **SET**
    - Request a change to the value of a single key. Depending on the daemon,
      this could result in a variety of behavior, from simply caching the value
      to slewing a telescope, and anything in-between. The final response
      indicates the request is complete but does not indicate what the new
      item value is.

      The :ref:`message_payload` for a SET request is the same as the payload
      for a GET response, except that the 'time' field is not required or
      expected, and there are additional fields set by the client to describe
      the origin of the request.

  * - **ACK**
    - Immediate acknowledgement of a request; this message type originates from
      a daemon, only in response to a request. If this response is not received
      with a very small time window after the initial request, the client can
      and should assume the daemon handling that request is offline. The
      acknowledgement confers no additional information beyond a positive
      affirmation that the request has been received.

  * - **REP**
    - A response to a direct request; this message type originates from a
      daemon, only in response to a request. This response will contain the
      full payload to satisfy the request, any error text related to a problem
      satisfying the request, or be an empty indication that the request has
      been completed.


.. _message_payload:

Message payload
---------------

The payload of a message is a JSON associative array. The fields will vary
depending on the message type, and are optional in nearly all circumstances,
but each field has a consistent meaning. Arbitrary additional fields are
allowed, but a standard mKTL client will not notice or handle them.

.. list-table::

  * - *Payload field*
    - *Description*

  * - **value**
    - The base representation of the value being transmitted in this message.
      For a GET response or a SET request, this would be the item value;
      depending on the item type this could be a boolean, numeric, or string
      value, or any valid data that can be serialized as JSON.

  * - **time**
    - The timestamp associated with the transmitted value. This should be
      interpreted as the "last modified" timestamp for an item, indicating
      when the item assumed the transmitted value. The timestamp is a numeric
      representation of UNIX epoch seconds.

  * - **error**
    - A JSON dictionary with information about any error that occurred while
      processing the request. If the value is not present or is the JSON null
      value, no error occurred. If it is present, it will have these fields:

      =========	============================================
      **type**	Analogous to the Python exception type
                (ValueError, TypeError, etc.).

      **text**	Descriptive text of the error.

      **debug**	Optional additional information about the
		error, such as a Python traceback.
      =========	============================================

      The intent of this error field is not to provide enough information for
      debugging of code, it is intended to provide enough information for the
      client to perform meaningful error handling.

  * - **prime**
    - An optional field associated with a GET request. If this field is
      present, and it is set to True, the daemon processing the request will
      initiate a series of priming broadcasts on its PUB/SUB interface for
      the requested item. These broadcasts look exactly like a regular
      :ref:`published broadcast <publish>` for an item, except the topic
      is prefixed with `prime:` to distinguish it from the regular message
      stream. This allows the requesting client to know, with confidence,
      that the PUB/SUB channel is fully operational for this item.

  * - **refresh**
    - An optional field associated with a GET request. If this field is
      present, and it is set to True, the daemon processing the request is
      expected to ignore cached data and retrieve the most current value
      for the target item. For example, if the item represents a temperature
      reading, the daemon would be expected to query the hardware controller,
      update its local cache, and return the result to the requesting client.

  * - **shape**
    - One of the two required fields in order to describe a bulk data array.
      This defines the dimensions of the bulk data array, and is interpreted
      the same way as the 'shape' parameter for a numpy ndarray.

  * - **dtype**
    - One of the two required fields in order to describe a bulk data array.
      This defines the data type of the bulk data array, and is interpreted
      the same way as the 'dtype' parameter for a numpy ndarray. If starting
      from an ndarray, the dtype is the string representation of the .dtype
      attribute of that array; when recreating an ndarray, this string is
      used to get the matching dtype attribute from the numpy module. In
      Python::

        payload['dtype'] = str(my_numpy_array.dtype)
        dtype = getattr(numpy, payload['dtype'])

  * - **_user**
    - The user name associated with a SET request.

  * - **_hostname**
    - The host name from which a SET request originated.

  * - **_pid**
    - The process identifier associated with a SET request.

  * - **_ppid**
    - The identifier for the parent process associated with a SET request.

  * - **_executable**
    - The executable running the process associated with a SET request.

  * - **_argv**
    - All additional command line arguments provided to the process associated
      with a SET request.


.. _publish:

Publish/subscribe
-----------------

The second socket type implements a publish/subscribe socket pattern. The
desired functionality in mKTL is a neat match for the PUB/SUB socket pattern
offered by ZeroMQ:

	* SUB clients subscribe to one or more topics from
	  a given PUB socket, or can subscribe to all topics
	  by subscribing to the empty string. This aligns well
	  with existing usage patterns, where KTL keyword
	  names and EPICS channel names are treated as unique
	  identifiers, and map easily to a PUB/SUB topic.

	* The filtering of topics occurs on the daemon side,
	  so if a PUB is publishing a mixture of high-frequency
	  values or large broadcasts, and a client is not
	  subscribed to those specific topics, the broadcasts
	  are never sent to the client.

.. mermaid:: protocol_pub_sub.mmd

The above diagram shows a typical pattern for publish/subscribe interactions.
A daemon periodically polls a hardware controller for a new value; the daemon
publishes any new values to any clients subscribed to updates for that item.
Subscribers may come and go at any time. Once the daemon issues the publication
it has no awareness of any subsequent processing or potential performance
bottlenecks on the client side; subscribed clients offer no feedback on their
receipt or handling of a published message.

The formatting of the PUB message is very similar to what is described
above for the :ref:`request/response multipart message format <request>`.
Some fields are not necessary for the PUB variant, and in order for the
topic matching to work the topic must be the first component of a multipart
message. The fields are as follows:

.. list-table::

  * - *Field*
    - *Data type*

  * - **topic**
    - For a typical broadcast the topic will be the full key for a single
      mKTL item. This is similar to the 'target' field in a
      :ref:`request/response message <request>`. For all mKTL addressing
      the topic appends a trailing '.' in order to prevent unwanted substring
      matching between similarly
      named keys. Likewise, because of the ZeroMQ behavior around leading
      substrings, any expanded use of mKTL PUB/SUB behavior will use a
      leading prefix to distinguish it from other message types. For example,
      broadcasting all SET requests with a leading 'set:' prefix, or
      broadcasting a bundle of related mKTL items with a leading 'bundle:'
      prefix. The response to a priming GET request does exactly this,
      issuing a stream of broadcasts with a leading 'prime:' prefix.

  * - **version**
    - A single ASCII character indicating the mKTL protocol version number.
      The initial release of the mKTL protocol uses the version character 'a'.

  * - **payload**
    - The message payload, with exactly the same contents as
      :ref:`described above <message_payload>`.

  * - **bulk**
    - A bulk byte sequence, with exactly the same contents as the
      :ref:`request/response message <request>`. This field will be
      omitted entirely if there is no bulk component.


.. _discovery:

Discovery
---------

The UDP discovery layer takes advantage of a feature of UDP listeners: not only
are you allowed to have multiple listeners on the same port, but they will all
respond to an incoming broadcast message. This feature allows every listening
participant to create a listener on the same port, which allows an arbitrary
number of processes to participate in discovery.

There are two phases to a client automatically discovering a daemon; these
phases are separated by an intermediary, the registry process
(:ref:`mkregistryd`). The first phase is for the registry daemon to discover
all local daemons; the second phase is for a client to discover available
registry daemons.

The discovery process is identical for the two phases, except that the listeners
are on different port numbers. Clients looking for registries, and registries
looking for daemons, they both start with a UDP broadcast; the recipient then
responds with a port number that can be used for REQ/REP mKTL queries.

.. mermaid:: protocol_discover_client_broadcast.mmd

The client/registry interaction is narrower than the registry/daemon
exchange: a client wants to know about a specific store,
and if the registry doesn't know about that store the query will fail. This
sequence is the only time a client will initiate or otherwise participate in
any form of broadcast discovery. Enumeration of available stores can also
occur in a form of directed discovery: if a client already knows where a
registry is running it can issue a directed UDP query to that host, though
instead of asking for a single store the client will ask for all available
stores. This specific exchange is intended to populate a local cache on
the client side, and will not occur unless explicitly requested.

.. mermaid:: protocol_discover_client_direct.mmd

The discovery exchange between a registry and a daemon uses the same structure,
but with the listener running on a different UDP port; a single port is used to
discover a registry daemon, and a single port is used to discover a daemon.

.. mermaid:: protocol_discover_registry.mmd

The registry will run this discovery process exactly once upon startup, in an
attempt to identify all available daemons on the local network, and cache their
locally authoritative catalog blocks.

.. mermaid:: protocol_discover_daemon.mmd

Daemons are expected to self-announce on startup,
which involves discovering local registries, and pushing the description of
the daemon's authoritative items to any discovered registries; the daemon
will issue a SET request to the registry's _catalog item to initiate further
processing. The registry will reconcile the new description against what it
already has cached, and will reject any announcement that introduces a
conflict.

There are four shared secrets used in the discovery exchange:

================= ==============================================================
*Secret*	  *Description*
================= ==============================================================
**registry port** The UDP port used to discover locally accessible
		  :ref:`mkregistryd` processes. Clients and daemons send
		  broadcasts to this port to initiate a discovery exchange.
		  The port number is 10103.

**daemon port**	  The UDP port used to discover locally accessible mKTL daemons.
		  :ref:`mkregistryd` sends a broadcast to this port to initiate
		  a discovery exchange. The port number is 10111.

**call**	  An arbitrary string used by the discoverer to trigger a
		  response from the listener. The string value is
		  ``Aloha mKTL``.

**response**	  An arbitrary string used by the listener to respond to any
		  received calls. The string value is ``E komo mai:``.

================= ==============================================================

The purpose of the discovery response is to convey a single piece of
information: the REQ/REP port number that process is using to receive
proper mKTL requests. The port
number, encoded as a string representation of an integer, is the sole additional
component of the response after the colon. For example, if a daemon has a
request port listening on port 10079, the full exchange (discovery request,
discovery response) would be::

    b'Aloha mKTL'

    b'E komo mai:10079'

