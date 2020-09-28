# Courier

Courier is LibrarianLib's network library. Courier is designed to solve some usability problems with 
Forge packets, specifically the fact that serialization can be a pain and the Forge API encourages
chucking all your packet implementations into one [god method][god_object].

[god_object]: https://en.wikipedia.org/wiki/God_object

Networking in Minecraft is done using “packets”, and those packets are sent on “channels.” A 
channel is like a radio channel, and packets are like the audio signal being sent on that channel.

## Channels

Creating a channel is simple. All you need to do is create a new `CourierChannel` object with a
channel ID and protocol version. The protocol version is used to check for client/server 
compatibility. By default, mods require identical versions on the client and server, so unless 
you're doing something special on that front this can be a static value.

```java
public static final CourierChannel channel = new CourierChannel(
    new ResourceLocation("your_mod:network"), "0"
);
```

## Packets 

Networking is hard, so there are a few things you need to keep in mind when using packets. 

First, packets are handled on a separate network thread. This means you can *not* interact with most
of the game directly in your packet handler. If you want to interact with the game you need to put
all that interaction inside a `context.enqueueWork(() -> {})`, which will call the given `Runnable`
on the main thread at the next opportunity.

Second, **NEVER EVER EVER** trust the client. Always assume the player is using a hacked client, and
program accordingly. Validate everything. If possible, compute things server-side instead of sending
them from the client. (e.g. compute the block they're looking at on the server instead of the client
sending the block in the packet. If the client can send the block they could do things through walls
or even from across the world. If you have a “fire spell” packet with a “charge” value that ranges
from 0 to 1, make sure it's *actually* between 0 and 1, otherwise a hacked client could casually 
send a packet with a charge of 500 and suddenly be able to fire a spell at 50,000% strength.)

Third, fourth, fifth, and sixth: ***NEVER TRUST THE CLIENT.*** The examples on this page include 
some basic validation, but you should always be thinking about ways people could hack their clients.

Networking is *hard.* Be careful.

## Creating manual packets

Once you've created your channel, it's time to register your packet. Whereas Forge uses callbacks
for serialization and handling, LibLib uses `PacketType` objects. Courier provides a system for 
automatic packet serialization, but I'll first cover entirely custom packets, since they're simpler,
if more manual.

```java
~import com.teamwizardry.librarianlib.core.util.Client;
~import com.teamwizardry.librarianlib.core.util.sided.SidedSupplier;
~import com.teamwizardry.librarianlib.courier.PacketType;
~import net.minecraft.block.Block;
~import net.minecraft.entity.player.PlayerEntity;
~import net.minecraft.network.PacketBuffer;
~import net.minecraft.util.math.BlockPos;
~import net.minecraftforge.fml.network.NetworkDirection;
~import net.minecraftforge.fml.network.NetworkEvent;
~import org.jetbrains.annotations.NotNull;
~
~import java.util.function.Supplier;
~
public class YourPacketType extends PacketType<YourPacketType.Packet> {
    public YourPacketType() {
        super(Packet.class);
    }

    // The actual packet
    public static class Packet {
        public final BlockPos pos;
        public final Block block;

        public Packet(BlockPos pos, Block block) {
            this.pos = pos;
            this.block = block;
        }
    }

    @Override
    public void encode(Packet packet, @NotNull PacketBuffer buffer) {
        buffer.writeBlockPos(packet.pos);
        buffer.writeRegistryId(packet.block);
    }

    @Override
    public Packet decode(@NotNull PacketBuffer buffer) {
        BlockPos pos = buffer.readBlockPos();
        Block block = buffer.readRegistryIdSafe(Block.class);
        return new Packet(pos, block);
    }

    @Override
    public void handle(Packet packet, @NotNull Supplier<NetworkEvent.Context> context) {
        PlayerEntity player;
        if (context.get().getDirection() == NetworkDirection.PLAY_TO_CLIENT) {
            // we can use client-only code in this block
            player = SidedSupplier.client(() -> Client.getPlayer());
        } else {
            player = context.get().getSender();
        }

        // run
        context.get().enqueueWork(() -> {
            // **NEVER** trust the client. If we don't do this it would allow a 
            // hacked client to generate and load arbitrary chunks.
            if (!player.world.isBlockLoaded(packet.pos)) {
                return;
            }
            if (player.world.getBlockState(packet.pos).getBlock() != packet.block) {
                // do something
            }
        });
    }
}
```

Once you've got your packet type, just call `channel.register(new YourPacketType())`. If you want
to limit the direction the packet is sent, you can add a `NetworkDirection` to that method, making 
it `channel.register(new YourPacketType(), NetworkDirection.PLAY_TO_SERVER)`.

