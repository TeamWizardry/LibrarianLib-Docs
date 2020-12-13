# Items

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

        ModItems.register(getRegistrationManager());
    }
}
```

## Basic item

```java
~import com.teamwizardry.librarianlib.foundation.registration.ItemSpec;
~import com.teamwizardry.librarianlib.foundation.registration.LazyItem;
~import com.teamwizardry.librarianlib.foundation.registration.RegistrationManager;
~
public class ModItems {
    public static final LazyItem coolItem = new LazyItem();

    public static void register(RegistrationManager registrationManager) {
        coolItem.from(registrationManager.add(
                new ItemSpec("cool_item")
                        .datagen(dataGen -> {
                            dataGen.name("en_US", "Cool Item");
                        })
        ));
    }
}
```

## Configuring `Item.Properties`

```java
~import com.teamwizardry.librarianlib.foundation.registration.ItemSpec;
~import com.teamwizardry.librarianlib.foundation.registration.LazyItem;
~import com.teamwizardry.librarianlib.foundation.registration.RegistrationManager;      
~import net.minecraft.item.Food;
~import net.minecraft.tags.ItemTags;
~
public class ModItems {
    public static final LazyItem fancyFish = new LazyItem();
    public static final Food fancyFishFood = (new Food.Builder()).hunger(2)
            .saturation(0.1F).build();

    public static void register(RegistrationManager registrationManager) {
        fancyFish.from(registrationManager.add(
                new ItemSpec("fancy_fish")
                        .maxStackSize(16)
                        .food(fancyFishFood)
                        .datagen(dataGen -> {
                            dataGen.tags(ItemTags.FISHES);
                            dataGen.name("en_US", "Fancy Fish");
                        })
        ));
    }
}
```

## Custom item class

The registration manager supports any `Item` subclass, but here I use `BaseItem` since it 
implements `IFoundationItem` to provide default model generation out of the box.

```java
import com.teamwizardry.librarianlib.foundation.registration.ItemSpec;
import com.teamwizardry.librarianlib.foundation.registration.LazyItem;
import com.teamwizardry.librarianlib.foundation.registration.RegistrationManager;

public class ModItems {
    public static final LazyItem coolItem = new LazyItem();

    public static void register(RegistrationManager registrationManager) {
        coolItem.from(registrationManager.add(
                new ItemSpec("cool_item")
                        .item(spec -> new CoolItem(spec.getItemProperties()))
                        .datagen(dataGen -> {
                            dataGen.name("en_US", "Cool Item");
                        })
        ));
    }
}
```