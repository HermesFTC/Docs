# Hermes
![Maven Central Version](https://img.shields.io/maven-central/v/gay.zharel.hermes/core?label=latest%20release&labelColor=darkBlue&color=yellow)

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
as it has all the necessary dependencies and code to get you started.
The QuickStart also contains some example OpModes to help you
tune your robot's PID gains, which aren't covered by the automatic tuners.

## Adding To An Existing Project

You will need to add the library as a dependency in your Gradle scripts.
In your `build.dependencies.gradle` file, add the following three lines
to the `dependencies` block:

```groovy
    implementation 'gay.zharel.hermes:core:<latest>>'
    implementation 'gay.zharel.hermes:actions:<latest>'
    implementation 'gay.zharel.hermes:ftc:<latest>'
```

Where `latest` is replaced by the latest version as displayed above.

Then, sync your project with Gradle files.

## Tuning 

To tune Hermes, simply connect to your robot's WiFi network and
go to [192.168.43.1:8080/hermes](192.168.43.1:8080/hermes)
in your browser!

The PID gains are not automatically tuned,
so you will need to tune them yourself.
The `ManualFeedbackTuner` OpMode from the QuickStart
can be used to tune your robot's PID gains using FTC Dashboard.

## API Documentation 

[API docs for Hermes can be found here](https://docs.hermes.zharel.gay/).
We recommend checking these docs pages, 
as they include many features not directly discussed here.

## Questions? Feedback?

If you have any questions or feedback about Hermes, 
feel free to reach out to us through our [Discord server](https://discord.gg/49C5epU22h)!

If you find a bug or have a feature request,
please open an issue on our [GitHub Issues page](https://github.com/HermesFTC/Hermes/issues);
if the issue is related to the QuickStart,
please open an issue on the [QuickStart Issues page](https://github.com/HermesFTC/Quickstart/issues) instead.

## Contributing

If you would like to contribute to Hermes,
please fork the repository and create a pull request.
We are open to all contributions,
and we will review your pull request as soon as possible.

Thank you!