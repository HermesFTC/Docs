# Creating Actions

Actions are the basic building blocks of Hermes's non-blocking control flow. 
They are composable, 
stateful commands that can be executed sequentially or in parallel.

There are a few ways to create actions:

## Action Interface

### Implementing Directly

The most basic way is to implement the `Action` interface. 
This is a functional interface with a single abstract method, `run(p: TelemetryPacket)`, 
which is called on every loop. 
It should return `true` if the action is still running, 
and `false` if it has completed. 
The `p` argument can be used to communicate information back to FTC Dashboard.

=== "Kotlin"

    ```kotlin
    class MyAction : Action {
        override fun run(p: TelemetryPacket): Boolean {
            // ... do something
            return true // or false
        }
    }
    ```

=== "Java"

    ```java
    class MyAction implements Action {
        @Override
        public boolean run(@NonNull TelemetryPacket p) {
            // ... do something
            return true; // or false
        }
    }
    ```

### Action Lambdas

Note that while it is possible to create an `Action` 
by making an implementation of the interface, 
it can be tedious to do this for every possible robot action.
Since `Action` is a functional interface, 
it is easy to create objects of it using lambda syntax.
This means you can create methods of your robot classes that 
return Actions without making an entire class.

> [!TIP]
> To learn more about lambdas, read 
> [this guide on lambdas in "Java"](https://dev.java/learn/lambdas/first-lambdas/),
> or [this guide on higher order functions and lambdas in "Kotlin"](https://kotlinlang.org/docs/lambdas.html).
> Hermes doesn't require any advanced knowledge about them, 
> but if you're interested in contributing to any NextFTC library,
> or programming in general, it might be useful to know.

=== "Kotlin"

    ```kotlin 
    class Subsystem {
        fun myAction(): Action {
            return Action { 
                //the TelemetryPacket paramemter is named `it` by default in the lambda
                // ... do something
                true // or false
            }
        }
    }
    ```

=== "Java"

    ```java
    class Subsystem {
        public Action myAction() {
            return p -> { 
                // here we named the TelemetryPacket parameter `p` 
                // ... do something
                return true; // or false
            };
        }
    }
    ```

### Previewing Actions

The `Action` interface also provides a `preview()` method 
that can be overridden to draw a preview of the action on the FTC Dashboard field overlay.
This is useful for visualizing the robot's path or other actions during autonomous.
The QuickStart's `FollowTrajectoryAction` implements it to draw the trajectory!

=== "Kotlin"

    ```kotlin
    class MyAction : Action {
        override fun run(p: TelemetryPacket): Boolean {
            // ... do something
            return true // or false
        }
    
        override fun preview(fieldOverlay: Canvas) {
            // ... draw on the field overlay
        }
    }
    
    //or 
    
    fun myAction(): Action {
        return Action {
            // ... do something 
            true // or false
        }.withPreview {
            // ... draw on the field overlay
        }
    }
    ```

=== "Java"

    ```java
    class MyAction implements Action {
        @Override
        public boolean run(@NonNull TelemetryPacket p) {
            // ... do something
            return true; // or false
        }
    
        @Override
        public void preview(@NonNull Canvas fieldOverlay) {
            // ... draw on the field overlay
        }
    }
    
    //or 
    
    Action myAction() {
        return p -> { 
            // ... do something 
            return true; // or false
        }.withPreview(c -> {
            // ... draw on the field overlay
        });
    }
    ```

### Action Requirements

Actions can also declare requirements. 
Requirements are a set of objects that the action needs to function.
The `ActionRunner` will ensure that no two actions with the same requirement can run at the same time.
This is useful for preventing two different actions from trying to control 
the same motor or servo simultaneously.

The type of `requirements` is `Set<Any>` to allow any object to be a requirement.
This can be a subsystem, a motor object, etc.

You can specify requirements by overriding the `requirements` property in your action, 
or by calling the `withRequirements` function, 
which returns a copy of the receiver with its argument
(either multiple objects or a set of objects) as the requirements.

=== "Kotlin"

    ```kotlin
    class MyAction(private val subsystem: Subsystem) : Action {
        override val requirements: Set<Any> = setOf(subsystem)
    
        override fun run(p: TelemetryPacket): Boolean {
            // ... do something with the subsystem
            return true // or false
        }
    }
    
    //or
    
    class MySubsystem {
        fun myAction(): Action {
            return Action { 
                // ... do something with the subsystem
                true // or false
            }.withRequirements(this)
        }
    }
    ```

=== "Java"

    ```java
    class MyAction implements Action {
        private final Subsystem subsystem;
        private final HashSet<Object> reqs;
        //we define reqs as a field so it is not re-initialized every time
    
        public MyAction(Subsystem subsystem) {
            this.subsystem = subsystem;
            this.reqs = new HashSet<>(Collections.singletonList(subsystem));
        }
    
        @Override
        public Set<Object> getRequirements() {
            return reqs;
        }
    
        @Override
        public boolean run(@NonNull TelemetryPacket p) {
            // ... do something with the subsystem
            return true; // or false
        }
    }
    
    //or
    
    class MySubsystem {
        public Action myAction() {
            return p -> { 
                // ... do something with the subsystem
                return true; // or false
            }.withRequirements(this);
        }
    }
    
    ```

## ActionEx

`ActionEx`, the expanded Action class, 
has two main methods of creating new Action types.

### Subclassing `ActionEx`

For more complex actions, you can use the `ActionEx` class. 
This class provides `init()`, `loop()`, and `end()` methods 
that are called at the beginning, during, and at the end of the action, respectively.

The `loop` method of an `ActionEx` follows the same logic as `Action`'s `run()`:
it should return `true` if the action is still running, 
and `false` if it has completed. 
Once it returns `false`, `end` is triggerred.

=== "Kotlin"

    ```kotlin
    class MyActionEx : ActionEx() {
        override fun init(packet: TelemetryPacket) {
            // ... called once at the beginning
        }
    
        override fun loop(packet: TelemetryPacket): Boolean {
            // ... called on every loop
            return true // or false
        }
    
        override fun end(packet: TelemetryPacket) {
            // ... called once at the end
        }
    }
    ```

=== "Java"

    ```java
    class MyActionEx extends ActionEx {
        @Override
        public void init(@NonNull TelemetryPacket packet) {
            // ... called once at the beginning
        }
    
        @Override
        public boolean loop(@NonNull TelemetryPacket packet) {
            // ... called on every loop
            return true; // or false
        }
    
        @Override
        public void end(@NonNull TelemetryPacket packet) {
            // ... called once at the end
        }
    }
    ```

### Building With `ActionEx`

`ActionEx` also provides a builder-style interface for creating actions. 
This is useful for creating complex actions without having to create a new class.

=== "Kotlin"

    ```kotlin
    val myAction = ActionEx()
        .withInit { /* ... called once at the beginning */ }
        .withLoop { /* ... called on every loop */ true }
        .withEnd { /* ... called once at the end */ }
        .withRequirements(/* optional */ )
    ```

=== "Java"

    ```java
    Action myAction = new ActionEx()
        .withInit(p -> { /* ... called once at the beginning */ })
        .withLoop(p -> { /* ... called on every loop */ return true; })
        .withEnd(p -> { /* ... called once at the end */ })
        .withRequirements(/* optional */ );
    ```

An `InstantAction` is a simple action that executes a given function once and then completes.

=== "Kotlin"

    ```kotlin
    val myInstantAction = InstantAction { 
        // ... do something 
    }
    ```

=== "Java"

    ```java
    InstantAction myInstantAction = new InstantAction(() -> {
        // ... do something
    });
    ```

## Interruptible

The `Interruptible` interface represents an `Action` that can be interrupted. 
When an interruption occurs, the `onInterrupt` method is called,
and the resulting `Action` is added to the queue.

We recommend defining it with the `interruptible` function, 
which accepts an Action:

=== "Kotlin"

    ```kotlin 
    class Subsystem {
        fun myAction(): Action {
            return Action { 
                //the TelemetryPacket paramemter is named `it` by default in the lambda
                // ... do something
                true // or false
            }.interruptible(Action {
                // ... do something (like stop the motors the original action needed)
                false
            })
        }
    }
    ```

===

    ```java
    class Subsystem {
        public Action myAction() {
            return p -> { 
                // ... do something
                return true; // or false
            }.interruptible(p -> {
                // ... do something (like stop the motors the original action needed)
                return false;
            });
        }
    }
    ```

Interruption is currently triggered by two things:
1. If `ActionRunner` detects a requirements collision
    (a newly added action has the same requirement as a previously added action),
    it will remove the old action from the queue, 
    and add its `onInterrupt` action instead.
2. If one action in a `RaceAction` ends,
    the other actions' `onInterrupt` actions are added to the queue.

## Action Groups

`Hermes` provides several ways to combine actions into groups.

- `SequentialAction`: Executes a series of actions one after the other.
- `ParallelAction`: Executes a group of actions at the same time. 
   The group completes when all actions have completed.
- `RaceAction`: Executes a group of actions at the same time. 
   The group completes when any one of the actions completes.
- `SleepAction` (not technically a group, but a similar utility): 
   does nothing and completes after the specified `Duration`.

You can also use the `then()`, `with()`, `race()`, and `delay()`  functions to chain actions together in a more readable way.

=== "Kotlin"

    ```kotlin
    val myAction = MyAction1()
        .then(MyAction2()) // run MyAction2 after MyAction1
        .with(MyAction3())  // run MyAction3 at the same time
        .delay(1.0) //wait 1 second before executing
    ```

=== "Java"

    ```java
    Action myAction = new MyAction1()
        .then(new MyAction2()) // run MyAction2 after MyAction1
        .with(new MyAction3())  // run MyAction3 at the same time
        .delay(1.0); //wait 1 second before executing
    ```

### `SleepAction`

The constructor for `SleepAction` actually accepts a `Duration` object,
which allows you to more easily specify durations:

=== "Kotlin"

    ```kotlin 
    val delay500ms = SleepAction(500.milliseconds) //extension function from "Kotlin" stdlib
    ```

=== "Java"

    ```java
    Action delay500ms = SleepAction(Duration.ofMillis(500));
    ```