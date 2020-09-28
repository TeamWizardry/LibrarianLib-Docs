# Event Bus

Etcetera provides a basic event bus with a reflection-based hooking system a la
`@SubscribeEvent`. This system was originally created for [Facade](../facade/README.md).

First, some terminology. **Events** are fired on an **event bus**. Other code can then **hook**
into that event type on that bus. Events can then be **fired** on that event bus and the bus will
run all of the hooks registered for that event type, passing the event object to each one in
succession.

## Event Bus

Creating an event bus just requires creating an `EventBus` object.

```java
~import com.teamwizardry.librarianlib.etcetera.eventbus.EventBus;
~
~public class ExampleBus {
    public final EventBus BUS = new EventBus();
~}
```

## Events

Events come in two flavors, cancelable and non-cancelable. I'll be covering the non-cancelable
events first since cancelable events just build on top of them.

Events typically have fields that hold details about what happened (e.g. what mouse button was
clicked). These are typically `final` fields, since events shouldn't be changing them. Some
events want something from the hooks (e.g. allowing event hooks to constrain a value before it's
used), so they will have non-final fields that the hooks can modify. The data from the event
object will then be read after firing it.

### Simple events

Simple events are, well, simple. However, they have a few tricks up their sleeve. The only thing
you need to do to create your own event is subclass `Event`.

```java
~import com.teamwizardry.librarianlib.etcetera.eventbus.Event;
~
public class ExampleEvent extends Event {
    public final int action;

    public ExampleEvent(int action) {
        this.action = action;
    }
}
```

### Cancelable events

Cancelable events are virtually identical to plain events, except that they have a `cancel()`
method that will stop any further processing of the event (unless they request otherwise. I'll
get into that) and the code firing the event may interpret the event being canceled in some special way.

```java
~import com.teamwizardry.librarianlib.etcetera.eventbus.CancelableEvent;
~
public class ExampleCancelableEvent extends CancelableEvent {
    public final int action;

    public ExampleCancelableEvent(int action) {
        this.action = action;
    }
}
```

## Hooks

Hooks into events can be made one of two ways. Either by manually calling the `hook` method on
the bus or by annotating a method with `@Hook` and calling the `register` method on the bus.

These two blocks are functionally equivalent:

```java
~public class ExampleHooks {
    public void manualHook() {
        BUS.hook(ExampleEvent.class, (ExampleEvent e) -> {
            // run code here
        });
    }
~}
```
```java
~import com.teamwizardry.librarianlib.etcetera.eventbus.Hook;
~
~public class ExampleHooks {
    public void autoHook() {
        BUS.register(this);
    }

    // hook methods don't need to be public
    @Hook
    private void example(ExampleEvent e) {
        // run code here
    }
~}
```

## Advanced events/hooks

### Hook options

Similarly to Forge event hooks, Etcetera event hooks can have a priority and can request to still 
recieve canceled events. The priority and `receiveCanceled` flag can be passed to the `hook` method
or in the `@Hook` annotation, as the case may be. The priority is useful for canceling events, since
you can request for your event to run before others.

### Per-hook state 

Each individual event hook can hold some state, and your event can act on that by overriding the
`storePerHookState` and `loadPerHookState` methods. Whatever value was returned from
`storePerHookState` is passed to `loadPerHookState` the next time *that hook* is called, meaning
each individual hook can have independent state.

```java
~import com.teamwizardry.librarianlib.etcetera.eventbus.Event;
~import org.jetbrains.annotations.Nullable;
~
public class ExampleEventState extends Event {
    public final int delta;
    public int accumulator;

    public ExampleEventState(int delta) {
        this.delta = delta;
    }

    @Override
    protected void loadPerHookState(@Nullable Object state) {
        if(state != null) {
            accumulator = (Integer)state;
        } else {
            accumulator = 0;
        }
        accumulator += delta;
    }

    @Nullable
    @Override
    protected Object storePerHookState() {
        return accumulator;
    }
}
```
```java
~import com.teamwizardry.librarianlib.etcetera.eventbus.Hook;
~
~public class ExampleHooks {
~    public void autoHook() {
~        BUS.register(this);
~    }
~
    // this will run every 100 "units" of delta. This is useful for mouse
    // scroll events, where lots of little deltas should add up to one step
    @Hook
    private void example(ExampleEventState e) {
        while(e.accumulator >= 100) {
            e.accumulator -= 100;
            // do the thing
        }
    }
~}
```
