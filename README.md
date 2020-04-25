# UUID Version 6 IETF Draft

This is the IETF draft for a version 6 UUID.  Various discussion will need to occur to arrive at a standard and this repo will be used to collect and organize that information.

The following is a list of relevant topics related to this draft.

Pull requests will be accepted for changes to Concerns and Possible Solutions or to introduce a new Topic if it is missing,  *as long as the text is concise, clear and objective.* PRs will not be accepted for changes to the decision made for the draft without full discussion.  Please make an issue to discuss such things.

- Topic: **Length**.  
  - Concerns:
    - A lot of existing code expects a UUID to be 128 bits.  Even when other aspects of the format are changed, this can provide a good deal of backward compatibility.
    - Some applications may need more than just 16 bytes to ensure uniqueness, depending on how many IDs they have to generate and under what circumstands.
    - Some applications may benefit from having shorter IDs when global uniqueness is not a requirement (e.g. local uniqueness will suffice) and easier human use of a shorter value is a priority.
  - Possible Solutions:
    - Keep the same length (16 bytes)
    - Change the size to something longer or shorter
    - Introduce a system for variable-length UUIDs
  - Current Decision Per Draft:
    - *Keep the same length.*  Introducing different length(s) would break backward compatibility and is not generally useful enough to be worth it.  If you need something other than 128 bits, it's not a UUID.

- Topic: **Text Encoding**
  - TODO

- Topic: **Timestamp**
  - TODO

- Topic: **Local/Global Uniqueness**
  - TODO
