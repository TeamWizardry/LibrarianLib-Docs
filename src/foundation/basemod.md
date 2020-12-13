# BaseMod

Most Foundation-based mods will extend `BaseMod`, which provides a lot of boilerplate code,
however most of the functionality it provides (e.g. [Registration
Managers](./registration_manager/README.md)) can be used even if you don't extend `BaseMod`.

A minimal mod class would look like this. Note that for Kotlin mods being loaded through Kottle,
you have to pass `true` to the `kottleContext` constructor parameter.

```java
~import com.teamwizardry.librarianlib.foundation.BaseMod;
~import net.minecraftforge.fml.common.Mod;
~
@Mod(ExampleMod.MODID)
public class ExampleMod extends BaseMod {
    public static final String MODID = "examplemod";
    public static ExampleMod INSTANCE;

    public ExampleMod() {
        INSTANCE = this;
        setLoggerBaseName("Example Mod");
    }
}
```

## `BaseMod` features

### Override points

Trying to remember what events to do what during can be confusing and annoying, so Foundation 
provides pre-built methods designed to be overridden by subclasses.

- `commonSetup` - Called on the `FMLCommonSetupEvent`
- `clientSetup` - Called on the `FMLClientSetupEvent`
- `dedicatedServerSetup` - Called on the `FMLDedicatedServerSetupEvent`
- `interModCommsEnqueue` - Send IMC messages here
- `interModCommsProcess` - Receive IMC messages here
- `modLoadingContextEventBus` - If you're using a custom mod language provider, override this
- `createRegistries` - Create any custom registries here
- `registerBlocks`**†**
- `registerItems`**†**
- `registerTileEntities`**†**
- `registerEntities`**†**
- `registerFluids`**‡**
- `registerContainers`**‡**
- `registerSounds`**‡**

**†** If you can, prefer using the [Registration Manager](./registration_manager/README.md) instead 
of registering manually.  
**‡** Support for these in the Registration Manager has not been implemented yet.

### Loggers

The default logger configuration for forge means that creating a logger for a specific class results 
in a useless mess of a logger name. For example, creating a logger for the `MessageScanner` class in 
Facade would result in a logger named `co.te.li.fa.co.me.MessageScanner`. That name is ugly and does
little to help identify the actual mod or class. Luckily, Foundation provides a system for making 
*useful* loggers.

At the beginning of your mod constructor, you should call `setLoggerBaseName` and give it a 
human-readable mod name (e.g. `"Example Mod"`). After you do that, you can get the root logger from
the mod using `mod.getLogger()` or create custom loggers using `mod.makeLogger("Subsystem name")` or
`mod.makeLogger(SomeClass.class)`. These two methods will produce loggers named `"Example Mod: 
Subsystem name"` and `"Example Mod: SomeClass"` respectively.

This logger functionality is also available independent of `BaseMod` in the `ModLogManager` class.

## Recommended mod structure

I won't prescribe much organization, but I would recommend you have a set of classes (e.g.
`ModBlocks`, `ModItems`, `ModTiles`) that handle registering and holding references to their 
specific types of object. The mod class would then call methods on these classes and pass them the
mod's Registration Manager.

```java
~import com.teamwizardry.librarianlib.foundation.registration.BlockSpec;
~import com.teamwizardry.librarianlib.foundation.registration.LazyBlock;
~import com.teamwizardry.librarianlib.foundation.registration.RegistrationManager;
~
public class ModBlocks {
    public static final LazyBlock awesomeBricks = new LazyBlock();

    public static void register(RegistrationManager registrationManager) {
        awesomeBricks.from(registrationManager.add(
                new BlockSpec("awesome_bricks")
                        // ...spec configuration
        ));
    }
}
```

You should also add everything to the Registration Manager during your mod's constructor. This is 
to ensure everything is set up before any of important events that the Registration Manager uses.
