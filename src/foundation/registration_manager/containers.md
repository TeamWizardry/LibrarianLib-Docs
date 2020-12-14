# Containers

Note that the Registration Manager only natively supports 
[Facade containers](/facade/containers). See that page for documentation on creating
the containers themselves. This page is dedicated to documenting the registration
process using Foundation.

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

        ModContainers.register(getRegistrationManager());
    }
}
```

## Registering

To register a container you just need to give it a name, tell Foundation the 
`FacadeContainer` class name, and give it the `FacadeContainerScreen` constructor.

```java
~import com.teamwizardry.librarianlib.foundation.registration.ContainerSpec;
~import com.teamwizardry.librarianlib.foundation.registration.LazyContainerType;
~import com.teamwizardry.librarianlib.foundation.registration.RegistrationManager;
~
public class ModContainers {
    public static final LazyContainerType<CoolContainer> coolContainer
            = new LazyContainerType<>();

    public static void register(RegistrationManager registrationManager) {
        coolContainer.from(registrationManager.add(
                new ContainerSpec<>("cool_container",
                        CoolContainer.class,
                        CoolContainerScreen::new
                )
        ));
    }
}
```

## Opening

Opening the container is simple. All you need is the container type, player, title, and
any additional constructor arguments your container uses:

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
~    public CoolItem(@NotNull Item.Properties properties) {
~        super(properties);
~    }
~
    @Override
    public ActionResultType onItemUse(ItemUseContext context) {
        if (!context.getWorld().isRemote) {
            ModContainers.coolContainer.get().open(
                    (ServerPlayerEntity) context.getPlayer(),
                    new TranslationTextComponent("examplemod.cool_container.title"),
                    // additional constructor arguments:
                    context.getPos()
            );
        }
        return ActionResultType.SUCCESS;
    }
}
```

## Container implementation

Your container's constructor should pull in the container type from the lazy when 
calling super:

```java
public CoolContainer(
        // built-in arguments
        int windowId, @NotNull PlayerEntity player,
        // additional arguments
        BlockPos pos
) {
    super(ModContainers.coolContainer.get(), windowId, player);
    this.pos = pos;
}
```
