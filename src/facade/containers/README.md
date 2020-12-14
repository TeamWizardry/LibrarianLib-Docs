# Containers

Containers are used wherever you need a GUI to do something on the server. This can be
accessing an inventory, configuring a block or item, or anything in between.

## Container classes

Facade containers must have a single constructor. That constructor's parameters must 
begin with `int windowId, PlayerEntity player`, followed by any number of 
[Prism](../../prism)-serializable parameters. This constructor should not be called 
directly, only indirectly via the `FacadeContainerType`'s `open` method.

## Opening containers

To open a container you call the `FacadeContainerType.open(player, title, args...)` 
method, passing the player, title, and the extra arguments in the container's 
constructor. Those arguments are used to call the container's constructor on the server, 
then serialized and sent to the client so it can create an identical container.

Note that containers can only be opened on the server.

When possible, you should pass specific values to the constructor instead of relying on 
it to get those values itself. For example, pass tile entity locations directly (you 
can't pass the tile entity itself), instead of having the client deduce what block the 
player is looking at.

## Container communication

Inventories in containers will automatically sync, but for manual/direct communication
between the client and the server, use messages. To create a message, create a method
with any number of Prism-serializable parameters and annotate it with `@Message`. 

To send a message from one side to the other (either the server container to the client 
or the client container to the server), call `sendMessage`, passing the method name and 
the arguments to pass to it. The arguments will be serialized into a packet, sent over
the network, and then the `@Message` method will be called.

***NOTE!*** These messages are still packets, so it's vital you take the 
[same precautions](../../courier#trust) as you would with any client-to-server 
communication.

```java
public class CoolContainer extends FacadeContainer {
    // in the container
    @Message
    private void doThing(String thing, int value) {
        // do something on the server
        this.sendMessage("serverToldMe", thing + " done");
    }

    @Message
    private void serverToldMe(String data) {
        // do something on the client, maybe update a field in a "value changed" message
        // this.sendMessage("doThing", ...); // we can send messages to the server too
    }
}

public class CoolContainerScreen extends ... {
    private void doThingButtonClicked() {
        this.container.sendMessage("doThing", "wow", 42);
    }
}
```

## Container registration

If you're using [Foundation](/foundation), the registration manager can [register 
containers for you](/foundation/registration_manager/containers). If you aren't using
Foundation, registering containers is relatively simple.

The first thing you'll need to do is create the `FacadeContainerType` and put it 
somewhere global. For simplicity in this example I'll put it in the mod class and
constructor, but you can put it wherever you feel works best. 

```java
~import com.teamwizardry.librarianlib.facade.container.FacadeContainerType;
~import net.minecraft.util.ResourceLocation;
~
public class ExampleMod {
    public static FacadeContainerType<CoolContainer> coolContainerType;

    public ExampleMod() {
        coolContainerType = new FacadeContainerType(CoolContainer.class);
        coolContainerType.setRegistryName(new ResourceLocation(
            "examplemod:cool_container"
        ));
    }
}
```

Next you need to register the type in the appropriate registry event and specify a 
`FacadeContainerScreen` type in the client setup event. In order to use a constructor
reference, the screen's constructor should have three parameters: your container,
the player's inventory, and the title 
(`CoolContainer container, PlayerInventory inventory, ITextComponent title`).


```java
~import com.teamwizardry.librarianlib.facade.container.FacadeContainerType;
~import net.minecraft.client.gui.ScreenManager;
~import net.minecraft.inventory.container.ContainerType;
~import net.minecraft.util.ResourceLocation;
~import net.minecraftforge.api.distmarker.Dist;
~import net.minecraftforge.api.distmarker.OnlyIn;
~import net.minecraftforge.event.RegistryEvent;
~import net.minecraftforge.fml.event.lifecycle.FMLClientSetupEvent;
~
public class ExampleMod {
    public static FacadeContainerType<CoolContainer> coolContainerType;

    public ExampleMod() {
        coolContainerType = new FacadeContainerType(CoolContainer.class);
        coolContainerType.setRegistryName(new ResourceLocation(
            "examplemod:cool_container"
        ));
    }

    @SubscribeEvent
    public void registerContainers(RegistryEvent.Register<ContainerType<?>> e) {
        e.getRegistry().register(coolContainerType);
    }

    @OnlyIn(Dist.CLIENT)
    public void clientSetup(FMLClientSetupEvent e) {
        ScreenManager.registerFactory(coolContainerType, CoolContainerScreen::new);
    }
}
```


