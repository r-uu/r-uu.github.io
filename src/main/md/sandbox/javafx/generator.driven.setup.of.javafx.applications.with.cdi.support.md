# Building large JavaFX applications with loosely coupled, self-contained visual components using CDI

Large systems are typically constructed from smaller systems. CDI facilitates loose coupling and efficient collaboration of independent systems. This article describes an approach to make CDI available in JavaFX applications.

When building JavaFX applications there are many recurring tasks that can be accomplished in a standardised and automatic manner. This can be achieved by using some abstractions of standard JavaFX classes and adherence of some naming conventions, that allow to automate these recurring tasks.

Moreover for large systems a way to colla loosely coupled way to  

# Setup JavaFX applications with support for ```@FXML``` and ```@Inject``` CDI injection automatically

JavaFX comes with support for convenient injection of UI components into controller instances that define the behaviour of JavaFX applications. ```@FXML``` annotated fields in controller classes do the trick. What is missing in vanilla JavaFX is support for CDI that allows to combine ```@FXML``` annotated fields with CDI ```@Inject``` annotated elements. This article describes how to automatically add CDI support together with some features that help developing JavaFX applications as reusable UI components.

## Standard building blocks of JavaFX applications

A typical JavaFX application is composed from multiple artifacts. Following some naming conventions helps to keep track in this fast spreading jungle of files. Let's say we want to build an app called ```XApp```. Here is what non-trivial JavaFX apps are typically built from:

* ```XApp.java```

  A class that is derived from JavaFX ```Application``` and that populates the JavaFX ```Stage``` with a JavaFX ```Scene```. JavaFX provides means to create such a ```Scene``` from a declarative UI description file with the name suffix ```.fxml```: 

* ```XApp.fxml```
 
   A ```XML``` file  that contains a declarative description of the visual parts of the UI. [Gluon Scenebuilder](https://gluonhq.com/products/scene-builder/) is a visual designer tool to create ```.fxml``` files.

* ```XAppController.java```

  A JavaFX controller is a class that initialises the UI elements defined in a JavaFX ```Scene``` and controls how the app reacts on UI events.

With JavaFX it is possible to create apps that consist of loosely coupled, reusable UI-components. Therefore these basic building blocks can be enhanced by additional parts such as:

* ```XAppService.java```

  An interface that defines the "external" component behaviour.

* ```XAppView.java```

  A view class that provides typical UI funcionality for other components.

We'll have a closer look at each of these building blocks in the following.

### Extending ```javafx.application.Application``` -  ```CDIApplication```

```CDIApplication``` is an abstract sub class of ```javafx.application.Application```.

 that support injection via @FXML annotations and CDI.

I built a couple of JavaFX components and found myself doing the same things again and again. So I decided to automate these efforts. Here is a description for a use of a generator I have built for these tasks.

A typical 