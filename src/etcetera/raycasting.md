# Raycasting

Etcetera provides highly optimized `Raycaster` class, which is designed to reduce the number of
short-lived objects and improve the performance of long raycasts. In order to avoid unnecessary
object allocation, the `Raycaster` object contains fields for the hit result, in contrast to
Minecraft's function, which returns a `RayTraceResult` object (and creates possibly *hundreds* of
temporary objects along the way).

## Casting rays

When appropriate, you should reuse a single `Raycaster` instance, and after you're done with the
hit result you should *always* call `reset()`. While the `Raycaster` class is very lightweight,
unnecessarily creating new instances somewhat defeats the minimal object allocation aspect of it.
Calling `reset()` is *vitally* important. When performing a raycast the result will be discarded
if it's farther away than the current result. This both simplifies the internal implementation
and allows you to combine the results of multiple raycasts with different settings.

In the interest of brevity these examples will use temporary objects (e.g. `Vec3d`), however if
you're writing performance-critical code you should avoid these and use primitive variables.

### Basic raycasting

```java
~import com.teamwizardry.librarianlib.etcetera.Raycaster;
~import net.minecraft.entity.Entity;
~import net.minecraft.util.math.BlockPos;
~import net.minecraft.util.math.Vec3d;
~import org.jetbrains.annotations.Nullable;
~
~public class BasicRaycastExample {
    // note: Raycaster is *not* thread-safe, though world should only be 
    // accessed from the main thread anyway.
    private static Raycaster raycaster = new Raycaster();

    @Nullable
    public BlockPos basicBlockRaycast(Entity entity) {
        Vec3d start = entity.getEyePosition(0);
        Vec3d look = entity.getLookVec();

        // cast the ray
        raycaster.cast(entity.getEntityWorld(), Raycaster.BlockMode.VISUAL,
                start.getX(), start.getY(), start.getZ(),
                start.getX() + look.getX() * 100,
                start.getY() + look.getY() * 100,
                start.getZ() + look.getZ() * 100
        );

        // get the result out of it
        BlockPos result = null;
        if (raycaster.getHitType() == Raycaster.HitType.BLOCK) {
            result = new BlockPos(
                    raycaster.getBlockX(),
                    raycaster.getBlockY(),
                    raycaster.getBlockZ()
            );
        }

        // it is VITALLY important that you do this
        raycaster.reset();
        return result;
    }
~}
```

### Advanced raycasting

While `Raycaster` has a convenience method for raycasting blocks, it can also cast against fluid or
entities using the full `cast` method.

```java
~import com.teamwizardry.librarianlib.etcetera.Raycaster;
~import net.minecraft.entity.Entity;
~import net.minecraft.entity.player.PlayerEntity;
~import net.minecraft.util.math.BlockPos;
~import net.minecraft.util.math.Vec3d;
~
~import java.util.function.Predicate;
~
~public class AdvancedRaycastExample {
    // note: Raycaster is *not* thread-safe, though world should only be 
    // accessed from the main thread anyway.
    private static Raycaster raycaster = new Raycaster();
    private static Predicate<Entity> isPlayerPredicate = 
            (entity) -> entity instanceof PlayerEntity;


    public void advancedRaycast(Entity entity) {
        double rayLength = 100;
        Vec3d start = entity.getEyePosition(0);
        Vec3d look = entity.getLookVec();
        look = new Vec3d(
                look.getX() * rayLength,
                look.getY() * rayLength,
                look.getZ() * rayLength
        );

        // cast the ray
        raycaster.cast(entity.getEntityWorld(),
                Raycaster.BlockMode.VISUAL,
                Raycaster.FluidMode.SOURCE,
                isPlayerPredicate,
                start.getX(), start.getY(), start.getZ(),
                start.getX() + look.getX(),
                start.getY() + look.getY(),
                start.getZ() + look.getZ()
        );

        // get the result out of it

        if(raycaster.getHitType() == Raycaster.HitType.NONE) {
            return;
        }

        // the fraction along the raycast that the hit occurred
        double distance = raycaster.getFraction() * rayLength;

        // normal and hit position apply to all the hit types
        Vec3d normal = new Vec3d(
                raycaster.getNormalX(),
                raycaster.getNormalY(),
                raycaster.getNormalZ()
        );
        Vec3d hitPos = new Vec3d(
                raycaster.getHitX(),
                raycaster.getHitY(),
                raycaster.getHitZ()
        );

        switch (raycaster.getHitType()) {
            // block and fluid hits both have block positions
            case BLOCK:
            case FLUID:
                BlockPos hitBlock = new BlockPos(
                        raycaster.getBlockX(),
                        raycaster.getBlockY(),
                        raycaster.getBlockZ()
                );
                break;
            // entity hits have the entity that was hit
            case ENTITY:
                Entity hitEntity = raycaster.getEntity();
                break;
        }

        // it is VITALLY important that you do this
        raycaster.reset();
    }
~}
```
