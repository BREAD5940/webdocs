# Learning how to program a robot

If you're reading this, it's probably your first time programming a robot. Congratulations. You are indeed cool.

Before coding a robot, you should already have a background in Object Oriented Programming of some kind. If not, refer to (TODO LINK OR SMTH)

We currently program our robots in Kotlin, a new language written for the Java Virtual Machine. It's similar to Java, with a focus on being more concise, null safe and interoperable with other JVM languages such as Java. We also use the Kotlin DSL for Gradle in our build scripts. Learn to code it at https://kotlinlang.org/docs/tutorials/, or [the X in Y minutes writeup](https://learnxinyminutes.com/docs/kotlin/). It's not necessary to know Kotlin in order to learn how to program (it's sometimes easier to learn about robot code and then learn Kotlin), but it's encouraged to know it in addition to Java. An example Kotlin codebase is available for download at the following link: <a href="files/Kotlin-Example-Command-Based.zip" download="Kotlin-Example-Command-Based.zip">Download it here.</a>

# WPILib and other libraries

If you already know OO Java/Kotlin, you're ready to get started messing with real robot code. Documentation of WPILib, the framework we use to program robots, is available [on FRC-docs](https://frc-docs.readthedocs.io/en/latest/index.html). While FRC-docs does have [a short intro to robot code](https://frc-docs.readthedocs.io/en/latest/docs/software/getting-started-with-benchtop/creating-benchtop-test-program-cpp-java.html), it's not great, so please refer to [Writing your first robot program](docs/guides/firstProgram).

We also use [FalconLibrary](docs/guides/falconlib/intro), the Kotlin-based library for FRC by Team 5190, to help program the robot. The library offers features such as a common API for all motors (including simulated motors), built in code for autonomous path following, a typesafe unit library and much more. Other external libraries used in 2019 include kotlinX for coroutines, jSerialComm for Jevois serial, kotson for JSON, mockito for mocking hardware in tests, ktlint for code formatting and Junit for unit testing.

# Code structure and design patterns

Our robot code is usually written using the Command-based pattern. Under Command-based, robot `Subsystems` are used by `Commands`, which use hardware from these subsystems to make the robot move and build more complicated command groups. Using Command-based improves code readability and removes gigantic, unreadable and confusing switch statements from a subsystem's code. More information on the Command-based robot framework is available [on FRC-docs](https://frc-docs.readthedocs.io/en/latest/docs/software/commandbased/index.html). A ZIP file of a bare bones Kotin command based framework is available for download <a href="files/Kotlin-Example-Command-Based.zip" download="Kotlin-Example-Command-Based.zip">here</a>. 

Code is structured in subfolders within the `src` folder in the root of the project. For more on code structure, see [this writeup](docs/guides/codeStructure).

In general, robot code is 10% coding and 90% debugging and tuning. Through resources such as WPILib and FalconLibrary, very little real and difficult code remains by the time you actually sit down to write a program. You'll spend most of your time getting familiar with the features offered by both libraries and how they integrate to create a moving and functioning robot. For example, we don't have to write code to interface directly with the [Pneumatics Control Module](https://frc-docs.readthedocs.io/en/latest/docs/software/actuators/pneumatics.html) to control solenoids, but instead use WPILib's `DoubleSolenoid` or FalconLibrary's `FalconDoubleSolenoid` classes. The code has already been written and includes features to make using them as simple as possible -- all that you have to do is give it the right settings, and the rest is handled for you, behind the scenes.
