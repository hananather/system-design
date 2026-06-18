## Data encloding formats

Producer

metadata: key-value-pair (content type, timestamp, payload size)

 + payload: sequence of bytes (byte array)

broker

Metadata + payload

Consumer: has to know how to parse the payload.
- both the consumer and the producer need to agree on the encoding format of the payload

Terms:

message content (objects): applications store data in memory in the form obobjects

when we need to send the data over the network, they have to convert objects into a format that can be transmitted across the network (and reconstructed later on the other side).

The process of converting an objected into a sequence of bytes is called **encoding** or **seralization**.

Encoding formats:
- **Textual** (JSON, XML, CSV)
  - Pros: human readible + widely supported by language and tools.
  - Cons: bigger messages (slower to transfer)
- **binary** (thrift, protobuf, avro)
  - Pros: smaller messages (faster to transfer and less space needed to store) (2) Fast to serialize/deseriallize
  - Cons: not human readble (harder to debug and test)

**Whats the cause of the biggest space saving?**

**Schema**
- A lot of space in a JSON document is occupied by field names

**Both produce and consumer need to share the schema**

Schema sharing options:
- **Through code:** (thrift and protobuf  are based on this principle)
- **Schema registry** (avro schema)
- **send along wiht the payload: **
  - Cons:** **increased message size (higher transmission latency and storage cost)
  - Pros: no need to build and maintain a schema registry (2) easier to re-process messages afterwards

Schema evolution: schema will change over time

In distributed systems when we need to deploy a new version of a message (schema) we cannot update producer and consumer instances all at once. For some period of time messages with different versions will coexist in the system. And all these messages must be successfully processed.
