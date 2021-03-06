## Analysis of current non-Standard Time-based K-sorted, Lexicographically Unique Identifiers to answer the questions:
---
### Length
**Q:** Are we sticking with 128-bits or are we introducing variable-length.
- The most common implementations utilize 128 bits for globally unique values. This should be kept for backwards compatibility.
- Where required, to best optimize database sizes, 64 bits is the second most common and provide a level of uniqueness sufficient to a specific applications context.
- Many articles exist on the general topic of "how long should an ID be to guarantee uniqueness" most agree that 128 is great, 64 is just as good if you only require uniqueness in the application context.
- Larger than 128 bits is not required from a collision avoidance perspective and adds extra unneeded data to the application/database/wire/etc 

Sources:
+ https://www.slideshare.net/davegardnerisme/unique-id-generation-in-distributed-systems
+ https://medium.com/javascript-in-plain-english/you-might-not-need-uuid-v4-for-generating-random-identifiers-89e8a28a7d77
+ https://towardsdatascience.com/are-uuids-really-unique-57eb80fc2a87
+ https://www.percona.com/blog/2019/11/22/uuids-are-popular-but-bad-for-performance-lets-discuss/
+ https://eager.io/blog/how-long-does-an-id-need-to-be/?hn

### Timestamp (and the other friends)
**Q:** Should a sort order be defined, if so what.
- The sort order should be up to the application and may sort slightly differently currently.
- This spec should aim to provide a better sortable timestamp UUID.
- UUIDv1 is technically sortable, just not in the most efficient way.
- An efficient format for k-sorting/k-ordering and Lexicographically sorting should be the goal.

---
**Q:** What should the format for the values be?
Most agree that "unique identifiers" such as a MAC address are:
- Flawed logic because they are not truly unique.
- Don't work well in virtual environments.
- They also pose an inherent security risk and should be avoided at all costs.

A note on Security:
- If security is of any concern or the UUID will be used for security operations then RFC 4122 UUIDv4 should be used. (This would be good to document in the security considerations)
- The timestamp embedded within an ID does expose some data about when the corresponding data was created and the time on the current server.
- However the timestamp alone articulates nothing about the data itself or the entire database or application as a whole.

The current UUIDv6 draft proposes:
- A 60 bit timestamp sourced from the UTC Gregorian calendar and big Indian encoding. This ultimately re-ranges the original spec to contain High, Mid, Low time values to preserve their position in time and promote better sorting ability
- The clock sequence sections and node are relaxed to include random bytes.
- Note that the clock sequence may be valuable to keep as per some discussion coming up later

A note on the time source for the timestamp
- UUIDv1 and proposed UUIDv6 utilize UTC Gregorian that are 100-nanosecond intervals.
- Many other implementations utilize Unix Epoch time
- Many others utilize a custom epoch date as the start
- Recommendation: Relax the spec and allow for any properly synced, monotonic time source as long as the required amount of time bits can be achieved.

In examining other implementations there are two outstanding approaches that can be found.
1. timestamp|random
- This approach takes an input timestamp of varying length (more on this in a moment) and concatenates random data amounting to the leftover space.
- The timestamp itself is usually epoch time as a standard rather than the UUIDv1 timestamp which uses Gregorian time.
- The timestamp is almost always provides milliseconds of accuracy and generally is either 48 or 64 bits.
- Many other timestamp bit sizes exist, for example 32, but are often used with ID implementations less than 128 bits to provide an accurate timesetamp and also leave room for random and other bits. For implementations that utilize 64 bits a smaller timestamp is acceptable.
- Downside: If more than one ID is generated at the same millisecond value the "random" value acts as collision avoidance. While this is good from a collision avoidance perspective now these values are not truly sortable.
- Note: Some libraries have built in "collision avoidance" where the ID generator that has created (and then detected) a duplicate ID in some way will increment a random bit, usually the least significant bit.
  Although the probably of a collision is low, I am not a fan of this approach which is sub-optimal for distributed environments where many nodes are creating IDs.
  Furthermore, What checks are in place to confirm this incremented value does not collide with another existing value somewhere on the system?

2. timestamp|random|sequence
- This has all of the same notes as the first method but also adds in a sequence counter which is used to solve the problem where multiple IDs are generated at the same millisecond
- The value of this sequence generally increases monotonically but the start is chosen at random.
- In almost every implementation the sequence counter is a set of bits at the end of the ID.
- This can be as small as 8 bits or as large as 24. The UUIDv1 uses spec uses 16 bits for clock sequence high and low which seems adequate for the collision avoidance purpose.
- Question: Is the best position for sorting with this at the end of the UUID (timestamp|random|sequence) or after timestamp (timestamp|sequence|random). Benchmark testing may be needed.

