Overview
========

mKTL is a distributed messaging protocol designed for command and telemetry exchange between software components in observatory control systems. It is intended as the successor to the :ref:`Keck Task Library <heritage>` (KTL) application programming interface (API) used at `W. M. Keck Observatory <https://keckobservatory.org/>`_ (WMKO).

The basic logical unit of communication in KTL is a key/value pair. Each key/value pair may support request/response interactions (for retrieving or setting values) and publish/subscribe interactions (for receiving asynchronous updates). KTL systems are highly distributed: individual software components interface directly with hardware controllers or services and expose their functionality within a locally unique namespace. No central broker or authority is required for message routing.

mKTL messaging preserves these core design concepts while defining a modern messaging protocol that can be implemented consistently across multiple programming languages.

The protocol provides:

* A key/value abstraction for commands and telemetry
* Request/response messaging for synchronous interactions
* Publish/subscribe messaging for asynchronous updates
* A distributed architecture with no required brokers or intermediaries
* A stable, versioned wire protocol suitable for multiple independent implementations


Core concepts
-------------

mKTL organizes commands and telemetry using a hierarchical namespace composed of :ref:`stores <store>` and :ref:`items <item>`. A store is a collection of items. It is effectively an associative array, providing little additional functionality beyond being a container. A store will have a unique name within its local namespace.

An item is the smallest addressable unit in mKTL, a single key/value pair that may correspond to a command, a configuration parameter, a telemetry value, an image buffer, and so on. Examples of items might include a power on/off control, a device operating mode, a temperature reading, a motor position, or the most recent image from a detector.

Each item has a key that uniquely identifies it within its store; the same item key can be reused in multiple stores. The store name and the item key combine to create the fully qualified key, which uniquely identifies an item within a local mKTL context.

A :ref:`daemon <daemon>` is any application that is authoritative for one or more items, meaning, the daemon is responsible for publishing any new values, and for handling any requests to change the value of the item. A :ref:`client <client>` is any application that interacts with an item for which it is not authoritative.

.. mermaid:: overview_client.mmd

From a high level view, an mKTL client solely interacts with mKTL items, contained within an mKTL store; no direct interactions with the wire protocol or the transport are required, though a client could forego all interface code and work strictly at the protocol+transport level.

.. mermaid:: overview_daemon.mmd

The perspective from an mKTL daemon is the mirror image, with daemons likewise not requiring any direct interactions with the protocol or transport, though again a daemon could bypass all interface code as long as it adheres to :ref:`the mKTL protocol <protocol>`.
