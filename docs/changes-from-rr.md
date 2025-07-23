# Changes From RoadRunner

Hermes has a few major differences from RoadRunner 1.0.

## 1. Package Layout

RoadRunner 1.0 places all classes in the `com.acmerobotics.roadrunner` package.
Hermes places classes in subpackages of `com.acmerobotics.roadrunner`,
such as `com.acmerobotics.roadrunner.paths`. 
This is similar to, but not the same as, the RoadRunner 0.5 package layout.

Unfortunately, due to this change, Hermes is not backwards compatible
with RoadRunner 1.0 (though migration is very easy!).
However, IntelliJ IDEA and Android Studio will automatically import classes,
so you do not need to worry about the package locations.

## 2. Trajectory Creation

The [`Drive` interface](https://docs.hermes.zharel.gay/ftc/com.acmerobotics.roadrunner.ftc/-drive/index.html)
has a `trajectoryBuilder` method, allowing you to create trajectories without `Action`s.
The API of the [`TrajectoryBuilder` class](https://docs.hermes.zharel.gay/core/com.acmerobotics.roadrunner.trajectories/-trajectory-builder/index.html)
is very similar to that of `TrajectoryActionBuilder`, 
but its `build` method returns a `List<Trajectory>` instead of an `Action`.
Alternatively, the `buildToComposite` method returns a `CompositeTrajectory` object,
which may be easier to use.

We also offer methods to create and profile paths without a builder, 
which is documented [here](concepts/traj-generation.md).

## 3. Localizers

In RoadRunner 1.0, localizers were part of the QuickStart.
Hermes includes localizers directly in the library, 
reducing the need for teams to copy and maintain localizer code from the QuickStart.
This eliminates the need for teams to copy and maintain localizer code from the QuickStart.

The API for localizers has also changed slightly,
with the `PARAMS` object being replaced by a fluent API for setting parameters.

The library provides several localizer implementations:
- **Two-wheel odometry** - For teams using two tracking wheels
- **Three-wheel odometry** - For teams using three tracking wheels  
- **Drive encoder localization** - Using drive motor encoders
- **Pinpoint localization** - For teams using the Pinpoint localization system

See the [Localizers](concepts/localizers.md) page for detailed information.

## 4. Trajectory Following

Hermes introduces a new **Follower** system that separates trajectory following from trajectory creation.
RoadRunner 1.0 combines these concepts within Actions, 
while Hermes provides dedicated follower objects for asynchronous trajectory execution.

Key improvements:
- **Asynchronous following**: Execute trajectories independently of other code
- **Multiple follower types**: `TimeFollower` (time-based) and `DisplacementFollower` (path-based)
- **Flexible end conditions**: Customizable conditions for when following should stop
- **Better disturbance handling**: DisplacementFollower can recover from path deviations

See the [Followers](concepts/followers.md) page for detailed information.

## 5. Action Improvements

Hermes includes multiple improvements to RoadRunner's Action framework.
Notably, it includes an `ActionRunner` object to manage asynchronous action queues,
and requirements and interruption features for `Action`s themselves.

The [Creating](actions/creating-actions.md) and [Using Actions](actions/using-actions.md)
guides explain many of these features.
