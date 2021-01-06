# Registration Manager

In the latest versions of Minecraft, registration has splintered into what seems like a thousand
different events, meaning the code for setting up a single block may be spread across almost a
dozen separate methods in half a dozen different classes in three different packages. This leads
to code that's…

- Difficult to read (you'll have to hop between multiple functions in multiple files in order to 
get a sense of a block or item's configuration),
- difficult to set up correctly (hope you remembered where all you need to put configurations!),
- and difficult to maintain (hope you remembered to update all the occurrences when you made that 
change!).

The Registration Manager's job is to handle all this for you. 

## Architecture

The Registration Manager is designed so you can define an entire "thing" in one place. Everything
about your block can be put together, meaning you only have to look in one place to get a full view
of it.

### Specs

Most of the Registration Manager uses what I call "specs". A spec is the pre-definition of
something which the Registration Manager then uses at the appropriate times to set up your
object. For example, the block spec contains block and item properties, a block instance callback, 
and data generation information, each of which is used in various locations.

The registration manager currently supports:

- Blocks
- Items
- Entities
- Tile Entities
- Capabilities
- GUI containers

In the future I hope to add support for:
- Fluids
- Sounds? (this is very much a maybe)

### Lazies

Most of the `add` methods in the registration manager will take in a spec and return out a "lazy",
which is a reference to a lazily-computed value. In this way, a block instance isn't created until 
it's needed. These lazy objects all have a `from` method, which will delegate the value to another
lazy. Doing it this way means you can still have a `final` field and just call
`theLazyField.from( registrationManager.add(...) );` to set its value.
