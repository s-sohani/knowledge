While developing system, some module may update, this update include code or database schema.  
In order for the system to continue running smoothly while updating, we need to maintain compatibility in both directions:
- **Backward compatibility** :Newer code can read data that was written by older code.
- **Forward compatibility**: Older code can read data that was written by newer code.
Backward compatibility is easy when writing new version of code but, Forward compatibility can be trickier, because it requires older code to ignore additions made by a newer version of the code.

Whenever you want to send data over the network or write it to a file—you need to encode it as a sequence of bytes. We then discussed a variety of different encodings for doing this.

In this chapter we will look at several formats for encoding data, including **JSON, XML, Protocol Buffers, Thrift, and Avro**. In particular, we will look at how they han‐ dle schema changes and how they support systems where old and new data and code need to coexist. We will then discuss how those formats are used for data storage and for communication: in web services, Representational State Transfer (**REST**), and remote procedure calls (**RPC**), as well as **message-passing** systems such as actors and message queues.

# Formats for Encoding Data
The translation from the in-memory representation to a byte sequence is called **encoding** (also known as **serialization** or **marshalling**), and the reverse is called decoding (**parsing**, **deserialization**, **unmarshalling**).

> Serialization is unfortunately also used in the context of transactions (see Chapter 7), with a completely different meaning.

## Language-Specific Formats
Popular languages have built-in encoding like Java has java.io.Serializable, Ruby has Marshal , Python has pickle. But they have **some problems**:
- The encoding is often tied to a particular programming language, and reading the data in **another language** is very difficult.
- In order to restore data in the same object types, the decoding process needs to be able to **instantiate arbitrary classes**.
- **Efficiency** problems in using CPU and RAM.

## JSON, XML, and Binary Variants
They are widely known, widely supported, and almost as widely disliked. JSON, XML, and CSV are **textual formats**, and thus somewhat **human-readable**.
they also have some subtle problems:
- In XML and CSV, you cannot **distinguish** between a **number** and a **string** and JSON doesn’t **distinguish** **integers** and **floating-point** numbers. This is a problem when dealing with large numbers, for example, integers greater than 2 pow 53 cannot be exactly represented in an IEEE 754 double-precision floating- point number, so such numbers become **inaccurate** when parsed in a language that uses floating-point numbers (such as JavaScript). A solution is use decimal string. 
- They **don’t** support **binary strings**, people get around this limitation by encoding the binary data as text **using Base64**. it’s somewhat hacky and **increases the data** size by 33%.
- These schema languages are quite powerful, and thus quite **complicated** to learn and **implement** (Converting data to object).

### Binary encoding
For internally communication you could choose a format that is more compact or faster to parse.
![[Pasted image 20240701190207.png|600]]

![[Pasted image 20240701190229.png|600]]

The binary encoding is 66 bytes long, which is only a little less than the 81 bytes taken by the textual JSON encoding (with whitespace removed). In the following sections we will see how we can do much better, and encode the same record in just 32 bytes.

## Thrift and Protocol Buffers
Apache Thrift and Protocol Buffers (protobuf) are binary encoding libraries that are based on the same principle. Protocol Buffers was originally developed at Google, Thrift was originally developed at Facebook, and both were made open source.
Both Thrift and Protocol Buffers require a schema for any data that is encoded.
### Thrift interface
![[Pasted image 20240701192841.png|400]]

### Protocol Buffers
![[Pasted image 20240701192925.png|400]]

Thrift has two different binary encoding formats called BinaryProtocol and CompactProtocol. 

### Thrift Binary Protocol
Each field has a type annotation (to indicate whether it is a string, integer, list, etc.) and, where required, a length indication (length of a string, number of items in a list). The strings that appear in the data (“Martin”, “daydream‐ ing”, “hacking”) are also encoded as ASCII (or rather, UTF-8), similar to before. The big difference compared to Json Binary is that there are no field names ( userName , favoriteNumber , interests ). Instead, the encoded data contains field tags, which are numbers ( 1 , 2 , and 3 ). Those are the numbers that appear in the schema definition.

![[Pasted image 20240701193356.png|600]]

### Thrift Compact Protocol
The Thrift CompactProtocol encoding is semantically equivalent to BinaryProtocol, it packs the same information into only 34 bytes. It does this by packing the field type and tag number into a single byte, and by using variable-length integers. Rather than using a full eight bytes for the number 1337, it is encoded in two bytes.

![[Pasted image 20240701193644.png|600]]

### Protocol Buffers
It's very similar to Thrift’s Compact Protocol. Protocol Buffers fits the same record in 33 bytes. In the schemas shown earlier, each field was marked either required or optional , but this makes no difference to how the field is encoded (nothing in the binary data indicates whether a field was required). The difference is simply that required enables a runtime check that fails if the field is not set, which can be useful for catching bugs.
![[Pasted image 20240701194041.png|600]]

### Field tags and schema evolution
How change schema in Thrift and Protocol Buffer while keeping backward and forward compatibility? 
- If a field value is not set, it is simply omitted from the encoded record.
- You can change the name of a field in the schema.
- You cannot change a field’s tag, since that would make all existing encoded data invalid.
- If old code (which doesn’t know about the new tag numbers you added) tries to read data written by new code, including a new field with a tag number it doesn’t recognize, it can simply ignore that field.
- The only detail is that if you add a new field, you cannot make it required.
- Every field you add after the initial deployment of the schema must be optional or have a default value.
- You can only remove a field that is optional.
- You can never use the same tag number again.

