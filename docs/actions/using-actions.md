# Using Actions

## `runBlocking`

The simplest way to execute an action is with `runBlocking()`. 
This function will run the given action to completion while blocking the current thread.
This is useful for simple, sequential autonomous op modes.

=== "Kotlin"

    ```kotlin
    runBlocking(MyAction())
    ```

=== "Java"

    ```java
    Actions.runBlocking(new MyAction());
    ```

While the action is running, telemetry packets are sent to FTC Dashboard, 
allowing you to visualize the action's state.

## `ActionRunner`

For more complex scenarios, like TeleOp, 
where you need to run multiple actions concurrently, 
you can use the `ActionRunner`.
It is a singleton object that manages a queue of asynchronous actions.

### Running Actions

To run an action with the `ActionRunner`, simply call `ActionRunner.run()`:

=== "Kotlin"

    ```kotlin
    ActionRunner.run(MyAction())
    ```

=== "Java"

    ```java
    ActionRunner.run(new MyAction());
    ```

You can also run multiple actions at once:

=== "Kotlin"

    ```kotlin
    ActionRunner.run(
        MyAction1(),
        MyAction2()
    )
    ```

=== "Java"

    ```java
    ActionRunner.run(
        new MyAction1(),
        new MyAction2()
    );
    ```

### Updating the Queue

The `ActionRunner` needs to be updated on every loop of your op mode. 
This is done by calling `ActionRunner.update()` at the end of your `loop()` method.

=== "Kotlin"

    ```kotlin
    override fun loop() {
        // ... your other loop code
    
        ActionRunner.update()
    }
    ```

=== "Java"

    ```java
    @Override
    public void loop() {
        // ... your other loop code
    
        ActionRunner.update();
    }
    ```

The `update()` method will execute the actions in the queue and send telemetry to FTC Dashboard.

### Action Interruption

When a new action is added to the queue, 
the `ActionRunner` will check if any of the currently running actions 
share any requirements with the new action.
If they do, and the running action is `Interruptible`, its `onInterrupt()` method will be called,
and it will be removed from the queue.
This allows you to create complex, priority-based behaviors.

### Op Mode Lifecycle

The `ActionRunner` automatically manages the action queue across different op mode stages. 
It ensures that the queue is empty before `opModePreInit` and clears it after `opModePostStop`.