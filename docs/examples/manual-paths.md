# Manually Making Paths Example

As discussed [earlier](../concepts/traj-generation.md),
making path and trajectory objects yourself 
offers more customization compared to the 
`trajectoryBuilder` API.

!!! CAUTION
    When creating paths in this way,
    make sure that the start and end positions and headings line up.
    Otherwise, the expected starting positions of each trajectory
    will not match where your robot actually is (unless your robot can teleport),
    which will confuse the followers.

We do not recommend this for new programmers.

The following example demonstrates creating `PositionPath`s,
turning them into `PosePath`s, adding the `PosePath`s,
and finally turning them into a followable `Trajectory`.

=== "Kotlin"

    ```kotlin 
    @Autonomous
    @Disabled
    class PathObjectsExampleKt : OpMode() {
        private lateinit var drive: MecanumDrive
        private lateinit var follower: Follower

        override fun init() {
            drive = MecanumDrive(hardwareMap, Pose2d(0.0, 0.0, 0.0))

            val path1: PosePath = Line(
                Vector2d(0.0, 0.0),
                Vector2d(10.0, 10.0)
            ).withTangentHeading()

            val path2: PosePath = fromPoints(
                Vector2d(10.0, 10.0),
                Vector2d(20.0, 20.0),
                Vector2d(10.0, 10.0)
            ).withLinearHeading(Math.PI / 4, 0.0)

            val trajectory = drive.createTrajectory(path1 + path2) 
            //the + operator was overloaded for the PosePath interface,
            //allowing us to make a CompositePosePath!

            follower = DisplacementFollower(trajectory, drive)
        }

        // the loop is the same as the TimeFollower example
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
    public class PathObjectsExample extends OpMode {
        private MecanumDrive drive;
        private Follower follower;

        @Override
        public void init() {
            drive = new MecanumDrive(hardwareMap, new Pose2d(0.0, 0.0, 0.0));

            PosePath path1 = new Line(
                    new Vector2d(0.0, 0.0),
                    new Vector2d(10.0, 10.0)
            ).withTangentHeading();

            PosePath path2 = BezierCurves.fromPoints(
                    new Vector2d(10.0, 10.0),
                    new Vector2d(20.0, 20.0),
                    new Vector2d(10.0, 10.0)
            ).withLinearHeading(Math.PI/4, 0.0);

            DisplacementTrajectory trajectory = drive.createTrajectory(path1.plus(path2));
            //the plus function for PosePaths returns a CompositePosePath

            follower = new DisplacementFollower(trajectory, drive);
        }

        //the loop is the same as the TimeFollower example
        @Override
        public void loop() {
            if (!follower.isDone()) {
                follower.follow();
            }
        }
    }
    ```
