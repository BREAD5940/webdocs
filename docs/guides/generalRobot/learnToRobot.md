# Learning how to program a robot

If you're reading this, it's probably your first time programming a robot. Congratulations. You are indeed cool.

Before coding a robot, you should already have a background in Object Oriented Programming of some kind. If not, refer to (TODO LINK OR SMTH)

We currently program our robots in Kotlin, a language written for the Java Virtual Machine. It's similar to Java, with a focus on being more concise, null safe and interoperable with other JVM languages such as Java. We also use the Kotlin DSL for Gradle in our build scripts. Learn to code it at https://kotlinlang.org/docs/tutorials/, or [the X in Y minutes writeup](https://learnxinyminutes.com/docs/kotlin/). It's not necessary to know Kotlin in order to learn how to program (it's sometimes easier to learn about robot code and then learn Kotlin), but it's encouraged to know it in addition to Java. An example Kotlin codebase is available for download from [it's Github repo](https://github.com/BREAD5940/Kotlin-Example).

## WPILib and other libraries

If you already know OO Java/Kotlin, you're ready to get started messing with real robot code. Documentation of WPILib, the framework we use to program robots, is available [on FRC-docs](https://frc-docs.readthedocs.io/en/latest/index.html). While FRC-docs does have [a short intro to robot code](https://frc-docs.readthedocs.io/en/latest/docs/software/getting-started-with-benchtop/creating-benchtop-test-program-cpp-java.html), it's not great, so please refer to [Writing your first robot program](docs/guides/firstProgram).

We also use [FalconLibrary](docs/guides/falconlib/intro), the Kotlin-based library for FRC by Team 5190, to help program the robot. The library offers features such as a common API for all motors (including simulated motors), built in code for autonomous path following, a typesafe unit library and much more. Other external libraries used in 2019 include kotlinX for coroutines, jSerialComm for Jevois serial, kotson for JSON, mockito for mocking hardware in tests, ktlint for code formatting and Junit for unit testing.

## Code structure and design patterns

Our robot code is usually written using the Command-based pattern. Under Command-based, robot `Subsystems` are used by `Commands`, which use hardware from these subsystems to make the robot move and build more complicated command groups. Using Command-based improves code readability and removes gigantic, unreadable and confusing switch statements from a subsystem's code. More information on the Command-based robot framework is available [on FRC-docs](https://frc-docs.readthedocs.io/en/latest/docs/software/commandbased/index.html). If you haven't yet, the bare bones example Kotlin code can be downloaded from [it's GitHub repo](https://github.com/BREAD5940/Kotlin-Example).

Code is structured in subfolders within the `src` folder in the root of the project. For more on code structure, see [this writeup](docs/guides/codeStructure).

In general, working on robot code is 10% coding and 90% debugging and tuning. Through resources such as WPILib and FalconLibrary, very little real and difficult code remains by the time you actually sit down to write a program. You'll spend most of your time getting familiar with the features offered by both libraries and how they integrate to create a moving and functioning robot. For example, we don't have to write code to interface directly with the [Pneumatics Control Module](https://frc-docs.readthedocs.io/en/latest/docs/software/actuators/pneumatics.html) to control solenoids, but instead use WPILib's `DoubleSolenoid` or FalconLibrary's `FalconDoubleSolenoid` classes. The code has already been written and includes features to make using them as simple as possible -- all that you have to do is give it the right settings, and the rest is handled for you, behind the scenes.

## Very Very Intro Stuff

The following articles hosted here should help get your development environment set up as well as get started writing robot code.

- [Download and Install WPILib](https://github.com/wpilibsuite/allwpilib/releases)
- [Install IntelliJ](https://www.jetbrains.com/idea/download)
- [How to use Git](http://imgs.xkcd.com/comics/git.png)
- [Importing a building projects with Gradle](docs/guides/generalRobot/introToGradle)

## Beginning Stuff
- [What is this WPILib library anyways?](https://frc-docs.readthedocs.io/en/latest/docs/software/wpilib-overview/what-is-wpilib.html)
- [What's "command-based programming" in WPILib?](https://frc-docs.readthedocs.io/en/latest/docs/software/commandbased/what-is-command-based.html)
- [What actuators can I use to make stuff move?](https://frc-docs.readthedocs.io/en/latest/docs/software/actuators/overview.html)
- [Robot Electronic Components](https://frc-docs.readthedocs.io/en/latest/docs/hardware/getting-started/control-system-hardware.html)
- [Making a motor spin](https://frc-docs.readthedocs.io/en/latest/docs/software/actuators/using-speed-controllers.html)
- [Making a piston fwoosh](https://frc-docs.readthedocs.io/en/latest/docs/software/actuators/pneumatics.html)
- [Driving around](docs/guides/generalRobot/codeARobot/driving)

## Stuff you Should do Second (Advanced Stuff)
- [What is Vision?](docs/guides/generalRobot/vision)
- [Arm PID -- a showcase of the different Talon modes](docs/guides/generalRobot/codeARobot/basicArm)
- [Driving at a velocity with PID+F on Talons](docs/guides/generalRobot/codeARobot/driveVeloPid)