# Custom Followers

While Hermes provides `TimeFollower` and `DisplacementFollower` for most use cases, 
you can create custom followers by implementing the `Follower` interface. 
This allows you to create specialized following behavior for unique requirements.

## Follower Interface

The `Follower` interface defines the contract that all followers must implement:

=== "Kotlin"

    ```kotlin
    interface Follower {
        val trajectory: Trajectory<*>
        val drive: Drive
        val endConditions: Set<EndCondition>

        val currentTarget: Pose2d
        val lastCommand: PoseVelocity2dDual<Time>
        val timer: ElapsedTime

        val isDone: Boolean

        fun follow()

        val points: List<Vector2d> get() = listOf(Vector2d.zero)
    }
    ```

=== "Java"

    ```java
    public interface Follower {
        Trajectory<?> getTrajectory();
        Drive getDrive();
        Set<EndCondition> getEndConditions();

        Pose2d getCurrentTarget();
        PoseVelocity2dDual<Time> getLastCommand();
        ElapsedTime getTimer();

        boolean isDone();

        void follow();

        default List<Vector2d> getPoints() {
            return List.of(Vector2d.zero);
        }
    }
    ```

## Custom Follower Examples

### Preview Follower

A follower that doesn't actually move the robot but logs the planned path:

=== "Kotlin"

    ```kotlin
    class PreviewFollower(
        override val trajectory: Trajectory<*>,
        override val drive: Drive,
        override val endConditions: Set<EndCondition> = EndCondition.default
    ) : Follower {
        override var currentTarget: Pose2d = trajectory[0.0].value()
            private set
        override var lastCommand: PoseVelocity2dDual<Time> = PoseVelocity2dDual.zero()
            private set
        override val timer: ElapsedTime = ElapsedTime()
        override var isDone: Boolean = false
            private set
        
        private var started = false
        private var targetIndex = 0.0

        override fun follow() {
            if (!started) {
                timer.reset()
                started = true
            }

            // Calculate current target based on time
            val dt = timer.seconds()
            val target = if (trajectory is TimeTrajectory) {
                trajectory[dt]
            } else {
                // For displacement trajectories, increment slowly
                targetIndex = minOf(targetIndex + 0.1, trajectory.length())
                trajectory[targetIndex]
            }

            currentTarget = target.value()
            
            // Don't actually move the robot, just log the path
            println("Would move to: ${currentTarget.position}, heading: ${currentTarget.heading}")
            
            // Check end conditions
            isDone = endConditions.any { it.shouldEnd(this) }
        }

        override val points: List<Vector2d> = baseFollower.points
    }
    ```

=== "Java"

    ```java
    public class PreviewFollower implements Follower {
        private final Trajectory<?> trajectory;
        private final Drive drive;
        private final Set<EndCondition> endConditions;
        
        private Pose2d currentTarget;
        private PoseVelocity2dDual<Time> lastCommand = PoseVelocity2dDual.zero();
        private final ElapsedTime timer = new ElapsedTime();
        private boolean isDone = false;
        
        private boolean started = false;
        private double targetIndex = 0.0;

        public PreviewFollower(Trajectory<?> trajectory, Drive drive, Set<EndCondition> endConditions) {
            this.trajectory = trajectory;
            this.drive = drive;
            this.endConditions = endConditions;
            this.currentTarget = trajectory.get(0.0).value();
        }

        public PreviewFollower(Trajectory<?> trajectory, Drive drive) {
            this(trajectory, drive, EndCondition.getDefault());
        }
        
        @Override
        public void follow() {
            if (!started) {
                timer.reset();
                started = true;
            }

            // Calculate current target based on time
            double dt = timer.seconds();
            Pose2dDual<Time> target;
            
            if (trajectory instanceof TimeTrajectory) {
                target = trajectory.get(dt);
            } else {
                // For displacement trajectories, increment slowly
                targetIndex = Math.min(targetIndex + 0.1, trajectory.length());
                target = trajectory.get(targetIndex);
            }

            currentTarget = target.value();
            
            // Don't actually move the robot, just log the path
            System.out.println("Would move to: " + currentTarget.position + 
                             ", heading: " + currentTarget.heading);
            
            // Check end conditions
            isDone = endConditions.stream().anyMatch(condition -> condition.shouldEnd(this));
        }

        // Implement remaining interface methods...
        @Override public Trajectory<?> getTrajectory() { return trajectory; }
        @Override public Drive getDrive() { return drive; }
        @Override public Set<EndCondition> getEndConditions() { return endConditions; }
        @Override public Pose2d getCurrentTarget() { return currentTarget; }
        @Override public PoseVelocity2dDual<Time> getLastCommand() { return lastCommand; }
        @Override public ElapsedTime getTimer() { return timer; }
        @Override public boolean isDone() { return isDone; }
        @Override public List<Vector2d> getPoints() { return baseFollower.getPoints(); }
    }
    ```

