# Followers

Followers are objects that enable your robot to execute trajectories asynchronously. 
They provide the bridge between your planned trajectories and your robot's actual movement,
continuously calculating and sending control commands to your drive system.

## What is a Follower?

A `Follower` is an interface that manages trajectory execution by:
- Tracking the current target position along a trajectory
- Computing control commands to reach that target
- Managing timing and completion status
- Handling end conditions for when to stop following

All followers implement the core `Follower` interface, which provides:

- `trajectory: Trajectory<*>` - The trajectory to follow
- `drive: Drive` - The drive system to control
- `endConditions: Set<EndCondition>` - Conditions that determine when to stop
- `currentTarget: Pose2d` - The current target position on the trajectory
- `isDone: Boolean` - Whether the follower has finished execution
- `follow()` - The main method called each loop to update robot control

## Types of Followers

Hermes provides two main types of followers, each with different approaches to trajectory following:

### TimeFollower

`TimeFollower` uses a time-based approach to trajectory following. It starts an internal timer when following begins and directs the robot to the pose that should be reached at the current elapsed time.

**How it works:**
1. Starts a stopwatch when `follow()` is first called
2. Uses elapsed time to determine the target pose from the trajectory
3. Directs the feedback controller to minimize the error between current and target poses

**Best for:**
- Predictable, smooth motion
- When timing is critical
- Trajectories where you want consistent velocity profiles

**Example usage:**

=== "Kotlin"

    ```kotlin
    val trajectory = drive.trajectoryBuilder()
        .forward(24.0)
        .splineTo(Vector2d(24.0, 24.0), Math.toRadians(90.0))
        .buildToComposite()

    val follower = TimeFollower(trajectory, drive)
    ```

=== "Java"

    ```java
    Trajectory<Arclength> trajectory = drive.trajectoryBuilder()
        .forward(24.0)
        .splineTo(new Vector2d(24.0, 24.0), Math.toRadians(90.0))
        .buildToComposite();

    Follower follower = new TimeFollower(trajectory, drive);
    ```

### DisplacementFollower

`DisplacementFollower` uses a displacement-based approach. 
Instead of following a time schedule, 
it finds the closest point on the trajectory's path to the robot's current position and targets that location.

**How it works:**
1. Continuously calculates the robot's position relative to the trajectory path
2. Finds the closest point on the path to the current robot position
3. Directs the robot to that closest point, naturally progressing along the path

**Best for:**
- Robust following that can recover from disturbances
- When the robot might get pushed off course
- Situations where timing is less important than path accuracy

**Example usage:**

=== "Kotlin"

    ```kotlin
    val trajectory = drive.trajectoryBuilder()
        .forward(24.0)
        .splineTo(Vector2d(24.0, 24.0), Math.toRadians(90.0))
        .buildToComposite()

    val follower = DisplacementFollower(trajectory, drive)
    ```

=== "Java"

    ```java
    Trajectory<Arclength> trajectory = drive.trajectoryBuilder()
        .forward(24.0)
        .splineTo(new Vector2d(24.0, 24.0), Math.toRadians(90.0))
        .buildToComposite();

    Follower follower = new DisplacementFollower(trajectory, drive);
    ```

### Choosing Between TimeFollower and DisplacementFollower


**TimeFollower**

- **Advantages:**
    - **Tested**: This follower is based on the widely used RoadRunner 1.0 following algorithm
    - **Time consistency**: Maintains predictable timing throughout trajectory execution
    - **Automatic speed adjustment**: Speeds up if behind schedule, slows down if ahead
- **Disadvantages:**
    - **Speed limitations**: Cannot run at full speed due to timing constraints
    - **Poor disturbance handling**: Gets disrupted easily by other robots or obstacles
    - **Quick failure**: Gives up quickly when paths fail or encounter issues
    - **Rigid timing**: Slowing down when ahead of schedule can be suboptimal

**DisplacementFollower**

- **Advantages:**
    - **Full speed capability**: Can run at maximum speed when correctly implemented
    - **Robust to disturbances**: Handles other robots and obstacles much better
    - **Flexible starting**: Can deal with starting paths from incorrect positions
    - **Better design**: Generally more robust and well-designed architecture
- **Disadvantages:**
    - **Less used**: Newer implementation with less real-world testing

**Use TimeFollower when:**

- You need proven, reliable behavior
- Timing consistency is critical
- Operating in controlled environments
- You're new to trajectory following

**Use DisplacementFollower when:**

- Maximum speed is important
- You expect disturbances (other robots, field elements)
- You need robust recovery from path deviations
- You want the most advanced following algorithm

**Migration Path:**

