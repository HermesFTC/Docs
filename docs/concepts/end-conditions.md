# End Conditions

End conditions determine when a follower should stop executing. 
The `EndCondition` interface allows you to define custom logic 
for when trajectory following should complete.

## What are End Conditions?

End conditions are functions that return `true` when 
a follower should stop following a trajectory. 
They provide a flexible way to control when trajectory execution completes, 
beyond just reaching the end of the planned path.

Every follower accepts a set of end conditions that are checked on each loop iteration. 
If any end condition returns `true`, 
the follower will stop and mark itself as done.

## Default End Conditions

By default, followers use `EndCondition.default`, which includes:
- **Overtime condition**: Stops if the trajectory runs 2 seconds longer than its planned duration
- **Distance from end**: Stops when the robot is within 2 inches of the trajectory's end position

## Custom End Conditions

You can create custom end conditions by implementing the `EndCondition` interface:

=== "Kotlin"

    ```kotlin
    val customEndCondition = EndCondition { follower ->
        // Custom logic here
        follower.timer.seconds() > 5.0 || someOtherCondition()
    }

    val follower = TimeFollower(trajectory, drive, setOf(customEndCondition))
    ```

=== "Java"

    ```java
    EndCondition customEndCondition = follower -> {
        // Custom logic here
        return follower.getTimer().seconds() > 5.0 || someOtherCondition();
    };

    Follower follower = new TimeFollower(trajectory, drive, Set.of(customEndCondition));
    ```

## Built-in End Condition Factory Methods

The `EndCondition` companion object provides several factory methods for common scenarios:

### Time-based Conditions

=== "Kotlin"

    ```kotlin
    // Stop if trajectory runs longer than planned duration + timeout
    EndCondition.overTime(duration: Duration)
    EndCondition.overTime(duration: java.time.Duration) 
    EndCondition.overTime(timeoutSeconds: Double)

    // Examples:
    val timeoutCondition = EndCondition.overTime(3.0) // 3 second timeout
    val kotlinDurationTimeout = EndCondition.overTime(5.seconds)
    ```

=== "Java"

    ```java
    // Stop if trajectory runs longer than planned duration + timeout
    EndCondition.overTime(Duration duration)
    EndCondition.overTime(java.time.Duration duration) 
    EndCondition.overTime(double timeoutSeconds)

    // Examples:
    EndCondition timeoutCondition = EndCondition.overTime(3.0); // 3 second timeout
    EndCondition javaDurationTimeout = EndCondition.overTime(Duration.ofSeconds(5));
    ```

### Position-based Conditions

=== "Kotlin"

    ```kotlin
    // Stop when robot is within specified distance of trajectory end
    EndCondition.dispFromEnd(distance: Double)

    // Example:
    val nearEndCondition = EndCondition.dispFromEnd(1.5) // Within 1.5 inches
    ```

=== "Java"

    ```java
    // Stop when robot is within specified distance of trajectory end
    EndCondition.dispFromEnd(double distance)

    // Example:
    EndCondition nearEndCondition = EndCondition.dispFromEnd(1.5); // Within 1.5 inches
    ```

### Velocity-based Conditions

=== "Kotlin"

    ```kotlin
    // Stop when robot velocity drops below threshold
    EndCondition.robotVel(maxVelocity: Double)

    // Example:
    val slowDownCondition = EndCondition.robotVel(2.0) // Below 2 in/s
    ```

=== "Java"

    ```java
    // Stop when robot velocity drops below threshold
    EndCondition.robotVel(double maxVelocity)

    // Example:
    EndCondition slowDownCondition = EndCondition.robotVel(2.0); // Below 2 in/s
    ```

## Combining Multiple Conditions

You can combine multiple end conditions to create sophisticated stopping logic:

=== "Kotlin"

    ```kotlin
    val customEndConditions = setOf(
        EndCondition.overTime(4.0),        // 4 second timeout
        EndCondition.dispFromEnd(1.0),     // Within 1 inch of end
        EndCondition.robotVel(1.5),        // Robot moving slower than 1.5 in/s
        EndCondition { gamepad1.a }        // Manual stop with A button
    )

    val follower = TimeFollower(trajectory, drive, customEndConditions)
    ```

