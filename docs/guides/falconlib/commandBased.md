# FalconLibrary Command Based

## FalconCommand and FalconSubsystems

FalconLibrary command based is a loose wrapper around the upcoming [new command based for 2020](https://frc-docs.readthedocs.io/en/latest/docs/software/commandbased/index.html). FalconCommand no longer provides coroutine based timing, but rather only reserves subsystems for you with constructor arguments (more on that later). An example of Falcon Command-based is avalible in the <a href="files/Kotlin-Example-Command-Based.zip" download="Kotlin-Example-Command-Based.zip">downlaodable example code</a> or in [Offseason-Croissant](https://github.com/bread5940/offseason-croissant) FalconSubsystem is simply `SendableSubsystemBase` with extra methods:

```Kotlin
abstract class FalconSubsystem : SendableSubsystemBase() {
    open fun lateInit() {} // run after robotInit
    open fun autoReset() {} // run when auto starts
    open fun teleopReset() {} // run when teleop starts
    open fun setNeutral() {} // run on robot disable
}
```

Similarly, `FalconCommand` is a light wrapper around WPI's `SendableCommandBase`. 

```Kotlin
package org.ghrobotics.lib.commands

abstract class FalconCommand(vararg requirements: Subsystem) : SendableCommandBase() {
    init {
        addRequirements(*requirements)
    }
}

class ExampleCommand : FaalconCommand(ExampleSubsystem) {}
```

## Building Command Groups in Kotlin with FalconLib

Command groups in FalconLibrary are simple to construct. Read more on Command groups in the [new command based for 2020 on frc-docs](https://frc-docs.readthedocs.io/en/latest/docs/software/commandbased/index.html). For example:

```Kotlin
+parallel {
    +followVisionAssistedTrajectory(mode.path1, pathMirrored, 4.feet, true)
    +sequential {
        +WaitCommand(mode.path1.duration.second - 3.5.second.second)
        +Superstructure.kHatchLow.withTimeout(2.0.second)
    }
}
```

In order to cleanly construct, new command groups, FalconLibrary offers four different builders -- one for each different kind of command group.

```Kotlin
package org.ghrobotics.lib.commands

fun sequential(block: BasicCommandGroupBuilder.() -> Unit) 
fun parallel(block: BasicCommandGroupBuilder.() -> Unit)
fun parallelRace(block: BasicCommandGroupBuilder.() -> Unit)
fun parallelDeadline(deadline: Command, block: ParallelDeadlineGroupBuilder.() -> Unit)
```

The `BasicCommandGroupBuilder` class contains an overridden unary plus (the + operator), which it uses to add the command to the current group. To add a command to a group being built within one of the builders listed above, simply call the `+` operator on the command and it will be added.

```Kotlin
operator fun Command.unaryPlus() = commands.add(this@unaryPlus)
```

Nore information on the qualified `this` keyword is available [here](https://kotlinlang.org/docs/reference/this-expressions.html).

In Kotlin lambda arguments should be outside of parenthesis. For this reason, while the first is _technically_ correct, it should never be used. Both end up building the same command group though.

```Kotlin
val group = sequential({
    +Foo()
    +Bar()
})

val anotherGroup = sequential {
    +Foo()
    +Bar()
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