If you're currently using TimeFollower and want better performance, 
consider testing DisplacementFollower in a controlled environment first. 
The developers of Hermes have found DisplacementFollower superior once properly tuned, 
but it has not been as widely tested in the competition environment.

Both followers can be used interchangeably in most situations, 
so you can experiment to see which works better for your specific robot and use case.

## Follower Parameters

`FollowerParams` allows you to configure how followers behave:

=== "Kotlin"

    ```kotlin
    data class FollowerParams(
        val profileParams: ProfileParams,     // Motion profile constraints
        val velConstraint: VelConstraint,     // Velocity limitations
        val accelConstraint: AccelConstraint  // Acceleration limitations
    )
    ```

=== "Java"

    ```java
    public class FollowerParams {
        public final ProfileParams profileParams;     // Motion profile constraints
        public final VelConstraint velConstraint;     // Velocity limitations
        public final AccelConstraint accelConstraint; // Acceleration limitations
        
        public FollowerParams(ProfileParams profileParams, 
                             VelConstraint velConstraint,
                             AccelConstraint accelConstraint) {
            this.profileParams = profileParams;
            this.velConstraint = velConstraint;
            this.accelConstraint = accelConstraint;
        }
    }
    ```

These parameters control:
- **Motion Profiling**: How velocities and accelerations are planned
- **Velocity Constraints**: Maximum speeds in different directions
- **Acceleration Constraints**: Maximum acceleration limits

## Using Followers in Practice

### Basic Usage Pattern

=== "Kotlin"

    ```kotlin
    class MyAutonomous : OpMode() {
        private lateinit var drive: MecanumDrive
        private lateinit var follower: Follower

        override fun init() {
            drive = MecanumDrive(hardwareMap, Pose2d(0.0, 0.0, 0.0))
            
            val trajectory = drive.trajectoryBuilder()
                .forward(24.0)
                .buildToComposite()
                
            follower = TimeFollower(trajectory, drive)
        }

        override fun loop() {
            if (!follower.isDone) {
                follower.follow()
            }
        }
    }
    ```

=== "Java"

    ```java
    public class MyAutonomous extends OpMode {
        private MecanumDrive drive;
        private Follower follower;

        @Override
        public void init() {
            drive = new MecanumDrive(hardwareMap, new Pose2d(0.0, 0.0, 0.0));
            
            Trajectory<Arclength> trajectory = drive.trajectoryBuilder()
                .forward(24.0)
                .buildToComposite();
                
            follower = new TimeFollower(trajectory, drive);
        }

        @Override
        public void loop() {
            if (!follower.isDone()) {
                follower.follow();
            }
        }
    }
    ```

### With Custom End Conditions

=== "Kotlin"

    ```kotlin
    val stopEarly = EndCondition { follower ->
        gamepad1.a // Stop if A button is pressed
    }

    val follower = DisplacementFollower(
        trajectory, 
        drive, 
        setOf(stopEarly)
    )
    ```

=== "Java"

    ```java
    EndCondition stopEarly = follower -> gamepad1.a; // Stop if A button is pressed

    Follower follower = new DisplacementFollower(
        trajectory, 
        drive, 
        Set.of(stopEarly)
    );
    ```

## Advanced Features

### Monitoring Progress

=== "Kotlin"

    ```kotlin
    // Check current target position
    val target = follower.currentTarget

    // Get the last command sent to the drive
    val command = follower.lastCommand

    // Check elapsed time
    val elapsedTime = follower.timer.seconds()
    ```

=== "Java"

    ```java
    // Check current target position
    Pose2d target = follower.getCurrentTarget();

    // Get the last command sent to the drive
    PoseVelocity2dDual<Time> command = follower.getLastCommand();

    // Check elapsed time
    double elapsedTime = follower.getTimer().seconds();
    ```

### Debugging and Visualization

Followers provide access to trajectory points for debugging:

=== "Kotlin"

    ```kotlin
    val trajectoryPoints = follower.points
    // Use these points for visualization or debugging
    ```

=== "Java"

    ```java
    List<Vector2d> trajectoryPoints = follower.getPoints();
    // Use these points for visualization or debugging
    ```

This makes it easy to visualize the planned path and understand what the follower is trying to achieve.

## End Conditions

End conditions determine when a follower should stop executing. 
They provide flexible control over when trajectory following completes.

For detailed information about end conditions, 
including built-in factory methods and custom implementations, 
see the [End Conditions](end-conditions.md) page.

## Custom Followers

For advanced use cases requiring specialized trajectory following behavior, 
you can create custom followers by implementing the `Follower` interface.

See the [Custom Followers](custom-followers.md) page for detailed examples and implementation guidance.
