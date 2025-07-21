# Localizers

Hermes supports many localization systems! 
Before tuning your robot, you must create a localizer object. 

This guide will not attempt to suggest which localizer you use;
there are many resources available for that.

## Drive Localizers

If you are using drive encoders,
you do not need to make any changes to your
`MecanumDrive` or `TankDrive` class,
as `MecanumDriveLocalizer` and `TankLocalizer`
are the defaults.

## Two-Deadwheel Localizer

Use this Localizer if your robot has two deadwheel pods,
and you are NOT using the Pinpoint localization system.

To create a `TwoDeadWheelLocalizer`, 
replace the `localizer = new MecanumDriveLocalizer ...` or 
`localizer = new TankLocalizer ...` with:

```java 
localizer = new TwoDeadWheelLocalizer(
                hardwareMap, lazyImu.get(), PARAMS.inPerTick,
                "frontLeft", "frontRight"
        ).withInitialPose(pose);
```

Replace `"frontLeft"` with the name of the motor your 
parallel pod is plugged into,
and `"frontRight"` with the name of the motor your
perpendicular pod is plugged into.

To reverse the directions of either encoder,
chain `.withDirections(<PARALLEL DIRECTION>, <PERPENDICULAR DIRECTION>)`
after the call to `withInitialPose`, before the ending semicolon.
Replace the placeholders with `DcMotorSimple.Direction.FORWARD` or
`DcMotorSimple.Direction.REVERSE`.

Finally, after you complete AngularRampLogger and
determine your wheel positions,
chain `.withLocations(<PARALLEL LOCATION>, <PERPENDICULAR LOCATION>)`
after the call to `withDirections`, before the semicolon.
Replace the placeholders with your tuned values.

## Three-Deadwheel Localizer

Use this Localizer if your robot has three deadwheel pods.

To create a `ThreeDeadWheelLocalizer`, 
replace the `localizer = new MecanumDriveLocalizer ...`
or `localizer = new TankLocalizer ...` with:

```java
localizer = new ThreeDeadWheelLocalizer(
                hardwareMap, PARAMS.inPerTick, 
                "frontLeft", "frontRight", "backLeft"
        ).withInitialPose(pose);
```

Replace `"frontLeft"` and `"frontRight"` with the names of the motors
your parallel pods are plugged into,
and `"backLeft"` with the name of the motor your perpendicular pod is plugged into.

To reverse the directions of either encoder,
chain `.withDirections(<PAR0 DIRECTION>, <PAR1 DIRECTION>, <PERPENDICULAR DIRECTION>)`
after the call to `withInitialPose`, before the ending semicolon.
Replace the placeholders with `DcMotorSimple.Direction.FORWARD` or
`DcMotorSimple.Direction.REVERSE`.

Finally, after you complete AngularRampLogger and
determine your wheel positions,
chain `.withLocations(<PAR0 LOCATION>, <PAR1 LOCATION>, <PERPENDICULAR LOCATION>)`
after the call to `withDirections`, before the semicolon.
Replace the placeholders with your tuned values.

## Pinpoint Localizer

Use this Localizer if you are using the goBILDA Pinpoint localization system 
with two deadwheel pods.

To create a `PinpointLocalizer`, 
replace the `localizer = new MecanumDriveLocalizer ...`
or `localizer = new TankLocalizer ...` with:

```java
localizer = new PinpointLocalizer(hardwareMap, PARAMS.inPerTick)
                .withInitialPose(pose);
```

This assumes your Pinpoint device is named `"pinpoint"` 
in your configuration. If it is not,
chain `.withName(<NAME>)` after `.withInitialPose` 
before the semicolon.

To reverse the directions of either encoder,
chain `.withDirections(<PARALLEL DIRECTION>, <PERPENDICULAR DIRECTION>)`
after the call to `withInitialPose`, before the ending semicolon.
Replace the placeholders with `DcMotorSimple.Direction.FORWARD` or
`DcMotorSimple.Direction.REVERSE`.

Finally, after you complete AngularRampLogger and
determine your wheel positions,
chain `.withOffsets(<PARALLEL LOCATION>, <PERPENDICULAR LOCATION>)`
after the call to `withDirections`, before the semicolon.
Replace the placeholders with your tuned values.

## OTOS Localizer

Use this Localizer if you are using the SparkFun Optical Odometry (OTOS) system.

To create an `OTOSLocalizer`,
replace the `localizer = new MecanumDriveLocalizer ...`
or `localizer = new TankLocalizer ...` with:

```java
localizer = new OTOSLocalizer(hardwareMap)
                .withInitialPose(pose);
```

This assumes your OTOS device is named `"sensor_otos"`
in your configuration. If it is not,
chain `.withName(<NAME>)` after `.withInitialPose`
before the semicolon.

After you complete the scalar tuning OpModes,
chain `.withScalars(<LINEAR>, <ANGULAR>)`,
after `.withName` before the semicolon,
replacing the placeholders with your tuned values.

The angular scalar will be 0.0 after you tune 
the linear scalar but before you tune the angular scalar.

After you complete the offset tuners,
chain `.withOffset(<X>, <Y>, <HEADING>)`
after the call to `.withScalars`,
before the semicolon.
Replace the placeholders with your tuned values.

The x and y offsets will be 0.0 after you tune the heading offset,
but before you tune the position offset.