### Slow-Motion Follower

A follower that executes trajectories at a reduced speed:

=== "Kotlin"

    ```kotlin
    class SlowMotionFollower(
        override val trajectory: Trajectory<*>,
        override val drive: Drive,
        private val speedMultiplier: Double = 0.5,
        override val endConditions: Set<EndCondition> = EndCondition.default
    ) : Follower {
        
        private val baseFollower = TimeFollower(trajectory, drive, emptySet())
        
        override var currentTarget: Pose2d = trajectory[0.0].value()
            private set
        override var lastCommand: PoseVelocity2dDual<Time> = PoseVelocity2dDual.zero()
            private set
        override val timer: ElapsedTime = ElapsedTime()
        override var isDone: Boolean = false
            private set

        override fun follow() {
            // Let the base follower do the work
            baseFollower.follow()
            
            // Scale down the command
            val baseCommand = baseFollower.lastCommand
            lastCommand = PoseVelocity2dDual(
                baseCommand.position,
                baseCommand.velocity.times(speedMultiplier),
                baseCommand.acceleration.times(speedMultiplier * speedMultiplier)
            )
            
            // Update our state
            currentTarget = baseFollower.currentTarget
            isDone = endConditions.any { it.shouldEnd(this) }
            
            // Send scaled command to drive
            drive.setDrivePowersWithFF(lastCommand)
        }

        override val points: List<Vector2d> = baseFollower.points
    }
    ```

=== "Java"

    ```java
    public class SlowMotionFollower implements Follower {
        private final Trajectory<?> trajectory;
        private final Drive drive;
        private final Set<EndCondition> endConditions;
        private final double speedMultiplier;
        
        private final TimeFollower baseFollower;
        
        private Pose2d currentTarget;
        private PoseVelocity2dDual<Time> lastCommand = PoseVelocity2dDual.zero();
        private final ElapsedTime timer = new ElapsedTime();
        private boolean isDone = false;

        public SlowMotionFollower(Trajectory<?> trajectory, Drive drive, 
                                 double speedMultiplier, Set<EndCondition> endConditions) {
            this.trajectory = trajectory;
            this.drive = drive;
            this.speedMultiplier = speedMultiplier;
            this.endConditions = endConditions;
            this.baseFollower = new TimeFollower(trajectory, drive, Set.of());
            this.currentTarget = trajectory.get(0.0).value();
        }

        public SlowMotionFollower(Trajectory<?> trajectory, Drive drive, double speedMultiplier) {
            this(trajectory, drive, speedMultiplier, EndCondition.getDefault());
        }
        
        @Override
        public void follow() {
            // Let the base follower do the work
            baseFollower.follow();
            
            // Scale down the command
            PoseVelocity2dDual<Time> baseCommand = baseFollower.getLastCommand();
            lastCommand = new PoseVelocity2dDual<>(
                baseCommand.position,
                baseCommand.velocity.times(speedMultiplier),
                baseCommand.acceleration.times(speedMultiplier * speedMultiplier)
            );
            
            // Update our state
            currentTarget = baseFollower.getCurrentTarget();
            isDone = endConditions.stream().anyMatch(condition -> condition.shouldEnd(this));
            
            // Send scaled command to drive
            drive.setDrivePowersWithFF(lastCommand);
        }

        // Implement remaining interface methods...
        @Override public Trajectory<?> getTrajectory() { return trajectory; }
        @Override public Drive getDrive() { return drive; }
        @Override public Set<EndCondition> getEndConditions() { return endConditions; }
        @Override public Pose2d getCurrentTarget() { return currentTarget; }
        @Override public PoseVelocity2dDual<Time> getLastCommand() { return lastCommand; }
        @Override public ElapsedTime getTimer() { return timer; }
        @Override public boolean isDone() { return isDone; }
        @Override public List<Vector2d> getPoints() { return baseFollower.getPoints(); }
    }
    ```

