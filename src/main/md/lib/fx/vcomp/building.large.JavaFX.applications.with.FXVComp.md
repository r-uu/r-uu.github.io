# Building large JavaFX applications with ```FXVComp```

Large systems are typically built from smaller components which themselves are built from even smaller components and so on. This article describes an approach called ```FXVComp```. ```FXVComp``` allows convenient building of complex JavaFX applications from components.

What is missing in vanilla JavaFX is built in support for CDI. And it can be challenging to properly bootstrap a CDI container without harmfully interfering JavaFX's startup procedure ...

---

## Abstract

```FXVComp``` provides support for building visual components with JavaFX that can be designed, run and tested on their own as well as be integrated into complex UI systems. In particular ```FXVComp``` leverages CDI that facilitates loose coupling and efficient collaboration of independent subsystems. Moreover ```FXVComp``` provides support for standardisation and automation for building reusable JavaFX components.

---

# Visual components in ```FXVComp```

A visual component in ```FXVComp``` typically consists at least of the following parts:

* A ```.fxml``` file with the declarative description of the UI,
* a component class implementing ```de.ruu.lib.fx.comp.FXCView```,
* a component controller class implementing ```de.ruu.lib.fx.comp.FXCViewController``` that defines the behaviour of the component and
* a component service implementing ```de.ruu.lib.fx.comp.FXCViewService```.

Each part will be described in detail in one of the following sections.

> In fact there can be even more artifacts that are used by ```FXVComp``` components. These will also be described later. 

---

# ```FXVComp``` framework

Before describing the naming conventions and showing the ease of use of ```FXVComp``` here is an overview of the framework:

![```FXVComp``` framework overview](de.ruu.lib.fx.comp.png)

Although they are rather tiny the interfaces on the left define the main parts of the framework. 

In the next section the ```FXVComp``` naming conventions will be described briefly.

---

# ```FXVComp``` naming conventions

The <code>FXComp</code> naming conventions support automation of recurring tasks when creating visual components. For a visual component named ```X``` the conventions expect:

* component class name: <code>X</code>
* component service interface name: <code>XService</code>
* component controller class name: <code>XController</code>
* application class name: <code>XApp</code>
* application runner class name: <code>XAppRunner</code>

These naming conventions can be overwritten in the classes described later.

Before going too much into details the following demonstrates the ease of use of ```FXVComp``` components.

---

# Using the ```FXVComp``` framework

The following picture shows how the types of the ```FXVComp``` framework were extended for a new ```FXVComp``` component named ```X```.

![```FXVComp``` framework overview](de.ruu.lib.fx.comp.X.png)

When complying to the ```FXVComp``` naming conventions building a small but complete visual component is really easy. For creating a component named ```X``` the following simple steps have to be made.

> Moreover ```FXVComp``` comes with a generator tool, that creates skeletons for the necessary artifacts. This tool will be described later on.

---

## Create ```X.fxml``` file

