# Tile Entities

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

        ModTiles.register(getRegistrationManager());
        ModBlocks.register(getRegistrationManager());
    }
}
```

## 1. Tile Entity

```java
~import com.teamwizardry.librarianlib.foundation.registration.LazyTileEntityType;
~import com.teamwizardry.librarianlib.foundation.registration.RegistrationManager;
~import com.teamwizardry.librarianlib.foundation.registration.TileEntitySpec;
~
public class ModTiles {
    public static LazyTileEntityType<CoolTileEntity> coolTile
            = new LazyTileEntityType<>();

    public static void register(RegistrationManager registrationManager) {
        coolTile.from(registrationManager.add(
                new TileEntitySpec<>("cool_item", () -> new CoolTileEntity())
                        .renderer(CoolTileEntityRenderer::new)
        ));
    }
}
```
```java
~import com.teamwizardry.librarianlib.foundation.tileentity.BaseTileEntity;
~import com.teamwizardry.librarianlib.prism.Save;
~import com.teamwizardry.librarianlib.prism.Sync;
~import net.minecraft.entity.Entity;
~import net.minecraft.util.text.StringTextComponent;
~
public class CoolTileEntity extends BaseTileEntity {
    @Save // save this field to the world file
    @Sync // sync this field with clients
    private int coolCount;

    public CoolTileEntity() {
        super(ModTiles.coolTile);
    }

    public void speak(Entity entity) {
        if(getWorld().isRemote) {
            entity.sendMessage(new StringTextComponent("Client Cool #" + coolCount));
        } else {
            entity.sendMessage(new StringTextComponent("Server Cool #" + coolCount));
        }

        coolCount++;
        markDirty(); // make sure the new state is saved
        notifyStateChange(); // send the new state to clients
    }
}
```

## 2. Block

```java
~import com.teamwizardry.librarianlib.foundation.registration.BlockSpec;
~import com.teamwizardry.librarianlib.foundation.registration.LazyBlock;
~import com.teamwizardry.librarianlib.foundation.registration.RegistrationManager;
~
public class ModBlocks {
    public static final LazyBlock coolTileBlock = new LazyBlock();

    public static void register(RegistrationManager registrationManager) {
        coolTileBlock.from(registrationManager.add(
                new BlockSpec("cool_tile")
                        .tileEntity(ModTiles.coolTile)
                        .block(spec -> new CoolTileBlock(spec.getBlockProperties()))
        ));
    }
}
```
```java
~import com.teamwizardry.librarianlib.foundation.block.BaseBlock;
~import net.minecraft.block.Block;
~import net.minecraft.block.BlockState;
~import net.minecraft.entity.player.PlayerEntity;
~import net.minecraft.tileentity.TileEntity;
~import net.minecraft.util.ActionResultType;
~import net.minecraft.util.Hand;
~import net.minecraft.util.math.BlockPos;
~import net.minecraft.util.math.BlockRayTraceResult;
~import net.minecraft.world.IBlockReader;
~import net.minecraft.world.World;
~import org.jetbrains.annotations.NotNull;
~import org.jetbrains.annotations.Nullable;
~
public class CoolTileBlock extends BaseBlock {
    public CoolTileBlock(@NotNull Block.Properties properties) {
        super(properties);
    }

    @Override
    public ActionResultType onBlockActivated(
            BlockState state, World worldIn, BlockPos pos, PlayerEntity player,
            Hand handIn, BlockRayTraceResult hit
    ) {
        TileEntity te = worldIn.getTileEntity(pos);
        if(!(te instanceof CoolTileEntity))
            return ActionResultType.FAIL;

        ((CoolTileEntity) te).speak(player);
        return ActionResultType.SUCCESS;
    }

    @Override
    public boolean hasTileEntity(BlockState state) {
        return true;
    }

    @Nullable
    @Override
    public TileEntity createTileEntity(BlockState state, IBlockReader world) {
        return new CoolTileEntity();
    }
}
```

## 3. Renderer

The renderer is specified using the `TileEntitySpec`'s `renderer` method.

```java
~import com.mojang.blaze3d.matrix.MatrixStack;
~import net.minecraft.client.renderer.IRenderTypeBuffer;
~import net.minecraft.client.renderer.tileentity.TileEntityRenderer;
~import net.minecraft.client.renderer.tileentity.TileEntityRendererDispatcher;
~
public class CoolTileEntityRenderer extends TileEntityRenderer<CoolTileEntity> {
    public CoolTileEntityRenderer(TileEntityRendererDispatcher rendererDispatcherIn) {
        super(rendererDispatcherIn);
    }

    @Override
    public void render(
            CoolTileEntity tileEntityIn, float partialTicks, MatrixStack matrixStackIn,
            IRenderTypeBuffer bufferIn, int combinedLightIn, int combinedOverlayIn
    ) {
        // render stuff here
    }
}
```