# Debugging robot code

Requirements: Command line access, [VisualVM](https://visualvm.github.io/download.html), a robot

Start by modifying the gradle build file's JVM arguments. The necessary arguments are:
```
"-Dcom.sun.management.jmxremote=true",
"-Dcom.sun.management.jmxremote.port=1099",
"-Dcom.sun.management.jmxremote.local.only=false",
"-Dcom.sun.management.jmxremote.ssl=false",
"-Dcom.sun.management.jmxremote.authenticate=false",
"-Djava.rmi.server.hostname=10.59.40.2"
```

For example, within the artifact block in the Kotlin DSL, ensure you have the following JVM args:
```
jvmArgs = listOf(
        "-Xmx20M",
        "-XX:+UseG1GC",
        "-Dcom.sun.management.jmxremote=true",
        "-Dcom.sun.management.jmxremote.port=1099",
        "-Dcom.sun.management.jmxremote.local.only=false",
        "-Dcom.sun.management.jmxremote.ssl=false",
        "-Dcom.sun.management.jmxremote.authenticate=false",
        "-Djava.rmi.server.hostname=10.59.40.2"
)
```