### Logging Follower

A follower that wraps another follower and adds detailed logging:

=== "Kotlin"

    ```kotlin
    class LoggingFollower(
        private val baseFollower: Follower,
        private val logPrefix: String = "Follower"
    ) : Follower by baseFollower {
        
        private var stepCount = 0

        override fun follow() {
            val startTime = System.nanoTime().nanoseconds
            
            val prevTarget = currentTarget
            val prevDone = isDone
            
            baseFollower.follow()
            
            val endTime = System.nanoTime().nanoseconds
            val executionTime = (endTime - startTime).toDouble(DurationUnit.MILLISECONDS) // Convert to milliseconds
            
            stepCount++
            
            println("$logPrefix Step $stepCount:")
            println("  Execution time: ${executionTime}")
            println("  Target: ${currentTarget}")
            println("  Command: ${lastCommand}")
            println("  Done: $isDone")
            
            if (isDone && !prevDone) {
                println("$logPrefix: Trajectory following completed!")
            }
        }
    }
    ```

=== "Java"

    ```java
    public class LoggingFollower implements Follower {
        private final Follower baseFollower;
        private final String logPrefix;
        private int stepCount = 0;

        public LoggingFollower(Follower baseFollower, String logPrefix) {
            this.baseFollower = baseFollower;
            this.logPrefix = logPrefix;
        }

        public LoggingFollower(Follower baseFollower) {
            this(baseFollower, "Follower");
        }

        @Override
        public void follow() {
            long startTime = System.nanoTime();
            
            Pose2d prevTarget = getCurrentTarget();
            boolean prevDone = isDone();
            
            baseFollower.follow();
            
            long endTime = System.nanoTime();
            double executionTime = (endTime - startTime) / 1_000_000.0; // Convert to milliseconds
            
            stepCount++;
            
            System.out.println(logPrefix + " Step " + stepCount + ":");
            System.out.println("  Execution time: " + executionTime + "ms");
            System.out.println("  Target: " + getCurrentTarget());
            System.out.println("  Command: " + getLastCommand());
            System.out.println("  Done: " + isDone());
            
            if (isDone() && !prevDone) {
                System.out.println(logPrefix + ": Trajectory following completed!");
            }
        }

        // Delegate all other methods to the base follower
        @Override public Trajectory<?> getTrajectory() { return baseFollower.getTrajectory(); }
        @Override public Drive getDrive() { return baseFollower.getDrive(); }
        @Override public Set<EndCondition> getEndConditions() { return baseFollower.getEndConditions(); }
        @Override public Pose2d getCurrentTarget() { return baseFollower.getCurrentTarget(); }
        @Override public PoseVelocity2dDual<Time> getLastCommand() { return baseFollower.getLastCommand(); }
        @Override public ElapsedTime getTimer() { return baseFollower.getTimer(); }
        @Override public boolean isDone() { return baseFollower.isDone(); }
        @Override public List<Vector2d> getPoints() { return baseFollower.getPoints(); }
    }
    ```

#### Understanding Delegation in LoggingFollower

The Kotlin version of `LoggingFollower` uses [interface delegation](https://kotlinlang.org/docs/delegation.html)
 (`Follower by baseFollower`) to automatically implement all interface methods 
 by forwarding them to the `baseFollower` object. 
 This eliminates the boilerplate code required in Java where 
 you must manually delegate every single interface method. 
You can still override specific methods (like `follow()`) 
to customize behavior while keeping automatic delegation for everything else.

## Best Practices

1. **Respect end conditions**: Always check and honor the end conditions
2. **Update state consistently**: Keep all interface properties up to date
3. **Handle edge cases**: Consider trajectory start/end and error conditions
4. **Provide debugging tools**: Include telemetry and visualization support
5. **Document behavior**: Clearly document what your custom follower does differently
6. **Test thoroughly**: Custom followers can have subtle bugs that are hard to debug
7. **Consider composition**: Often better to wrap existing followers than rewrite from scratch

Custom followers provide flexibility to adapt the following behavior to your specific needs, 
whether it's for advanced control, logging, or unique trajectory handling. 
