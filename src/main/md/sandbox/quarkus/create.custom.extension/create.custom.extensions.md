One day I decided to explore quarkus.

# exploring quarkus

 Quickly I arrived at a point where I wanted to reuse a custom java library. It turned out that this involves more than just the usual adding of another dependency to a maven `pom.xml`. What you need is to create a custom quarkus extension for your library. The reasons for this are explained [here](https://quarkus.io/guides/writing-extensions).

I decided to create a minimal POC for a minimal custom quarkus extension. So I started with the inevitable - hello world:

## create a minimal quarkus app - hello world

Thanks to [quarkus application generator](https://code.quarkus.io/) I got a minimal quarkus app running within minutes. Alternatively you can create the app with maven from the command line:

> [Create and ]Change to a (new) directory of your choice.

```
mvn io.quarkus:quarkus-maven-plugin:1.10.5.Final:create -DprojectGroupId=ruu.io -DprojectArtifactId=app -DclassName="app.GreetingResource" -Dpath="/hello"
```

This is the resulting minimal quarkus app:

```java
package app;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

@Path("/hello")
public class GreetingResource
{
	@GET
	@Produces(MediaType.TEXT_PLAIN)
	public String hello()
	{
		return "Hello World";
	}
}
```

In the app's `pom.xml` I had to deactivate two execution goals in `project.build.plugins.plugin.quarkus-maven-plugin` to make my IDE (eclipse) happy:

```xml
...
<build>
	<plugins>
		<plugin>
		<groupId>io.quarkus</groupId>
		<artifactId>quarkus-maven-plugin</artifactId>
		<version>${quarkus-plugin.version}</version>
		<extensions>true</extensions>
		<executions>
			<execution>
			<goals>
				<goal>build</goal>
<!--               <goal>generate-code</goal> -->
<!--               <goal>generate-code-tests</goal> -->
			</goals>
			</execution>
		</executions>
		</plugin>
		...
	</plugins>
</build>
...
```

>I think somewhere I saw that this is for now an unresolved issue for the quarkus maven goal ... Please let me know if there is a fix availabel for this.

To continue my POC for reusing custom java libraries in quarkus apps I also created a maven project for a minimal java library and made it available in my local maven repository:

## create minimal custom library

```java
package lib;

public class MathOps
{
	public final static int add(int a, int b) { return a + b; }
}
```

## include library to dependencies of quarkus app

To make my library available in my quarkus app I added it as a maven dependency into `pom.xml` of my app.

```xml
<dependency>
    <groupId>ruu.io</groupId>
    <artifactId>lib</artifactId>
    <version>0.0.1-SNAPSHOT</version>
</dependency>
```

## use custom library in quarkus app

Then I modified my quarkus app to use my custom library like that:

```java
package app;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.Produces;
import javax.ws.rs.core.MediaType;

import lib.MathOps;

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

At first glance everything seemed to be fine. At least this compiled perfectly in eclipse.

However, as mentioned above, this turned out to be a too simplistic approach for quarkus. Using a regular library as regular maven dependency is not the quarkus-style of using libraries!

## quarkus complaints

Trouble started when I compiled the app project from the command line with:

```mvn compile```

Maven output was something like this:

![](screenshot%20-%20compile%20error%20with%20regular%20lib.png)

Soon I suspected that it would be necessary to create a custom quarkus extension to overcome these issues.

## migrate a regular java library to a custom quarkus extension

For my POC I planned to work with the following maven project (directory) structure that keeps all related parts together in a common maven parent project:

```
- project   parent project for the related maven child modules
  - app       module for app
  - lib       module for lib
  - lib-qxt   module for qxt
```

The `lib-qxt` (`qxt` standing for `q`uarkus e`xt`ension) module was generated with the following command issued in the project directory:

```
mvn io.quarkus:quarkus-maven-plugin:1.10.5.Final:create-extension -N -DgroupId=ruu.io -DartifactId=lib-qxt -Dversion=0.0.1-SNAPSHOT -Dquarkus.nameBase="lib-qxt"
```

The generated name of the new project directory corresponds to the maven artifact name `ruu.io-sandbox-quarkus-create.custom.extension-lib-qxt`. This clumsy name corresponds to my personal convention for maven artifact names. However maven used this name for the creation of a directory structure. Because I did not want such a long name for the directories I manually changed the directory name to `lib-qxt`. To keep the maven project hierarchy in sync I also had to manually adjust

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
- project
  - app
  - lib
  - lib-qxt        parent module for quarkus extension       (generated)
    - deployment     module for quarkus extension deployment (generated)
    - runtime        module for quarkus extension runtime    (generated)
```

In the generated deployment module there are two JUnit test classes. I expected to be able to run them for a just generated, empty project but I was wrong. The error I saw was:

```
java.lang.RuntimeException: org.junit.jupiter.api.extension.TestInstantiationException: Failed to create test instance
Caused by: org.junit.jupiter.api.extension.TestInstantiationException: Failed to create test instance
Caused by: java.lang.RuntimeException: java.lang.reflect.InvocationTargetException
Caused by: java.lang.reflect.InvocationTargetException
Caused by: java.lang.IllegalStateException: Unable to locate CDIProvider
```

This behaviour changed when I added another dependency to the `pom.xml` of the deployment module:

```xml
<dependency>
	<groupId>io.quarkus</groupId>
	<artifactId>quarkus-arc</artifactId>
	<scope>test</scope>
</dependency>
```
Now the output changed to:

```
[ERROR] Tests run: 1, Failures: 0, Errors: 1, Skipped: 0, Time elapsed: 0.052 s <<< FAILURE! - in ruu.io.sandbox.quarkus.create.custom.extension.lib.qxt.test.RuuIoSandboxQuarkusCreateCustomExtensionLibQxtDevModeTest
[ERROR] test  Time elapsed: 0.034 s  <<< ERROR!
org.junit.jupiter.api.extension.TestInstantiationException: Unable to create test proxy
Caused by: java.lang.IllegalAccessException: class io.quarkus.test.QuarkusDevModeTest cannot access a member of class ruu.io.sandbox.quarkus.create.custom.extension.lib.qxt.test.RuuIoSandboxQuarkusCreateCustomExtensionLibQxtDevModeTest with modifiers ""
```
The above was output from
Unfortunately r
 Unfortunately until now I was not able to 


To my surprise they do not 