```.fxml``` files contain declarative descriptions of the visual parts of the UI. [Gluon Scenebuilder](https://gluonhq.com/products/scene-builder/) is a visual designer tool to create ```.fxml``` files. The picture shows the visual appearance of the X component in scenebuilder.

![```X.fxml``` in scenebuilder](scenebuilder.png)

Here is the ```xml``` source code of ```X.fxml```generated by scenebuilder:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<?import javafx.scene.control.Button?>
<?import javafx.scene.control.Label?>
<?import javafx.scene.layout.AnchorPane?>

<AnchorPane maxHeight="-Infinity" maxWidth="-Infinity" minHeight="-Infinity" minWidth="-Infinity" prefHeight="165.0" prefWidth="228.0" xmlns="http://javafx.com/javafx/11.0.1" xmlns:fx="http://javafx.com/fxml/1">
   <children>
      <Label layoutX="33.0" layoutY="14.0" text="this is FXComp component X" />
      <Button fx:id="button" layoutX="80.0" layoutY="41.0" mnemonicParsing="false" text="click me" />
      <AnchorPane fx:id="anchorPane" layoutX="22.0" layoutY="79.0" prefHeight="73.0" prefWidth="174.0" />
   </children>
</AnchorPane>
```

---

## Implement ```FXCView```

If the default naming conventions suffice implementing a ```FXCView``` is as simple as:

```java
public class X extends DefaultFXCView { }
```

There is nothing more to be done but creating an (initially) empty class that ```extends DefaultFXCView```. Through extending ```DefaultFXCView``` the new ```X``` components inherits functionality that lets it directly be available for integration into large JavaFX applications.

---

## Implement ```FXCViewController```

```X.fxml``` contains the declaration of four JavaFX components. Two of them (```button``` and ```anchorPane```) have a ```fx:id``` which makes them available in a controller class via JavaFX injection and the ```@FXML``` annotation.

Here is an implementation for ```XController```:

```java
public class XController extends DefaultFXCViewController
{
	@FXML private Button button;
	@FXML private AnchorPane anchorPane;

	@Override
	@FXML
	protected void initialize()
	{
		button.setOnAction(e -> System.out.println("button pressed in X component"));
	}
}
```

When clicking the button a message is printed to the console.

---

## Implement ```FXCViewService```

As long as a ```FXVComp``` component does not provide a particular serice, writing a service interface is as simple as:

```java
public interface XService extends FXCViewService { }
```

Obviously this is the case only for the most simple use cases.

---

## Implement ```FXCApp```

Implementing a ```FXCApp``` is pretty easy again. ```FXCApp``` is a sub class of ```javafx.application.Application``` so this class is the entry point for a JavaFX application that can be run on its own.

```java
public class XApp extends FXCApp { }
```

As the code snippet shows if the default functionality inherited by ```FXCApp``` suffices there is nothing left to do. The next section will show how to run the new ```FXVComp``` component.

## Implement ```FXCAppRunner```

Finally there is a "runner" class that starts the new ```FXVComp``` component as JavaFX application:

```java
public class XAppRunner extends FXCAppRunner 
{
	public static void main(String[] args)
	{
		FXCAppRunner.run(XApp.class, args);
	}
}
```

---

## Summary

The preceding sections show that implementing a new ```FXVComp``` component is pretty easy. In fact for a simple example as above there are only a few lines of code that have to be written in a few classes or interfaces. However, ```FXVComp``` comes with a code generator that creates code skeletons for all the necessary classes and interfaces. The next section describes the code generator.

---

# ```FXVComp``` Code Generator

Though implementing a new ```FXVComp``` component is really easy it involves the creation of quite a few artifacts that should comply to naming conventions. Because complex JavaFX applications easily consist of many dozens of components things become a little hard to keep track of quickly.

To accomplish this ```FXVComp``` comes with a generator tool that automatically creates skeletons for the described artifacts and thereby relieve the developer from a lot of recurring and error prone manual tasks.

To automatically create artifact skeletons a generator can be created by extending ```GeneratorFXCompBundle``` as follows:

```java
public class XCompGenerator extends GeneratorFXCompBundle
{
	public XCompGenerator(String packageName, String simpleFileName)
	{
		super(packageName, simpleFileName);
	}

	public static void main(String[] args) throws GeneratorException
	{
		XCompGenerator generator;
		generator = new XCompGenerator(XCompGenerator.class.getPackageName(), "X");
		generator.run();
	}
}
```

> Note that by default ```GeneratorFXCompBundle``` generates artifact skeletons into directories under ```src/gen/java``` and ```src/gen/resource``` to avoid that manual adjustments to the artifacts be overwritten. Therefore generated artifacts have to be checked and copied carefully by hand into ```src/main/java``` and ```src/main/resource```!

---

# Brief description of the framework types

---

## ```FXCView```

In ```FXVComp``` a visual component implements ```FXCView```. Therefore it has to provide only two methods: ```getLocalRoot()``` and ```getService()```. In case the visual component does not expose any specific services the latter method can be implemented by just returning an empty implementation of ```FXCViewService```.

```getLocalRoot()``` provides a ```javafx.scene.Parent``` object that represents the root of the component's tree of nodes. The tree of nodes defines the visual appearance of the component.

---

## ```DefaultFXCView```

```DefaultFXCView``` is an abstract implementation of ```FXCView``` that leverages the ```FXVComp``` naming conventions to automate the bootstrapping of all the parts that contribute to the overall features of a ```FXVComp``` component. You can adjust the bootstrapping mechanisms by overriding protected methods.

In this implementation ```getLocalRoot()``` loads the component's tree of nodes from an <code>.fxml</code> file. It looks for the file by leveraging the ```FXVComp``` default naming conventions (see
javadoc for package ```de.ruu.lib.fx.comp```) or the overridden return value from ```getFXLMResourceName()```.

You can run and test implementations of ```DefaultFXCView``` conveniently with ```FXCApp``` and ```FXCAppRunner```.

---

## ```FXCApp```

```FXCApp``` is an abstract sub class of JavaFX ```javafx.application.Application```s with CDI support. <code>FXCApp</code>s provide convenient support for automatic initialisation of various parts of JavaFX applications with CDI support.

CDI is bootstrapped from ```FXCAppRunner```s (see below).

While ```FXCApp``` instances itself are not CDI managed, the above mentioned ```DefaultFXCView``` and its ```FXCViewController``` objects can be managed by CDI. This makes CDI available for JavaFX applications while preserving the benefits of JavaFX injection via <code>@FXML</code> annotations.

---

## ```FXCAppRunner```

```FXCAppRunner``` is an abstract base class for classes that launch ```FXCApp```s. They intercept the usual JavaFX application start procedure in order to bootstrap CDI before they call ```Application#launch(Class, String...)``` of ```FXCApp```. This continues the usuall JavaFX start procedure for JavaFX applications.

As a result ```FXCApp```s do not have to take care of any CDI bootstrapping and still have full CDI support. The same applies to the other parts of the ```FXVComp``` framework.

---

# Aggregating ```FXVComp``` components to create large JavaFX applications

To demonstrate aggregation of ```FXVComp``` components to bigger systems the following shows, how to populate the ```AnchorPane``` of the ```X``` component from the above example with tree of nodes from a second component called ```Y```.

```Y``` can be created in the same way as ```X``` by generating skeleton artifacts with another ```FXVComp``` code generator and adjusting these artifacts to meet the requirements of the new ```FXVComp``` component.

Here is how an instance of ```Y``` is CDI injected into the controller f 

```java
public class XController extends DefaultFXCViewController 
{
	@FXML private Button button;
	@FXML private AnchorPane anchorPane;

	@Inject private Y y;

	@Override
	@FXML
	protected void initialize()
	{
		Parent root = y.getLocalRoot();
		anchorPane.getChildren().add(root);
		button.setOnAction(e -> onAction(e));
	}

	private void onAction(ActionEvent e)
	{
		CDI.current().getBeanManager().fireEvent(new XButtonPressed(this, "hello world"));
	}
}
```
xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx

a second component named ```Y``` is being generated. In the ```X``` component there is an ```AnchorPane``` that is not populated so far.

## create event type
## fire event
## observe event