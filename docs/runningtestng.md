#Run TestNG

TestNG can be invoked in different ways:

* Command line
* [ant](/ant)
* [Eclipse](/eclipse)
* [IntelliJ's IDEA](/idea)

This section only explains how to invoke TestNG from the command line. Please click on one of the links above if you are interested in one of the other ways.

Assuming that you have TestNG in your class path, the simplest way to invoke TestNG is as follows:

```java
java org.testng.TestNG testng1.xml [testng2.xml testng3.xml ...]
```

You need to specify at least one XML file describing the TestNG suite you are trying to run. Additionally, the following command-line switches are available:

### Command Line Parameters

Option	| Argument |	Documentation
--|----|--------------
-configfailurepolicy|	skip/continue	|Whether TestNG should continue to execute the remaining tests in the suite or skip them if an @Before* method fails. Default behavior is skip.
-d |	A directory |	The directory where the reports will be generated (defaults to test-output).
-dataproviderthreadcount	|The default number of threads to use for data providers when running tests in parallel.|This sets the default maximum number of threads to use for data providers when running tests in parallel. It will only take effect if the parallel mode has been selected (for example, with the -parallel option). This can be overridden in the suite definition.
-excludegroups|	A comma-separated list of groups.	|The list of groups you want to be excluded from this run.
-groups |	A comma-separated list of groups.	|The list of groups you want to run (e.g. "windows,linux,regression").
-listener	|A comma-separated list of Java classes that can be found on your classpath. |	Lets you specify your own test listeners. The classes need to implement org.testng.ITestListener
-usedefaultlisteners|	true/false |	Whether to use the default listeners
-methods |	A comma separated list of fully qualified class name and method. For example com.example.Foo.f1,com.example.Bar.f2. |	Lets you specify individual methods to run.
-methodselectors|	A comma-separated list of Java classes and method priorities that define method selectors.|	Lets you specify method selectors on the command line. For example: com.example.Selector1:3,com.example.Selector2:2
-parallel	| methods/tests/classes	 |If specified, sets the default mechanism used to determine how to use parallel threads when running tests. If not set, default mechanism is not to use parallel threads at all. This can be overridden in the suite definition.
-reporter	| The extended configuration for a custom report listener.	| Similar to the -listener option, except that it allows the configuration of JavaBeans-style properties on the reporter instance.
Example: -reporter com.test.MyReporter:methodFilter=*insert*,enableFiltering=true
You can have as many occurrences of this option, one for each reporter that needs to be added.
-sourcedir | 	A semi-colon separated list of directories.	| The directories where your javadoc annotated test sources are. This option is only necessary if you are using javadoc type annotations. (e.g. "src/test" or "src/test/org/testng/eclipse-plugin;src/test/org/testng/testng").
-suitename	| The default name to use for a test suite.	 |This specifies the suite name for a test suite defined on the command line. This option is ignored if the suite.xml file or the source code specifies a different suite name. It is possible to create a suite name with spaces in it if you surround it with double-quotes "like this".
-testclass	| A comma-separated list of classes that can be found in your classpath.|	A list of class files separated by commas (e.g. "org.foo.Test1,org.foo.test2").
-testjar|	A jar file.	| Specifies a jar file that contains test classes. If a testng.xml file is found at the root of that jar file, it will be used, otherwise, all the test classes found in this jar file will be considered test classes.
-testname	|The default name to use for a test.|	This specifies the name for a test defined on the command line. This option is ignored if the suite.xml file or the source code specifies a different test name. It is possible to create a test name with spaces in it if you surround it with double-quotes "like this".
-testnames |	A comma separated list of test names.	| Only tests defined in a <test> tag matching one of these names will be run.
-testrunfactory	| A Java classes that can be found on your classpath.	| Lets you specify your own test runners. The class needs to implement org.testng.ITestRunnerFactory.
-threadcount	| The default number of threads to use when running tests in parallel. |	This sets the default maximum number of threads to use for running tests in parallel. It will only take effect if the parallel mode has been selected (for example, with the -parallel option). This can be overridden in the suite definition.
-xmlpathinjar	| The path of the XML file inside the jar file.	 | This attribute should contain the path to a valid XML file inside the test jar (e.g. "resources/testng.xml"). The default is "testng.xml", which means a file called "testng.xml" at the root of the jar file. This option will be ignored unless -testjar is specified.



This documentation can be obtained by invoking TestNG without any arguments.


You can also put the command line switches in a text file, say c:\command.txt, and tell TestNG to use that file to retrieve its parameters:

```cmd
C:> more c:\command.txt
-d test-output testng.xml
C:> java org.testng.TestNG @c:\command.txt

```
Additionally, TestNG can be passed properties on the command line of the Java Virtual Machine, for example

```java
java -Dtestng.test.classpath="c:/build;c:/java/classes;" org.testng.TestNG testng.xml
```
Here are the properties that TestNG understands:

#### System properties

Property| Type|	Documentation
---|---|---
testng.test.classpath |	A semi-colon separated series of directories that contain your test classes.|	If this property is set, TestNG will use it to look for your test classes instead of the class path. This is convenient if you are using the package tag in your XML file and you have a lot of classes in your classpath, most of them not being test classes.

Example:
```java 
java org.testng.TestNG -groups windows,linux -testclass org.test.MyTest
```

The ant task and testng.xml allow you to launch TestNG with more parameters (methods to include, specifying parameters, etc...), so you should consider using the command line only when you are trying to learn about TestNG and you want to get up and running quickly.

Important: The command line flags that specify what tests should be run will be ignored if you also specify a testng.xml file, with the exception of -includedgroups and -excludedgroups, which will override all the group inclusions/exclusions found in testng.xml.

