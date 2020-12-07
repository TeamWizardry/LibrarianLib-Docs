# Blocks

## Basic block

```java
~import com.teamwizardry.librarianlib.foundation.block.BaseSimpleBlock;
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
                        .withProperties(BaseSimpleBlock.STONE_DEFAULTS)
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

```java
~import com.teamwizardry.librarianlib.foundation.block.BaseSimpleBlock;
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
                        .withProperties(BaseSimpleBlock.STONE_DEFAULTS)
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
~import com.teamwizardry.librarianlib.foundation.block.BaseSimpleBlock;
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
public class AwesomeBricks extends BaseSimpleBlock {
    public AwesomeBricks(@NotNull Block.Properties properties) {
        super(properties);
    }

    @Override
    public ActionResultType onBlockActivated(
            BlockState state, World worldIn, BlockPos pos,
            PlayerEntity player, Hand handIn, BlockRayTraceResult hit
    ) {
        // do stuff
        return ActionResultType.CONSUME;
    }
}
```
