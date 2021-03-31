# Using maven-jlink-plugin with JavaFX

This is a playground to use [maven-jlink-plugin](https://github.com/apache/maven-jlink-plugin) with JavaFX application.

## Problem

Using v3.1.0 of the plugin, `mvn jlink:jlink` fails with the following exception:

```
Error: automatic module cannot be used with jlink: javafx.graphicsEmpty
```

Plugin logs have the following lines in them:

```
[INFO] --- maven-jlink-plugin:3.1.0:jlink (default-cli) @ hellofx ---
[INFO]  -> module: javafx.base ( ~/.m2/repository/org/openjfx/javafx-base/13/javafx-base-13-linux.jar )
[INFO]  -> module: javafx.graphicsEmpty ( ~/.m2/repository/org/openjfx/javafx-graphics/13/javafx-graphics-13.jar )
[INFO]  -> module: hellofx ( ~/Documents/git/samples/IDE/IntelliJ/Modular/Maven/hellofx/target/classes )
[INFO]  -> module: javafx.baseEmpty ( ~/.m2/repository/org/openjfx/javafx-base/13/javafx-base-13.jar )
[INFO]  -> module: javafx.controls ( ~/.m2/repository/org/openjfx/javafx-controls/13/javafx-controls-13-linux.jar )
[INFO]  -> module: javafx.graphics ( ~/.m2/repository/org/openjfx/javafx-graphics/13/javafx-graphics-13-linux.jar )
[INFO]  -> module: javafx.fxml ( ~/.m2/repository/org/openjfx/javafx-fxml/13/javafx-fxml-13-linux.jar )
[INFO]  -> module: javafx.fxmlEmpty ( ~/.m2/repository/org/openjfx/javafx-fxml/13/javafx-fxml-13.jar )
[INFO]  -> module: javafx.controlsEmpty ( ~/.m2/repository/org/openjfx/javafx-controls/13/javafx-controls-13.jar )
```

JavaFX tries to be platform independent by hiding the platform specific jars behind empty dependencies. If we add the following dependency to an application running on Linux:

```
<dependency>
    <groupId>org.openjfx</groupId>
    <artifactId>javafx-graphics</artifactId>
    <version>13</version>
</dependency>
```

It will be resolved into `javafx-graphics-13.jar` and `javafx-graphics-13-linux.jar`. The former is an empty jar with `Automatic-Module-Name` set as `javafx.graphicsEmpty`. More information on this can be found here: http://mail.openjdk.java.net/pipermail/openjfx-dev/2018-August/022313.html

## Solution

Most Java(FX) applications with a main class will add a `launcher` configuration to the plugin. If a launcher is found, `maven-jlink-plugin` can skip adding the entire module-path to the add-modules list and add only the base module to it. This is how [javafx-maven-plugin](https://github.com/openjfx/javafx-maven-plugin)'s `jlink` goal works.

This approach may have some side-effects and needs to be explored.