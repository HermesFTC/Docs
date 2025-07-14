# Follower Example

If you are writing more complicated autonomous routines, 
you can use a `Follower` to follow a path asynchronously.
This can, for example, be used with a finite state machine.

The QuickStart comes with two types of followers:
1. `TimeFollower`, which essentially starts a stopwatch and directs the feedback controller to close
    the gap between the estimated pose and the desired pose at the current time from the trajectory.
2. `DisplacementFollower`, which finds the closest point on the trajectory's path to your robot,
    and directs the feedback controller to close the gap based on that instead.

Both types of followers have their benefits, 
which will eventually be discussed on a page of this guide.

The following (haha) example uses `TimeFollower`, 
but it can be switched out for `DisplacementFollower` with no other changes to the `OpMode`.

[All of the `TrajectoryBuilder` functions are documented here](http://rr.zharel.me/core/com.acmerobotics.roadrunner.trajectories/-trajectory-builder/index.html).

=== "Kotlin"

    ```kotlin 
    @Autonomous
    @Disabled
    class FollowerExampleKt : OpMode() {
        private lateinit var drive: MecanumDrive
        private lateinit var follower: Follower

        override fun init() {
            drive = MecanumDrive(hardwareMap, Pose2d(0.0, 0.0, 0.0))
            val traj: Trajectory<Arclength> = drive.trajectoryBuilder()
                .forward(10.0)
                .splineTo(Vector2d(10.0, 10.0), Math.toRadians(90.0))
                .buildToComposite()

            follower = TimeFollower(traj, drive)
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
    @Autonomous
    @Disabled
    public class FollowerExample extends OpMode {
        private MecanumDrive drive;
        private Follower follower;

        @Override
        public void init() {
            drive = new MecanumDrive(hardwareMap, new Pose2d(0.0, 0.0, 0.0));
            Trajectory<Arclength> traj = drive.trajectoryBuilder()
                    .forward(10.0)
                    .splineTo(new Vector2d(10.0, 10.0), Math.toRadians(90.0))
                    .buildToComposite();

            follower = new TimeFollower(traj, drive);
        }

        @Override
        public void loop() {
            if (!follower.isDone()) {
                follower.follow();
            }
        }
    }
    ```

