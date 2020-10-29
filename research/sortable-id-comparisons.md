## A comparison of non-Standard Time-based K-sorted, Lexicographically Unique Identifiers
---
### Name: Example Formatting
- Full ID Format (with all the moving parts concatenated)
- Total Length (and encoding)
- Timestamp Format (and format)
- Node Format (extra data in the ID)
- Collision Handling
- Accuracy (of the timestamp)
- Downsides (if there are any glaring errors)
- Source (of the analysis)
---
### Name: Twitter's Cassie (LexicalUUID)
- Full ID Format: timestamp|machineHostnameHash
- Total Length: 128 Bits
- Timestamp Format: 64 Bits, Epoch
- Node Format: 64 bit hash of the machine's hostname
- Accuracy: MicrosecondEpochClock
- Collision Handling: n/a
- Downsides: 
- Source: https://github.com/twitter-archive/cassie
---
### Name: Twitter's Snowflake
- Full ID Format: timestamp|machineID|sequence
- Total Length: 64 bits, unsigned integers
- Timestamp Format: 41 bits, NTP Synced, bespoke epoch
- Node Format: 10 bit configured machine id, 12 bit sequence number
- Collision Handling: Explicit Sequence Number
- Accuracy: Milliseconds
- Downsides: 
- Source: https://github.com/twitter-archive/snowflake/releases/tag/snowflake-2010
---
### Name: Boundary's Flake (inspired by Snowflake)
- Full ID Format: timestamp|mac|sequence
- Total Length: 128 Bits, Base62
- Timestamp Format: 64 Bits, Epoch 
- Node Format: 48 Bit Worker ID (MAC), 16 Bit sequence number
- Accuracy: Milliseconds
- Collision Handling: Explicit Sequence Number
- Downsides: MAC Address in the node
- Source: https://web.archive.org/web/20131231222024/http://boundary.com/blog/2012/01/12/flake-a-decentralized-k-ordered-unique-id-generator-in-erlang/, https://github.com/boundary/flake
---
### Name: Instagram's Sharding ID (inspired by Snowflake)
- Full ID Format: timestamp|shardID|sequence
- Total Length: 64 bits
- Timestamp Format: 41 bits (epoch)
- Node Format: 13 bits for shard ID, 10 bits for sequence counter
- Accuracy: Milliseconds
- Collision Handling: Explicit Sequence Number
- Downsides:
- Source: https://instagram-engineering.com/sharding-ids-at-instagram-1cf5a71e5a5c
---
### Name: Segment's K-Sortable Unique IDentifier (KSUID) (inspired by Snowflake)
- Full ID Format: timestamp|random
- Total Length: 160 Bits, base62 text format
- Timestamp Format: 32 bits, UTC timestamp (epoch)
- Node Format: 128 random payload
- Accuracy: Seconds
- Collision Handling: Random Value only
- Downsides: Larger than other ID's resulting in more data per ID
- Source: https://github.com/segmentio/ksuid, https://segment.com/blog/a-brief-history-of-the-uuid/
---
### Name: Elasticsearch's Elasticflake (inspired by Flake)
- Full ID Format: timestamp|mac|sequence
- Total Length: 120 Bits, Base64
- Timestamp Format: 48 Bits
- Node Format: 48 Bits for MAC, 24 Bits for Sequence Counter
- Accuracy: Milliseconds
- Collision Handling: Explicit Sequence Number
- Downsides: MAC Address in the node
- Source: https://github.com/ppearcy/elasticflake
---
### Name: Flake ID (Inspired by Flake?)
- Full ID Format: timestamp|datacenter|worker|sequence
- Total Length: 64 bits, Big Endian, 
- Timestamp Format: 42 Bits, Epoch
- Node Format: 5 bit datacenter, 5 bit worker, 12 bit counter
- Accuracy: Milliseconds 
- Collision Handling: Explicit Sequence Number
- Downsides: 
- Source: https://github.com/T-PWK/flake-idgen
---
### Name: Sony's Sonyflake (inspired by Flake?)
- Full ID Format: timestamp|sequence|machineID
- Total Length: 63 bits?
- Timestamp Format: 39 bits
- Node Format: 8 bit sequence number, 16 bit machine id
- Accuracy: 10 milliseconds
- Collision Handling: Explicit Sequence Number
- Downsides: Interesting number of total bits in the ID length
- Source: https://github.com/sony/sonyflake
---
### Name: UUIDv1
- Format: timeLow|timeMid|timeHigh|version|clockSeqHigh|Variant|clockSeqLow|NodeID
- Total Length: 128 characters, UUIDv1 format
- Timestamp Format: 60 bit timestamp (32 bits timeLow, 16 bits timeMid, 12 bits timeHigh)
- Node Format: 4 bit UUID Version, 3 Bit Variant, 5 bit clockSeqHigh,  8 bits clockSeqLow, 48 bit MAC Address
- Accuracy: 100-nanosecond intervals
- Collision Handling: Explicit Sequence Number (clockSeqHigh and clockSeqLow)
- Downsides: Timestamp arranged in a sub-optimal way, MAC included in the calculation, Gregorian epoch + 100-nanosecond increment rather than standard epoch + sec/ms
- Notes: Also used by Microsoft's NEWSEQUENTIALID (UuidCreateSequential) [Source](https://devblogs.microsoft.com/oldnewthing/20191120-00/?p=103118). Also used by Apache's Cassandra (TIMEUUID) [Source](https://cassandra.apache.org/doc/latest/cql/functions.html?highlight=timeuuid)
- Source: http://www.ietf.org/rfc/rfc4122.txt
---
### Name: Laravel's Str::orderedUuid()
- Full ID Format: timestamp|random (Looks like UUIDv4 format)
- Total Length: 128 Bits
- Timestamp Format: 48 bits, Server Time (Epoch?)
- Node Format: 72 Random Bits
- Accuracy: Milliseconds
- Collision Handling: Random Value only
- Downsides:
- Source: https://itnext.io/laravel-the-mysterious-ordered-uuid-29e7500b4f8
---
### Name: COMB GUID
- Full ID Format: timestamp|random (Looks like UUIDv4 format)
- Total Length: 128 Bits
- Timestamp Format: 48 Bits, Epoch format
- Node Format: 74 Random Bits
- Accuracy: Milliseconds
- Collision Handling: Random Value only
- Downsides: Date may be formatted at the start or the end to accommodate SQL Sort or PostgreSQL Sort. The DateTime strategies described above are limited to 1-3ms resolution, which means if you create many COMB values per second, there is a chance you'll create two with the same timestamp value. This won't result in a database collision--the remaining random bits in the GUID protect you there. But COMBs with exactly the same timestamp value aren't guaranteed to sort in order of insertion, because once the timestamp bytes are sorted, the sort order will rely on the random bytes after that.
- Source: https://github.com/richardtallent/RT.Comb
---
### Name: Universally Unique Lexicographically Sortable Identifier (ULID)
- Full ID Format: timestamp|random
- Total Length: 128 bits, Crockford's Base32 text format
- Timestamp Format: 48 bits (Unix Time)
- Node Format: 80 bits
- Accuracy: Milliseconds
- Collision Handling: Random Value only
- Downsides: 26 character Base32 can contain 130 bits where this UUID is 128 resulting in a potential decode error on a base32 string.
- Source: https://github.com/ulid/spec
---
### Name: Sortable Identifier (SID)
- Full ID Format: timestamp|random
- Total Length: 128 Bits, Hex, Base64, base32
- Timestamp Format: 64 bit timestamp
- Node Format: 64 bit random
- Accuracy: Nanoseconds
- Collision Handling: If (by any chance) this is called in the same nanosecond, the random number is incremented instead of a new one being generated.
- Downsides: 
- Source: https://github.com/chilts/sid
---
### Name: Better GUID
- Full ID Format: timestamp|random
- Total Length: 136 bits, base64 web-safe chars
- Timestamp Format: 64 bits
- Node Format: 72 bits of random
- Accuracy: millisecond
- Collision Handling: Monotonically incrementing "random"
- Downsides: Monotonically incrementing "random", second largest of all IDs
- Source: https://github.com/kjk/betterguid
---
### Name: Google's Firebase pushID
- Full ID Format: timestamp|random
- Total Length: 120 Bits, Modified Base64 ASCII encoded so the ascii can be sorted as well
- Timestamp Format: 48 bit timestamp
- Node Format: 72 bits of randomness
- Accuracy: Milliseconds
- Collision Handling: Built-in least significant bit increment in the event of multiple generations at the same millisecond
- Downsides: Client side implementation where clock skews can be prevalent
- Source: https://firebase.googleblog.com/2015/02/the-2120-ways-to-ensure-unique_68.html, https://gist.github.com/mikelehen/3596a30bd69384624c11, https://github.com/firebase/firebase-js-sdk/blob/master/packages/database/src/core/util/NextPushId.ts
---
### Name: XID
- Full ID Format: timestamp|machineID|processID|sequence
- Total Length: 96 bits, Base32
- Timestamp Format: 32 bit timestamp, Unix Epoch
- Node Format: 24 bit machine ID, 16 bit process ID, 24 bit counter (random start)
- Accuracy: Seconds
- Collision Handling: Explicit Sequence Number
- Downsides:
- Source: https://github.com/rs/xid
---
### Name:  mongoDB's ObjectID
- Full ID Format: timestamp|random|sequence
- Total Length: 96 bits
- Timestamp Format: 32 bit timestamp
- Node Format: 40 bit random, 24 bit sequence (random start)
- Accuracy: Seconds
- Collision Handling: Explicit Sequence Number
- Downsides:
- Source: https://docs.mongodb.com/manual/reference/method/ObjectId/
---
### Name: unique collision-resistant ID (CUID)
- Format: c|timestamp|sequence|clientFingerprint|random
- Total Length: ~124 bits, Base36
- Timestamp Format: ~62 bits (timestamp|sequence)
- Node Format: ~62 bits (clientFingerprint|random)
- Accuracy: Milliseconds
- Collision Handling: Explicit Sequence Number
- Downsides: math.random
- Source: https://github.com/ericelliott/cuid