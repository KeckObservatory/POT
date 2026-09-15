.. _heritage:

Heritage
========

History
-------

The development of the Keck Task Library (KTL) has its origins in the early 1990’s,  motivated by the need for a common API to standardize access to commands and telemetry from disparate systems across `W. M. Keck Observatory <https://keckobservatory.org/>`_ (WMKO). The first light instruments at WMKO were HIRES, LRIS, and NIRC; HIRES and LRIS had some measure of common heritage and relied on MUSIC messaging, developed by `University of California Observatories <https://www.ucobservatories.org/>`_ at the `UC Santa Cruz <https://www.ucsc.edu/>`_ campus, for interprocess communications; NIRC used a system based on the `remote procedure call <https://en.wikipedia.org/wiki/Sun_RPC>`_ (RPC) library from `Sun Microsystems <https://en.wikipedia.org/wiki/Sun_Microsystems>`_; the telescope control system uses the `Experimental Physics and Industrial Control System <https://epics-controls.org/>`_ (EPICS). The KTL API is a set of C routines, implemented as a shared library, that provides a common key/value behavior layered on top of these disparate communication APIs.

The entry point for any KTL client is to load a shared library unique to the KTL service of interest, where a KTL service is a collection of individual KTL keywords, and where a keyword is a single key/value pair. This KTL client library can be any C code, so long as it implements the methods that the KTL API expects to find. The earliest instruments took full advantage of this opportunity, with custom code implementing the interface to each of their respective communication methods; later instruments relied on common, configuration-driven libraries that were shared across instruments; each of the three communication styles (EPICS, MUSIC, and RPC) has an implementation intended to be shared across many services and/or instruments, though there isn’t much common heritage between the three, and certain optional aspects of the KTL API are poorly supported, or not supported at all, in some of the variants; as a result, shared KTL tools are required to be aware of implementation-specific variances in order to function correctly, and some specific applications cannot be shared because they rely on implementation-specific behavior.

With that background in mind, let it be stated clearly that KTL was unambiguously successful in its broader objectives. New instruments progressively leaned further on KTL as a key technology, with more services, more dispatchers, more keywords, and overall, more KTL-driven information flow though each system. The telescope control system and adaptive optics benches at WMKO did not realize the same level of benefit, as the development for those systems focused on using EPICS directly as a first-class interface, enabled KTL-based access to a limited subset of commands and telemetry, and often do not use the KTL abstraction for internal tools. KTL also saw widespread adoption at Lick Observatory, where it is used as a first-class interface for all commands and telemetry; University of California Observatories' oversight of LicK Observatory and the UC participation in WMKO encouraged this practice, and it yielded substantial improvements to both KTL and common KTL tools across both observatories.

In order for mKTL to succeed it must be better than KTL. While this assertion is simple to write down there will be both qualitative and quantitative metrics that must be met in order to achieve that broader objective. Three decades of KTL development, maintenance, and support provide a wealth of experience to draw from; care must be taken to emphasize the successful design choices made with KTL, while minimizing or eliminating the areas where it was less successful. mKTL is not positioned as a rejection of KTL: mKTL is an evolution, drawing from an established heritage, and avoiding the missteps of the past must not come at the expense of making new missteps that KTL successfully evaded.


Strengths
---------

The strengths identified here highlight areas to emphasize when making design decisions for mKTL.

Key/value pairs
^^^^^^^^^^^^^^^

Representing commands and telemetry as key/value pairs has been an excellent match for the usage patterns at WMKO. Sensor values, motor positions, filter names, etc., all lend themselves to having a single value, or family of values, representing vital state information for a given system. Being able to address keywords individually is an important part of this approach, as it allows client applications to work with exactly the subset of commands and telemetry they need, instead of requiring manipulation of a bulkier, more complex structure.

Request/response
^^^^^^^^^^^^^^^^

A request/response pattern is fundamental to both synchronous and asynchronous usage in KTL. The KTL command exchange involves two steps: a first stage notification, acknowledging that a request has been made, and a second stage notification, a full response indicating the completion of the request. The base pattern for this request/response exchange underpins the different handling options available to a KTL client: ignoring any response, continuing with execution before checking for a response, and blocking execution until a response is received. Each of these patterns is essential for different types of client interactions.

Publish/subscribe
^^^^^^^^^^^^^^^^^

A publish/subscribe pattern allows for asynchronous handling of new values; this is particularly valuable for event-driven use cases, where recording, displaying, or otherwise reacting to a new value should occur immediately rather than wait for a client polling cycle to occur. Publish/subscribe also unlocks additional efficiency, in that a single publish event can be distributed to all subscribers in a single pass, rather than requiring subscribers to individually poll for new values via the request/response interface.

Flexible values
^^^^^^^^^^^^^^^