A third option does exist for distributed nodes that usually removes random and replaces it with a machineID
3. timestamp|machineID|sequence or timestamp|sequence|machineID
- This machineID is usually a variable length from 8 to 16 bits and is locally unique to a node involved in UUID creation.
- This helps for collision avoidance due to a machine's given context being captured within the UUID ensuring locally generated IDs have a very low chance of colliding with another distributed nodes IDs.
- Note that this method removes random as they are also smaller than 128 bits and need to make the most of the available bits.
- Again, is the best position for the sequence at the end of the UUID or after the timestamp?
---
**Q:** How should this new timestamp 128-bit UUID be identified?
- The Variant bits should be set to 10x (hex 8, 9, A, B)
- The Version should be set to 0110 for Version 6
- If we decide to implement more than one time-sortable UUID such as a 48 and 64 bit timestamp as separate specs the next available version should be used. (7 as 0111, 8 as 1000 and so on) 
---

### Text Encoding
**Q:** Should we have alternate text formats or just the existing hex with dashes?
- The existing hex format with the dashes is great for immediate readability by humans but terrible for computers.
- They should almost never be stored as this format in database applications and instead should be stored as binary data and as such the 128 characters they represent.
- It may be with an implementation note in the RFC on this very point.
- Based on the data it seems like the most common are base64, base64 safe-alphabet and base32 representations.
- Perhaps there is an easy way to mutate the 128 bits into any format desired?
- Possible solution: A descriptor prefix on a the mutated value when these are in use. Example using the draft, b32a:base32UUID to indicate how to pack/unpack these bits.
- Another Possible Solution: The last few bits can be used for identification for packing/unpacking purposes, say 0001 is b32a, 0010 is b32b, and so on
- Ideally the focus is on the standard 128 bit format with the dashes and then can circle to how that may be represented as an alternative formats later.

**Q:** Alternative, could we define a UUID prefix descriptor for edge-case scenarios where the current UUID format is not acceptable?
- Some APIs do not play well with dashes, rather than mutate to a new format or different encoding; could a safe 'UUIDvX' prefix, where X is the current version, be used on the current UUID and dashes omitted?
- This would allow for backwards compatibility with existing UUIDs while not over complicating the UUID too much.
- This would also ensure pre-HTML5 IDs can utilize UUIDs as HTML4.x and lower have strict rules about an ID not starting with a number. Here they would start with a 'U'
- https://www.w3.org/TR/html401/types.html#type-name

Example:

 b3ebb6c8-19a3-11eb-adc1-0242ac120002
 
 UUIDv1b3ebb6c819a311ebadc10242ac120002
 
 --
 
 824c8abb-2774-4ee0-887c-3501501c1be1
 
 UUIDv4824c8abb27744ee0887c3501501c1be1

---
### Local/Global Uniqueness
**Q:** Is this a solution only for globally unique IDs, or does it include locally unique IDs as well (unique within one system, not the whole planet/solar system/galaxy/universe)
- For 128 bit UUID the spec should still aim to be globally unique.
- If a 64bit variant alternative is introduced this should be explicitly called out that it is likely unique within a given application context.
- That being said, one could explore the act of an "as required" mutations for a 64 bit variant when a situation arises where UUID is required to be transmitted outside of the application context.

Potential Mutation Example
1. Compute a hash on the 64 bit variant's text representation
2. Truncate the resulting hash to 64 bits by removing the most least significant bits (Truncation process in RFC 2104, Section 5)
3. Post-fix the trimmed hash at the end of the 64bit-UUID
4. Mutate the variant bits to '111' (hex E or F) as per the leftover Variant in RFC 4122, Section 4.1.1. this is "Reserved for future definition." and is perfect for a new 64bit variant UUID.
5. The Version can be mutated to 0001 as the first version of the 64to128 conversion Variant

Other Thoughts
- Problem: This is a one-way conversion as bits have been mutated and there is not a good way to undo the conversion.
- Alternatively, the prefix solution described could be used to identify that the 64 bit value is actually UUID 64.
- Thought, this could also be used as a method to create a larger than 128 bit UUID as well using the same steps but on a 128bit UUID in step 1 and then prefixing on the current UUID behind a dash.
- Input: b3ebb6c8-19a3-11eb-adc1-0242ac120002
- SHA256: 99a65514eca05bfa1ccaa95bd2510d889f1bfea20ffc2aa57a739e1bbce060c7
- Combined: b3ebb6c8-19a3-11eb-adc1-0242ac120002-99a65514eca05bfa1ccaa95bd2510d889f1bfea20ffc2aa57a739e1bbce060c7
- There may be no use case for this but if a larger more unique UUID is required this could be a potential method for creating one. The hash method could also be variable in length. sha1, md5, sha256, etc.