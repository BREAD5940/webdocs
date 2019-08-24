# Writing Code to Control a Drivetrain

Here we'll start writing code to start driving Croissant around. We'll start with the [Kotlin example codebase](https://github.com/BREAD5940/Kotlin-Example). By the end of this you should be familiar with making CAN motor controllers tell motors to spin, interfacing with xbox controllers and creating a basic Command for driving. 

However, this example code comes with an `ExampleSubsystem` and `ExampleCommand`. The subsytem as a FalconSRX on an arbitrary port which may or may not exist on the robot you're using it on, and may in fact cause undesired behavior. For this reason, before starting on making a Drivetrain subsystem, remove references to either from Controls and Robot, and delete both the `ExampleSubsystem` and `ExampleCommand`. 

## Creating a Subsystem for the Drivetrain

Start by making a [new package](https://www.youtube.com/watch?v=EBWmlcMXxXc) under `frc.robot.subsystems` called `drive`, and create a Kotlin class there called `DriveSubsystem` that extends `FalconSubsystem`. The convention is to have the robot's Subsystems end with Subsystems, Commands' names end with Command and routines/Command group's names end with Routine. 

Note that While FalconLibrary does offer a TankDriveSubsystem abstract class, it also hides much of what is actually going on behind the scenes. For this reason, we will not be using it here. Instead, we will be using FalconLibrary's [FalconSubsystem](docs/guides/falconlib/commandBased?id=falconcommand-and-falconsubsystems)

Next, we're going to give it a motor. On the robot, each side of the drivetrain has two motors. But to start, we're only going to use one motor. Give the drivetrain one member variable -- a FalconSRX on port 1 using the `DefaultNativeUnitModel`. You can also make sure that the output is not inverted by adding `motor.outputInverted = false` to the constructor, or add it to a `run` block applied to the FalconSRX that returns `this`. Your DriveSubsystem should now look something like this:

```Kotlin
object DriveSubsystem : FalconSubsystem() {
    
    val leftMotor: FalconSRX<NativeUnit> = FalconSRX(1, DefaultNativeUnitModel).apply { /* this: FalconSRX<NativeUnit> */
        this.outputInverted = false
    }
}
```

Next, let's make our motor actually spin. The motor can be commanded to spin at a [duty cycle](https://en.wikipedia.org/wiki/Duty_cycle) as a double between -1.0 and 1.0. To set the motor's duty cycle, use `motor.setDutyCycle(dutyCycle)`. To do this, we'll create a command which we will eventually use for driving. Also in the `frc/robot/subsystems/drive` package, create a class named something along the lines of `TeleopDriveCommand` which extends `FalconCommand(DriveSubsystem)`. When we pass the DriveSubsystem to the FalconCommand argument, it will automatically add the requirement of DriveSubsystem to the command. When the command initializes, it should set the DriveSubsystem's duty cycle to half, and when it ends, it should stop the motor regardless of if the commanded ended interrupted or not. The motor can be stopped either by setting the duty cycle to 0.0, or by calling `setNeutral`. Furthermore, make the command run perpetually for now -- that is, `isFinished()` should always return false. Note also that unless you override the `runsWhenDisabled` disabled method, the command will not run while the robot is disabled. Your command should look something like this:

```Kotlin
class TeleopDriveCommand : FalconCommand(DriveSubsystem) {
    override fun isFinished() = false

    override fun initialize() {
        DriveSubsystem.leftMotor.setDutyCycle(0.5)
    }

    override fun end(interrupted: Boolean) {
        DriveSubsystem.leftMotor.setNeutral()
    }
}
```

We also have to add this command to `Controls`, so that we can start and stop it with an xbox controller. FalconLibrary's XboxController takes a block, in which buttons are bound. Within this block, we register emergency mode (more on that later I swear) and can also bind buttons. In the example, we only want to be able to start and stop commands if we aren't currently climbing. 

If you haven't yet, remove lines 20 through 24 if they reference ExampleCommand. The `state({!isClimbing})` state block should now be empty. We'll bind our command to the `A` button on the xbox controller by adding `button(kA).change((TeleopDriveCommand()))` anywhere within xboxController block, either within our without the isClimbing state block. 

Now we can test our code on Croissant. Put the robot up on blocks or a piece of wood such that the wheels do not contact the ground. [Build and deploy your code to the robot](http://127.0.0.1:4000/#/docs/guides/generalRobot/introToGradle?id=building-testing-checking-and-deploying-robot-code-through-the-command-line), and acquire a driver station and xbox controller.

Now, when the `A` button of the xboxController on port 0 of the [driver station USB Devices tab](https://frc-docs.readthedocs.io/en/latest/docs/software/driverstation/driver-station.html#usb-devices-tab), the TeleopDriveCommand command will be scheduled and the motor will spin at half power. When you release the button, the command will end and the motor should stop.

Of course, in our drivetrain, each side doesn't have just one motor -- it has two! We'll add another motor to our drivetrain. On Croissant, the left side of the drivetrain consists of the CAN motor controllers on ports 1 and 2, so we'll add another motor to the DriveSubsystem that's the same as the `leftMotor`, but on port 2 instead of port 1. We'll also tell the new motor to do exactly the same thing that we tel the leftMotor to do using the `follow` method.

However, simply adding another motor can be really dangerous. Motors can spin opposite directions despite the talons applying the same voltage in the same direction to each. For this reason, when coding gearboxes or linkages with more than one motors as inputs, follow a procedure such as the following:
- Instantiate the first motor and un-invert the output (set inverted to false)
- Using Phoenix Tuner, ensure that positive voltages make the mechanism spin "forwards" (for drivetrains, drive forwards, arms spin "positive angle", etc)
    - If not, invert the output of the first motor in code or using Phoenix Tuner and re-confirm inversion
- Instantiate the second motor and do **not** set it to follow the first motor
- Using Phoenix Tuner, ensure that positive voltages make the mechanism spin "forwards" (for drivetrains, drive forwards, arms spin "positive angle", etc)
    - If not, invert the output of the second motor in code or using Phoenix Tuner and re-confirm inversion
- *After* confirming inversion, set the second motor to follow the first motor

Once again, if the motors attempt to spin opposite directions, the motors will stall, try to rip each other apart and will likely damage the gearbox. Worst of all build team will be mad at you.

The DriveSubsystem should now look something like this:

```Kotlin
    val leftMotor: FalconSRX<NativeUnit> = FalconSRX(1, DefaultNativeUnitModel).apply { /* this: FalconSRX<NativeUnit> */
        this.outputInverted = false // Replace me with what you found works for the leftMotor
    }

    val leftFollower: FalconSRX<NativeUnit> = FalconSRX(2, DefaultNativeUnitModel).apply { /* this: FalconSRX<NativeUnit> */
        this.outputInverted = false // Replace me with what you found works for the leftFollower
        this.follow(leftMotor)
    }
```

When we redeploy, enable and start our command again, both motors should now spin the left side of the drivetrain. The wheels should now spin quite a bit faster. If it doesn't move or moves less, disable and check motor heat -- you may have accidentally messed up inversion settings!

Cool! Now you have your first command which makes half of the robot drivetrain spin. Repeat the motor inversion checks for a `rightMotor` on port 3, and `rightFollower` on port 4, set the right follower to follow the right motor, and set the `rightMotor`'s duty cycle in the TeleopDriveCommand too. Now, when you press the xbox controller's A button, both sides of the drivetrain should spin such that the robot would drive forward if it wasn't on blocks (you did put it on blocks, _didn't you?_ The code still tells the robot to _yeet forward regardless of the people or walls in the way_).

Now that we have four motors which spin the correct direction when you tell them to go forward, let's make it obey the joystick. Let's change our manual drive command a bit to take joystick input. As we drive using "Arcade Drive" (left stick forward for linear motion, right stick left/right for turn rate), we'll get the forward velocity of the robot with `Controls.driverFalconXbox.getY(GenericHID.Hand.kLeft).withDeadband(kDeadband)`, and the turn rate with `Controls.driverFalconXbox.getX(GenericHID.Hand.kRight)`. Firstly, both of these return DoubleSources (They're called DoubleSuppliers in Java) -- to get the current Joystick angle, you need to `invoke` these DoubleSources. Note also that the Xbox Y axis is negative when you push it forward. We'll do it by storing these controls within the companion object of ManualDriveCommand, and then invoking them in the execute() method to get the current position of the joysticks. For now, we will just set our motors to the forward component of the angle of the joysticks. Our ManualDriveCommand will now look something like this:

```Kotlin
class TeleopDriveCommand : FalconCommand(DriveSubsystem){

    override fun isFinished() = false

    override fun execute() {
        // negative because xbox is negative Y when pushed forward
        val forward = -speedSource() // same as -1 * speedSource.invoke()
        val turn = rotationSource()

        val wantedLeftOutput = forward
        val wantedRightOutput = forward

        DriveSubsystem.leftMotor.setDutyCycle(wantedLeftOutput)
        DriveSubsystem.rightMotor.setDutyCycle(wantedRightOutput)
    }

    override fun end(interrupted: Boolean) {
        DriveSubsystem.leftMotor.setNeutral()
    }

    companion object {
        private const val kDeadband = 0.05
        val speedSource by lazy { Controls.driverFalconXbox.getY(GenericHID.Hand.kLeft).withDeadband(kDeadband) }
        private val rotationSource by lazy { Controls.driverFalconXbox.getX(GenericHID.Hand.kRight) }
    }
}
```

The implementation of the rest of arcade drive (getting the robot to turn by adding or subtracting the requested turn speed from the left or right motor output) is trivial and left as an exercise to the reader. If you're stumped, good luck finding it somewhere [in the source code](https://github.com/wpilibsuite/allwpilib/blob/master/wpilibj/src/main/java/edu/wpi/first/wpilibj/drive/DifferentialDrive.java) or [on frc-docs](https://docs.wpilib.org/en/latest/docs/software/actuators/wpi-drive-classes.html?highlight=differentialdrive#drive-modes).

Now that we have a Command which let's our driver drive the robot, let's make sure that it's always running. We'll do this as adding it as the default command of the DriveSubsystem. To do this, add the following to the DriveSubsystem's lateInit method:

```Kotlin
    override fun lateInit() {
        defaultCommand = TeleopDriveCommand()
    }
```

### An aside on motor brake mode

By default the TalonSRX, and therefore the motor, will be in "coast" mode when their duty cycle is set to zero or `setNeutral` is called. The TalonSRX (and also the Spark MAX) have a neutral mode called "brake," where the leads of the motor are communized electrically to create effectively an electric brake. If you set the neutral mode of the talons to this, the robot will roll to a stop much more quickly but will be more difficult to push by hand for the same reason. Simply set the `brakeMode` property of a FalconMotor to true to enable it.
