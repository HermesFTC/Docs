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
which is documented [here](traj-generation.md).

## 3. Action Improvements

Hermes includes multiple improvements to RoadRunner's Action framework.
Notably, it includes an `ActionRunner` object to manage asynchronous action queues,
and requirements and interruption features for `Action`s themselves.

The [Creating](actions/creating-actions.md) and [Using Actions](actions/using-actions.md)
guides explain many of these features.
