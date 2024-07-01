While developing system, some module may update, this update include code or database schema.  
In order for the system to continue running smoothly while updating, we need to maintain compatibility in both directions:
- Backward compatibility :Newer code can read data that was written by older code.
- Forward compatibility: Older code can read data that was written by newer code.
Backward compatibility is easy when writing new version of code but, Forward compatibility can be trickier, because it requires older code to ignore additions made by a newer version of the code.

In this chapter we will look at several formats for encoding data, including JSON, XML, Protocol Buffers, Thrift, and Avro. In particular, we will look at how they han‐ dle schema changes and how they support systems where old and new data and code need to coexist. We will then discuss how those formats are used for data storage and for communication: in web services, Representational State Transfer (REST), and remote procedure calls (RPC), as well as message-passing systems such as actors and message queues.

# Formats for Encoding Data
The translation from the in-memory representation to a byte sequence is called encoding (also known as serialization or marshalling), and the reverse is called decoding (parsing, deserialization, unmarshalling).

> Serialization is unfortunately also used in the context of transactions (see Chapter 7), with a completely different meaning.

## Language-Specific Formats
Popular languages have built-in encoding like Java has java.io.Serializable, Ruby has Marshal , Python has pickle. But they have some problems:
- The encoding is often tied to a particular programming language, and reading the data in another language is very difficult.
- In order to restore data in the same object types, the decoding process needs to be able to instantiate arbitrary classes.
- Efficiency problems in using CPU and RAM.

## JSON, XML, and Binary Variants
They are widely known, widely supported, and almost as widely disliked. JSON, XML, and CSV are textual formats, and thus somewhat human-readable.
they also have some subtle problems:
- In XML and CSV, you cannot distinguish between a number and a string and JSON doesn’t distinguish integers and floating-point numbers. This is a problem when dealing with large numbers, for example, integers greater than 2 pow 53 cannot be exactly represented in an IEEE 754 double-precision floating- point number, so such numbers become inaccurate when parsed in a language that uses floating-point numbers (such as JavaScript). A solution is use decimal string. 
- They don’t support binary strings, people get around this limitation by encoding the binary data as text using Base64. it’s somewhat hacky and increases the data size by 33%.
- These schema languages are quite powerful, and thus quite complicated to learn and implement.

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

Thrift has two different binary encoding formats, iii called BinaryProtocol and CompactProtocol. 

### Thrift Binary Protocol
Each field has a type annotation (to indicate whether it is a string, integer, list, etc.) and, where required, a length indication (length of a string, number of items in a list). The strings that appear in the data (“Martin”, “daydream‐ ing”, “hacking”) are also encoded as ASCII (or rather, UTF-8), similar to before. The big difference compared to Json Binary is that there are no field names ( userName , favoriteNumber , interests ). Instead, the encoded data contains field tags, which are numbers ( 1 , 2 , and 3 ). Those are the numbers that appear in the schema definition.

![[Pasted image 20240701193356.png|600]]

### Thrift Compact Protocol
The Thrift CompactProtocol encoding is semantically equivalent to BinaryProtocol, it packs the same information into only 34 bytes. It does this by packing the field type and tag number into a single byte, and by using variable-length integers. Rather than using a full eight bytes for the number 1337, it is encoded in two bytes.

![[Pasted image 20240701193644.png|600]]

### Protocol Buffers
It's very similar to Thrift’s CompactProtocol. Protocol Buffers fits the same record in 33 bytes.
