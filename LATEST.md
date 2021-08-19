# Latest

## Overall

- Goals:
  - Introduce new UUIDs which make good database or other keys.
  - Sorts by time (and ideally monotonically when generated from one environment) as a set of raw bytes (this is part of being able to treat UUIDs as opaque values in as many situations as possible and reducing implementation complexity).
  - Reduce complexity in the spec and implementations
  - Remove MUSTs where they are not needed - there is a lot of implementation detail in RFC4122 which does not need to be forced on every implementation and then leaving people wondering if their implementation is correct or not
  - Make UUIDs as opaque as possible (but 100% opaque is not practical or desirable, more below)
- Focus on defining the minimum requirments for a compatible implementation.
  - I.e. "any implementation which produces values with the following properties is correct"
  - And the include the implementation recommendations as clearly separate.
  - Things like sequence field for monotonicity or random number source requirements are implementation-specific.
    People SHOULD use a CSPRNG to generate random values, but it doesn't necessarily mean the implementation is broken if it does not.
- Clarify global vs local uniqueness.  This gets tricky because this idea of guaranteeing a universally fully unique identifier is factually impossible - the spec just tries to do the best it can.  But then when we to get implementations, you might have a database system that doesn't care about globally universally unique identifiers that are unique everywhere in the universe - they just need a database key that isn't in use.  The spec should clearly indicate what is allow/not allowed for valid implementations.  Part of it might be that if an implementation provides a more narrow definition "unique" for it's specific situation, then it should state this clearly but otherwise it's okay as long as the value can reasonably be expected to live in the intended environment (i.e. the database example given is fine as long as it's clear the implementation is just for generating datebase pks)
- UUIDs should be as opaque as possible, but we necessarily have exceptions.
  - Reading the version is an example.
  - I think we should make an exception for the timestamp, because it is useful for practical purposes to know when a UUID was generated.
  - If you want to obfuscate the timestamp because you don't want this property, just use UUIDv8 where you can do whatever you want.
  - Things like the sequence counter and random bytes should be considered opaque, I can't think of any reason an external application would read the sequence counter from a UUID generated somewhere else and do anything useful with it.  It's only used internally by the implementation, and even then implementations should be compared against a strict criteria of what is needed to produce a valid UUID, not fiddly rules about sequence counters that may or may not be useful/applicable to every scenario.
- For v7 and v8 merge variant and version fields into one single 8-bit field. (new variant value = 0b111 means version field immediately follows in the next 5 bits). Reason: simplicity.  I don't see any practical reason to keep the version field where it is other than fear of breaking backward compatability with BROKEN implementations.  Correct RFC4122 implementations should be checking the var field and if it doesn't match an expecting value not interpreting anything else.  (See RFC4122 section 4.1.1)
- Mention backward compatability: Specifically what this means is:
  - Will new UUIDs break existing implementations and if so how often/bad.  (In general I think the changes described here are acceptable because they will not break correct implementations - I realize the real world is more complicated though.)
  - Can we add new things in a way that breaks as little as possible (like allowing variable length but still having 128-bits as the default).
- For v7 and v8 we can add the capabiliy of variable length.  For existing implementations that expect 128-bits, it should be valid to zero-pad shorter values, and longer values obviously will break.  128-bits should still probably be the recommended default unless there is a VERY compelling reason to do otherwise.  Reason: Additional (or less) random bytes allowed, based on application's requirements for uniqueness.
  - Minimum is 9 bytes (up to the variant+version byte)
  - Should we specify a maximum, e.g 64 or 128 or 256 bytes?  It would almost certainly be useful for implementations to know what the upper limit is.
  - Implementations are not forced to support non-128-bit values.  It is perfectly valid for an implementation to say that it implements the spec but choses not to implement variable length for whatever reason (legacy code, performance/size considerations, etc.)
- Introduce Crockford base32 as a standard text encoding (variable length).  Existing hex encoding stays as-is, although a definition of what to do with variable length UUIDs encoded as hex is needed.). This has the added benefit of being able to use the "-" character to positively identify which text format is in used (hex always has them, crockford base32 never does)

## UUIDv6

- Is just the re-arranged byte sequence as originally described at http://gh.peabody.io/uuidv6/
- But recommendation is to not use MAC address but CSPRNG bytes instead.
- The point is to be easy to adapt from an existing UUIDv1 implementation.
- Anything more complicated or different goes elsewhere.

## UUIDv7

New layout:
```
   0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    unix_nano_timestamp_u64                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    unix_nano_timestamp_u64                    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | var |  ver    |         seq_rand_etc                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                         seq_rand_etc                          |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- Goal: Allow nanosecond-precise timestamps with a familiar unix epoch and in a simpler format than UUIDv6 and earlier UUIDs.
- Timestamp is nanoseconds since the Unix epoch as an unsigned int64 - max date is 2554-07-21T23:34:33.999999999Z
- Uses new combined var_ver byte for simplicity.
- Doesn't have strict requirements for sequence counter and "node", etc. but will provide implementation recommendations.
- Implementations like JS that don't have (or others that don't want) more than ms (or second or whatever lower) time precision can just fill in the least significant bytes with random data.  If this breaks monotonicity (some numbers generated will be out of sequence for the same millisecond unless you guarantee the random data you insert always counts up), then that's up to the implementation.  Monotonicity is a SHOULD, not a MUST, with implementation recommendations.
- All the way down to the var-ver byte can be truncated or it can be made longer (9 bytes would be the minimum otherwise the var-ver field disappears).  Since the exact contents of everything after the 9th byte are implementation-specific, I think this would work.

## UUIDv8

Layout:
```
   0                   1                   2                   3
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   | var |  ver    |                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                                                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
```

- Goal: Make a valid version where people can do whatever they want and it's still a valid implementation.  It basically means "if you see a UUID wiht this version, there is no guarantee of what's in it".

## FAQ

* Do we need three new formats?  Yes. They each have different tradeoffs.  6 is easy to implement given a v1 implementaitons.  7 is more time precision but simpler and provides variable length.  8 lets you do whatever you want.
* Do we need to introduce a new variant?  Need... hard to say, but the point is to make the spec simpler by tucking the variant and version fields into a single byte.  Compare the UUIDv7 layout above to RFC4122 - is it worth it?  I think so, but it's also subjective.  Also, by doing this we introduce another bit which is technically unused (the most significant bit of the version field will always be 0 for now) - so we have some flexibility for the future if later it is decided this specification messed everything up and should have never been born.
* Why do we need/want to introduce variable length? The basic problem is there is no one-size-fits-all level of uniqueness and collision resistance.  There will always be some applications that want more bytes to increase the level of uniqueness/unguessability/collision-resistance.  So if we introduce the concept here that UUIDs can be longer and implementations adapt to that idea - while still being compatible with existing 128-bit implementations where possible (i.e. I can still make a 128-bit ID with v6, 7 or 8 and chuck it in Cassandra and it's UUID code will at least not break - I've actually tested this on Cassandra for v6 but not for this new v7 or v8 proposal). 
