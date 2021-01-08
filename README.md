# MDBee Whitepaper

## Abstract

With this solution we want to implement an experimental application that can publish, and keep synced over time, an existent MongoDB data set on the decentralized Swarm network. These data must be accessible to everyone that know how to get and read them.

## Introduction

Documental databases are receiving more and more interest today on IT solutions. High performance and high scalability are their primary advantages over relational databases.

Documental databases can have several scopes, but in general they are primary used when we don’t care about a schema enforcement, we need high write/read performance, and application doesn’t require a lot of data updates. If we are in this case, we are in the situation that permits to denormalize documents, optimize reading operations, and avoid performing a lot of expensive tables JOIN operations, that require several disk accesses, a lot of memory consume and CPU time.

Another trend that is starting, and that we think will lead the web development on next years, is an ethical approach to data management. Transparency and verifiability through use of open data will be probably near to mandatory on services of next decades. An important step in this direction has been performed with introduction of blockchain technologies, that since 2008 with birth of Bitcoin, is revolutionizing the concept of trustability on the Internet. This set of new practices come together under the name of Web 3.0, or Web3.

But if blockchain have started a philosophical change of the net, it is not the only operator. Indeed, if we assume that Web3 will be virtually based on a globally distributed computer, we can see the blockchain as a CPU of this, able to perform applications state management with the implementation of a consensus system, but in general it is not able to distribute large amount of data. For this scope, new tools must be put beside, for represent the mass storage of this global computer.