Every KTL keyword value has two representations: ‘ascii’ and ‘binary’, which have different meanings depending on the defined keyword type. This behavior, while optional, has powerful practical use: a boolean keyword, for example, may have 0 and 1 as its available binary values, but the ascii values could be anything: off and on, no and yes, false and true, or whatever other values might be handled by that service’s KTL client library. This behavior extends to enumerated and mask keyword types, where a binary integer value can be interpreted as arbitrary strings. This allows the possibility that a client can work directly with human-readable values for command and telemetry instead of relying on the correct handling of magic numbers to achieve the same goal. This same basic feature is used to render numeric values in multiple different units, such as the binary value being in radians and the ascii value in degrees, or as sexagesimal.

Distributed control
^^^^^^^^^^^^^^^^^^^

There is no single source of authority for KTL; there is no federated naming scheme, or any requirement that what is installed on one computer matches what is installed on another. Any naming scheme must only be unique locally, and does not register itself with a persistent registry running elsewhere in order to function correctly. In practice, most KTL services have unique names, and uniqueness is generally a virtue, though for specific use cases, such as testing, there is merit in being able to run any KTL service in an isolated environment. Any KTL client, as long as it satisfies its prerequisites, can communicate with any KTL dispatcher, without invoking an intermediary; any one KTL dispatcher (or family of dispatchers) does not have common infrastructure that might be bottlenecked by some other dispatcher (or dispatchers). In practice, this allows instruments to operate independently of each other, limiting the likelihood that a failure affecting one system might degrade performance for another.

Multiple languages
^^^^^^^^^^^^^^^^^^

One of the virtues of a C-based API is that every other language has a mechanism to access that API; some languages have several. Regardless, the ubiquitous ability to access C code from other languages has been of enduring value to expand KTL access beyond its original intended use. Language-specific KTL interfaces for Tcl, Java, and Python all see production use.


Weaknesses
----------

The weaknesses identified here highlight areas to avoid when making design decisions for mKTL.

C-based API
^^^^^^^^^^^

While a C-based API has merit it has also proven to be a weakness for KTL. The KTL API relies on an ioctl interface for much of its optional functionality; the arguments to the defined ioctl commands vary in both number and type, and switching ioctl behavior is governed by a single 32-bit integer, which limits the number of optional commands that might be added to the API. Any expansion of the API beyond its current footprint creates binary incompatibility, where interoperability between systems becomes critically impaired and could motivate a mass rebuild across any interconnected systems.  And, just because access to a C-based API is possible from a given language doesn’t mean it’s easy; for example, KTL relies on a union type (a KTL polymorph) to contain telemetry values, which must be handled with care when the value is exposed to a strongly typed language, like Java.

Interface layer
^^^^^^^^^^^^^^^

With KTL being primarily a C-based API, the key interface layer between a client application and the dispatcher receiving the command is this C API. A client’s application code is custom up until that point; on the other side of the interface, the code associated with the persistent daemon takes over, including all aspects of on-the-wire communication between the KTL client library and the dispatcher. This design decision means that client interactions cannot be cleanly isolated from dispatcher handling, because the behavior of the dispatcher-specific messaging requires code specific to that messaging on the client side.

Code complexity
^^^^^^^^^^^^^^^

The implementation of the KTL API is a complex collection of C code, having grown organically from the base concepts of the API to include logging behavior, backgrounded queueing and dispatching of events, and additional hooks for optional behavior added to the API over time. Support exists for different types of messaging, of which only one style of messaging is ever used. Making changes in any one area often has unintended consequences elsewhere in the code base; eliminating memory leaks is one example, where memory allocation occurs in one file, but deallocation must occur elsewhere, or in a different context entirely; some allocations occur in the client library but are freed by the KTL API, and the reverse also occurs. So, this code complexity extends to the KTL client libraries themselves, and the amount of close coupling across that API layer is high.

Multiple communcation styles
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

One of the key design goals of KTL was to implement a common abstraction to manage differences in communication style between different systems, each potentially employing a unique protocol and transport for inter-process communication. While it was successful in this regard, it also created circumstances leading to some of its largest flaws: any communication failures require deep protocol-specific knowledge in order to diagnose and remedy the failure; any client wishing to communicate with a given system must locally install all libraries appropriate for the communication method(s) used by that system; each KTL client library is an isolated code base, and is on its own to successfully map KTL calls to its native method of communication, with significant variance in how successful they actually are. The net result is that it is not sufficient to be an expert in an application-specific protocol and transport, it is not sufficient to be an expert in the KTL API, proper maintenance requires an individual to be expert in both areas in order to succeed.

Prerequisite knowledge
^^^^^^^^^^^^^^^^^^^^^^

Having the correct KTL client library installed locally is generally insufficient to access the KTL service; some amount of additional information must be provided, such as environment variables defining a target host name, or configuration files describing how the client library should connect to the waiting KTL dispatcher. These configuration management quirks are a barrier to usage, and depending on their nature, delicate and prone to configuration mismatches.

Exotic dependencies
^^^^^^^^^^^^^^^^^^^

