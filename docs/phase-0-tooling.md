# Phase 0 – Tooling

**Time:** about 6–8 hours  
**Goal:** prepare a Java/Kotlin development environment and build a small Java program and a small Kotlin program from the command line.

This phase is for an experienced .NET and TypeScript developer. Keep the .NET comparison in mind, but focus on learning the Java and Kotlin toolchain rather than installing every optional IDE feature.

## 1. Install the prerequisites

Install the following using the official installation instructions for your operating system:

- [Eclipse Temurin JDK 21](https://adoptium.net/temurin/releases/). Install a **JDK**, not just a JRE: the JDK includes the compiler.
- [Git](https://git-scm.com/downloads).
- [Docker Desktop](https://www.docker.com/products/docker-desktop/) (or Docker Engine on Linux).
- [Azure CLI](https://learn.microsoft.com/cli/azure/install-azure-cli).
- [IntelliJ IDEA](https://www.jetbrains.com/idea/download/) and [Android Studio](https://developer.android.com/studio).
- The [Kotlin command-line compiler](https://kotlinlang.org/docs/command-line.html), for the Kotlin command-line exercise below.

You do not need an Azure subscription or a running Docker container to complete this phase. They will be used in later phases.

Open a new terminal after installation and check that the tools are available:

```shell
java -version
javac -version
git --version
docker --version
az version
kotlinc -version
```

Confirm that `java` and `javac` report version 21. If a command is not found, check the installation and `PATH` configuration, then open a fresh terminal. Do not configure production credentials or put secrets in source control.

## 2. Choose and configure an IDE

Use **IntelliJ IDEA** for the Java backend and **Android Studio** for Kotlin and Android work. Android Studio is based on IntelliJ IDEA and includes Android-specific tools such as the SDK manager, emulator, layout inspector, and Compose preview. The IDEs provide code completion, navigation, refactoring, debugging, test running, and direct access to Maven or Gradle tasks.

In IntelliJ IDEA, open a project from its directory, select the installed JDK 21 as the project SDK, and use the Maven tool window for Maven projects. In Android Studio, create or open an Android project, install the Android SDK components it requests, and use the Gradle tool window to sync and run Gradle tasks. In both IDEs, use the integrated terminal to run the same commands you would use in a standalone shell; this makes IDE and command-line results easier to compare.

Useful starting shortcuts (the exact key combination can vary by operating system and keymap):

| Action | IntelliJ IDEA / Android Studio |
|---|---|
| Search actions and settings | Double Shift |
| Find a class or file | `Ctrl+N` / `Ctrl+Shift+N` (`⌘+O` / `⌘+Shift+O` on macOS) |
| Find usages | `Alt+F7` (`⌥+F7` on macOS) |
| Rename symbol | `Shift+F6` |
| Reformat code | `Ctrl+Alt+L` (`⌥+⌘+L` on macOS) |
| Run the current configuration | `Shift+F10` (`Ctrl+R` on macOS) |

Use the IDE's **Find Action** feature to look up an action if your keymap differs.

### VS Code and plugins

VS Code remains a reasonable alternative for editing, Git, and small command-line projects, especially when using a familiar setup. IntelliJ IDEA and Android Studio are recommended here because their Java, Spring, Kotlin, Gradle, and Android workflows are integrated and need less manual configuration. Prefer Android Studio for Android-specific tasks even if you use VS Code elsewhere.

Install only the support you need:

- Use IntelliJ IDEA's Spring support for backend navigation and configuration assistance. Spring support is available through the IDE's bundled features or plugins depending on edition.
- Kotlin support is included in IntelliJ IDEA and Android Studio.
- Android tools are provided by Android Studio; install SDK platforms and emulator images from its SDK Manager.
- Use the IDE's database tools to inspect a database when available in your edition, or use a separate database client.
- For code style, choose one formatter/linter for a project: for example, Spotless to apply formatting or Checkstyle to enforce Java rules. These are normally configured in the build, so command-line builds and CI use the same rules; don't add both without a project need.

## 3. Understand Maven and Gradle

Both tools resolve dependencies, run tests, and produce build outputs, but each project uses one build definition:

| | Maven | Gradle with Kotlin DSL |
|---|---|---|
| Main configuration | `pom.xml` | `build.gradle.kts` (and often `settings.gradle.kts`) |
| Typical source layout | `src/main/java`, `src/test/java` | JVM projects use `src/main/kotlin` and `src/test/kotlin`; Android code is usually under `app/src/main/` |
| Familiar comparison | Convention-driven build, somewhat like MSBuild with NuGet | Flexible task graph and build scripts, also serving the role of MSBuild |
| Dependency declaration | Coordinates and scopes in the POM | Dependencies in Gradle configurations such as `implementation` and `testImplementation` |
| Typical commands | `./mvnw test`, `./mvnw package` | `./gradlew test`, `./gradlew build` |
| Project wrapper | `mvnw` / `mvnw.cmd` | `gradlew` / `gradlew.bat` |

NuGet is closest to the dependency-resolution part of Maven or Gradle; it is not itself the .NET build engine. Prefer a project's checked-in Maven or Gradle wrapper over a globally installed build tool. The wrapper selects the project’s expected tool version, so the IDE and command line run consistent builds. The IDE can expose build tasks in a tool window, but the wrapper command is the portable way to run them and the one used by CI.

Later phases will add the real `backend/` Maven and `android/` Gradle projects. For now, notice that a dependency is declared in the build file rather than downloaded and copied into the source tree. Build files are also where project-wide compiler, test, and formatting settings belong.

## 4. Build Java from the command line

Create a working directory and save this as `Hello.java`:

```java
public class Hello {
    public static void main(String[] args) {
        System.out.println("Hello, Java!");
    }
}
```

Compile and run it:

```shell
javac Hello.java
java Hello
```

`javac` compiles the source to a `.class` file; `java` runs that class. This small example uses the JDK directly, without Maven.

## 5. Build Kotlin from the command line

Save this as `Hello.kt` in a working directory:

```kotlin
fun main() {
    println("Hello, Kotlin!")
}
```

Compile it into a runnable JAR and run it:

```shell
kotlinc Hello.kt -include-runtime -d hello.jar
java -jar hello.jar
```

`kotlinc` compiles Kotlin, and `-include-runtime` packages the Kotlin runtime so the JAR can be run with Java. This exercise uses the Kotlin compiler directly, without Gradle.

## 6. Practice with the tools

1. Use the IDE project wizard to create a basic Java project in IntelliJ IDEA and an Empty Activity project in Android Studio. You do not need to add app features yet; the Android application is built in Phase 4.
2. Change each greeting, rebuild, and run again.
3. Locate the terminal, project view, and build-tool window in each IDE. Use Find Action to locate the formatting action, then use Rename on a symbol in a small edit.
4. Locate the Maven and Gradle wrapper commands in the table above. Explain which project in the planned course will use each one and why a wrapper is useful.
5. In your own words, compare NuGet, Maven, and Gradle dependency declarations and identify where each tool's build configuration lives.

## Definition of done

- The prerequisite version commands succeed; Java and `javac` report JDK 21.
- Both programs print their greeting when built and run from the command line.
- You can open a Java project in IntelliJ IDEA and an Android/Kotlin project in Android Studio, and can find the terminal and build-tool window in each.
- You can explain when you would use Maven versus Gradle, what the wrapper does, and how their roles relate to MSBuild and NuGet.
