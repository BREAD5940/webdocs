# FalconLibrary Command Based

## FalconCommand and FalconSubsystems

FalconLibrary command based is a loose wrapper around the upcoming [new command based for 2020](https://frc-docs.readthedocs.io/en/latest/docs/software/commandbased/index.html). FalconCommand no longer provides coroutine based timing, but rather only reserves subsystems for you with constructor arguments (more on that later). FalconSubsystem is simply `SendableSubsystemBase` with extra methods:

```kotlin
abstract class FalconSubsystem : SendableSubsystemBase() {
    open fun lateInit() {}
    open fun autoReset() {}
    open fun teleopReset() {}
    open fun setNeutral() {}
}
```

### Building Command Groups in Kotlin with FalconLib

## FalconTimedRobot

[`FalconTimedRobot`](https://github.com/5190GreenHopeRobotics/FalconLibrary/blob/2020/wpi/src/main/kotlin/org/ghrobotics/lib/wrappers/FalconTimedRobot.kt) is a wrapper around [WPI's `TimedRobot` class](https://first.wpi.edu/FRC/roborio/release/docs/java/edu/wpi/first/wpilibj/TimedRobot.html), and is at the heart of any FalconLibrary based robot. It includes the same methods as the WPI `TimedRobot` abalible to be overridden, automatically runs the CommandScheduler and 