=== "Java"

    ```java
    Set<EndCondition> customEndConditions = Set.of(
        EndCondition.overTime(4.0),        // 4 second timeout
        EndCondition.dispFromEnd(1.0),     // Within 1 inch of end
        EndCondition.robotVel(1.5),        // Robot moving slower than 1.5 in/s
        follower -> gamepad1.a             // Manual stop with A button
    );

    Follower follower = new TimeFollower(trajectory, drive, customEndConditions);
    ```

## Common Use Cases

### Emergency Stop

Create an end condition for emergency stops 
(note that gamepad input is not legal in the autonomous period of FTC matches):

=== "Kotlin"

    ```kotlin
    val emergencyStop = EndCondition { follower ->
        gamepad1.back || gamepad2.back // Either controller back button
    }
    ```

=== "Java"

    ```java
    EndCondition emergencyStop = follower -> gamepad1.back || gamepad2.back;
    ```

### Time Limits for Autonomous

Set strict time limits for autonomous periods:

=== "Kotlin"

    ```kotlin
    val autonomousTimeLimit = EndCondition { follower ->
        follower.timer.seconds() > 25.0 // Stop 5 seconds before autonomous ends
    }
    ```

=== "Java"

    ```java
    EndCondition autonomousTimeLimit = follower -> 
        follower.getTimer().seconds() > 25.0; // Stop 5 seconds before autonomous ends
    ```

### Position Tolerance

Create custom position tolerances:

=== "Kotlin"

    ```kotlin
    val tightTolerance = EndCondition { follower ->
        val target = follower.currentTarget
        val current = follower.drive.localizer.pose
        val error = target - current
        error.position.norm() < 0.5 && abs(error.heading.log()) < Math.toRadians(2.0)
    }
    ```

=== "Java"

    ```java
    EndCondition tightTolerance = follower -> {
        Pose2d target = follower.getCurrentTarget();
        Pose2d current = follower.getDrive().getLocalizer().getPose();
        Pose2d error = target.minus(current);
        return error.position.norm() < 0.5 && 
               Math.abs(error.heading.log()) < Math.toRadians(2.0);
    };
    ```

## Best Practices

### Always Include a Timeout

Never rely solely on position or velocity conditions - 
always include a timeout to prevent infinite loops:

=== "Kotlin"

    ```kotlin
    val safeConditions = setOf(
        EndCondition.dispFromEnd(1.0),     // Primary condition
        EndCondition.overTime(5.0)         // Safety timeout
    )
    ```

=== "Java"

    ```java
    Set<EndCondition> safeConditions = Set.of(
        EndCondition.dispFromEnd(1.0),     // Primary condition
        EndCondition.overTime(5.0)         // Safety timeout
    );
    ```

### Test End Conditions

Test your end conditions thoroughly, especially custom ones:

=== "Kotlin"

    ```kotlin
    // Test the condition logic separately
    fun testEndCondition() {
        val condition = EndCondition.robotVel(2.0)
        // Simulate follower state and verify condition behavior
    }
    ```

=== "Java"

    ```java
    // Test the condition logic separately
    public void testEndCondition() {
        EndCondition condition = EndCondition.robotVel(2.0);
        // Simulate follower state and verify condition behavior
    }
    ```

### Use Appropriate Tolerances

Choose tolerances that match your robot's capabilities and requirements:

- **Precise positioning**: Use smaller distance tolerances (0.5-1.0 inches)
- **General movement**: Use larger distance tolerances (1.0-3.0 inches)  
- **Fast movements**: Use velocity-based conditions to detect when movement stops
- **Critical timing**: Use strict time limits with appropriate buffers

End conditions are a powerful tool for creating robust and reliable autonomous routines. 
By combining different types of conditions, 
you can ensure your robot behaves predictably in various scenarios.