Swarm is a new technology that permits to do this ([https://swarm.ethereum.org/](https://swarm.ethereum.org/)). It uses blockchain to share incentives, but data is shared directly across nodes, point-to-point, without use of transactions. It doesn’t need a consensus layer, so it’s not required that every node agree on same history. This is a focal point, because it permits to storage a large amount of information, every node owns its data independently and doesn’t need to agree with others on them. This also permits to let nodes to forget data, when there will be no one that cares to maintain it. Swarm is implemented over a pure p2p network, and everyone that know how to reach and interpretate uploaded data can access it. It’s a distributed and unstoppable technology, like blockchain.

These two kinds of instruments can collaborate for build a new paradigm of open data management and distribution. If a classical centralized database permits to an application maintainer to keep control over data, a new distribution layer based on Swarm, built over the database, can replicate and distribute data to everyone who is interested to receive it.

The automatic distribution system implemented by Swarm is perfect for avoid a lot of clients connecting directly to the database. It also keeps the communication private, encrypted, incensurable and unalterable. It also can scale very well, with an incentive system financed directly by who is interested into receive data, and encouraging more nodes to replicate it, proportionally to demand. So, it appears that Swarm can be a perfect solution for this scope.

Moreover, a documental database is a perfect candidate to be replicated over Swarm. A relational database also could be replicated, but a documental have some more advantages:

* A document is more expressive than a record. It doesn’t require to perform JOIN ops, so also doesn’t require different reads from different tables. Indeed, very often documents are already denormalize and optimized for the application use, and because every internet request is extremely time consuming, less requests we can perform, better it is.
* A document is very easy to transpose on an independent file. It can be easily serialized in binary or string and distributed as is. A table record instead have a lot of consistency constraints with the table containing it, and this force to keep important relations between a table, and all its records.
* A documental database doesn’t implement schema restrictions, so it’s defined schemaless. Because of this property, we can manage schemas only on application side, and don’t have to upgrade documents if our data schema evolves. This permits to keep already distributed data as is and doesn’t require to upload old documents again even after a schema upgrade.

Choosing what documental database is best to use today, we think that MongoDB is a great candidate. It’s a native documental database, it’s the most diffuse, and it is totally open source under Server Side Public License ([https://www.mongodb.com/licensing/server-side-public-license](https://www.mongodb.com/licensing/server-side-public-license)), a license directly derived from GNU AGPL v3.0.

## Objectives

With MDBee we want to accomplish different target points:

* It must work with any (or near any) current MongoDB deployment, possibly with as little as possible modifications, or none at all
* It must use MongoDB’s public interface as is, without any change
* It doesn’t have to affect native database performance, or the least as possible
* Public data must be exposed only through Swarm, so MongoDB’s entities must be converted into Swarm’s entities. This involves databases, collections, indexes, and documents
* Involved Swarm nodes must be fully compliant to Swarm’s network specifications
* Progressively any database evolution must be published and reachable on Swarm network. Periodic snapshots are acceptable, frequency is left to preferred configuration
* MongoDB’s nodes must keep full control on data evolution; public Swarm access will permit only read-only operations on a given database, except for database forking operations, that permits to start from a given state a full independent database instance owned by the forker
* Last snapshots, guaranteed by the database maintainer, must be always reachable on Swarm network. When a snapshot become obsolete and not guaranteed by defined policies, it could disappear from network
* Documents must be accessible individually on Swarm
* Any document update must involve as less as possible changes on already published data, keeping updates as close as possible to only diff updates
* Data should leave Swarm’s node only when requested, with a lazy publishing paradigm. This permit to not flood Swarm network with unwanted data updates
* Any entity that knows how to access the database must be able to download and replicate it, with a full functional similar copy of the source database
* Data must be accessible on Swarm also from live applications. Snapshots frequency will determine the data publication lag, but any application request must be satisfied with live responses from the network
* Human readability of data must be possible, depending on specific configuration. This in necessary to permit to everyone to simply understand contents of a document without use of external applications, and improve data accessibility
* Data can be published encrypted or in clear on Swarm network
* Simple addresses must be used for access data from a Swarm gateway, involving use of manifests. For example:
  * Database root manifest with `/<dbAddress>`
  * Database meta with `/<dbAddress>/$meta`
  * Collection meta with `/<dbAddress>/<collectionName>/$meta`
  * A document with `/<dbAddress>/<collectionName>/<docId>`
  * A document list with a specific index key, on an index different than Id `/<dbAddress>/<collectionName>/$indexes/<indexName>/<indexKey>`

## Architecture

Several kinds of architecture have been evaluated. Here will see them with related considerations.

First proposal, because MongoDB is an open-source project, try to fork it replacing the storage layer with a Swarm node. It could be executed as a secondary node on a replica set, and this would preserve performance on the primary one. This could have been done, but there are different issues:

* MongoDB code is very complex, and to maintain it would have requested high investments in development resources. Even because we should have had to align our releases with new MongoDB’s releases, with new feature and bugfixes
* Our custom version of MongoDB should have had to be used instead of the official one in clusters. Even if they both would be compatible, this could be a very difficult decision to take in production environments, even more if this can’t be a choose at all, like into cloud managed services that offers MongoDB as a SaaS
* A high performance and heavily optimized storage layer has been implemented with MongoDB, and to replace it could be not a good idea. This should have been used also for Swarm’s storage layer, and so in the end insert Swarm  as a layer between MongoDB’s orchestration logic, and the native storage layer. Even more complex

These considerations have started to indicate that to fork MongoDB could be not the best solution.

Second proposal is to develop this solution has a fake replica node, and to connect it to an existent MongoDB’s replica set. The node would be hidden, so client would never know not even the existence of this node, and it never would have to reply to queries. This would be a simpler solution, but with some disadvantages:

* The replica set protocol would have had to be reimplemented and maintained as is with every new release. It is a small portion of all Mongo’s code, but still a relevant dependency to align to
* Replica set is a delicate environment, and even in this case can be hard to change a current architecture for add another custom node. Even because replica’s node quantity is relevant with system working, and because could be very difficult to config a custom node on-premises with cloud SaaS offers
* A replica set is an architecture that doesn’t know nothing about sharding. Mongo’s sharding can implement inside of each shard an independent replica set, but this requires that each shard runs a distinct instance of the custom node in each set. Not impossible, but maybe could be resolved with a better solution

At this point we have chosen to make a step back and ask ourselves if to replicate a Mongo’s node is really the best choose on Swarm, and we have discovered that it isn’t. We have discovered that both storages of Mongo and of Swarm would had to contain same data, but with different approaches, for full different scopes:

* MongoDB is oriented to store documents, Swarm is oriented to store chunks
* Documents can change and be relocated, Swarm chunks instead never change
* MongoDB is full oriented to read/write performance for application clients, Swarm instead must orchestrate with network-oriented features, like chunk caching with request forwarding
* MongoDB’s data isn’t deduplicated on disk, instead Swarm natively do it
* We will publish data with consistent snapshots on Swarm, but this aren’t well supported with Mongo. If I remove a document on Mongo, but I’d like to request it on Swarm network from a previous snapshot, the requiring user needs a way for identify and get it. If the document’s chunks are not found on net, but I’m guaranteeing that they must be available, I would have had to build a system that doesn’t really delete them on Mongo, where with Swarm I only need to pin the snapshot manifest, and unpin it when outdated

So, these two must be totally distinct solutions, with different goals.

The last propose is to connect a simple controller application between a Mongo and a Swarm node instances. We can also use Mongo’s official drivers, that permits to handle complexity of communication protocol. This application can use Mongo’s "change stream" ([https://docs.mongodb.com/manual/changeStreams/](https://docs.mongodb.com/manual/changeStreams/)) for receive updates, and like a replica node to update data on Swarm’s node with every change performed on database. Change streams permits to communicate only diff between nodes, and don’t have to read the full database for each snapshot.

This solution has advantages; it doesn’t require to handle complexity of MongoDB’s implementation, and the only dependency is to official drivers, that can be upgraded as a simple external library. This still have disadvantages, but very smaller if compared to others:

* This requires duplicating data, but like any replica node would had required. Advantage of this is that differently from a new replica node, this solution doesn’t require to add a machine on a replica set, and instead will work simply connecting like a client
* Mongo’s change stream is supported only on replica set (and shards), because it’s derived from Mongo’s “oplog”, that is how replica nodes propagate data changes. This is not a big issue, because like every production distribution of MongoDB is implemented with a replica set. Apart from that, ANY replica set is supported
* A sync protocol must be implemented, but it can be simpler and independent from the one implemented natively by Mongo's replica nodes

Other advantages with this solution are that we can talk directly with “mongos” instances, so don’t have to worry about different Mongo shards, if our Swarm node can handle changes from all shards by itself. If it can’t, solution is more complex. In this case we can connect a different Swarm node to each different Mongo shard, and eventually merge them from an additional node, with common manifests.

More research must be executed on how to shard Swam nodes with this application.

## Protocol

Here we will see more about how MDBee will work.

### Initial sync

When we execute for the first time the software, we must connect it to a Mongo node, operating into a replica set or a sharded system, and to a Swarm node. So, we must choose what databases and collections we want to publish on Swarm.

After this it will start to sync, transferring from Mongo node to the Swarm node all selected data. The protocol used will be very similar to the protocol implemented by MongoDB itself when it must sync with a replica node. It’s described here ([https://github.com/mongodb/mongo/blob/master/src/mongo/db/repl/README.md#initial-sync](https://github.com/mongodb/mongo/blob/master/src/mongo/db/repl/README.md#initial-sync)).

After that sync has been completed, the application will move into the data maintenance mode.

### Data maintenance

When the Swarm node is synced, the application must stay in maintenance mode, where it will receive any data change using the “change stream” from the Mongo node for update the local store. This stream is built with the “oplogs” built from Mongo for keep synced replica nodes and shards, and reproduce any single change performed on data sets.

With this stream it will be able to update current documents on Swarm node and build new manifests of active indexes that must be published with the database snapshot. Every change will be pinned locally on Swarm node, but for avoid flooding the network with unwanted chunks, they shouldn’t be synced until the first request to access them is received.

If we will keep receiving for example continuous updates on a document, but this document is never accessed in read from Swarm between an update and the other, it’s not useful for the network to keep pushing updates on it. Every chunk would fall into an outdated state, and this would only consume bandwidth and disk space on receiving nodes. Instead, we should follow a lazy paradigm, where any chunk will be uploaded only when a user requests it. This can be done with use of the global pinning.

Global pinning, or recovery protocol, is a Swarm feature that permits to request to specific nodes to reupload a chunk if this have been removed from the neighbourhood where it should have been recorded. Obviously, the request destination must maintain a copy of the chunks, for example having pinned them, and the requiring user must know who the node is.

Every document update changes the related Swarm address, so every manifest linking it must change its references. For how are built manifests on Swarm, every document change has a complexity of log(n) on manifest updates, where n is the number of total stored documents. This due to the hierarchical organization of manifests of Swarm, that works like a Merkle tree.

### Snapshot publication

Periodically, with a frequency that can be configured, a snapshot of the database will be published.

We are keeping an always updated local copy of documents and manifests, and data will be pushed with laziness on network, only when nodes request them. So, what we need to do at each snapshot is only to publish a Single Owner Chunk (SOC) containing references to:

* The address of the root manifest of last database snapshot, the entry point
* The address of node that is pinning chunks, for use with global pinning

The root database manifest will be locally pinned, and when we have an outdated manifest it can be unpinned from local node.

A SOC will be published with a reachable address using a periodic Swarm Feed schema, where we can deduce what the address is knowing the publisher’s address, the database identifier, and the timestamp of the publication, adjusting its precision with the frequency of publication.

### Data access

When users want to access data, they must know:

* The publisher’s address
* The database identifier
* The interested timestamp of publication

With these information it will be able to calculate address of the SOC with root manifest information. An application can be provided for calculate the address.

When user has found the SOC, inside will find the address of the root manifest, its entry point to the database, and the address of node that exposes the global pinning. Requiring the chunk with a global pinning information, if chunk exists on Swarm network it will be searched on Kademlia and so downloaded, instead if it doesn’t exist it will be requested by Swarm protocol to the node that owns it. In any way, if snapshot is not outdated and is so available, it will be downloaded.

From root manifest the user will be able to get database metadata, list of collections and list of indexes. These will be navigable with url composition.

Because with this process we will be able to publish potentially a full database, with all its information, we can also perform a full download of data, and replicate it locally. It can be used for execute independent backups, and for keep decentralized history of its evolution. Will be also possible to fork it, creating a local copy of a snapshot, for example using a local instance of MongoDB, and to start to work on the copy as an independent clone of the original.

### Data encoding

Data will be encoded into documents and manifests. Manifests are used for composing hierarchical navigation structure and indexes. Documents instead will be the real contained data of database. Each published MongoDB’s document will be a different document on Swarm, with its unique address. Every time that a document changes, also its address does.

Inside a document we will find the related Mongo’s document serialized with a JSON-like language. This is chosen because primary scope of this project is to enhance transparency, and with a small cost on document size, if compared with binary encoding, we gain human readability. "Human before machine" must be the motto.

Documents could also be published compressed, but compression is expensive and must be performed server side, and can be performed only on single documents. This is necessary because we need to maintain possibility to upload only upgraded documents at each snapshot, reusing what is already been uploaded, and compression for example of the full collection would require the full collection to be uploaded each time that a new snapshot is requested. Obviously, without possibility to compress the full collection, the compression rate would be very low, and it eliminates any kind of chunk deduplication on documents. Because of this, compression isn’t an our goal currently.

## Future developments

First version of application will support only an index on Id per collection, so documents will be reachable only if we know its Id, or with a full manifest scan. On future versions we can build manifests with secondary indexes, and these can be mapped on arbitrary fields. Also different from the Mongo's indexes.

This will permit also to arbitrary user applications to perform searches on document keys, and maybe directly use this distributed database for perform read only queries to use for example with dapps, without need to request them to API of a service provider.

Other developments will lead to capability to scale data publication to shards of Swarm nodes, where each shard will be able to handle a portion of data to publish. All published data will then be merged on united manifests, and the database on Swarm will be accessible as always, without the clients need to know that behind the scenes there were shards of nodes to operate on documents. MongoDB’s shards don’t need to be ported on Swarm; Swarm’s shards should be a totally different concept.

On a secondary moment we can also create a tool that perform inverse process, so from a Swarm snapshot is able to build a local instance of MongoDB, that is a clone of the original source. This is an independent fork, and can be used for various scopes, like backup, service replication or data analysis.

We could also publish a secondary stream with all oplogs, so a listening process can handle all data updates without have to scan every snapshot for discover diffs.
