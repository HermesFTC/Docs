# Custom Actions

## Basic Examples

In this guide, you will learn how to create your own custom actions for your subsystems. 
We will be creating a simple intake subsystem with a single motor.

### Step 1: Create your subsystem

First, we need to create our subsystem. 
NextRunner does not include any formal Subsystem class or interface,
but we can create a class and pass in a `HardwareMap` instance to the constructor
in order to access hardware.

=== "Kotlin"

    ```kotlin
    class Intake(hwMap: HardwareMap) {
        private val motor = hwMap[DcMotorEx::class.java, "intake_motor"]
    }
    ```

=== "Java"

    ```java
    public class Intake {
        private DcMotorEx motor;
        public Intake(HardwareMap hwMap) { 
            motor = hwMap.get(DcMotorEx.class, "intake_motor");
        }
    }
    ```

### Step 2: Create your actions

Now, we will create two actions: `run()` and `stop()`. The `run()` action will turn the intake motor on, and the `stop()` action will turn it off.

#### `run()` action

To create the `run()` action, we will create a new `Action` that sets the motor power to 1.

=== "Kotlin"

    ```kotlin
    fun run(): Action = Action { 
        motor.power = 1.0
        true
    }
    ```

=== "Java"

    ```java
    public Action run() {
        return p -> {
            motor.setPower(1.0);
            return true;
        };
    }
    ```

Note that we return `true` at the end of the action. This is because we want the action to continue running until it is interrupted by another action (like `stop()`).

#### `stop()` action

Now, let's create the `stop()` action. This action will set the motor power to 0 and then complete.

=== "Kotlin"

    ```kotlin
    fun stop(): Action = Action { 
        motor.power = 0.0
        false
    }
    ```

=== "Java"

    ```java
    public Action stop() {
        return p -> {
            motor.setPower(0.0);
            return false;
        };
    }
    ```

This time, we return `false` at the end of the action. This tells the `ActionRunner` that the action is complete and can be removed from the queue.

### Final Result

Here is the final code for our `Intake` subsystem:

=== "Kotlin"

    ```kotlin
    class Intake(hwMap: HardwareMap) {
        private val motor = hwMap[DcMotorEx::class.java, "intake_motor"]

        fun run(): Action = Action { 
            motor.power = 1.0
            true
        }

        fun stop(): Action = Action { 
            motor.power = 0.0
            false
        }
    }
    ```

=== "Java"

    ```java
    public class Intake {
        private DcMotorEx motor;
        public Intake(HardwareMap hwMap) {
            motor = hwMap.get(DcMotorEx.class, "intake_motor");
        }

        public Action run() {
            return p -> {
                motor.setPower(1.0);
                return true;
            };
        }

        public Action stop() {
            return p -> {
                motor.setPower(0.0);
                return false;
            };
        }
    }
    ```

## More Complex Actions

The `run()` and `stop()` actions are simple, but what if we want to create more complex actions? Let's take a look at a few examples.

### Lift

Let's create a lift subsystem that can move to a specific position. This is useful for things like moving a lift to a scoring position.

First, let's create our subsystem. We will need a motor for the lift.

=== "Kotlin"

    ```kotlin
    class Lift(hwMap: HardwareMap) {
        private val motor = hwMap[DcMotorEx::class.java, "lift_motor"]
    }
    ```

=== "Java"

    ```java
    public class Lift {
        private DcMotorEx motor;
        public Lift(HardwareMap hwMap) { 
            motor = hwMap.get(DcMotorEx.class, "lift_motor");
        }
    }
    ```

Now, let's create an action that moves the lift to a specific position. We will use a simple P controller with a gravity feedforward to move the motor to the target position. The feedforward term will help to counteract the force of gravity on the lift.

=== "Kotlin"

    ```kotlin
    fun goToPosition(targetPosition: Double): Action = Action {
        val kP = 0.01 // Proportional gain, this should be tuned
        val kG = 0.1 // Gravity feedforward, this should be tuned

        val error = targetPosition - motor.currentPosition
        motor.power = (error * kP) + kG
        
        // continue if error is greater than 10 ticks
        abs(error) > 10 
    }
    ```

