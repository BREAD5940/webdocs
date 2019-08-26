# Arm PID -- a showcase of the different Talon modes

PID control is a simple method of controlling one state of a robot mechanism, such as wheel velocity on a drivetrain or the position of a joint. 2019 was our first year ever doing proper position PID on a joint, and we had not one but three of them. To control them we used the built-in PID features of the TalonSRX. The Talon can use it's onboard PID loops on velocity and position closed loop, or motion magic mode. To get a feel for the internal workings of a PID controller, let's create our own in a blank robot project. You can start with the [example project](https://github.com/BREAD5940/Kotlin-Example) and delete all the subsystems and stuff. We'll try using Croissant's proximal joint for this by creating a SendableSubsystemBase for the Proximal with the following motor settings:

|   | Master | Follower  |  
|---|---|---|
| Port |  31 | 32  |   |   |
| Output Inverted | True | False  |   
| Follower Inversion |  |  InvertType.OpposeMaster |   
| Feedback sensor | FeedbackDevice.CTRE_MagEncoder_Relative |   |   
| Sensor phase |  False |   |   
| Ticks per rotation | 4096 * 28 / 3 NativeUnits | |

Your code might look something like this:

```Kotlin
object Proximal {
    
    val master = FalconSRX(31,
            NativeUnitRotationModel(4096.nativeUnits * 28 / 3)).apply {
                outputInverted = true
                feedbackSensor = FeedbackDevice.CTRE_MagEncoder_Relative
                talonSRX.setSensorPhase(false)
            }

    @Suppress("unused")
    val follower = FalconSRX(31, DefaultNativeUnitModel).apply {
                this.follow(master)
                outputInverted = false
                talonSRX.setInverted(InvertType.OpposeMaster)
            }

    val position: SIUnit<Radian> // note that this is like a unit circle, so "up" is positive angle and "down" is negative angle
        get() = master.encoder.position

}
```

We can get the joint's position with `Proximal.position`, and can use the normal FalconMotor methods for setting the outputs on the motor. Let's start with writing our own basic angle control code and make it more confusing from there.

## How do we usually control our linkages?

We often use a form of control called PID control. Page 11 of Tyler's amazing [State-space guide](https://file.tavsys.net/control/state-space-guide.pdf) has a wonderful explication of the PID controller that you should at least try to read. 

Below we're going to present a number of different methods of controlling the position of the arm (or whatever) using feedback about it's position.

## Bang-bang control 

Bang-bang control is a method of controlling the position, velocity, angle or any other state of a system by turning it on and off again. For example, if we wanted our joint to be at 0 degrees, we would turn the motor onto a set percent when it was below 0 degrees and turn it off (or run it backwards) if it was above 0 degrees. This is also effectively what happens if you have the P term on a P controller too large, which is what we'll look at next. In general, bang-bang is the worst form of control because it results in a mechanism that "bounces" around the desired position rather than "settling" at the correct location.

## Proportional Control

The Next Best Thing from Bang-bang is arguably Proportional control, or P control. Based upon how far away the arm is from where it wants to be, it will apply more power. You can think of Proportional control like a spring, pulling the system towards the setpoint. For example, if the arm was close but just below the wanted position, we would apply a small amount of power, and if the arm was further away below the wanted position, we would apply a larger amount of power. The "error" geometrically how far away you currently are is from where you want to be, or mathematically `error = current - desired`. The power would be given by `Output = error * kP`, where kP is some arbitrary gain that's used to convert error into a motor output or other form of output. We might use this in our code to control the arm inside the `execute()` method of a Command, or pass it as a `Runnable` to a `RunCommand`:

```Kotlin
val proportional = (Proximal.position - 0.degree /* our setpoint */).radian * kP
Proximal.master.setDutyCycle(proportional)
```

The value of kP is chosen to make the arm or mechanism react quickly to changes in the setpoint but to avoid rapid osculations about the setpoint. If you observe the setpoint and actual encoder position in Phoenix Tuner, you should see something like this:

![](https://3l4sbp4ao2771ln0f54chhvm-wpengine.netdna-ssl.com/wp-content/uploads/2017/10/Settling-Time-Graph.gif)

You want your system to be critically dampened so that it doesn't overshoot (much, a little is ok) but reaches the setpoint quickly. If your Kp is too great your system will be underdampened and there will be a large amount of overshoot as shown above, and values of Kp that are too small (or too much Derivative, as discussed below) will result in an over-dampened response. Try as best you can with just Kp to get the system to look like the critically dampened curve, with Kp as great as it can be without oscillations.

The issue arises when large values of P are necessary to react quickly like the critically dampened curve but produce lots of "bouncing" around the setpoint, and when values of P which don't osculate aren't large enough to cause the mechanism to react quickly. This issue can be at least partially solved with derivative control.

## Proportional plus Feedforward control

P control works really well if it's the only force acting on the *system* (a general word used to describe the Thing that you're controlling and the Thing you're using to control it), however it starts to break down when external forces start to come into play. For example, an elevator must fight the force of gravity going upwards but not downwards, and the torque exerted on the pivot of an arm changes as the angle of the arm changes relative to the ground. In cases like these, we often use [tools such as Oblarg's Characterization Toolsuite](docs/guides/generalRobot/characterization) to determine how much motor output we need to counteract forces such as static friction and the pull of gravity. For our arm, we can run the arm characterization toolsuite using the process described in the tutorial there to determine out system's constants and add that to our control commands. For example:

```Kotlin
val proportional = (Proximal.position - 0.degree /* our setpoint */).radian * kP
val feedForward = Proximal.position.radian * kCos + kStatic.withSign(proximal.master.encoder.velocity.radian)
val voltageToApply = proportional + feedForward

Proximal.master.setDutyCycle(voltageToApply)
```

Now, the P loop is no longer having to compensate for the force of gravity or the static friction of the arm -- that's what the `feedForward` does. This should make the controller behave more "symmetrically," having the same response regardless of position or direction of travel.

## Proportional + Derivative Control

While a "pure p" control loop like the one discussed above does a good job for some tasks, there may be cases where a value of P sufficient to cause the actuator to actuate to the setpoint aggressively enough also causes osculations about the setpoint, and values of P that remove the osculation produce sluggish responses. In this case we can use a Derivative (D) term to help counteract that. Tyler's [State-space guide](https://file.tavsys.net/control/state-space-guide.pdf) describes Derivative term on page 11 as acting to drive the velocity error of the controller to zero, much like a P controller on velocity. A large value of D will act to dampen any velocity at all, and the P term must to some extent overcome the D term to get the system to move towards the setpoint. D should be added *last*, after tuning P and feedforward. The D output of the controller is calculated by `D_output = Kd * de/dt`, or `D_output = Kd * (0 - velocity)/dt`. Because command-based should run the command's `update()` method at 50hz (or once ever 0.02 seconds), you can assume that dt will always be 0.02 seconds. Your code might now look like the following:

```Kotlin
val proportional = (Proximal.position - 0.degree /* our setpoint */).radian * Kp
val derivative = (0 - Proximal.master.encoder.velocity).radian / 0.02 * Kd
val feedForward = Proximal.position.radian * kCos + kStatic.withSign(proximal.master.encoder.velocity.radian)
val voltageToApply = proportional + derivative + feedForward

Proximal.master.setDutyCycle(voltageToApply)
```

When tuning D, start with a value 2x Kp and make sure that you aren't overdampening the system. Increase Kd just enough to prevent oscolations. At this point you can increase Kp if you want the system to respond even faster, keeping in mind that *if you crank both of these gains in like a see-saw your system may become unstable*.

![](https://3l4sbp4ao2771ln0f54chhvm-wpengine.netdna-ssl.com/wp-content/uploads/2017/10/Settling-Time-Graph.gif)

## What do Talon SRXes do?

As mentioned above, Talon SRXes can do PID onboard even faster than the robot code (at 1,000 hertz). They also support cascaded PID control through Motion Magic, which is what we used on our joints this year. Essentially, you find a Feedforward (Kf) term to convert a velocity setpoint into a motor output, and run PID to convert position error into a velocity error to feed to the inner loop. This is a form of cascaded PID control. When you tell the TalonSRX to go to, say, 45 degrees, the TalonSRX calculates a profile between where it is and where it wants to be so that it has time to accelerate at the beginning and decelerate at the end. We used a similar method to drive a set distance forward last year by running a P loop on distance error and feeding the output to our drivetrain Talons to do velocity PID on. **Note that the units that you've been using for PID up until now are *different* than the units that the TalonSRX uses!**

### How would I convert my above code to use the Talon's PID instead of my PID?

It's actually fairly simple! We'll modify the Proximal file we already made to configure the PID gains of our Talon. Note that these settings can also bet set via Phoenix Tuner

```Kotlin
val master = FalconSRX(31,
        NativeUnitRotationModel(4096.nativeUnits * 28 / 3)).apply {
            outputInverted = true
            feedbackSensor = FeedbackDevice.CTRE_MagEncoder_Relative
            talonSRX.setSensorPhase(false)
        }.apply {
            talonSRX.config_kP(/* PID Slot */ 0, /* value */ 0.0)
            talonSRX.config_kI(/* PID Slot */ 0, /* value */ 0.0)
            talonSRX.config_IntegralZone(/* PID Slot */ 0, /* value */ 0)
            talonSRX.config_kD(/* PID Slot */ 0, /* value */ 0.0)
            talonSRX.config_kF(/* PID Slot */ 0, /* value */ 0.0)
        }
```

Now we'll just change the execute() method of our command to use the onboard PID:

```Kotlin
val setpoint = 0.degree

val feedForwardDutyCycle = Proximal.position.radian * kCos + kStatic.withSign(proximal.master.encoder.velocity.radian)
val feedForwardVoltage = (feedForwardDutyCycle / 12.0).volt

Proximal.master.setPosition(setpoint, feedForwardVoltage)
```

We'll start tuning Kp. For the arm, start with a gain of 0.1 and observe what happens. Continue to increase the P gain in increments of 0.1 until the arm responds quickly but jiggles around the setpoint a bit like an under dampened setpoint. Back off the Kp by 0.05 to 0.1 (ish) and start with a Kd of 0.05, until the jiggle is removed or within acceptable tolerance.