### Datatypes and schema evolution
What about changing the datatype of a field?
- There is a risk that values will lose precision or get truncated. For example, say you change a 32-bit integer into a 64-bit integer. the old code is still using a 32-bit variable to hold the value. If the decoded 64-bit value won’t fit in 32 bits.

## Apache Avro
- It is a subproject of Hadoop.
- It has two schema languages: one (Avro IDL) intended for human editing, and one (based on JSON) that is more easily machine-readable.
![[Pasted image 20240707061758.png||600]]
- There is nothing to identify fields or their datatypes. The encoding simply consists of values concatenated together.
![[Pasted image 20240707062000.png|600]]
### The Reader’s schema
- The binary data can only be decoded correctly if the code reading the data is using the exact same schema as the code that wrote the data.
- When data is decoded (read), the Avro library resolves the differences by looking at the writer’s schema and the reader’s schema side by side and translating the data from the writer’s schema into the reader’s schema.
- To maintain compatibility, you may only add or remove a field that has a default value.
- It’s no problem if the writer’s schema and the reader’s schema have their fields in a different order, because the schema resolution matches up the fields by field name.
- Avro doesn’t have optional and required markers in the same way as Protocol Buffers and Thrift do.
- Changing the datatype of a field is possible, provided that Avro can convert the type.
- Changing the name of a field is possible but a little tricky: the reader’s schema can contain aliases for field names, so it can match an old writer’s schema field names against the aliases. This means that changing a field name is backward compatible but not forward compatible.

### The Writer’s schema
Avro use case:
- Large file with lots of records: The writer of that file can just include the writer’s schema once at the beginning of the file.
- Database with individually written records: Include a version number at the beginning of every encoded record and to keep a list of schema versions in your data base. A reader can fetch a record, extract the version number, and then fetch the writer’s schema for that version number from the database.
- Sending records over a network connection: Two process can negotiate the schema version on connection setup and then use that schema for the lifetime of the connection.

# Modes of Dataflow
In the rest of this chapter we will explore some of the most common ways how data flows between processes:
- Via databases
- Via service calls
- Via asynchronous message passing

## Dataflow Through Databases
Several applications or single application (running multi instance of an application) may access to database. It is necessary consider Backward and forward compatibility. If the application changes, some process runs the new version and other runs an old version of the application. This means that a value in the database may be written by a newer version of the code, and subsequently read by an older version of the code that is still running. Thus, forward compatibility is also often required for databases. For example you add new field in new version of an application, The old version wants to update a record, in this situation, the unknown field might be lost in that translation process. 
![[Pasted image 20240729092247.png|600]]
### Different values written at different times
May be you have large database since five years ago, update  application change entire schema. In this situation the whole data should be update, this is an expensive process for large database so most databases avoid it if possible. 
## Dataflow Through Services: REST and RPC
Communicate process over a network. The arrangement calls `Client` and `Server`. 
There are variance client like web-browser, native app running on a mobile, java script, web service. 
Moreover, a server can itself be a client to another service.
When HTTP is used as the underlying protocol for talking to the service, it is called a web service.
There are two popular approaches to web services: REST and SOAP.
The API of a SOAP web service is described using an XML-based language called the Web Services Description Language, or WSDL. WSDL enables code generation so that a client can access a remote service using local classes and method calls.

### The problems with remote procedure calls (RPCs)
EJB is Java Rpc that limited for Enterprise java bean, RMI also limited for java, DCOM is limited to Microsoft platform. All of thease are based on the RPC idea that limit to a programming language. 
Despite all these problems, RPC isn’t going away. Various RPC frameworks have been built on top of all the encodings mentioned in this chapter: for example, Thrift and Avro come with RPC support included, gRPC is an RPC implementation using Protocol Buffers, Finagle also uses Thrift, and Rest.li uses JSON over HTTP.
Custom RPC protocols with a binary encoding format can achieve better perfor‐ mance than something generic like JSON over REST. However, a RESTful API has other significant advantages: it is good for experimentation and debugging.
For these reasons, REST seems to be the predominant style for public APIs. The main focus of RPC frameworks is on requests between services owned by the same organi‐ zation, typically within the same datacenter.

## Message-Passing Dataflow
Send message through broker. Send async message. 

### Distributed actor frameworks
The actor model is a programming model for concurrency in a single process. Rather than dealing directly with threads (and the associated problems of race conditions, locking, and deadlock), logic is encapsulated in actors. Each actor typically represents one client or entity, it may have some local state (which is not shared with any other actor), and it communicates with other actors by sending and receiving asynchro‐ nous messages. Message delivery is not guaranteed: in certain error scenarios, mes‐ sages will be lost. Since each actor processes only one message at a time, it doesn’t need to worry about threads, and each actor can be scheduled independently by the framework. In distributed actor frameworks, this programming model is used to scale an applica‐ tion across multiple nodes. The same message-passing mechanism is used, no matter whether the sender and recipient are on the same node or different nodes. If they are on different nodes, the message is transparently encoded into a byte sequence, sent over the network, and decoded on the other side.
Actor frameworks such as `Akka` and `Orleans`.