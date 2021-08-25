# Outline

Here's what I'm thinking for another outline.  Instead of going down more or less in sequence of version and field by field, instead give right at the top (after a brief sumary) the core pieces of info required to produce a correct implementation, including the bit layout.  Then cover the text formats.  And the following that, go through each of the various application-specific concerns - keeping it succinct and so each section you can easily read the first line or two and decide if it matters for your application and skip if not.  Don't repeat RFC4122 stuff unless it's vital to cover in this doc.

## Background

(more or less what is in the current -02 draft, edit as needed)

## Motivation

- The need for unique IDs in applications is great
- Time-ordered values are useful as database keys and important for performance (more in section ...)
- It really shouldn't be over-specified.  Application requirements are vastly different.  Generated values should generally just be opaque.
- But, it is helpful to have a document which explains the tradeoffs and gives guidance.  (Picking a good unique value generation scheme is non-trivial.)
- And provides for backward compatability with RFC4122.

## Summary of Changes

- Three new UUIDs with different tradeoffs:
  - v6 is easy to adapt from v1
  - v7 is easy to implement for new time-ordered UUID implementations (particularly which don't care about implementing prior UUID versions)
  - v8 allows you to do whatever you want
- Crockford base32 text format
- Variable length is allowed (optional, 128-bits remains the default)
- Requirements for storing vs generating UUIDs are separated out for clarity and simplicity (and because they are different)
- Other related implementation concerns are each covered separately.

## Storing UUIDs

- UUIDs SHOULD be treated as opaque values unless there is a good reason to do otherwise (i.e. never examine the bits in a UUID unless you really have no other choice)
- As such, any storage mechanism capable of storing a series of bytes (minimum 9, maximum 64) is a valid storage mechanism for UUIDs.  If an implementation chooses to only support 128-bit UUIDs, then anything that can store 16 bytes is valid.  THERE IS NO REQUIREMENT ABOUT IMPLEMENTATIONS STORING UUIDS TO HAVE AN UNDERSTANDING OF THEIR CONTENTS.
- A UUID with all zero bytes is never valid (per this doc and the allowed values per RFC4122), so instead of checking "is this UUID valid", compare against all zeros.  THE LESS UUID INSPECTION IS DONE, THE BETTER (resulting code is simpler, more obvious, less likely to be broken and probably faster).
- If you really must inspect a UUID, the procedure for determining the version is (see RFC4122 for normative reference on UUIDs version <= 5):
  - Read the two most significant bits from the octect 8 (zero-based) and if it's 0x10 the check the most significant 4 bits of octect 6 for the version number (per RFC4122)
  - Otherwise, per this new spec, check octect 8 for the value 0xE7 meaning version 7 or 0xE8 meaning version 8 (we can mention that we're using variant = 0b111 here and refer to RFC4122 section 4.1.1 for the background). (note that this is the only step needed if you only care about versions 7 or 8)
  - Once you know the version refer to the spec of that particular version about the contents.

## Length

- Default is stil 128-bits, 16 bytes.  Implementations SHOULD default to this.
- Implementations MAY choose to allow generation UUIDs of anywhere from 9 to 64 bytes.  (decide if we want to elaborate on the various possible motivations for this here, e.g. shorter for better human use, longer for lower collision probability)
- Implementations that store UUIDs of variable length SHOULD support any length from 9 to 64 bytes (decide if this is the right range - <9 bytes and we lose the variant/version, and 64 is arbitrary but we should have some upper limit in case it has an impact on physical storage requirements)
- Variable length is optional and implementations MAY opt-out and only support 128-bits.
- Recommendation is for implementations to start defining UUID as an array of bytes (length + pointer or whatever language mechanism) instead of a fixed set of 16 bytes.
- I think we need a table in here that gives bit/byte lengths and collision probability.  So someone can look down the numbers and pick the one that is right for their app.

## Text Format

- Existing hex format is still valid.  Shorter or longer values just omit or add hex characters (give some examples at different lengths)
- Crockford base32 is allowed also (briefly summarize reasons - shorter, more compact if UUIDs are stored as text, crockford version's benefits vs).
- Parsers can use the presence of '-' to determine if it's hex or crockford base32 format.
- Compatible parsers MUST support crockford base32 with or without padding, and allow a checksum part (verifyin the checksum is optional)
- Base32 text encoder SHOULD output crockford base32 values without padding or checksum by default.  The padding or checksum features MAY be used if warranted for a specific application.

## UUIDv6

- Recommended for implementations that already implement UUIDv1 and want a low-impact way to implement time-ordered values.
- Refer to RFC4122 for v1, timestamp bits reordered (describe in more detail)
- Recommend "Node" field be filled with random bytes instead of MAC address for the same reasons stated in Shared Knoweldge section, but this is optional.
- Be sure to remove any new language related to the sequence counter, we want to keep that as it was https://github.com/uuid6/uuid6-ietf-draft/issues/38

(from the existing draft, might tweak it a bit)
```
        0                   1                   2                   3
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                           time_high                           |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |           time_mid            |      time_low_and_version     |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |clk_seq_hi_res |  clk_seq_low  |         node (0-1)            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                         node (2-5)                            |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```
FIXME: I think this would be easier to read if we split out the bit fields, e.g. "var" as it's own few bits, "ver" as 4 bits, etc.

- Anything else is the same as RFC4122.  (Don't bother describing the old fields in detail - the old RFC does that)

## UUIDv7

- Recommended for NEW implementations that are not concerned with existing UUIDv1 code.
- Bit layout:

```
   0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    unix_nano_timestamp_u64                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    unix_nano_timestamp_u64                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |     0xE7      |         seq_rand_etc                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         seq_rand_etc                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- `unix_nano_timestamp_u64` is big endian uint64 of nanoseconds since the Unix epoch (same as "unix timestamp" [link?] but with nano-second granularity). Don't have/want nanosecond-precision - see below
- Mention max datetime
- Implementations SHOULD use the current timestamp to provide values that are time-ordered and continually increasing.
- It's okay to "fuzz" timestamp values here for various reasons (security, clock inaccuracy, etc.), there is no absolute guarantee about how close the clock value needs to be to actual time - that's implementation-specific.

## UUIDv8

- Recommended when you want an implementation that does something different than laid out here.
- It means that other than octet 8 being 0xE8, there are no guarantees about the contents.
- Recommend using in place of UUIDv4 to achieve similar result in a simpler way.

```
   0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |     0xE8      |                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

## Sorting

- Implementations that require sorting (e.g. database indexes) SHOULD sort as opaque raw bytes (UUIDv6 and 7 are designed for this), without examining the contents at all.
- Implementations MAY implement more complex sorting rules for UUID versions < 6 (or strictly speaking they MAY do this for any version, but shouldn't need to).
- One of the big benefits of time ordering is "index locality" - new values are near each other in the index and can much more easily be clustered together for better performance.  Real-world differences vs random data can be quite large.

## Timestamp Granularity

- Implementations SHOULD use the current timestamp to provide values that are time-ordered and continually increasing.
- It's okay to "fuzz" timestamp values here for various reasons (security, clock inaccuracy, etc.), there is no absolute guarantee about how close the clock value needs to be to actual time - that's implementation-specific.
- Give some examples

## Monotonicity

- This is really just for v7:
- defined simply as "each value returned is greater than the last"
- Implementations SHOULD return monotonic values where it is feasible. E.g. in a single library, effort should be made to return successive values that count up.
- You can do this with an emedded sequence counter, by waiting for the next clock tick, or by checking that in case of next value has same timestamp see that the random bytes part is greater than prior value and generate new random bytes and check again.  Describe trade offs of these approaches, but it's implemenatation-specific - no mandedated sequence counter.
- Provide a clear suggestion of a recommended way (e.g. something as simple as: "if next UUID is <= last, generate again until timestamp or random bytes provide a value that is higher" - maybe consider a max limit for cases where the system clock rolls back).
- The monotonicity properties of an particular UUID generator SHOULD stated in it's documentation.

## Global vs Local Uniqueness

- Global uniquess is impossible to guarantee without shared knowledge.  I.e. two systems cannot come up with two numbers completely independently without some possibility of collission UNLESS they agree on some mechanism to ensure uniqueness ahead of time (e.g. your numbers always end with ... and mine always end with... or whatever)
- RFC4122 tried to use MAC address as shared knowledge - it sort of worked but had problems (security issues with exposing MAC addresses, guarantee of actual global uniqueness is questionable)
- So instead just decide if you need 100% guarantee of uniqueness.  If so, implement shared knowledge approach (see section below)
- If not, reduce probability of uniqueness to acceptable level for your application.
- It is okay if implementations only provide "local" uniqueness, e.g. unique within one database instance or cluster of machines - as long as the implementation A) states that and B) the value can be reasonably expected to remain only in that system.  I.e. don't use this unique-within-this-database UUID across databases and the docs need to state this.
- Implementations which do not know the uniqueness requirements of the final application and cannot implement shared knowledge should just be made as unique as possible and state that.

## Collision Resistance

- In the absence of shared knowledge, collisions cannot be fully prevented, only the probability reduced.
- Give the math and some examples.
- Variable length allows you to further reduce collision probably beyond 128-bits if needed.  If we give the math, then people can make their own decisions based on their app.
- v7 is time ordered for better index locality/database performance, v8 has lower collision resistance, pick your poison.

## Unguessability

- Some applications acquire security benefits from generating values that cannot be predicted (this is related to collision resistance, but not the same thing).
- Using a cryptographically secure pseudo-random number generator (CSPRNG) provides values that are both difficult to predict ("unguessable") and have a low likelyhood of collision ("unique").  And so this is the recommended approach.

## Shared Knowledge

- Means a pre-arranged agreement about how different systems or pieces of code will each produce different values.
- Examples: A section of the "random" part gets devoted to: database node number, MAC or IP address, manually entered ID, etc.  Or it could be something like the UUID generator in a database implementation simply checks to ensure the UUID is not in use before generating.  Regardless, the concept is that an implementation MAY just make up a rule that ensures uniqueness, at the cost of some guessability.
- Using a shared knowledge pattern with the same length of UUID increases guessability (the more bits that fit a known value or pattern, the easier a value is to guess).
- Shared knowledge solutions are okay and MAY be done as long as this is stated in the UUID implementation docs.
- Talk a bit about the cost of collision, e.g. https://github.com/uuid6/uuid6-ietf-draft/issues/36#issuecomment-903295070
- Mention that the reason this spec does not endorse any specific global registry is because if something goes wrong with it (like the fact that MAC addresses used to be more or less unique but with cloud computing and software network interfaces being commonplace that assumption changed) - in this case random data results in lower collision probability.  So basically we're saying: Global registries, aside from being inconvenient, can still have problems and thus the collision probability jumps way up above the random data approach - so let's not even bother.  If an application wants a "perfect, guaranteed unique" solution, it provide it within it's own application via shared knowledge.

## Documentation

- We say in various places that certain things should be stated in the implementation docs.  So we should probably have a list here... If you make a UUID implementation, provide a clear statement in the documentation about each of these points:
  - Timestamp granularity (v7 only)
  - Monotoncity (v7 only do the values always count up)
  - Uniqueness scope (are these values supposed to be globally unique or unique with a specific context, if so which context)
  - Shared knowledge system (if any)
  - Collision resistance math (use the table in this spec if it helps)
  - Unguessability (how strong is the random number generator for how many bits)

## Minimal Practical Implementations (Generation)

(We don't care about storing because it should be opaque, and parsing is trivial and standardized)

### UUIDv6

- Generate a UUIDv1
- Reorder the time bits
- Change the version

### UUIDv7

```
bytes[0:8] = big_endian(unix_timestamp_nano())
bytes[9] = 0xE7
bytes[10:end] = crypto_rand()
// TODO: should we put examples of monotonicity logic (or other things above) in here?
```

### UUIDv8

```
bytes[0:end] = crypto_rand()
bytes[9] = 0xE8
```

## Implementation Scenarios

Give examples of the bit layout and/or algorithm used to address various situations that have come up.  Briefly cover the motivation for each case as the first thing.
- Only millisecond clock precision available (JS) - algo to fill in the rest with random (motivation: no choice, environment limitation OR precise clock readings present security risk)
- Monotonic guarantee: Retry method  - algorithm for generator to retry when it gets a value with the same clock tick (motivation: ease of implementation, (describe why monontoicity is desirable, i forget but there are cases))
- Monotonic guarantee: Sequence counter method - assign 12 or maybe 16 bits of the random area as a sequence counter.  (motivation: you have a case where monotonicity is vital (e.g. an algorithm that depends on looking at a prior value and comparing greater than to determine if something is new) and you want to be able to generate up to X values per clock tick without waiting)
- 160-bit values - example of bit layout with more random bytes at the end, generation is trivial (motivation: reduce collision probability)
- Shared-knowledge approach to guarantee uniquness - embedded database node number.
TODO: more examples?