=== "Java"

    ```java
    public Action goToPosition(double targetPosition) {
        return p -> {
            double kP = 0.01; // Proportional gain, this should be tuned
            double kG = 0.1; // Gravity feedforward, this should be tuned

            double error = targetPosition - motor.getCurrentPosition();
            motor.setPower((error * kP) + kG);
            
            // continue if error is greater than 10 ticks
            return Math.abs(error) > 10;
        };
    }
    ```

This action will move the motor to the `targetPosition` and then complete once it is within 10 ticks of the target.

### Claw

Now, let's create a claw subsystem that can open and close.

=== "Kotlin"

    ```kotlin
    class Claw(hwMap: HardwareMap) {
        private val servo = hwMap[Servo::class.java, "claw_servo"]
    }
    ```

=== "Java"

    ```java
    public class Claw {
        private Servo servo;
        public Claw(HardwareMap hwMap) { 
            servo = hwMap.get(Servo.class, "claw_servo");
        }
    }
    ```

Now, let's create the `open()` and `close()` actions. These actions will set the servo position and then complete immediately.

=== "Kotlin"

    ```kotlin
    fun open(): Action = Action {
        servo.position = 0.1
        false // complete immediately
    }

    fun close(): Action = Action {
        servo.position = 0.8
        false // complete immediately
    }
    ```

=== "Java"

    ```java
    public Action open() {
        return p -> {
            servo.setPosition(0.1);
            return false; // complete immediately
        };
    }

    public Action close() {
        return p -> {
            servo.setPosition(0.8);
            return false; // complete immediately
        };
    }
    ```

This time, we return `false` at the end of the action. This tells the `ActionRunner` that the action is complete and can be removed from the queue.

### Final Result

Here is the final code for our `Lift` and `Claw` subsystems:

=== "Kotlin"

    ```kotlin
    class Lift(hwMap: HardwareMap) {
        private val motor = hwMap[DcMotorEx::class.java, "lift_motor"]

        fun goToPosition(targetPosition: Double): Action = Action {
            val kP = 0.01 // Proportional gain, this should be tuned
            val kG = 0.1 // Gravity feedforward, this should be tuned

            val error = targetPosition - motor.currentPosition
            motor.power = (error * kP) + kG
            
            // continue if error is greater than 10 ticks
            abs(error) > 10 
        }
    }

    class Claw(hwMap: HardwareMap) {
        private val servo = hwMap[Servo::class.java, "claw_servo"]

        fun open(): Action = Action {
            servo.position = 0.1
            false // complete immediately
        }

        fun close(): Action = Action {
            servo.position = 0.8
            false // complete immediately
        }
    }
    ```

=== "Java"

    ```java
    public class Lift {
        private DcMotorEx motor;
        
        public Lift(HardwareMap hwMap) { 
            motor = hwMap.get(DcMotorEx.class, "lift_motor");
        }

        public Action goToPosition(double targetPosition) {
            return p -> {
                double kP = 0.01; // Proportional gain, this should be tuned
                double kG = 0.1; // Gravity feedforward, this should be tuned
        
                double error = targetPosition - motor.getCurrentPosition();
                motor.setPower((error * kP) + kG);
                
                // continue if error is greater than 10 ticks
                return Math.abs(error) > 10;
            };
        }
    }

    // in java these should be placed in separate files

    public class Claw {
        private Servo servo;
        public Claw(HardwareMap hwMap) { 
            servo = hwMap.get(Servo.class, "claw_servo");
        }

        public Action open() {
            return p -> {
                servo.setPosition(0.1);
                return false; // complete immediately
            };
        }

        public Action close() {
            return p -> {
                servo.setPosition(0.8);
                return false; // complete immediately
            };
        }
    }
    ```

## Next Steps

Now that you have created your custom actions, 
you can learn how to use them in your TeleOp OpMode in the [next guide](./teleop-actions.md).