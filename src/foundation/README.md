# Foundation

Creating mods has become progressively more complicated and convoluted after every update, which
means you have to spend more and more of your time wrestling with a dozen different registry
events, blockstate generators, tag generation, capability and tile entity serialization, and even
more that you likely missed or forgot about.

This is unacceptable. Every hour you spend wrestling with an event is an hour you aren't spending
*creating*. That's where Foundation comes in. Its goal is to maximize the amount of time you spend 
creating by handling the boilerplate and bullshit for you.

…and oh boy, the bullshit. Minecraft's codebase is a never-ending source of the stuff, which can
often be overwhelming or disheartening for someone who just wants to *create*. For example…

Did you know that at bare minimum blocks have to be configured in at least eight separate
locations? Or that when generating tags you have to list all the blocks with the tag, as opposed
to adding tags to the block? Or that generating models properly requires wrapping the default
`ExistingFileHelper` so it doesn't crash when generating `BlockItem` models? Or that generating
loot tables involves a list of pairs of biconsumer consumer suppliers and loot parameter sets,
and also requires overriding the validation function so the generator doesn't just crash?

***This*** is why LibrarianLib, and specifically Foundation exists. I deal with all the bullshit
so you don't have to.
