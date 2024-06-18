Swift currently **powers the largest object storage clouds**, including Rackspace Cloud Files, the HP Cloud, IBM Softlayer Cloud and countless private object storage clusters
- Swift can be used as a stand-alone storage system or as part of a cloud compute environment.
- Swift runs on standard Linux distributions and on standard x86 server hardware
- Swift—like Amazon S3—has an eventual consistency architecture, which make it ideal for building massive, highly distributed + infrastructures with lots of unstructured data serving global sites.
- All objects (data) stored in Swift **have a URL**
- Applications store and retrieve data in Swift via an industry-standard **RESTful HTTP API**
- Objects can have extensive metadata, which can be indexed and searched
- All objects are stored with multiple copies and are replicated in as-unique-as-possible availability zones and/or regions
- Swift is scaled by adding additional nodes, which allows for a cost-effective l**inear storage expansion**
- When adding or replacing hardware, data **does not have to be migrated to a new storage system**, i.e. there are no fork-lift upgrades
- Failed nodes and drives can be swapped out while the cluster is running with no downtime. New nodes and drives can be added the same way.
- Swift client libraries such as Java, Python, Ruby, or JavaScript.

The storage URL has two basic parts:
- cluster location
- location -> acount(split per user)/container(bucket)/object

![[Pasted image 20240615150536.png]]

### The Swift HTTP API
- GET—downloads objects, lists the contents of containers or accounts
- PUT—uploads objects, creates containers, overwrites metadata headers
- POST—creates containers if they don't exist, updates metadata (accounts or containers), overwrites metadata (objects)
- DELETE—deletes objects and containers that are empty
- HEAD—retrieves header information for the account, container or object.

## Swift Overview
A Swift cluster is the distributed storage system used for object storage. Each machine running **one or more** Swift’s **processes** and services is called a node.
- Proxy Node
- Storage Node
	- Account
	- Container
	- Object

If a valid request is sent to Swift then the proxy server will verify the request, determine the correct storage nodes responsible for the data (based on a hash of the object name) and send the request to those servers concurrently.

The proxy server process is looking up multiple locations because Swift provides data durability by writing multiple–typically three complete copies of the data and storing them in the system.

#### Account Layer
The account server process handles requests regarding metadata for the individual accounts or the list of the containers within each account. This information is stored by the account server process in SQLite databases on disk.

#### Container Layer
 Like accounts, the container information is stored as SQLite databases.
 
#### Object Layer
Objects are stored as binary files on the drive using a path. The timestamp is important as it allows the object server to store multiple versions of an object. the data and metadata are stored together and copied as a single unit.

### Consistency Services
The two main consistency services are auditors and replicators.

#### Auditors
Auditors run in the background on every storage node in a Swift cluster and continually scan the disks to ensure that the data stored on disk. If an error is found, the auditor moves the corrupted object to a quarantine area.
There are account auditors, container auditors and object auditors.

#### Replicators
Account, container, and object replicator processes run in the background on all nodes that are running the corresponding services. A replicator will continuously examine its local node and compare the accounts, containers, or objects against the copies on other nodes in the cluster. If one of other nodes has an old or missing copy, then the replicator will send a copy of its local data out to that node.

## Cluster Architecture
Within a cluster the nodes will also belong to two logical groups: regions and nodes. Regions and nodes are user-defined and identify unique characteristics about a collection of nodes.
Regions are user-defined and usually indicate when parts of the cluster are physically separate --usually a geographical boundary.

