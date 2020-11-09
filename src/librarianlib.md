# LibrarianLib

LibrarianLib is a suite of libraries designed to help you express your creativity without having
to sweat the details. You don’t need to keep track of all the events that go into registering a
block, just hand it to LibLib, and we’ll take care of it for you.

LibrarianLib is fundamentally a Java library. The library itself may be written in Kotlin, but
it's been built with the focus on being clean and easy to use from Java code. If you encounter
any [rough APIs](#rough-apis), please join [our Discord server][discord] and let me know. I do my
best to present a clean API, but some stuff inevitably slips through the cracks.

[main_issues]: https://github.com/TeamWizardry/LibrarianLib/issues

If you aren't familiar with Kotlin, don't worry. Kotlin is structurally almost identical to Java,
and I've written up a [quick guide](./reading_kotlin.md) to help get you up to speed on reading
Kotlin.

## Documentation completeness

LibrarianLib is rather large, and there's a lot of complex topics to document, so expect this
documentation to be very incomplete for quite a while. I'm going to write the documentation in a
tutorial style, with each module a self-contained unit that is expected to be read in order. 

I try to make the APIs as obvious as I can and prioritize documentation comments for the most
confusing stuff, but there's only so much documentation I can take before I go crazy, sometimes
it's hard to tell from the inside what's confusing, and sometimes I just forget.

If you find any of the documentation (whether that be this guide or the documentation comments)
or APIs especially confusing please contact me (@thecodewarrior) on [our Discord server][discord]
so I can help clarify for you and learn how to improve the docs.

[discord]: https://discord.gg/KPaZHvq

## Documentation Examples

This guide contains a lot of examples, so it'll be useful to go over a couple things about how
the code blocks work. A lot of examples will have irrelevant lines (e.g. imports) hidden away. If
they do, there will be a small eye icon in the top right that can be clicked to show them.
Clicking the "copy" icon beside it will copy them, even if hidden. Most of the complete examples
(i.e. not code snippets) can also be viewed in the [main repository](liblib_repo). They will be
located in the `com.teamwizardry.librarianlib.<modulename>.example` package under the
`modules/<modulename>/src/test/java` directory. Other example-related code (e.g. animated
demonstrations for Facade) will be located in the same package under the `.../src/test/java`
directory.

[liblib_repo]: https://github.com/TeamWizardry/LibrarianLib

## Rough APIs

Writing Kotlin code that's clean from the Java side, while easy most of the time, requires some
specific considerations. If you run into any rough APIs in LibrarianLib please tell me about them
[on Discord][discord] so I can resolve them.

Here are a few ways that the Kotlin–Java interface can go wrong:

- Needing to use `SomeClass.INSTANCE.*`  
  (this means I forgot to add `@JvmStatic`)
- Needing to use `SomeClass.Companion.INSTANCE.*`  
  (this means I forgot to add `@JvmStatic`)
- Needing to use a class called `SomeNameKt`  
  (this means I forgot to set the `@JvmName` of the file)
- Getters for "constant" values, like `SomeClass.getSOME_CONSTANT()`  
  (this means I forgot to add `@JvmField` or need to rename it to use `camelCase`)
- A public method that accepts a Kotlin `Function1`/`Function2`/etc.  
  (this means I forgot to use a Java functional interface instead of a Kotlin function type)
- *Both* a public method that accepts a `Consumer`/`Supplier`/etc. *and* one that accepts a Kotlin
  `Function1`/`Function2`/etc.  
  (there are some cases this is used, and it means I forgot to mark the Kotlin one as 
  `@JvmSynthetic`)
- A public method with a `$` in the name, like `someMethod$librarianlib`  
  (this means I forgot to add `@JvmSynthetic` to an internal member)

If you ever run across an API that seems… "funky" from a java perspective, ask about it!

