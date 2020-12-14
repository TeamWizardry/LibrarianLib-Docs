# Non-Inventory Container

In this example I'll be creating an item that allows you to set blocks adjacent to where
you clicked to dirt:

<video width="750" autoplay loop><source src="non_inventory.mp4" type="video/mp4"></video>

### Opening the Container

```java
~import com.teamwizardry.librarianlib.foundation.item.BaseItem;
~import net.minecraft.entity.player.ServerPlayerEntity;
~import net.minecraft.item.Item;
~import net.minecraft.item.ItemUseContext;
~import net.minecraft.util.ActionResultType;
~import net.minecraft.util.text.TranslationTextComponent;
~import org.jetbrains.annotations.NotNull;
~
public class CoolItem extends BaseItem {
    public CoolItem(@NotNull Item.Properties properties) {
        super(properties);
    }

    @Override
    public ActionResultType onItemUse(ItemUseContext context) {
        if (!context.getWorld().isRemote) {
            ExampleModContainers.coolContainer.get().open(
                    (ServerPlayerEntity) context.getPlayer(),
                    new TranslationTextComponent("modid.container.cool_item.title"),
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
~import net.minecraft.block.Blocks;
~import net.minecraft.entity.player.PlayerEntity;
~import net.minecraft.util.math.BlockPos;
~import org.jetbrains.annotations.NotNull;
~
public class CoolContainer extends FacadeContainer {
    private final BlockPos pos;

    public CoolContainer(int windowId, @NotNull PlayerEntity player, BlockPos pos) {
        super(ExampleModContainers.coolContainer.get(), windowId, player);
        this.pos = pos;
    }

    @Message
    private void setToDirt(int offset) {
        // NEVER trust the client
        if(offset > 1) offset = 1;
        if(offset < -1) offset = -1;
        getPlayer().world.setBlockState(
                pos.add(0, offset, 0),
                Blocks.DIRT.getDefaultState()
        );
    }

    @Override
    public boolean canInteractWith(PlayerEntity playerIn) {
        return true;
    }
}
```

### The Screen

```java
~import com.teamwizardry.librarianlib.facade.container.FacadeContainerScreen;
~import com.teamwizardry.librarianlib.facade.layers.StackLayout;
~import com.teamwizardry.librarianlib.facade.pastry.components.PastryButton;
~import com.teamwizardry.librarianlib.facade.pastry.layers.PastryBackground;
~import com.teamwizardry.librarianlib.math.Align2d;
~import com.teamwizardry.librarianlib.math.Vec2d;
~import net.minecraft.entity.player.PlayerInventory;
~import net.minecraft.util.text.ITextComponent;
~import org.jetbrains.annotations.NotNull;
~
public class CoolContainerScreen extends FacadeContainerScreen<CoolContainer> {
    public CoolContainerScreen(
            @NotNull CoolContainer container,
            @NotNull PlayerInventory inventory,
            @NotNull ITextComponent title
    ) {
        super(container, inventory, title);

        getMain().setSize(new Vec2d(100, 50));
        getMain().add(new PastryBackground(0, 0, 100, 50));

        PastryButton plusOne = new PastryButton("Set Y+1 to dirt",
                () -> container.sendMessage("setToDirt", 1)
        );
        PastryButton zero = new PastryButton("Set Y+0 to dirt",
                () -> container.sendMessage("setToDirt", 0)
        );
        PastryButton minusOne = new PastryButton("Set Y-1 to dirt",
                () -> container.sendMessage("setToDirt", -1)
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