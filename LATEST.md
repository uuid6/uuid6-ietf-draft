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
- Clarify global vs local uniqueness.  This gets tricky because this idea of guaranteeing a universally fully unique identifier is factual impososible - the spec just tries to do the best in can.  But then when we to get implementations, you might have a database system that doesn't care about globally universally unique identifiers that are unique everywhere in the universe - they just need a database key that isn't in use.  The spec should clearly indicate what is allow/not allowed for valid implementations.  Part of it might be that if an implementation provides a more narrow definition "unique" for it's specific situation, then it should state this clearly but otherwise it's okay as long as the value can reasonably be expected to live in the intended environment (i.e. the database example given is fine as long as it's clear the implementation is just for generating datebase pks)
- UUIDs should be as opaque as possible, but we necessarily have exceptions.
  - Reading the version is an example.
  - I think we should make an exception for the timestamp, because it is useful for practical purposes to know when a UUID was generated.
  - If you want to obfuscate the timestamp because you don't want this property, just use UUIDv8 where you can do whatever you want.
- For v7 and v8 merge variant and version fields into one single 8-bit field. (new variant value = 0b111 means version field immediately follows in the next 5 bits). Reason: simplicity.  I don't see any practical reason to keep the version field where it is other than fear of breaking backward compatability with BROKEN implementations.  Correct RFC4122 implementations should be checking the var field and if it doesn't match an expecting value not interpreting anything else.  (See RFC4122 section 4.1.1)
- Backward compatability: Specifically what this means is:
  - Will new UUIDs break existing implementations and if so how often/bad.
  - Can we add new things in a way that breaks as little as possible (obvoiusly UUID implementations will be
- For v7 and v8 we can add the capabiliy of variable length.  For existing implementations that expect 128-bits, it should be valid to zero-pad shorter values, and longer values obviously will break.  128-bits should still probably be the recommended default unless there is a VERY compelling reason to do otherwise.
- Introduce Crockford base32 as a standard text encoding (variable length).  Existing hex encoding stays as-is, although a definition of what to do with variable length UUIDs encoded as hex is needed.)

## UUIDv6

- Is just the re-arranged byte sequence as originally described at http://gh.peabody.io/uuidv6/
- The point is to be easy to adapt from an existing UUIDv1 implementation.
- Anything more complicated or different goes elsewhere.

## UUIDv7

- Goal: Allow nanosecond-precise timestamps with a familiar unix epoch and in a simpler format than UUIDv6 and earlier UUIDs.
- Uses new var_ver field 

## UUIDv8

- Goal: Make a valid version where people can do whatever they want and it's still a valid implementation.  It basically means "if you see a UUID wiht this version, there is no guarantee ".

## FAQ

* Do we need three new formats?  Yes. They each have different tradeoffs.
* 
