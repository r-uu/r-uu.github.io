# Building large JavaFX applications with CDI

Large systems are typically built from smaller systems. CDI facilitates loose coupling and efficient collaboration of independent subsystems. This article describes an approach to make CDI conveniently available in JavaFX applications.

When building JavaFX applications there are many recurring tasks to be accomplished. Another goal of the approach described here is to show how automation and standardisation of these tasks can be achieved by
* following a few naming conventions for typical JavaFX artifacts such as JavaFX application and controller classes, ```.fxml``` files, ...,
* making use of some predefined framework classes that provide useful additional functionality instead of using JavaFX classes such as JavaFX application class ... directly and
* leveraging a code generation tool that automates the creation of skeletons for the aforementioned artifacts.

Code examples can be downloaded from [xxx](xxx).

## Setting up vanilla JavaFX applications

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