# Localizers

Hermes supports many localization systems! 

This guide will not attempt to suggest which localizer you use;
there are many resources available for that.

## What is a localizer?

A localizer is an object that provides the robot's position,
orientation, and velocity in the field.

The most basic localizer is the drive localizer,
which uses the encoders on your drive motors
to determine the robot's position and orientation.
However, many robots use additional sensors,
such as deadwheel encoders, IMUs, or optical odometry,
to improve localization accuracy.

The `Localizer` interface in Hermes
provides a common API for all localizers,
and has two properties and one method:

- `pose: Pose2d` (mutable): the robot's position and orientation in the field
- `vel: PoseVelocity2d` (read-only): the robot's velocity in the field
- `update(): PoseVelocity2d`: updates the localizer's pose and returns the robot's velocity

For Java users, the `pose` property is translated to
`Pose2d getPose()` and `void setPose(Pose2d pose)` methods,
and the `vel` property is translated to the `PoseVelocity2d getVel()` method.

## Built-in Localizers

Hermes automatically detects which localizer your robot is using based on your hardware configuration!

### Drive Localizers

**MecanumDriveLocalizer** and **TankLocalizer** use the encoders on your drive motors
to determine the robot's position and orientation.

### Two-Deadwheel Localizer

**TwoDeadWheelLocalizer** is for robots with two deadwheel pods
(one parallel and one perpendicular to the robot's forward direction).
Do not use this if you have the Pinpoint localization system.

### Three-Deadwheel Localizer

**ThreeDeadWheelLocalizer** is for robots with three deadwheel pods
(two parallel and one perpendicular to the robot's forward direction).

### Pinpoint Localizer

**PinpointLocalizer** is for the goBILDA Pinpoint localization system 
with two deadwheel pods.

### OTOS Localizer

**OTOSLocalizer** is for the SparkFun Optical Odometry (OTOS) system.

## Custom Localizers

If you want to create a custom localizer,
you can implement the `Localizer` interface 
and provide your own implementation of the properties and method
described above:

=== "Kotlin"

    ```kotlin
    class CustomLocalizer : Localizer {
        override var pose: Pose2d = Pose2d.zero

        override var vel = PoseVelocity2d.zero
            private set

        override fun update(): PoseVelocity2d {
            pose = // calculate new pose
            vel = // calculate velocity
            return vel
        }
    }
    ```

    Note that we create both `pose` and `vel` as mutable properties,
    despite being read-only in the interface. 
    This means that we can update both the `pose` and `vel` properties
    internally in the `update` method,
    but only `pose` can be set externally.

=== "Java"

    ```java
    public class CustomLocalizer implements Localizer {
        private Pose2d pose = Pose2d.zero();
        private PoseVelocity2d vel = PoseVelocity2d.zero();

        @Override
        public Pose2d getPose() {
            return pose;
        }

        @Override
        public void setPose(Pose2d pose) {
            this.pose = pose;
        }

        @Override
        public PoseVelocity2d getVel() {
            return vel;
        }

        @Override
        public PoseVelocity2d update() {
            pose = // calculate new pose
            vel = // calculate velocity
            return vel;
        }
    }
    ```