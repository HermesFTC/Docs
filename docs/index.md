# Hermes
![Maven Central Version](https://img.shields.io/maven-central/v/me.zharel.hermes/core?label=latest%20release&labelColor=darkBlue&color=yellow)

Hermes offers a revolutionary motion profiling and planning
algorithm that builds on RoadRunner from ACME Robotics,
FTC Team 8367. 

## Features
- **Path Planning** with smooth continuity, such as Bézier curves,
  hermite splines, and more! 
- **Motion Profiling** paths to ensure the most optimal
  trajectory for your robot, using either time or displacement
- **Action Framework** for easily scalable robot actions
  for asynchronous robot control
- **Advanced Following** that uses PID control and 
  voltage-compensated feedforward to accurately follow
  trajectories
- **Unparalleled Customization** in path generation, 
  profiling, and following

## QuickStart

We recommend using the [QuickStart](https://github.com/HermesFTC/Quickstart),
as it already has `Localizer`, `Drive`, and `Follower` implementations, 
the required tuning OpMode setup, and some convenient examples!

## Adding To An Existing Project

If you are installing Hermes to an existing project, 
we recommend downloading the QuickStart and copying *its* 
`TeamCode` module package to the `TeamCode` module in your project. 

You will also need to add the library as a dependency in your Gradle scripts.
In your `build.dependencies.gradle` file, add the following three lines
to the `dependencies` block:

```groovy
    implementation 'me.zharel.hermes:core:<latest>>'
    implementation 'me.zharel.hermes:actions:<latest>'
    implementation 'me.zharel.hermes:ftc:<latest>'
```

Where `latest` is replaced by the latest version as displayed above.

Then, sync your project with Gradle files.

## Updating From RoadRunner 1.0

Hermes's API is mostly compatible with RoadRunner code,
though package locations were changed.
Simply replace your RoadRunner imports with the following:

```groovy
    implementation 'me.zharel.hermes:core:<latest>>'
    implementation 'me.zharel.hermes:actions:<latest>'
    implementation 'me.zharel.hermes:ftc:<latest>'
```

And sync your project with Gradle files. 

> [!WARNING]
> You will need to remove all of the RoadRunner imports from your files
> and replace them with Hermes's.
> Once you delete the import statements, IntelliJ and Android Studio
> can automatically replace them.

## Tuning 

If you are using the QuickStart, the tuning process is
the exact same as the [RoadRunner 1.0 tuning process](https://rr.brott.dev/docs/v1-0/tuning/).
Future changes to the tuning process will be listed here.

## KDoc 

[KDoc for Hermes can be found here](http://docs.hermes.zharel.gay/).
We recommend checking the KDoc pages, 
as they include many features not directly discussed here.
