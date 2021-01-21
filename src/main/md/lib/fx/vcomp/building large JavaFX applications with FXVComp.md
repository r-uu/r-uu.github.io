# Building large JavaFX applications with ```FXVComp```

Large systems are typically built from smaller components which themselves are built from even smaller components and so on. This article describes an approach how to build JavaFX applications from components conviently.

## Abstract

```FXVComp``` provides support for building visual components with JavaFX that can be designed, run and tested on their own as well as be integrated into complex UI systems. In particular ```FXVComp``` leverages CDI that facilitates loose coupling and efficient collaboration of independent subsystems. Moreover ```FXVComp``` provides support for standardisation and automation of reusable JavaFX components.

## Visual components in ```FXVComp```

A visual component in ```FXVComp``` typically consists at least of the following parts:

* A component class derived from ```de.ruu.lib.fx.comp.FXCView```,
* a ```.fxml``` file,
* a component controller class derived from ```de.ruu.lib.fx.comp.FXCViewController``` and
* a component service implementation extending ```de.ruu.lib.fx.comp.FXCViewService```.

The super types of these artifacts are provided by the ```FXVComp``` framework and will be described in the following section.

In fact there can be even more artifacts that are used by ```FXVComp``` components. These will also be described later. 

## ```FXVComp``` framework

The following picture gives an overview of the ```FXVComp``` framework:

xxxxxxxxxxxxxxxx TODO xxxxxxxxxxxxxxxx

Next there is a detailed description of the framework types.

### ```FXCView```

xxxxxxxxxxxxxxxx TODO xxxxxxxxxxxxxxxx

## Automation of recurring tasks

Regardless of using FXVComp or not ...

However, developers of non trivial JavaFX applications always have to deal with pretty much recurring tasks. ```FXVComp``` accomplishes this by a tool that automates the initial generation of skeletons for the artifacts in a very flexible manner. By default it will introduce a naming convention for the artifacts that standardises naming and helps to keep track of all the parts needed for ```FXVComp``` components. The conventions can be overwritten to meet specific needs.

In the following this article describes the mentioned framework and tooling in more detail and finally it will show how to build a small sample ```FXVComp``` component.

Code examples can be downloaded from [xxx](xxx).

A typical JavaFX application is composed from multiple artifacts. Following some naming conventions helps to keep track in this fast spreading jungle of files. Let's say we want to build an app called ```XApp```. Here is what non-trivial JavaFX apps are typically built from:

* ```XApp.java```

  A class that is derived from ```javafx.application.Application``` and that populates the ```javafx.stage.Stage``` with a ```javafx.scene.Scene```. JavaFX provides means to create a ```Scene``` instance from a declarative UI description file with the name suffix ```.fxml```: 

* ```XApp.fxml```
 
   ```.fxml``` files contain declarative descriptions of the visual parts of the UI. [Gluon Scenebuilder](https://gluonhq.com/products/scene-builder/) is a visual designer tool to create ```.fxml``` files.

* ```XAppController.java```

  A JavaFX controller is a class that initialises the UI elements defined in a ```Scene``` and controls how the app reacts on UI events.

JavaFX comes with support for convenient injection of UI components into controller instances that define the behaviour of JavaFX applications. ```@FXML``` annotated fields in controller classes do the trick. 

However, the bigger applications grow the more urgent becomes the need to split it up into smaller, better maintainable pieces. The following shows how a JavaFX app can be built from smaller apps where each app can be run and tested on its own.

## Extending basic JavaFX building blocks

With JavaFX it is possible to create large UI applications from smaller JavaFX apps. This helps to make testing much easier and contributes to overall robustness.

This approach enhances the before mentioned basic building blocks of a JavaFX application:

* ```XAppService.java```

  An interface that defines the "external" component behaviour.

* ```XAppView.java```

  A view class that provides typical UI funcionality that may be useful for collaboration with other components.

* ```XAppRunner.java```

We'll have a closer look at each of these building blocks in the following.

### Extending ```javafx.application.Application``` -  ```CDIApplication```

```CDIApplication``` is an abstract sub class of ```javafx.application.Application```.


What is missing in vanilla JavaFX is built in support for CDI. And it can be challenging to properly bootstrap a CDI container without harmfully interferring JavaFX's startup procedure. The following chapter describes an approach that hides CDI bootstrapping and takes care of recurring tasks for JavaFX development.

that support injection via @FXML annotations and CDI.

I built a couple of JavaFX components and found myself doing the same things again and again. So I decided to automate these efforts. Here is a description for a use of a generator I have built for these tasks.

A typical 