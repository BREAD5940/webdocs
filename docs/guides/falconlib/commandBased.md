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

## Building Command Groups in Kotlin with FalconLib

blah blah. This example is from 5190's Cargo Ship routine:

```Kotlin
override val routine
    get() = sequential {

        +parallel {
            +followVisionAssistedTrajectory(mode.path1, pathMirrored, 4.feet, true)
            +sequential {
                +WaitCommand(mode.path1.duration.second - 3.5.second.second)
                +Superstructure.kHatchLow.withTimeout(2.0.second)
            }
        }

        val path2 = followVisionAssistedTrajectory(mode.path2, pathMirrored, 4.feet)

        +parallel {
            +path2
            +sequential {
                +IntakeHatchCommand(true).withTimeout(0.5.second)
                +IntakeCloseCommand()
                +Superstructure.kBackHatchFromLoadingStation
                +IntakeHatchCommand(false).withExit { path2.isFinished }
            }
        }

        +relocalize(TrajectoryWaypoints.kLoadingStation, false, pathMirrored)

        +parallel {
            +IntakeHatchCommand(false).withTimeout(0.75.second)
            +followVisionAssistedTrajectory(mode.path3, pathMirrored, 4.feet, true)
            +sequential {
                +WaitCommand(1.0)
                +Superstructure.kHatchLow.withTimeout(4.second)
            }
        }

        +parallel {
            +IntakeHatchCommand(true).withTimeout(0.5.second)
            +object : FalconCommand(DriveSubsystem) {
                override fun execute() {
                    DriveSubsystem.tankDrive(-0.3, -0.3)
                }

                override fun end(i: Boolean) {
                    DriveSubsystem.zeroOutputs()
                }
            }.withTimeout(1.second)
        }
    }
}
```

## FalconTimedRobot

[`FalconTimedRobot`](https://github.com/5190GreenHopeRobotics/FalconLibrary/blob/2020/wpi/src/main/kotlin/org/ghrobotics/lib/wrappers/FalconTimedRobot.kt) is a wrapper around [WPI's `TimedRobot` class](https://first.wpi.edu/FRC/roborio/release/docs/java/edu/wpi/first/wpilibj/TimedRobot.html), and is at the heart of any FalconLibrary based robot. It includes the same methods as the WPI `TimedRobot` abatable to be overridden, automatically runs the CommandScheduler, handles entering and exiting emergency modes as well as manages the `FalconSubsystemHandler` (which calls the extra methods in `FalconSubsystem` automatically). Subsystems are added to the `FalconSubsystemHandler` and the emergency ready subsystem list with the unary plus (+) operator; an example of this follows:

```Kotlin
object Robot: FalconTimedRobot() {
    override fun robotInit() {
        // add the DriveSubsystem to both the subsystem handler 
        // and (if it's EmergencyHandleable) to the emergency list
        +DriveSubsystem
    }
}
```