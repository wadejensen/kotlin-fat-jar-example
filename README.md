### Kotlin build fat jar in Gradle DSL

To build a fat jar which includes all Kotlin project dependencies, we need to add 
a "fatJar" task as a dependency to the gradle build step.

Add the following to your `build.gradle.kts` file: 

```
import org.gradle.jvm.tasks.Jar

val fatJar = task("fatJar", type = Jar::class) {
    baseName = "${project.name}-fat"
    // manifest Main-Class attribute is optional.
    // (Used only to provide default main class for executable jar)
    manifest {
        attributes["Main-Class"] = "example.HelloWorldKt" // fully qualified class name of default main class
    }
    from(configurations.runtime.map({ if (it.isDirectory) it else zipTree(it) }))
    with(tasks["jar"] as CopySpec)
}

tasks {
    "build" {
        dependsOn(fatJar)
    }
}
```

`baseName` is the name of the resulting fat jar.
If a project is called `hello-kotlin`, the default skinny jar is `hello-kotlin.jar`.
The fat jar will in turn be named `hello-kotlin-fat.jar` in this example.

To change the ${project.name} variable you can override it in settings.gradle.kts 

```settings.gradle.kts
rootProject.name = "hello-kotlin"
```

Then run `gradle build` from the project root to compile the project and create the jar artifact.
By default it is created under: "build/libs/project-name-fat.jar"

To run the default main class in the fat jar file:

```
> java -jar build/libs/hello-kotlin-fat.jar
Hello, world!
```


To run another main class:

```
> java -cp build/libs/hello-kotlin-fat.jar example.GoodbyeWorldKt
Goodbye, world!
```

Note that because Kotlin does not require a main function to be declared within a class,
the class name for the function in GoodbyeWorld.kt is just the filename with 'Kt' appended.

#### How does the dependency packaging work?
Black magic. Sorry, I was just starting with Kotlin and Gradle when this was written.
It seems to recursively copy directories of runtime dependencies into the output 
jar artifact.
