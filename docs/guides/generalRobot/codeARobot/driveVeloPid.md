# Driving at a specificized velocity using PID and Other Fancy Stuff

Driving around is cool and all, but what would I do if I wanted to go 5.2 feet per second? I could figure out how much throttle I usually have to apply to bo 5 feet per second and then just increase it by (5.2 / 5), but I still don't _know_ that I'm going 5.2 feet per second.

This is where closed loop control comes in. Because our robots have encoders of the drivetrain gearboxes, the TalonSRXes the encoders are connected to know how far the wheels have moved and the rate at which they are spinning. We can use this information to guarantee that our robot is going exactly 5.2 feet per second, using something called PID control. PID control tries to make the value that the sensor reading ("input") match the desired value ("setpoint") by computing the difference between the setpoint and the input ("error") and varying the motor speed based on the error. A more in depth writeup on what PID control is can be found [from this FRC programming done right article](https://frc-pdr.readthedocs.io/en/latest/control/pid_control.html) or [on FRC-Docs' Testing and Tuning PID Loops](https://docs.wpilib.org/en/latest/docs/software/wpilib-tools/shuffleboard/advanced-usage/shuffleboard-tuning-pid.html).

We'll go through using the Talon's onboard PID loop to make the drivetrain drive at exactly the speed that we specify.

## TalonSRX's onboard PID 

The TalonSRX have basically two discrete PID loops that run the PID algorithm based on readings either from sensors plugged directly into them or from Gadgeteers, Pigeons or other sensors over [the robot CAN network](https://docs.wpilib.org/en/latest/docs/software/can-devices/index.html). We will only ever use 1, but users can switch between them or mix their output to do things such as keeping the robot flat while climbing up on two stilts [like 5190 did in 2019](https://github.com/5190GreenHopeRobotics/2019CompetitionSeason/blob/master/src/main/kotlin/org/ghrobotics/frc2019/subsystems/climb/ClimbSubsystem.kt). The TalonSRX also can store four different sets of PID constants which can be quickly cycled through to change PID gains without setting all of them again over CAN. But you totally don't need to know this.

### Getting set up

Pull up the driving code you made in the last tutorial -- if you haven't yet, you should go through that [here](docs/guides/generalRobot/codeARobot/driving) first. We're going to make a couple of simple changes to it to allow us to use the [FalconMotor's setPosition method](docs/guides/falconlib/falconmotor). To do this, we're going to change from the `DefaultNativeUnitModel` that we currently have set to a `LengthNativeUnitModel`. The length native unit model calculates the distance a mechanism such as the wheels on a drivetrain have traveled based on the wheel radius and the number of sensor "ticks" in one rotation of the wheels. For Croissant, there are `4096.nativeUnits` in one rotation of the `2.inch` radius wheels. We can now get the current distance that the left wheels have moved from the motor's encoder as an `SIUnit<Meter>` using `leftMotor.encoder.position`. Try putting the left distance in inches [onto SmartDashboard](https://frc-docs.readthedocs.io/en/latest/docs/software/wpilib-tools/smartdashboard/displaying-expressions.html) in the DriveSubsystems' [periodic method](https://frc-docs.readthedocs.io/en/latest/docs/software/commandbased/subsystems.html), which you likely haven't yet overridden. This method gets called 50 times a second. The DriveSubsystem should now looks something like this:

```Kotlin
    val leftMotor = FalconSRX(1, NativeUnitLengthModel(4096.nativeUnits, 2.inch))
        .apply { ... code ... }

    ... more code ...

    override fun periodic() {
        SmartDashboard.putNumber("leftPosition", leftMotor.encoder.position.inch)
    }
```

When you deploy this code, a new box should pop up on the SmartDashboard tab of Shuffleboard that shows the current position of the motor It should change as you push the robot (or spin the wheels using the driving code from the last driving tutorial) forward and backward, increasing in value as the robot moves forward and decreasing in value as the robot moves backward. If you haven't use Shuffleboard before, read more about it [here](https://frc-docs.readthedocs.io/en/latest/docs/software/wpilib-tools/shuffleboard/getting-started/shuffleboard-tour.html). If the value is inverted -- that is, it gets greater when the robot moves backwards -- try adding `talonSRX.setSensorPhase(true)` or `talonSRX.setSensorPhase(false)` to the leftMotor's apply block so that the value gets greater as the robot moves forward. Repeat the same thing with the right motor so that it's sensor position increases as the robot moves forwards. 

Cool! Now we have our sensors all set up to read the robot's velocity. We now need to set the PID gains of the wheels. We'll apply the gains to the motor in (you guessed it) the motor's apply block, and for now just set them all to zero using the config_kP method and friends [from the talonSRX](https://www.ctr-electronics.com/downloads/api/java/html/interfacecom_1_1ctre_1_1phoenix_1_1motorcontrol_1_1_i_motor_controller.html). You can do this with the following code snippet in a motor's apply block:

```Kotlin
talonSRX.config_kP(/* PID Slot */ 0, /* value */ 0.0)
talonSRX.config_kI(/* PID Slot */ 0, /* value */ 0.0)
talonSRX.config_IntegralZone(/* PID Slot */ 0, /* value */ 0)
talonSRX.config_kD(/* PID Slot */ 0, /* value */ 0.0)
talonSRX.config_kF(/* PID Slot */ 0, /* value */ 0.0)
```

### Tuning Feedforward

We will start tuning the left motor. [CTRE suggests](https://phoenix-documentation.readthedocs.io/en/latest/ch16_ClosedLoop.html#calculating-velocity-feed-forward-gain-kf) starting with pure F gain and adding P once that's tuned for velocity control. F gain is a "guess" at how much voltage is required to move the motor a speed. It assumes that the motor is linear -- that is, zero speed is zero voltage, and to double the speed you can simply double the applied voltage (and interpolate linearly anywhere in between). To do this to the left motor, do the following *with the robot on blocks* (we wouldn't want to kill anyone or break a window, would we?):

- Add a call that puts the left motor's `encoder.rawVelocity.value` onto SmartDashboard
  - Raw velocity is just the number of "ticks" the sensor sees in 100ms and isn't a real value like distance, although you can convert it to distance with a NativeUnitModel.
- Run the wheels forward at a known duty cycle (such as 50%) using either the manual drive command you've already written or Something Else (an example is below)
- Record in code the maximum raw velocity the wheel reached
- Divide the duty cycle you applied by the maximum raw velocity of the wheels
- If you're doing this before the 2020 season, multiply that value by 1023 "throttle units" (because CTRE is bad)
- The resulting number should be your F gain
  - It should be about the same for both sides of the drivetrain -- if it's not, something else is wrong

To just drive forward, you *could* inline the command using WPILib's new StartEndCommand. The first Runnable will be run when the Command is initialized, the second when it ends, and the [vararg](https://kotlinlang.org/docs/reference/functions.html#variable-number-of-arguments-varargs) at the end specifies requirements:

```Kotlin
val YeetForwardCommand = StartEndCommand(
        Runnable { DriveSubsystem.leftMotor.setDutyCycle(0.5); DriveSubsystem.rightMotor.setDutyCycle(0.5) },
        Runnable { DriveSubsystem.leftMotor.setNeutral(); DriveSubsystem.rightMotor.setNeutral() },
        DriveSubsystem
)
```

Apply the calculated F gain to both motors either by changing and redeploying code, or by changing it from [CTRE's Phoenix Tuner](https://phoenix-documentation.readthedocs.io/en/latest/ch03_PrimerPhoenixSoft.html?highlight=tuner). 

Now that you have a good first guess for your F-gain, let's test it out! We'll modify the command above to set the motor's velocity instead of duty cycle by simply changing `setDutyCycle` to `setVelocity`:

```Kotlin
val YeetForwardCommand = StartEndCommand(
        Runnable { DriveSubsystem.leftMotor.setVelocity(4.feet.velocity); DriveSubsystem.rightMotor.setVelocity(4.feet.velocity) },
        Runnable { DriveSubsystem.leftMotor.setNeutral(); DriveSubsystem.rightMotor.setNeutral() },
        DriveSubsystem
)
```

Run this command and observe how fast the left and right motors spin. The velocity of the left and right motors can be obtained a double with the following code snippet: `val ftPerSecLeft = leftMotor.encoder.velocity.feetPerSecond`. Scale your F gain on the left motor by the expected speed of the robot divided by the left speed while the robot is moving and is done accelerating. Say we used a F gain of 500, expected 5ft/sec and observed a speed of 6 -- our new F gain would be given by `new_f = old_f * (5ft_sec / 6ft_sec)`. We do the same thing for the right motor as well, using it's observed speed and scaling it's F gain by the ratio described above. This doesn't have to be _dead_ accurate, but it should be "close-ish" (I guess += 0.5ft/sec or so?) -- the remaining error will be taken up by the P gain.

Next we'll start tuning the Proportional gain. Proportional gain can be thought of a bit like the sensitivity of the system -- if the P gain is large the controller will react more aggressively to error, while small P gains don't react as strongly to error. If P is too small the controller may take a long time to converge but shouldn't bounce about the setpoint (desired velocity), while large values of P will make the controller "bounce" around the setpoint. Read the linked documents on PID above for a better explication of the role of the gains. For our drivetrain, the job of the P term is to "take up the slack" left by the F term.

Someone with a lot of fancy math and who knows a lot more about control systems proved that the optimal PID controller for a motor which is being asked to go a velocity will have a D gain of zero, so we'll ignore it for now. 

### Tuning the P Gain

For the TalonSRX, start with a P gain of 0.1 applied either through Phoenix Tuner or the apply block of the motor. Open up Phoenix tuner, select TalonSRX 1 for the left motor or 3 for the right motor, navigate to the Plot tab (the very last one), and plot motor velocity, error and motor output. Note that these units are different -- they are in encoder ticks per 100ms, not meters (or inches) per second. For example, Phoenix Tuner would show show a speed of about 2000 ticks per 100ms if the robot was driving 5 feet per second. Once the plot is moving, try rolling the robot around and make sure you can see the velocity data. Now run the StartEndCommand from above, and watch the plot. The wheel velocity should rise and error should decrease, hover near or around the setpoint, and velocity will go to zero (and error will increase) when the command ends. Observe how long it takes the robot to reach the desired velocity as well as how much motor output is being applied. If the robot's wheel velocity bounces quickly around the setpoint enough that you can year it (or relatedly, error bounces around zero) or the deviation of error from zero is more than about 0.2ft/sec (or about 100 ticks per 100ms above or below zero error), decrease P by a factor of 0.75 or so. If the error remains non-zero (that is, it stays either above or below zero) for the whole time, you should for sure increase P gain by a factor of 1.5 to 2. **In general, P gain should be as large as possible without causing osculations greater than 0.2ft/sec or so.**

Assuming that you found a combination of PID + F gains such that the robot drivetrain stays close to the requested velocity, congratulations! The robot now doesn't kill itself, and you're ready to try some more advanced autonomous code or mess with closed loop driving!
