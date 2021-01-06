# Non-Inventory Container

In this example I'll be creating an item that allows you to set blocks adjacent to where
you clicked to dirt:

<video width="750" autoplay loop><source src="non_inventory.mp4" type="video/mp4"></video>

### Opening the Container

```java
~import com.teamwizardry.librarianlib.facade.example.ExampleModContainers;
~import net.minecraft.entity.player.ServerPlayerEntity;
~import net.minecraft.item.Item;
~import net.minecraft.item.ItemUseContext;
~import net.minecraft.util.ActionResultType;
~import net.minecraft.util.text.TranslationTextComponent;
~import org.jetbrains.annotations.NotNull;
~
public class DirtSetterItem extends Item {
    public DirtSetterItem(@NotNull Item.Properties properties) {
        super(properties);
    }

    @Override
    public ActionResultType onItemUse(ItemUseContext context) {
        if (!context.getWorld().isRemote) {
            ExampleModContainers.dirtSetterContainerType.open(
                    (ServerPlayerEntity) context.getPlayer(),
                    new TranslationTextComponent("modid.container.dirt_setter"),
                    // additional constructor arguments:
                    context.getPos()
            );
        }
        return ActionResultType.SUCCESS;
    }
}
```

### The Container

```java
~import com.teamwizardry.librarianlib.facade.container.FacadeContainer;
~import com.teamwizardry.librarianlib.facade.container.messaging.Message;
~import com.teamwizardry.librarianlib.facade.example.ExampleModContainers;
~import net.minecraft.block.Blocks;
~import net.minecraft.entity.player.PlayerEntity;
~import net.minecraft.util.math.BlockPos;
~import org.jetbrains.annotations.NotNull;
~
public class DirtSetterContainer extends FacadeContainer {
    private final BlockPos pos;

    public DirtSetterContainer(
            int windowId, @NotNull PlayerEntity player,
            BlockPos pos
    ) {
        super(ExampleModContainers.dirtSetterContainerType, windowId, player);
        this.pos = pos;
    }

    @Message
    private void setBlockPressed(int offset) {
        if(isClientContainer())
            return; // don't actually set the block on the client

        // NEVER trust the client
        if(offset > 1) offset = 1;
        if(offset < -1) offset = -1;
        getPlayer().world.setBlockState(
                pos.add(0, offset, 0),
                Blocks.DIRT.getDefaultState()
        );
    }

    @Override
    public boolean canInteractWith(PlayerEntity player) {
        return true;
    }
}
```

```java
~import com.teamwizardry.librarianlib.facade.container.FacadeContainerType;
~import com.teamwizardry.librarianlib.facade.example.containers.DirtSetterContainer;
~import com.teamwizardry.librarianlib.facade.example.containers.DirtSetterContainerScreen;
~import net.minecraft.client.gui.ScreenManager;
~import net.minecraft.inventory.container.ContainerType;
~import net.minecraftforge.event.RegistryEvent;
~import net.minecraftforge.fml.event.lifecycle.FMLClientSetupEvent;
~
public class ExampleModContainers {
    public static FacadeContainerType<DirtSetterContainer> dirtSetterContainerType =
            new FacadeContainerType<>(DirtSetterContainer.class);

    static {
        dirtSetterContainerType.setRegistryName("modid:dirt_setter");
    }

    public static void registerContainers(RegistryEvent.Register<ContainerType<?>> e) {
        e.getRegistry().register(dirtSetterContainerType);
    }

    public static void clientSetup(FMLClientSetupEvent e) {
        ScreenManager.registerFactory(
                dirtSetterContainerType,
                DirtSetterContainerScreen::new
        );
    }
}
```

### The Screen

```java
~import com.teamwizardry.librarianlib.facade.container.FacadeContainerScreen;
~import com.teamwizardry.librarianlib.facade.layers.StackLayout;
~import com.teamwizardry.librarianlib.facade.pastry.layers.PastryButton;
~import com.teamwizardry.librarianlib.math.Align2d;
~import com.teamwizardry.librarianlib.math.Vec2d;
~import net.minecraft.entity.player.PlayerInventory;
~import net.minecraft.util.text.ITextComponent;
~import org.jetbrains.annotations.NotNull;
~
public class DirtSetterContainerScreen extends FacadeContainerScreen<DirtSetterContainer> {
    public DirtSetterContainerScreen(
            @NotNull DirtSetterContainer container,
            @NotNull PlayerInventory inventory,
            @NotNull ITextComponent title
    ) {
        super(container, inventory, title);

        getMain().setSize(new Vec2d(100, 50));

        PastryButton plusOne = new PastryButton("Set Y+1 to dirt",
                () -> sendMessage("setBlockPressed", 1)
        );
        PastryButton zero = new PastryButton("Set Y+0 to dirt",
                () -> sendMessage("setBlockPressed", 0)
        );
        PastryButton minusOne = new PastryButton("Set Y-1 to dirt",
                () -> sendMessage("setBlockPressed", -1)
        );

        getMain().add(StackLayout.build()
                .align(Align2d.CENTER)
                .size(getMain().getSize())
                .spacing(1)
                .add(plusOne, zero, minusOne)
                .build()
        );
    }
}
```