## Creating prism packets

You can also use the [Prism](../prism/README.md) module to automatically serialize your packets. To
take advantage of this you need to implement `CourierPacket` and [make it serializable](../prism/refract_class.md). 

```java
~import com.teamwizardry.librarianlib.core.util.Client;
~import com.teamwizardry.librarianlib.core.util.sided.SidedSupplier;
~import com.teamwizardry.librarianlib.courier.CourierPacket;
~import dev.thecodewarrior.prism.annotation.Refract;
~import dev.thecodewarrior.prism.annotation.RefractClass;
~import dev.thecodewarrior.prism.annotation.RefractConstructor;
~import net.minecraft.block.Block;
~import net.minecraft.entity.player.PlayerEntity;
~import net.minecraft.network.PacketBuffer;
~import net.minecraft.util.math.BlockPos;
~import net.minecraftforge.fml.network.NetworkDirection;
~import net.minecraftforge.fml.network.NetworkEvent;
~import org.jetbrains.annotations.NotNull;
~
@RefractClass
public class YourCourierPacket implements CourierPacket {
    @Refract
    public final BlockPos pos;
    @Refract
    public final Block block;

    // parameter types and names match fields
    @RefractConstructor
    public YourCourierPacket(BlockPos pos, Block block) {
        this.pos = pos;
        this.block = block;
    }

    // optionally write anything not supported by Prism.
    @Override
    public void writeBytes(@NotNull PacketBuffer buffer) {
    }

    // optionally read anything not supported by Prism.
    // you'll need to use a non-final field and initialize it in this method.
    @Override
    public void readBytes(@NotNull PacketBuffer buffer) {
    }

    @Override
    public void handle(@NotNull NetworkEvent.Context context) {
        PlayerEntity player;
        if (context.getDirection() == NetworkDirection.PLAY_TO_CLIENT) {
            // we can use client-only code in this block
            player = SidedSupplier.client(() -> Client.getPlayer());
        } else {
            player = context.getSender();
        }

        // run
        context.enqueueWork(() -> {
            // **NEVER** trust the client. If we don't do this
            // it would allow a hacked client to generate and load
            // arbitrary chunks.
            if (!player.world.isBlockLoaded(this.pos)) {
                return;
            }
            if (player.world.getBlockState(this.pos).getBlock() != this.block) {
                // do something
            }
        });
    }
}
```

Once you've got your courier packet class, just call `channel.registerCourierPacket(YourCourierPacket.class)`. 
You can optionally give it a `NetworkDirection` to limit the direction the packet is sent and a 
`BiConsumer` for any inline processing you want to do.

## Sending packets

To send a packet all you have to do is call `channel.send(target, packetInstance)`. The only 
complex part here is the packet target, so I'll go over them briefly.

- `PacketDistributor.SERVER.noArg()`  
  The simplest target, `SERVER` sends a packet from the client to the server. This is the 
  *only* target that's usable on the client, and it isn't usable on the server.
- `PacketDistributor.ALL.noArg()`  
  The simplest server-side target, `ALL` sends a packet to every connected client.
- `PacketDistributor.PLAYER.with(() -> thePlayer)`  
  Sends a packet directly to the supplied player.
- `PacketDistributor.DIMENSION.with(() -> theDimensionType)`  
  Sends a packet to all the players in the supplied dimension.
- `PacketDistributor.NEAR.with(() -> targetPoint)`  
  Sends a packet to every player near the supplied point `TargetPoint`. A `TargetPoint` consists
  of a dimension, xyz coordinates, radius, and optionally an "excluded" player, which won't be sent
  the packet. The excluded player is useful if the packet is related to one player and you only 
  need to send packets to everyone else.
- `PacketDistributor.TRACKING_ENTITY.with(() -> theEntity)`  
  Sends a packet to every player that is currently tracking the supplied entity. If the supplied 
  entity is a player, this will *not* send the packet to that player. If you need that, use the
  next target.
- `PacketDistributor.TRACKING_ENTITY_AND_SELF.with(() -> theEntity)`  
  Sends a packet to every player that is currently tracking the supplied entity. If the supplied 
  entity is a player, this will send the packet to that player as well.
- `PacketDistributor.NMLIST.with(() -> theNetworkManagers)`  
  You will almost never need this one. Sends the packet directly to the supplied `NetworkManager`s
  (you can get a player's network manager using `serverPlayer.connection.getNetworkManager()`).
  
## Replying to packets

If you want to reply to a packet you just received, just pass the packet you would like to send and
the network context to `channel.reply(packet, context)`.