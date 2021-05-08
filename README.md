# Update

A new draft was submitted 26 April 2021: https://tools.ietf.org/html/draft-peabody-dispatch-new-uuid-format-01 Feedback welcome :)

# New UUID Formats
This is the github repo for the IETF draft surrounding the topic of new time-based UUID formats.
Various discussion will need to occur to arrive at a standard and this repo will be used to collect and organize that information.

Pull requests will be accepted for changes to Concerns and Possible Solutions or to introduce a new Topic if it is missing,  *as long as the text is concise, clear and objective.* PRs will not be accepted for changes to the decision made for the draft without full discussion. Please make an issue to discuss such things.

## Drafts
- The XML draft in the root folder is the most recent working draft for resubmission to the IETF.
- An HTML and Textual (.txt) RFC representation will be provided in the root folder to ease reader input and discussion.
- Older drafts can be viewed in the "old drafts" folder

## Other
- Research efforts can be found in the "research" within the root directory.
- Prototype Implementations for these drafts can be found in the prototypes section: https://github.com/uuid6/prototypes

## RFC Scope
In order to keep things on track the following topics have been decided as in-scope or out of scope for this particular RFC.
For more information on any of these items refer to the XML, TXT, HTML draft, research and the issue tracker.

### In Scope Topics
- Timestamp format, length, accuracy and bit layout
- Sequence counter position and length
- Pseudo-random Node formatting
- Big Endian vs Little Endian bit layout
- Sorting/Ordering techniques
- Multi-node and clustering UUID Genration best practices
- UUID Database Storage best practices
- Any and all UUID security concerns
- Same Timestamp-tick collision handling best practices

### Out of Scope Topics
- Total length (128 bits retained for backwards compatibility)
- UUID textual layout changes (anything other than 8-4-4-4-12 isn't a UUID)
- Alternative text encoding techniques (Crockfords Base32, Base64, etc) [Possibly a future RFC!]
- Local/Global Uniqueness (New UUIDs should provide global uniqueness the same as RFC 4122)
