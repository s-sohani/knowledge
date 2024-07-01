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
