- Swift is an object storage system that is part of the OpenStack project
- Swift is open-source and freely available
- Swift currently **powers the largest object storage clouds**, including Rackspace Cloud Files, the HP Cloud, IBM Softlayer Cloud and countless private object storage clusters
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

