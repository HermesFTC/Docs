# Using ActionRunner in TeleOp

This guide will show you how to use `ActionRunner` to run actions in your TeleOp OpMode. 
We will be using the `Lift` and `Claw` subsystems from the [custom actions guide](./custom-actions.md),
and a `Drive` class from the QuickStart.

## Step 1: Create your OpMode

First, let's create a new TeleOp OpMode.

=== "Kotlin"

    ```kotlin
    @TeleOp
    class MyTeleOp : OpMode() {
        override fun init() {
            // initialization code
        }

        override fun loop() {
            // main loop
        }
    }
    ```

=== "Java"

    ```java
    @TeleOp
    public class MyTeleOp extends OpMode {
        @Override
        public void init() {
            // initialization code
        }

        @Override
        public void loop() {
            // main loop
        }
    }
    ```



## Step 2: Initialize subsystems

Now, let's initialize our `Lift` and `Claw` subsystems.

=== "Kotlin"

    ```kotlin
    @TeleOp
    class MyTeleOp : OpMode() {
        private lateinit var lift: Lift
        private lateinit var claw: Claw
        private lateinit var drive: Drive 

        override fun init() {
            lift = Lift(hardwareMap)
            claw = Claw(hardwareMap)
            drive = MecanumDrive(hardwareMap, Pose2d(0.0, 0.0, 0.0))
        }

        override fun loop() {
            // main loop
        }
    }
    ```

=== "Java"

    ```java
    @TeleOp
    public class MyTeleOp extends OpMode {
        private Lift lift;
        private Claw claw;
        private Drive drive;

        @Override
        public void init() {
            lift = new Lift(hardwareMap);
            claw = new Claw(hardwareMap);
            drive = new MecanumDrive(hardwareMap, new Pose2d(0.0, 0.0, 0.0));
        }

        @Override
        public void loop() {
            // main loop
        }
    }
    ```



## Step 3: Map controller buttons to actions

Now, let's map some controller buttons to actions. 
We'll use the `gamepad1` object to get button presses.

To prevent an action from being triggered every loop cycle while a button is held down,
we can use the `aWasPressed()`, `bWasPressed()`, etc. methods.
These methods return true only on the first frame the button is pressed.

`ActionRunner` is a named object, 
meaning there is one instance of it that is accessed globally.
Its functions are called like static methods in Java.

=== "Kotlin"

    ```kotlin
    // in loop()
    if (gamepad1.aWasPressed()) {
        ActionRunner.run(claw.open())
    }

    if (gamepad1.bWasPressed()) {
        ActionRunner.run(claw.close())
    }

    if (gamepad1.xWasPressed()) {
        ActionRunner.run(lift.goToPosition(1000.0))
    }

    if (gamepad1.yWasPressed()) {
        ActionRunner.run(lift.goToPosition(0.0))
    }
    ```

=== "Java"

    ```java
    // in loop()
    if (gamepad1.aWasPressed()) {
        ActionRunner.run(claw.open());
    }

    if (gamepad1.bWasPressed()) {
        ActionRunner.run(claw.close());
    }

    if (gamepad1.xWasPressed()) {
        ActionRunner.run(lift.goToPosition(1000));
    }

    if (gamepad1.yWasPressed()) {
        ActionRunner.run(lift.goToPosition(0));
    }
    ```



## Step 4: Set Drive Powers and Update the ActionRunner

We set the drive powers based on the gamepad's left and right sticks,
but we do not run an action for this.
Instead, we directly set the drive power using the `setDrivePower()` method of the
`Drive` class. 
One benefit of `ActionRunner` is that it allows you to run actions in parallel with other code,
such as setting drive powers.

Finally, we need to call `ActionRunner.update()` at the end of our `loop()` function. This will run the actions and keep them updated.

=== "Kotlin"

    ```kotlin
    // at the end of loop()

    drive.setDrivePower(PoseVelocity2d(
        Vector2d(
            gamepad1.left_stick_y,
            gamepad1.left_stick_x
        ),
        gamepad1.right_stick_x,
    ))

    ActionRunner.update()
    ```

=== "Java"

    ```java
    // at the end of loop()

    drive.setDrivePower(new PoseVelocity2d(
        new Vector2d(
            gamepad1.left_stick_y,
            gamepad1.left_stick_x
        ),
        gamepad1.right_stick_x,
    ));

    ActionRunner.update();
    ```



## Final Result

Here is the final code for our TeleOp OpMode:

=== "Kotlin"

    ```kotlin
    @TeleOp
    class MyTeleOp : OpMode() {
        private lateinit var lift: Lift
        private lateinit var claw: Claw
        private lateinit var drive: Drive

        override fun init() {
            lift = Lift(hardwareMap)
            claw = Claw(hardwareMap)
            drive = MecanumDrive(hardwareMap, Pose2d(0.0, 0.0, 0.0))
        }

        override fun loop() {
            if (gamepad1.aWasPressed()) {
                ActionRunner.run(claw.open())
            }

            if (gamepad1.bWasPressed()) {
                ActionRunner.run(claw.close())
            }

            if (gamepad1.xWasPressed()) {
                ActionRunner.run(lift.goToPosition(1000.0))
            }

            if (gamepad1.yWasPressed()) {
                ActionRunner.run(lift.goToPosition(0.0))
            }

            drive.setDrivePower(PoseVelocity2d(
                Vector2d(
                    gamepad1.left_stick_y,
                    gamepad1.left_stick_x
                ),
                gamepad1.right_stick_x,
            ))

            ActionRunner.update()
        }
    }
    ```

=== "Java"

    ```java
    @TeleOp
    public class MyTeleOp extends OpMode {
        private Lift lift;
        private Claw claw;
        private Drive drive;

        @Override
        public void init() {
            lift = new Lift(hardwareMap);
            claw = new Claw(hardwareMap);
            drive = new MecanumDrive(hardwareMap, new Pose2d(0.0, 0.0, 0.0));
        }

        @Override
        public void loop() {
            if (gamepad1.aWasPressed()) {
                ActionRunner.run(claw.open());
            }

            if (gamepad1.bWasPressed()) {
                ActionRunner.run(claw.close());
            }

            if (gamepad1.xWasPressed()) {
                ActionRunner.run(lift.goToPosition(1000));
            }

            if (gamepad1.yWasPressed()) {
                ActionRunner.run(lift.goToPosition(0));
            }

            drive.setDrivePower(new PoseVelocity2d(
                new Vector2d(
                    gamepad1.left_stick_y,
                    gamepad1.left_stick_x
                ),
                gamepad1.right_stick_x,
            ));

            ActionRunner.update();
        }
    }
    ```


