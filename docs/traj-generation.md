# Path and Trajectory Generation

While the `TrajectoryBuilder` API is great for making continuous and connected paths, 
it does not allow as much customization as manually creating the Path
and `Trajectory` objects.

Hermes actually has two types of path: `PosePath` and `PositionPath<Param>`
(the `Param` type parameter will be explained in a different guide).
The main difference is that `PositionPath`s are only for, well,
positions, while `PosePath`s also include headings. 
An example of a `PositionPath` is a `Line`, 
which simply connects two `Vector2d`s.

## Types of `PositionPath`

The primary types of `PositionPath` are
1. `Line`s, which are straight lines that connect `Vector2d`s,
2. Quintic hermite splines, which are used by `splineTo` and its variants, and
3. Bézier curves, which can only be created manually.

Creating a `Line` is easy! 
Simply pass the two points you are creating into its constructor:

=== "Kotlin"

    ```kotlin 
    val line = Line(Vector2d(0.0, 0.0), Vector2d(10.0, 10.0))
    ```

=== "Java"

    ```java
    PositionPath<Arclength> line = 
        new Line(new Vector2d(0.0, 0.0), new Vector2d(10.0, 10.0));
    ```

Splines and Bézier curves, however, require a little more work.
Both curve types are implemented parametrically, 
meaning the x and y curves are separate objects that both depend on a parameter `t`.
In addition, the `QuinticSpline1d` and `BezierCurve1d` classes
expect `t` to range solely from 0.0 to 1.0.
This means more math (integration, for those who know calculus) 
is required to allow `t` to range through the full length of the curve.

The library conveniently provides a function to generate a Bézier curve 
using `Vector2d`s as control points, 
allowing end users to skip the extra steps described above:

=== "Kotlin"

    ```kotlin 
    val curve = fromPoints(
        Vector2d(0.0, 0.0),
        Vector2d(10.0, 10.0),
        Vector2d(10.0, 30.0)
    )
    ```

    The `fromPoints` function comes from the file `BezierCurves.kt`.

=== "Java"

    ```java
    PositionPath<Arclength> curve = BezierCurves.fromPoints(
            new Vector2d(0.0, 0.0),
            new Vector2d(10.0, 10.0),
            new Vector2d(10.0, 30.0)
    );
    ```

The official type returned by `fromPoints` is `ArcLengthReparamCurve2d`,
which is essentially a fancy name for a curve that ranges from 0.0 to its full length.

Due to the nature of hermite splines, 
it is difficult to create them by themselves.
We recommend using the `trajectoryBuilder` API's
`splineTo` and related methods instead.

## Types of `PosePath`

Hermes also includes [three default functions](http://rr.zharel.me/core/com.acmerobotics.roadrunner.paths/-position-path/index.html#1182961430%2FFunctions%2F-431355489)
in the `PositionPath` interface to add heading paths,
which creates `PosePath` objects!

The four major types of heading paths include:
1. Tangent heading, where the heading is based off the tangent of the curve,
2. Constant heading,
3. Linear heading interpolation, and
4. Spline heading interpolation, which actually uses a `QuinticSpline1d` to smoothly interpolate headings

The link to the KDoc provides an explanation on how to add heading paths 
to a `PositionPath` to create a `PosePath`.

## Turning A `PosePath` into a `Trajectory`

A trajectory is simply a path and a profile!

Motion profiling in Hermes is based on your tuning,
so the easiest way to create a profile is with the `createProfile(path: PosePath)` 
function of `Drive`. 
It uses your tuned constants and parameters to generate a forward profile of your path,
and returns a `DisplacementProfile` object.
To create a `TimeProfile` object, you can simply pass the displacement-based profile
into `TimeProfile`'s constructor.

Similarly, the `createTrajectory(path: PosePath)` function of `Drive` uses your
tuned constants to create a profile of your path, and then returns a `DisplacementTrajectory` object based on it.

