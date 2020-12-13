# Blocks

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

        ModBlocks.register(getRegistrationManager());
    }
}
```

## Basic block

```java
~import com.teamwizardry.librarianlib.foundation.block.BaseBlock;
~import com.teamwizardry.librarianlib.foundation.registration.BlockSpec;
~import com.teamwizardry.librarianlib.foundation.registration.LazyBlock;
~import com.teamwizardry.librarianlib.foundation.registration.RegistrationManager;
~import com.teamwizardry.librarianlib.foundation.registration.RenderLayerSpec;
~import net.minecraft.block.material.MaterialColor;
~import net.minecraft.tags.BlockTags;
~
public class ModBlocks {
    public static final LazyBlock awesomeBricks = new LazyBlock();

    public static void register(RegistrationManager registrationManager) {
        awesomeBricks.from(registrationManager.add(
                new BlockSpec("awesome_bricks")
                        .renderLayer(RenderLayerSpec.SOLID)
                        .withProperties(BaseBlock.STONE_DEFAULTS)
                        .mapColor(MaterialColor.ADOBE)
                        .datagen(dataGen -> {
                            dataGen.tags(BlockTags.STONE_BRICKS);
                            dataGen.name("en_US", "Awesome Bricks");
                        })
        ));
    }
}
```

## Custom block class

The registration manager supports any `Block` subclass, but here I use `BaseBlock` since it 
implements `IFoundationBlock` to provide default item and block model generation out of the box.

```java
~import com.teamwizardry.librarianlib.foundation.block.BaseBlock;
~import com.teamwizardry.librarianlib.foundation.registration.BlockSpec;
~import com.teamwizardry.librarianlib.foundation.registration.LazyBlock;
~import com.teamwizardry.librarianlib.foundation.registration.RegistrationManager;
~import com.teamwizardry.librarianlib.foundation.registration.RenderLayerSpec;
~import net.minecraft.block.material.MaterialColor;
~import net.minecraft.tags.BlockTags;
~
public class ModBlocks {
    public static final LazyBlock awesomeBricks = new LazyBlock();

    public static void register(RegistrationManager registrationManager) {
        awesomeBricks.from(registrationManager.add(
                new BlockSpec("awesome_bricks")
                        .renderLayer(RenderLayerSpec.SOLID)
                        .withProperties(BaseBlock.STONE_DEFAULTS)
                        .mapColor(MaterialColor.ADOBE)
                        .block(spec -> new AwesomeBricks(spec.getBlockProperties()))
                        .datagen(dataGen -> {
                            dataGen.tags(BlockTags.STONE_BRICKS);
                            dataGen.name("en_US", "Awesome Bricks");
                        })
        ));
    }
}
```
```java
~import com.teamwizardry.librarianlib.foundation.block.BaseBlock;
~import net.minecraft.block.Block;
~import net.minecraft.block.BlockState;
~import net.minecraft.entity.player.PlayerEntity;
~import net.minecraft.util.ActionResultType;
~import net.minecraft.util.Hand;
~import net.minecraft.util.math.BlockPos;
~import net.minecraft.util.math.BlockRayTraceResult;
~import net.minecraft.world.World;
~import org.jetbrains.annotations.NotNull;
~
public class AwesomeBricks extends BaseBlock {
    public AwesomeBricks(@NotNull Block.Properties properties) {
        super(properties);
    }

    @Override
    public ActionResultType onBlockActivated(
            BlockState state, World worldIn, BlockPos pos,
            PlayerEntity player, Hand handIn, BlockRayTraceResult hit
    ) {
        // do stuff
        return ActionResultType.SUCCESS;
    }
}
```

## Custom model generation

Block classes can implement and override `IFoundationBlock` to handle model generation themselves
or the model can be configured directly in the spec, as I'll show here.

In this example I'm configuring the block to use a custom texture location. The 
`BlockStateProvider` also has pre-built methods for things like fences or stairs, however those all
require specific block classes. Plus, for things like that you should probably be using Foundation's
[base blocks](base_blocks.md) and [block collections](block_collections.md).

```java
~import com.teamwizardry.librarianlib.foundation.block.BaseBlock;
~import com.teamwizardry.librarianlib.foundation.registration.BlockSpec;
~import com.teamwizardry.librarianlib.foundation.registration.LazyBlock;
~import com.teamwizardry.librarianlib.foundation.registration.RegistrationManager;
~import com.teamwizardry.librarianlib.foundation.registration.RenderLayerSpec;
~import net.minecraft.block.material.MaterialColor;
~import net.minecraft.tags.BlockTags;
~import net.minecraftforge.client.model.generators.ModelFile;
~
public class ModBlocks {
    public static final LazyBlock awesomeBricks = new LazyBlock();

    public static void register(RegistrationManager registrationManager) {
        awesomeBricks.from(registrationManager.add(
                new BlockSpec("awesome_bricks")
                        .renderLayer(RenderLayerSpec.SOLID)
                        .withProperties(BaseBlock.STONE_DEFAULTS)
                        .mapColor(MaterialColor.ADOBE)
                        .datagen(dataGen -> {
                            dataGen.tags(BlockTags.STONE_BRICKS);
                            dataGen.name("en_US", "Awesome Bricks");
                            dataGen.model(provider -> {
                                ModelFile model = provider.models().cubeAll(
                                        dataGen.getBlock().getRegistryName().getPath(),
                                        provider.modLoc("block/custom_texture_name")
                                );
                                provider.simpleBlock(dataGen.getBlock(), model);
                            });
                        })
        ));
    }
}
```