# Welcome 

!!! note 
    This is not official documentation . Visit [https://testng.org/](https://testng.org/) for official documentation .Its just a stylized version of the official documentation .

TestNG is a testing framework inspired from JUnit and NUnit but introducing some new functionalities that make it more powerful and easier to use, such as:

* Annotations.
* Run your tests in arbitrarily big thread pools with various policies available (all methods in their own thread, one thread per test class, etc...).
* Test that your code is multithread safe.
* Flexible test configuration.
* Support for data-driven testing (with @DataProvider).
* Support for parameters.
* Powerful execution model (no more TestSuite).
* Supported by a variety of tools and plug-ins (Eclipse, IDEA, Maven, etc...).
* Embeds BeanShell for further flexibility.
* Default JDK functions for runtime and logging (no dependencies).
* Dependent methods for application server testing.
* TestNG is designed to cover all categories of tests:  unit, functional, end-to-end, integration, etc...

### Requirements

TestNG requires JDK 7 or higher.

### Mailing-lists

* The users mailing-list can be found on [Google Groups](http://groups.google.com/group/testng-users).
* If you are interested in working on TestNG itself, join the [developer mailing-list](http://groups.google.com/group/testng-users).
* If you are only interested in hearing about new versions of TestNG, you can join the low volume [TestNG announcement mailing-list](http://groups.google.com/group/testng-announcements).

### Locations of the projects

If you are interested in contributing to TestNG or one of the IDE plug-ins, you will find them in the following locations:

* [TestNG](https://github.com/cbeust/testng/)
* [Eclipse plug-in](https://github.com/cbeust/testng-eclipse/)
* [IDEA IntelliJ plug-in](https://github.com/JetBrains/intellij-community/tree/master/plugins/testng)
* [NetBeans plug-in](http://wiki.netbeans.org/TestNG)

### Bug reports

If you think you found a bug, here is how to report it:

* Create a small project that will allow us to reproduce this bug. In most cases, one or two Java source files and a testng.xml file should be sufficient. Then you can either zip it and email it to the [testng-dev mailing-list](http://groups.google.com/group/testng-dev) or make it available on an open source hosting site, such as [github](https://github.com/) and email testng-dev so we know about it. Please make sure that this project is self contained so that we can build it right away (remove the dependencies on external or proprietary frameworks, etc...).
* If the bug you observed is on the Eclipse plug-in, make sure your sample project contains the .project and .classpath files.
* [File a bug](https://github.com/cbeust/testng/issues).

For more information, you can either [download](download.md) TestNG, read the [manual](gettingstarted.md).

### License
Apache 2.0