The use of KTL is tied to a WMKO standard deployment of kroot; this includes the KTL API shared libraries, the KTL client libraries for the specific KTL services of interest, configuration files used by those KTL client libraries, and command line utilities providing common functionality for scripts as well as interactive use. Any KTL-based software must therefore have a full kroot install available in order to function normally; this is perhaps not a burden for systems that must have kroot regardless, but the close coupling between KTL and kroot is an easy example of an exotic dependency. Individual client libraries create their own burdens in this respect: an EPICS-based client library must have enough EPICS available locally to build and run; likewise for MUSIC, which is bundled in kroot, and RPC, which despite being an industry standard, WMKO’s KTL RPC implementation relies on GNU libc extensions that were deprecated and removed, and multithreaded behavior that was present in Solaris but not ported to Linux. Some dependencies are less exotic, but can still cause problems at build and run time, such as libxml2.

Performance
^^^^^^^^^^^

Typical KTL requests must pass through several handling steps on their way to their destination, and the same is true for any response. The handoff between the C API and any KTL client library implementation establishes multiple places where queueing and threading are natural structures to control the flow of data; these queues, signaling steps, and translations result in lost time, either to increased latency, or a need for processing power, or both. This effect is not egregious, in that the original use cases likely did not intend for KTL to achieve frequencies higher than 100 Hz; performance up to an appreciable fraction of a kilohertz has been attained, though not relied on. Nevertheless, KTL is actively avoided for high frequency or low latency communications.

Bundling keywords
^^^^^^^^^^^^^^^^^

The KTL API treats keywords as isolated entities; there is no provision for linking multiple keywords into an atomic unit, either for commands or telemetry. Consider the case where a telescope is commanded to move: the right ascension and declination must both be specified as separate keywords, and then a third keyword triggered in order to begin the move. Consider the case of a motorized mechanism, which may have multiple encoders, motor power feedback, and other telemetry; KTL does not provide a method for a client to confidently assert that a set of telemetry is a self consistent snapshot of a system’s state.

Bulk data
^^^^^^^^^

The KTL API provides limited support for numeric arrays, which are effectively associative arrays; while this does in part address the bundling concern noted directly above, that is only true if the keywords are all numeric, and either all integers or all floating point numbers. Support for these array types is limited, both in terms of array size and in which KTL client libraries allow their use. The transport of an image buffer is one example of the type of data that falls outside what the KTL API can support.


Decision criteria
-----------------

The strengths and weaknesses described above can be distilled to a handful of guiding principles, listed in order of decreasing significance.

Ubiquity
^^^^^^^^

It is inevitable that mKTL will need to be implemented in multiple languages, not just Python, in order to maintain acceptance as an observatory standard over a multi-decade time span. Any approaches embedded in mKTL, and any key external technologies leveraged by mKTL, must therefore be ubiquitous and readily available across not just languages of choice, but generally available as standard options for a broad range of languages. This increases the odds that future languages are likely to provide similar support. Widespread adoption also implies widespread familiarity with the technologies and approaches, which could give new developers a head start on their introduction to mKTL.

Portability
^^^^^^^^^^^

The concept of portability may be derivative of ubiquity; regardless, mKTL and its dependencies must be readily buildable on a diverse set of potential platforms and architectures. This increases the odds that future platforms and architectures are likely to provide similar support.

Simplicity
^^^^^^^^^^

mKTL and its dependencies must not impose undue complexity in order to leverage their key functionality. Simple dependencies will allow mKTL code to remain clean and easy to follow, rather than impose their own structure on the design or implementation; mKTL itself should likewise seek to minimize boilerplate requiredfor its use.

Features
^^^^^^^^

mKTL and its dependencies must provide compelling features. For dependencies, if the cost of reimplementing specific functionality is low it is likely preferable to avoid the additional dependency entirely; for mKTL, the feature set should remain targeted to its core functionality, largely in support of achieving the other goals outlined here.

Performance
^^^^^^^^^^^

mKTL and its dependencies must be adequately performant such that the overall performance is not degraded by their use.


Core dependencies
-----------------

With these principles as guideposts, mKTL has adopted two key external technologies as foundational components for its functional goals: `ZeroMQ <https://zeromq.org/>`_ as a network transport, and `JSON <https://www.json.org/>`_ as a data interchange format.

Support for ZeroMQ spans virtually every programming language in use today, well beyond the set of languages of potential interest to the mKTL community; because of its widespread use and extensive history it is also the type of technology that will have a long tail of maintenance as it ages, even if it is no longer being actively developed. The features of ZeroMQ are also well aligned with the goals of mKTL, with respect to enabling a distributed architecture, having low overhead and high performance, transparent reconnect logic, and usage patterns that mimic simple sockets. By a happy coincidence ZeroMQ also implements an efficient PUB/SUB pattern well-suited to mKTL's functional requirements.

Similarly, JSON enjoys ubiquitous support for all modern programming languages, surpassed possibly only by XML; XML's gains in language support are offset by its bulky structure and inefficiency of parsing. JSON parsing is likewise inefficient and represents a significant source of processing overhead for mKTL messages; this inefficiency is accepted as it is not onerous enough to cause mKTL to miss its performance goals. Being able to natively represent numeric, string, boolean, and sequence data in JSON relieves mKTL of the need to invent its own parsing or complex message payload scheme; being able to represent the description of a store's items in JSON also eliminates ambiguity about the proper formatting of mKTL metadata.

