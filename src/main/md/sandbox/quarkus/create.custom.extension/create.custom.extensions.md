One day I decided to explore quarkus.

# exploring quarkus

First things first - hello world:

## create minimal quarkus app

Thanks to [quarkus application generator](https://code.quarkus.io/) I got my first app running within minutes.

```java
package app;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/hello-resteasy")
public class GreetingResource
{
	@GET
	@Produces(MediaType.TEXT_PLAIN)
	public String hello()
	{
		return "Hello RESTEasy";
	}
}
```

For my new app I wanted to reuse a regular custom library that happily lives in a project outside my quarkus application and is available in my local maven repository.

## create minimal custom library

```java
package lib;

public class MathOps
{
	public final static int add(int a, int b) { return a + b; }
}
```

## include library to dependencies of quarkus app

To make my library available in my quarkus app I added it as a maven dependency into pom.xml of my app.

```xml
<dependency>
    <groupId>ruu.io</groupId>
    <artifactId>sandbox-quarkus-create.custom.extension-lib</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

## use custom library in quarkus app

I modified my quarkus app to use my custom library like that:

```java
package app;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import de.ruu.sandbox.quarkus.extension.lib.MathOps;

@Path("/hello")
public class GreetingResource
{
	@GET
	@Produces(MediaType.TEXT_PLAIN)
	public String hello()
	{
		return "1 + 1 = " + MathOps.add(1, 1);
	}
}
```

At first glance everything seemed to be fine. At least this compiled perfectly in my IDE.

## quarkus complaints

However, this turned out to be a too simplistic approach. Using a regular library as regular maven dependency is not the quarkus-style of using libraries!

Trouble started when I compiled the project with:

> `mvn compile`

Maven output was something like this:

!["could not show maven complaints"](https://lh3.googleusercontent.com/-qzsR7aahX6M/X_Gvs_IoiMI/AAAAAAAAAGg/-ww2AMg_TJcg5M4FLede06mI1So3wqbSACLcBGAsYHQ/h120/mvn-output-compile.with.regular.lib.png "maven complaints")

I suspected that it would be necessary to create a custom quarkus extension to overcome these issues.

## quarkus rescue

I planned to work wiht the following maven project (directory) structure that keeps all related parts of my app together in a common maven parent project:

```
- project   maven parent project (packaging "pom") for the related maven child projects
  - app       maven child project for app (packaging "jar")
  - lib       maven child project for lib (packaging "jar")
  - lib-qxt   maven child project for qxt (`q`uarkus e`xt`ension, packaging "pom")
```

The `qxt` project was generated with the following command issued in the project directory:

```
mvn io.quarkus:quarkus-maven-plugin:1.10.5.Final:create-extension -N -DgroupId=ruu.io -DartifactId=ruu.io-sandbox-quarkus-create.custom.extension-lib-qxt -Dversion=0.0.1-SNAPSHOT -Dquarkus.nameBase="lib-qxt"
```

The generated name of the new project directory corresponds to the maven artifact name `ruu.io-sandbox-quarkus-create.custom.extension-lib-qxt`. For me this is ok for the artifact name but I dislike such a long name for a sub directory so I manually changed the directory name to `lib-qxt` (standing for `q`uarkus e`xt`ension). To keep the maven project hierarchy in sync I also had to manually adjust

```xml
<modules>
	<module>app</module>
	<module>lib</module>
	<module>ruu.io-sandbox-quarkus-create.custom.extension-lib-qxt</module>
</modules>
```

in `project/pom.xml` like this:

```xml
<modules>
	<module>app</module>
	<module>lib</module>
	<module>lib-qxt</module>
</modules>
```

After this the maven project (directory hierarchy) looked like this:

```
- project        parent project for the related maven child projects
  - app            project for app
  - lib            project for lib
  - qxt            project for qxt (generated, artifactId ...-parent)
    - deployment      project for qxt deployment (generated, artifactId ...-deployment)
    - runtime         project for qxt runtime    (generated, artifactId ...-runtime)
```

I hoped to have an empty but 