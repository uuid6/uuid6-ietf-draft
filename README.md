# Updates

- This work has been officially adopted by the IETF!
- Work on the base document has been reset to Draft 00 and moved to https://github.com/ietf-wg-uuidrev/rfc4122bis
- Full Details and further updates will be posted in [Post-IETF 114 and the future of this Draft](https://github.com/uuid6/uuid6-ietf-draft/issues/122)

```
Name:		draft-ietf-uuidrev-rfc4122bis
Revision:	09
Title:		Universally Unique IDentifiers (UUID)
Document date:	2023-08-04
Group:		uuidrev
Pages:		51
URL:            https://www.ietf.org/archive/id/draft-ietf-uuidrev-rfc4122bis-09.txt
Status:         https://datatracker.ietf.org/doc/draft-ietf-uuidrev-rfc4122bis/
Html:           https://www.ietf.org/archive/id/draft-ietf-uuidrev-rfc4122bis-09.html
Htmlized:       https://datatracker.ietf.org/doc/html/draft-ietf-uuidrev-rfc4122bis
Diff:           https://author-tools.ietf.org/iddiff?url2=draft-ietf-uuidrev-rfc4122bis-09
```

---

# New UUID Formats
This is the GitHub repo for the IETF draft surrounding the topic of new UUID formats.
Various discussion will need to occur to arrive at a standard and this repo will be used to collect and organize that information.

## High Level Overview
1. **UUID version 6**: A re-ordering of UUID version 1 so it is sortable as an opaque sequence of bytes. Easy to implement given an existing UUIDv1 implementation.

    `time_high|time_mid|time_low_and_version|clk_seq_hi_res|clk_seq_low|node`
2. **UUID version 7**: An entirely new time-based UUID bit layout sourced from the widely implemented and well known Unix Epoch timestamp source.

    `unix_ts_ms|ver|rand_a|var|rand_b`

3. **UUID version 8**: A free-form UUID format which has no explicit requirements except maintaining backward compatibility.

    `custom_a|ver|custom_b|var|custom_c`

5. **Max UUID**: A specialized UUID which is the inverse of the Nil UUID from RFC4122.

    `FFFFFFFF-FFFF-FFFF-FFFF-FFFFFFFFFFFF`

---

# RFC Scope
In order to keep things on track the following topics have been decided as in-scope or out of scope for this particular RFC.
For more information on any of these items refer to the XML, TXT, HTML draft, research and the issue tracker for a particular discussion (follow hyperlinks below.)

### In Scope Topics - UUID Generation
- [Timestamp Granularity](https://github.com/uuid6/uuid6-ietf-draft/issues/23)
   - Sub-Topics: Timestamp epoch source, format, length, accuracy and bit layout
- [Monotonicity and Counters for same Timestamp-tick collision avoidance during batch UUID creation](https://github.com/uuid6/uuid6-ietf-draft/issues/60)
   - Sub-Topics: Counter position, length, rollover handling and seeding.
- [Pseudo-random formatting, length and generation methods](https://github.com/uuid6/uuid6-ietf-draft/issues/55)
- Distributed UUID Generation best practices
  - Sub-Topics: [Shared Knowledge Schemes and embedded nondescript node identifiers](https://github.com/uuid6/uuid6-ietf-draft/issues/36) 
- [Max UUID Usage](https://github.com/uuid6/uuid6-ietf-draft/issues/62)

### In Scope Topics - UUID Best Practices as it relates to the previous topics
- Global and Local Uniqueness (collision resistance mechanisms)
- Unguessability
- Sorting/Ordering techniques
- Storage and Opacity best practices
- Big Endian vs Little Endian bit layout
- Any and all UUID security concerns!
  - Sub-Topics: [MAC address usage in next-generation UUIDs](https://github.com/uuid6/uuid6-ietf-draft/issues/13)


---

### Out of Scope Topics (Rolled into a new Draft that can be found here: [New UUID Encoding Techniques](https://github.com/uuid6/new-uuid-encoding-techniques-ietf-draft))
- [URN Modifications](https://github.com/uuid6/new-uuid-encoding-techniques-ietf-draft/issues/4)
- [Alternative text encoding techniques (Crockfords Base32, Base64, etc)](https://github.com/uuid6/new-uuid-encoding-techniques-ietf-draft/issues/3)
- [Variable length UUIDs | UUID Long](https://github.com/uuid6/new-uuid-encoding-techniques-ietf-draft/issues/2)

### Out of Scope Topics
- [Variant Bit E Usage](https://github.com/uuid6/uuid6-ietf-draft/issues/26)

---

### Out of Scope Topics (as as the result of discussion threads)
- [Variable length subsecond precision encoding](https://github.com/uuid6/uuid6-ietf-draft/issues/24)

### Out of Scope Topics (for backwards compatibility)
- Changing the default 8-4-4-4-12 UUID text layout
- Changing anything about RFC4122's UUID versions 1 through 5
- [Changing too much about UUIDv6 that would otherwise inhibit porting v1 to v6](https://github.com/uuid6/uuid6-ietf-draft/issues/52)

---

# Contributing
- The XML draft in the root folder is the most recent working draft for re-submission to the IETF.
  - An HTML and Textual (.txt) RFC representation will be provided in the root folder to ease reader input and discussion.
  - [Older drafts](https://github.com/uuid6/uuid6-ietf-draft/tree/master/old%20drafts) are available for view here or on the [IETF Datatracker](https://datatracker.ietf.org/doc/draft-peabody-dispatch-new-uuid-format/).
- The RFC Draft utilize an XML formatted document that follows [RFC7742 markup](https://xml2rfc.tools.ietf.org/rfc7749.html). All XML changes MUST follow this format and pass conversion to `.txt` and `.html` via https://xml2rfc.tools.ietf.org/
- Utilize the issue tracker to discuss topics, solutions, problems, typos and anything else.
  - Where possible contribute to an existing [Discussion Thread](https://github.com/uuid6/uuid6-ietf-draft/issues?q=is%3Aissue+is%3Aopen+label%3ADiscussion) vs creating a new thread.
  - Reviewing is the pre-Draft 01 [Research efforts](https://github.com/uuid6/uuid6-ietf-draft/tree/master/research) is encouraged before diving into discussion threads.
  - New threads that propose alternative text SHOULD utilize `Proposed Draft Change` GitHub issue template to ensure proper information is captured for the draft authors.
  - Be civil!
- Pull requests will be accepted  *as long as the text is concise, clear and objective.* 
  - PRs will not be accepted for changes to the decision made for the draft without full discussion. 
  - PRs MUST include the updated `.xml` and xml2rfc generated `.txt` and `.html` documents.
  - Draft versions are frozen until submission to the IETF; at which point new work constitutes a new draft version.

---

# Prototyping
Remember first and foremost that this specification is still a draft. Breaking changes are to be expected. Prototypes SHOULD only be implemented to verify or discredit topics of the draft text.
- [Prototype Implementations](https://github.com/uuid6/prototypes) are available via this repro.
