# Build systems and Gradle

To compile user written code into code that the [roboRIO](https://frc-docs.readthedocs.io/en/latest/docs/hardware/getting-started/control-system-hardware.html#national-instruments-roborio), we use a compiler program called [Gradle](https://gradle.org/). The program is available using the `gradlew` or `gradlew.bat` scripts bundled with the code on Mac/Linux and Windows, respectively. Gradle is responsible for managing the plugins which compile the Java or Kotlin code into .jar and .class files, and the [GradleRIO plugin](https://github.com/wpilibsuite/GradleRIO), which deploys the code to the robot and handles things like remote debugging, running other WPI tools such as Shuffleboard from the command line and code simulation.

Gradle knows what to do based on settings configured in the project's `build.gradle` or `build.gradle.kts` folders. Users can [add dependencies on external libraries](https://docs.gradle.org/current/userguide/dependency_management_for_java_projects.html), manage GradleRIO settings such as debug and simulation options, [add custom repositories for dependencies](https://docs.gradle.org/current/userguide/declaring_repositories.html), and add JVM arguments for things such as heap size or debugging.

```
Note: If you're using a build.gradle, you're using Groovy; If you have a build.gradle.kts file, you're using the Gradle Kotlin DSL.
```
```
Note: when adding dependencies to be used on the RoboRIO, use the compile keyword in place of the implementation keyword. If you don't the RoboRIO will throw ClassNotFoundExceptions.
```

## Importing a project into IntelliJ

To import an existing Gradle project such as robot code into IntelliJ to work on, either click Import Project (Or File -> Open with intelliJ already open), and select the project's `build.gradle` or `build.gradle.kts` file. For the Gradle JVM, use 11 if the option exists. A video version of this is available [here](https://www.youtube.com/watch?v=wQyDk4Ji1Gk). 

## Building, testing, checking and deploying robot code through the command line

Requirements: Command line access, a robot, IntelliJ or VSCode

All of the operations needed in development can be accessed through the command line. In VSCode, click on the problem counter symbols in the bottom left corner and select `Terminal`. In IntelliJ the option is located on the bottom bar under the same name. This will spawn a terminal session currently in the root directory of your project. To get Gradle started, execute the `gradlew` or `gradlew.bat` files. On Mac, Linux and Window's Powershell program, this is done using `./gradlew`; on Windows' Terminal program, this is done with simply `gradlew`. From now on, I will assume the first case, but remove the ./ prefix if you're using Terminal, and if one doesn't work try the other. On the back, this command will tell the Gradle wrapper to create a Gradle [daemon process](https://en.wikipedia.org/wiki/Daemon_(computing)) or use an existing, idle daemon. On a Chromebook this can take over two minutes, but usually takes no more than a dozen seconds on any modern computer.

Next, build your code. If this is your first time building, Gradle will download the necessary libraries and save them for later. This can take some on slow internet connections. Building code is done with the `./gradlew build` command. The build command will build the main code and run all checks and [unit tests](https://docs.gradle.org/current/userguide/java_testing.html) against it. In the example code, this includes the `ExampleTest`'s `exampleTest()` method and the `ktlint` plugin for kotlin code format checks. If you want to exclude linting and unit tests, use the command `./gradlew build -x check`.

Once your code has built successfully, it's time to deploy it to the robot. Connect via Wifi (or USB, if on Windows) and run `./gradlew deploy` after building code. If you have not yet built or your code has changed, new changes _may_ not be deployed (but sometimes are). And that's it!

### Linting and code format checks

Gradle can run checks to make sure that code conforms to formatting standards. If your code fails to either compile or pass unit tests or format checks, you will likely see a message such as the following:

```
FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':ktlintMainSourceSetCheck'.
```

If the failure is with the `ktlint` plugin, run `./gradlew ktlintformat` to format the code automatically. If it complains about wildcard imports, disable them (it's a bug, sorry) by adding a comment to the import like this: 
```
import org.ghrobotics.lib.wrappers.hid.* // ktlint-disable no-wildcard-imports
```

If a unit test fails, the reason will be printed above. **Scroll up in the terminal to find out more**

### Side note: File permissions

A common problem on Unix-based (Linux and Mac) systems involves file permissions. By default, files are not allowed to be executed as a script unless explicitly allowed to -- hence, the `gradlew` script may not have these permissions marked by default on new projects. If so, you will see an error such as the following: `bash: ./gradlew: Permission denied`. If you do, simply give it executable permissions with the chmod program: `chmod +x gradlew`. This is not necessary on Windows. For more on file permissions, take a look [here](https://www.guru99.com/file-permissions.html) or at the first thing to come up when you google it.