# Test methods, classes and groups

### Test methods

Test methods are annotated with @Test. Methods annotated with @Test that happen to return a value will be ignored, unless you set allow-return-values to true in your testng.xml:

```xml 
<suite allow-return-values="true">

or

<test allow-return-values="true">
```
### Test groups

TestNG allows you to perform sophisticated groupings of test methods. Not only can you declare that methods belong to groups, but you can also specify groups that contain other groups. Then TestNG can be invoked and asked to include a certain set of groups (or regular expressions) while excluding another set.  This gives you maximum flexibility in how you partition your tests and doesn't require you to recompile anything if you want to run two different sets of tests back to back.

Groups are specified in your testng.xml file and can be found either under the &lt;test&gt; or  &lt;suite&gt; tag. Groups specified in the &lt;suite&gt; tag apply to all the &lt;test&gt; tags underneath. 
> Note that groups are accumulative in these tags: if you specify group "a" in &lt;suite&gt; and "b" in  &lt;test&gt;, then both "a" and "b" will be included.

For example, it is quite common to have at least two categories of tests

* Check-in tests :- These tests should be run before you submit new code.  They should typically be fast and just make sure no basic functionality was broken.
 
* Functional tests :-  These tests should cover all the functionalities of your software and be run at least once a day, although ideally you would want to run them continuously.

Typically, check-in tests are a subset of functional tests.  TestNG allows you to specify this in a very intuitive way with test groups.  For example, you could structure your test by saying that your entire test class belongs to the "functest" group, and additionally that a couple of methods belong to the group "checkintest":

```java 
public class Test1 {
  @Test(groups = { "functest", "checkintest" })
  public void testMethod1() {
  }
 
  @Test(groups = {"functest", "checkintest"} )
  public void testMethod2() {
  }
 
  @Test(groups = { "functest" })
  public void testMethod3() {
  }
}
```

Invoking TestNG with

```xml
<test name="Test1">
  <groups>
    <run>
      <include name="functest"/>
    </run>
  </groups>
  <classes>
    <class name="example1.Test1"/>
  </classes>
</test>
```

will run all the test methods in that classes, while invoking it with checkintest will only run testMethod1() and testMethod2().

Here is another example, using regular expressions this time.  Assume that some of your test methods should not be run on Linux, your test would look like:

```java
@Test
public class Test1 {
  @Test(groups = { "windows.checkintest" })
  public void testWindowsOnly() {
  }
 
  @Test(groups = {"linux.checkintest"} )
  public void testLinuxOnly() {
  }
 
  @Test(groups = { "windows.functest" )
  public void testWindowsToo() {
  }
}
```

You could use the following testng.xml to launch only the Windows methods:

```xml
<test name="Test1">
  <groups>
    <run>
      <include name="windows.*"/>
    </run>
  </groups>
 
  <classes>
    <class name="example1.Test1"/>
  </classes>
</test>
```

> Note: TestNG uses [regular expressions](http://en.wikipedia.org/wiki/Regular_expression), and not [wildmats](http://en.wikipedia.org/wiki/Wildmat). Be aware of the difference (for example, "anything" is matched by ".*" -- dot star -- and not "*").
#### Method groups

You can also exclude or include individual methods:
```xml
<test name="Test1">
  <classes>
    <class name="example1.Test1">
      <methods>
        <include name=".*enabledTestMethod.*"/>
        <exclude name=".*brokenTestMethod.*"/>
      </methods>
     </class>
  </classes>
</test>
```

This can come in handy to deactivate a single method without having to recompile anything, but I don't recommend using this technique too much since it makes your testing framework likely to break if you start refactoring your Java code (the regular expressions used in the tags might not match your methods any more).

### Groups of groups

Groups can also include other groups. These groups are called "MetaGroups".  For example, you might want to define a group "all" that includes "checkintest" and "functest".  "functest" itself will contain the groups "windows" and "linux" while "checkintest will only contain "windows".  Here is how you would define this in your property file:

```xml 
<test name="Regression1">
  <groups>
    <define name="functest">
      <include name="windows"/>
      <include name="linux"/>
    </define>
  
    <define name="all">
      <include name="functest"/>
      <include name="checkintest"/>
    </define>
  
    <run>
      <include name="all"/>
    </run>
  </groups>
  
  <classes>
    <class name="test.sample.Test1"/>
  </classes>
</test>
``` 

### Exclusion groups

TestNG allows you to include groups as well as exclude them.

For example, it is quite usual to have tests that temporarily break because of a recent change, and you don't have time to fix the breakage yet.  4 However, you do want to have clean runs of your functional tests, so you need to deactivate these tests but keep in mind they will need to be reactivated.
A simple way to solve this problem is to create a group called "broken" and make these test methods belong to it.  For example, in the above example, I know that testMethod2() is now broken so I want to disable it:

```java 
@Test(groups = {"checkintest", "broken"} )
public void testMethod2() {
}
```

All I need to do now is to exclude this group from the run:

```xml
<test name="Simple example">
  <groups>
    <run>
      <include name="checkintest"/>
      <exclude name="broken"/>
    </run>
  </groups>
  
  <classes>
    <class name="example1.Test1"/>
  </classes>
</test>
``` 

This way, I will get a clean test run while keeping track of what tests are broken and need to be fixed later.

> Note:  you can also disable tests on an individual basis by using the "enabled" property available on both @Test and @Before/After annotations.

### Partial groups

You can define groups at the class level and then add groups at the method level:

```java
@Test(groups = { "checkin-test" })
public class All {
 
  @Test(groups = { "func-test" )
  public void method1() { ... }
 
  public void method2() { ... }
}
```

In this class, method2() is part of the group "checkin-test", which is defined at the class level, while method1() belongs to both "checkin-test" and "func-test".

### Parameters

Test methods don't have to be parameterless.  You can use an arbitrary number of parameters on each of your test method, and you instruct TestNG to pass you the correct parameters with the @Parameters annotation.

There are two ways to set these parameters:  with testng.xml or programmatically.

#### Parameters from testng.xml

If you are using simple values for your parameters, you can specify them in your testng.xml:

```java
@Parameters({ "first-name" })
@Test
public void testSingleString(String firstName) {
  System.out.println("Invoked testString " + firstName);
  assert "Cedric".equals(firstName);
}
```

In this code, we specify that the parameter firstName of your Java method should receive the value of the XML parameter called first-name.  This XML parameter is defined in testng.xml:

```xml
<suite name="My suite">
  <parameter name="first-name"  value="Cedric"/>
  <test name="Simple example">
  <-- ... -->
```
The same technique can be used for @Before/After and @Factory annotations:

```java
@Parameters({ "datasource", "jdbcDriver" })
@BeforeMethod
public void beforeTest(String ds, String driver) {
  m_dataSource = ...;    // look up the value of datasource
  m_jdbcDriver = driver;
}
```

This time, the two Java parameter ds and driver will receive the value given to the properties datasource and jdbc-driver respectively. 

Parameters can be declared [optional](https://jitpack.io/com/github/cbeust/testng/master/javadoc/org/testng/annotations/Optional.html) with the Optional annotation:

```java
@Parameters("db")
@Test
public void testNonExistentParameter(@Optional("mysql") String db) { ... }
```

If no parameter named "db" is found in your testng.xml file, your test method will receive the default value specified inside the @Optional annotation: "mysql".

The @Parameters annotation can be placed at the following locations:

* On any method that already has a @Test, @Before/After or @Factory annotation.
* On at most one constructor of your test class.  In this case, TestNG will invoke this particular constructor with the parameters initialized to the values specified in testng.xml whenever it needs to instantiate your test class.  This feature can be used to initialize fields inside your classes to values that will then be used by your test methods.

> Notes:
> The XML parameters are mapped to the Java parameters in the same order as they are found in the annotation, and TestNG will issue an error if the numbers don't match.
> Parameters are scoped. In testng.xml, you can declare them either under a &lt;suite&gt; tag or under &lt;test&gt;. If two parameters have the same name, it's the one defined in &lt;test&gt; that has precedence. This is convenient if you need to specify a parameter applicable to all your tests and override its value only for certain tests.

#### Parameters with DataProviders

Specifying parameters in testng.xml might not be sufficient if you need to pass complex parameters, or parameters that need to be created from Java (complex objects, objects read from a property file or a database, etc...). 

In this case, you can use a Data Provider to supply the values you need to test.  A Data Provider is a method on your class that returns an array of array of objects.  This method is annotated with @DataProvider:

```java
//This method will provide data to any test method that declares that its Data Provider
//is named "test1"
@DataProvider(name = "test1")
public Object[][] createData1() {
 return new Object[][] {
   { "Cedric", new Integer(36) },
   { "Anne", new Integer(37)},
 };
}
 
//This test method declares that its data should be supplied by the Data Provider
//named "test1"
@Test(dataProvider = "test1")
public void verifyData1(String n1, Integer n2) {
 System.out.println(n1 + " " + n2);
}
```

will print

```
Cedric 36
Anne 37

```
A @Test method specifies its Data Provider with the dataProvider attribute.  This name must correspond to a method on the same class annotated with @DataProvider(name="...") with a matching name.

By default, the data provider will be looked for in the current test class or one of its base classes. If you want to put your data provider in a different class, it needs to be a static method or a class with a non-arg constructor, and you specify the class where it can be found in the dataProviderClass attribute:

```java
public class StaticProvider {
  @DataProvider(name = "create")
  public static Object[][] createData() {
    return new Object[][] {
      new Object[] { new Integer(42) }
    };
  }
}
 
public class MyTest {
  @Test(dataProvider = "create", dataProviderClass = StaticProvider.class)
  public void test(Integer n) {
    // ...
  }
}

```

The data provider supports injection too. TestNG will use the test context for the injection. The Data Provider method can return one of the following types:

* An array of array of objects (Object[][]) where the first dimension's size is the number of times the test method will be invoked and the second dimension size contains an array of objects that must be compatible with the parameter types of the test method. This is the case illustrated by the example above.
* An Iterator&lt;Object[]&gt;. The only difference with Object[][] is that an Iterator lets you create your test data lazily. TestNG will invoke the iterator and then the test method with the parameters returned by this iterator one by one. This is particularly useful if you have a lot of parameter sets to pass to the method and you don't want to create all of them upfront.
  * An array of objects (Object[]). This is similar to Iterator&lt;Object[]&gt; but causes the test method to be invoked once for each element of the source array.
  * An Iterator&lt;Object&gt;. Lazy alternative of Object[]. Causes the test method to be invoked once for each element of the iterator.


It must be said that return type is not limited to Object only thus MyCustomData[][] or Iterator&lt;Supplier&gt; are also possible. The only limitation is that in case of iterator its parameter type can't be explicitly parametrized itself. Here is an example of this feature:

```java
@DataProvider(name = "test1")
public Iterator<Object[]> createData() {
  return new MyIterator(DATA);
}
```
Using MyCustomData[] as a return type

```java 
@DataProvider(name = "test1")
public MyCustomData[] createData() {
  return new MyCustomData[]{ new MyCustomData() };
}
```
Or its lazy option with Iterator&lt;MyCustomData&gt;
```java
@DataProvider(name = "test1")
public Iterator<MyCustomData> createData() {
  return Arrays.asList(new MyCustomData()).iterator();
}
```

Parameter type (Stream) of Iterator can't be explicitly parametrized

```java 
@DataProvider(name = "test1")
public Iterator<Stream> createData() {
  return Arrays.asList(Stream.of("a", "b", "c")).iterator();
}
```

If you declare your @DataProvider as taking a java.lang.reflect.Method as first parameter, TestNG will pass the current test method for this first parameter. This is particularly useful when several test methods use the same @DataProvider and you want it to return different values depending on which test method it is supplying data for.

For example, the following code prints the name of the test method inside its @DataProvider:

```java 
@DataProvider(name = "dp")
public Object[][] createData(Method m) {
  System.out.println(m.getName());  // print test method name
  return new Object[][] { new Object[] { "Cedric" }};
}
 
@Test(dataProvider = "dp")
public void test1(String s) {
}
 
@Test(dataProvider = "dp")
public void test2(String s) {
}
```
and will therefore display:

```
test1
test2
```

Data providers can run in parallel with the attribute parallel:

```java 
@DataProvider(parallel = true)
// ...
``` 

Parallel data providers running from an XML file share the same pool of threads, which has a size of 10 by default. You can modify this value in the &lt;suite&gt; tag of your XML file:

```xml 
<suite name="Suite1" data-provider-thread-count="20" >
...
``` 

If you want to run a few specific data providers in a different thread pool, you need to run them from a different XML file.
####  Parameters in reports

Parameters used to invoke your test methods are shown in the HTML reports generated by TestNG. Here is an example:

``` java
test.dataprovider.Sample1Test.verifyNames(java.lang.String,java.lang.Integer)
// Parameters Cedric, 36 
test.dataprovider.Sample1Test.verifyNames(java.lang.String,java.lang.Integer)
// Parameters Anne Marie , 37 
```

### Dependencies

Sometimes, you need your test methods to be invoked in a certain order.  Here are a few examples:

To make sure a certain number of test methods have completed and succeeded before running more test methods.

To initialize your tests while wanting this initialization methods to be test methods as well (methods tagged with @Before/
After will not be part of the final report).

TestNG allows you to specify dependencies either with annotations or in XML.

#### Dependencies with annotations

You can use the attributes dependsOnMethods or dependsOnGroups, found on the @Test annotation.

There are two kinds of dependencies:

* Hard dependencies. All the methods you depend on must have run and succeeded for you to run. If at least one failure occurred in your dependencies, you will not be invoked and marked as a SKIP in the report.
* Soft dependencies. You will always be run after the methods you depend on, even if some of them have failed. This is useful when you just want to make sure that your test methods are run in a certain order but their success doesn't really depend on the success of others. A soft dependency is obtained by adding "alwaysRun=true" in your @Test annotation.

Here is an example of a hard dependency:

```java
@Test
public void serverStartedOk() {}
 
@Test(dependsOnMethods = { "serverStartedOk" })
public void method1() {}

``` 

In this example, method1() is declared as depending on method serverStartedOk(), which guarantees that serverStartedOk() will always be invoked first.

You can also have methods that depend on entire groups:

```java 
@Test(groups = { "init" })
public void serverStartedOk() {}
 
@Test(groups = { "init" })
public void initEnvironment() {}
 
@Test(dependsOnGroups = { "init.*" })
public void method1() {}

``` 

In this example, method1() is declared as depending on any group matching the regular expression "init.*", which guarantees that the methods serverStartedOk() and initEnvironment() will always be invoked before method1(). 

Note:  as stated before, the order of invocation for methods that belong in the same group is not guaranteed to be the same across test runs.

If a method depended upon fails and you have a hard dependency on it (alwaysRun=false, which is the default), the methods that depend on it are not marked as FAIL but as SKIP.  Skipped methods will be reported as such in the final report (in a color that is neither red nor green in HTML), which is important since skipped methods are not necessarily failures.

Both dependsOnGroups and dependsOnMethods accept regular expressions as parameters.  For dependsOnMethods, if you are depending on a method which happens to have several overloaded versions, all the overloaded methods will be invoked.  If you only want to invoke one of the overloaded methods, you should use dependsOnGroups.

For a more advanced example of dependent methods, please refer to this [article](https://beust.com/weblog/archives/000171.html), which uses inheritance to provide an elegant solution to the problem of multiple dependencies.

By default, dependent methods are grouped by class. For example, if method b() depends on method a() and you have several instances of the class that contains these methods (because of a factory of a data provider), then the invocation order will be as follows:

``` 
a(1)
a(2)
b(2)
b(2)
```

TestNG will not run b() until all the instances have invoked their a() method.

This behavior might not be desirable in certain scenarios, such as for example testing a sign in and sign out of a web browser for various countries. In such a case, you would like the following ordering:

```
signIn("us")
signOut("us")
signIn("uk")
signOut("uk")
```

For this ordering, you can use the XML attribute group-by-instances. This attribute is valid either on &lt;suite&gt; or &lt;test&gt;:

```xml
<suite name="Factory" group-by-instances="true">
or
  <test name="Factory" group-by-instances="true">
``` 
#### Dependencies in XML

Alternatively, you can specify your group dependencies in the testng.xml file. You use the &lt;dependencies&gt; tag to achieve this:

```xml 
<test name="My suite">
  <groups>
    <dependencies>
      <group name="c" depends-on="a  b" />
      <group name="z" depends-on="c" />
    </dependencies>
  </groups>
</test>
``` 

The &lt;depends-on&gt; attribute contains a space-separated list of groups.

### Factories

Factories allow you to create tests dynamically. For example, imagine you want to create a test method that will access a page on a Web site several times, and you want to invoke it with different values:

```java 
public class TestWebServer {
  @Test(parameters = { "number-of-times" })
  public void accessPage(int numberOfTimes) {
    while (numberOfTimes-- > 0) {
     // access the web page
    }
  }
}
```
```xml
<test name="T1">
  <parameter name="number-of-times" value="10"/>
  <classes>
    <class name= "TestWebServer" />
  </classes>
</test>
 
<test name="T2">
  <parameter name="number-of-times" value="20"/>
  <classes>
    <class name= "TestWebServer"/>
  </classes>
</test>
 
<test name="T3">
  <parameter name="number-of-times" value="30"/>
  <classes>
    <class name= "TestWebServer"/>
  </classes>
</test>
``` 

This can become quickly impossible to manage, so instead, you should use a factory:

```java 
public class WebTestFactory {
  @Factory
  public Object[] createInstances() {
   Object[] result = new Object[10]; 
   for (int i = 0; i < 10; i++) {
      result[i] = new WebTest(i * 10);
    }
    return result;
  }
}
```

and the new test class is now:

```java
public class WebTest {
  private int m_numberOfTimes;
  public WebTest(int numberOfTimes) {
    m_numberOfTimes = numberOfTimes;
  }
 
  @Test
  public void testServer() {
   for (int i = 0; i < m_numberOfTimes; i++) {
     // access the web page
    }
  }
}
```

Your testng.xml only needs to reference the class that contains the factory method, since the test instances themselves will be created at runtime:

```xml 
<class name="WebTestFactory" />
``` 

Or, if building a test suite instance programatically, you can add the factory in the same manner as for tests:

```java 
TestNG testNG = new TestNG();
testNG.setTestClasses(WebTestFactory.class);
testNG.run();
``` 

The factory method can receive parameters just like @Test and @Before/After and it must return Object[].  The objects returned can be of any class (not necessarily the same class as the factory class) and they don't even need to contain TestNG annotations (in which case they will be ignored by TestNG).

Factories can also be used with data providers, and you can leverage this functionality by putting the @Factory annotation either on a regular method or on a constructor. Here is an example of a constructor factory:

```java 
@Factory(dataProvider = "dp")
public FactoryDataProviderSampleTest(int n) {
  super(n);
}
 
@DataProvider
static public Object[][] dp() {
  return new Object[][] {
    new Object[] { 41 },
    new Object[] { 42 },
  };
}
``` 

The example will make TestNG create two test classes, on with the constructor invoked with the value 41 and the other with 42.

### Class level annotations

The @Test annotation can be put on a class instead of a test method:

```java 
@Test
public class Test1 {
  public void test1() {
  }
 
  public void test2() {
  }
}
``` 

The effect of a class level @Test annotation is to make all the public methods of this class to become test methods even if they are not annotated. You can still repeat the @Test annotation on a method if you want to add certain attributes.

For example:

```java 
@Test
public class Test1 {
  public void test1() {
  }
 
  @Test(groups = "g1")
  public void test2() {
  }
}
``` 

will make both test1() and test2() test methods but on top of that, test2() now belongs to the group "g1".

### Ignoring tests

TestNG lets you ignore all the @Test methods :

* In a class (or)
* In a particular package (or)
* In a package and all of its child packages

using the new annotation @Ignore.

When used at the method level @Ignore annotation is functionally equivalent to @Test(enabled=false). Here's a sample that shows how to ignore all tests within a class.

```java 
import org.testng.annotations.Ignore;
import org.testng.annotations.Test;
 
@Ignore
public class TestcaseSample {
 
    @Test
    public void testMethod1() {
    }
 
    @Test
    public void testMethod2() {
    }
}
``` 

The @Ignore annotation has a higher priority than individual @Test method annotations. When @Ignore is placed on a class, all the tests in that class will be disabled.

To ignore all tests in a particular package, you just need to create package-info.java and add the @Ignore annotation to it. Here's a sample :

```java 
@Ignore
package com.testng.master;
 
import org.testng.annotations.Ignore;

``` 

This causes all the @Test methods to be ignored in the package com.testng.master and all of its sub-packages.

###  Parallelism and time-outs

You can instruct TestNG to run your tests in separate threads in various ways.

#### Parallel suites

This is useful if you are running several suite files (e.g. "java org.testng.TestNG testng1.xml testng2.xml") and you want each of these suites to be run in a separate thread. You can use the following command line flag to specify the size of a thread pool:

```java 
java org.testng.TestNG -suitethreadpoolsize 3 testng1.xml testng2.xml testng3.xml

``` 

The corresponding ant task name is suitethreadpoolsize.

#### Parallel tests, classes and methods

The parallel attribute on the <suite> tag can take one of following values:

```xml
<suite name="My suite" parallel="methods" thread-count="5">
<suite name="My suite" parallel="tests" thread-count="5">
<suite name="My suite" parallel="classes" thread-count="5">
<suite name="My suite" parallel="instances" thread-count="5">
``` 

* parallel="methods": TestNG will run all your test methods in separate threads. Dependent methods will also run in separate threads but they will respect the order that you specified.
* parallel="tests": TestNG will run all the methods in the same &lt;test&gt; tag in the same thread, but each &lt;test&gt; tag will be in a separate thread. This allows you to group all your classes that are not thread safe in the same &lt;test&gt; and guarantee they will all run in the same thread while taking advantage of TestNG using as many threads as possible to run your tests.
* parallel="classes": TestNG will run all the methods in the same class in the same thread, but each class will be run in a separate thread.
* parallel="instances": TestNG will run all the methods in the same instance in the same thread, but two methods on two different instances will be running in different threads.

Additionally, the attribute thread-count allows you to specify how many threads should be allocated for this execution.

> Note: the @Test attribute timeOut works in both parallel and non-parallel mode.

You can also specify that a @Test method should be invoked from different threads. You can use the attribute threadPoolSize to achieve this result:
```java
@Test(threadPoolSize = 3, invocationCount = 10,  timeOut = 10000)
public void testServer() {
``` 

In this example, the function testServer will be invoked ten times from three different threads. Additionally, a time-out of ten seconds guarantees that none of the threads will block on this thread forever.

### Rerunning failed tests

Every time tests fail in a suite, TestNG creates a file called testng-failed.xml in the output directory. This XML file contains the necessary information to rerun only these methods that failed, allowing you to quickly reproduce the failures without having to run the entirety of your tests.  Therefore, a typical session would look like this:

```
java -classpath testng.jar;%CLASSPATH% org.testng.TestNG -d test-outputs testng.xml
java -classpath testng.jar;%CLASSPATH% org.testng.TestNG -d test-outputs test-outputs\testng-failed.xml

```

> Note that testng-failed.xml will contain all the necessary dependent methods so that you are guaranteed to run the methods that failed without any SKIP failures.


Sometimes, you might want TestNG to automatically retry a test whenever it fails. In those situations, you can use a retry analyzer. When you bind a retry analyzer to a test, TestNG automatically invokes the retry analyzer to determine if TestNG can retry a test case again in an attempt to see if the test that just fails now passes. 

Here is how you use a retry analyzer:

1. Build an implementation of the interface org.testng.IRetryAnalyzer
2. Bind this implementation to the @Test annotation for e.g., @Test(retryAnalyzer = LocalRetry.class)

Following is a sample implementation of the retry analyzer that retries a test for a maximum of three times.
```java 
import org.testng.IRetryAnalyzer;
import org.testng.ITestResult;
 
public class MyRetry implements IRetryAnalyzer {
 
  private int retryCount = 0;
  private static final int maxRetryCount = 3;
 
  @Override
  public boolean retry(ITestResult result) {
    if (retryCount < maxRetryCount) {
      retryCount++;
      return true;
    }
    return false;
  }
}
import org.testng.Assert;
import org.testng.annotations.Test;
 
public class TestclassSample {
 
  @Test(retryAnalyzer = MyRetry.class)
  public void test2() {
    Assert.fail();
  }
}

``` 

### JUnit tests

TestNG can run JUnit 3 and JUnit 4 tests.  All you need to do is put the JUnit jar file on the classpath, specify your 

JUnit test classes in the testng.classNames property and set the testng.junit property to true:

```xml
<test name="Test1" junit="true">
  <classes>
    <!-- ... -->
``` 

The behavior of TestNG in this case is similar to JUnit depending on the JUnit version found on the class path:

##### JUnit 3:

All methods starting with test* in your classes will be run
* If there is a method setUp() on your test class, it will be invoked before every test method. 
* If there is a method tearDown() on your test class, it will be invoked before after every test method. 
* If your test class contains a method suite(), all the tests returned by this method will be invoked. 
#### JUnit 4:
* TestNG will use the org.junit.runner.JUnitCore runner to run your tests. 


### Running TestNG programmatically

You can invoke TestNG from your own programs very easily:

```java 
TestListenerAdapter tla = new TestListenerAdapter();
TestNG testng = new TestNG();
testng.setTestClasses(new Class[] { Run2.class });
testng.addListener(tla);
testng.run();
``` 
This example creates a TestNG object and runs the test class Run2. It also adds a TestListener. You can either use the adapter class org.testng.TestListenerAdapter or implement org.testng.ITestListener yourself. This interface contains various callback methods that let you keep track of when a test starts, succeeds, fails, etc...

Similarly, you can invoke TestNG on a testng.xml file or you can create a virtual testng.xml file yourself. In order to do this, you can use the classes found the package org.testng.xml: XmlClass, XmlTest, etc... Each of these classes correspond to their XML tag counterpart.

For example, suppose you want to create the following virtual file:
```xml
<suite name="TmpSuite" >
  <test name="TmpTest" >
    <classes>
      <class name="test.failures.Child"  />
    <classes>
    </test>
</suite>
``` 

You would use the following code:

```java 
XmlSuite suite = new XmlSuite();
suite.setName("TmpSuite");
 
XmlTest test = new XmlTest(suite);
test.setName("TmpTest");
List<XmlClass> classes = new ArrayList<XmlClass>();
classes.add(new XmlClass("test.failures.Child"));
test.setXmlClasses(classes) ;
``` 
And then you can pass this XmlSuite to TestNG:

``` java 
List<XmlSuite> suites = new ArrayList<XmlSuite>();
suites.add(suite);
TestNG tng = new TestNG();
tng.setXmlSuites(suites);
tng.run();
``` 
Please see the JavaDocs for the entire API.

### BeanShell and advanced group selection

If the &lt;include&gt; and &lt;exclude&gt; tags in testng.xml are not enough for your needs, you can use a BeanShell expression to decide whether a certain test method should be included in a test run or not. You specify this expression just under the &lt;test&gt; tag:

```xml
<test name="BeanShell test">
   <method-selectors>
     <method-selector>
       <script language="beanshell"><![CDATA[
         groups.containsKey("test1")
       ]]></script>
     </method-selector>
   </method-selectors>
  <!-- ... -->
```

When a &lt;script&gt; tag is found in testng.xml, TestNG will ignore subsequent &lt;include&gt; and &lt;exclude&gt; of groups and methods in the current &lt;test&gt; tag:  your BeanShell expression will be the only way to decide whether a test method is included or not.

Here are additional information on the BeanShell script:

* It must return a boolean value.  Except for this constraint, any valid BeanShell code is allowed (for example, you might want to return true during week days and false during weekends, which would allow you to run tests differently depending on the date).

* TestNG defines the following variables for your convenience:
  * java.lang.reflect.Method method:  the current test method.
  * org.testng.ITestNGMethod testngMethod:  the description of the current test method.
  * java.util.Map&lt;String, String&gt; groups:  a map of the groups the current test method belongs to.
 
You might want to surround your expression with a CDATA declaration (as shown above) to avoid tedious quoting of reserved XML characters).
 
### Annotation Transformers

TestNG allows you to modify the content of all the annotations at runtime. This is especially useful if the annotations in the source code are right most of the time, but there are a few situations where you'd like to override their value.
In order to achieve this, you need to use an Annotation Transformer.

An Annotation Transformer is a class that implements the following interface:

```java 
public interface IAnnotationTransformer {
 
  /**
   * This method will be invoked by TestNG to give you a chance
   * to modify a TestNG annotation read from your test classes.
   * You can change the values you need by calling any of the
   * setters on the ITest interface.
   *
   * Note that only one of the three parameters testClass,
   * testConstructor and testMethod will be non-null.
   *
   * @param annotation The annotation that was read from your
   * test class.
   * @param testClass If the annotation was found on a class, this
   * parameter represents this class (null otherwise).
   * @param testConstructor If the annotation was found on a constructor,
   * this parameter represents this constructor (null otherwise).
   * @param testMethod If the annotation was found on a method,
   * this parameter represents this method (null otherwise).
   */
  public void transform(ITest annotation, Class testClass,
      Constructor testConstructor, Method testMethod);
}
``` 

Like all the other TestNG listeners, you can specify this class either on the command line or with ant:

```
java org.testng.TestNG -listener MyTransformer testng.xml
``` 
or programmatically:

```java 
TestNG tng = new TestNG();
tng.setAnnotationTransformer(new MyTransformer());
// ...'
```
When the method transform() is invoked, you can call any of the setters on the ITest test parameter to alter its value before TestNG proceeds further.

For example, here is how you would override the attribute invocationCount but only on the test method invoke() of one of your test classes:

```java
public class MyTransformer implements IAnnotationTransformer {
  public void transform(ITest annotation, Class testClass,
      Constructor testConstructor, Method testMethod)
  {
    if ("invoke".equals(testMethod.getName())) {
      annotation.setInvocationCount(5);
    }
  }
}
```

IAnnotationTransformer only lets you modify a @Test annotation. If you need to modify another TestNG annotation (a configuration annotation, @Factory or @DataProvider), use an IAnnotationTransformer2.

### Method Interceptors
Once TestNG has calculated in what order the test methods will be invoked, these methods are split in two groups:
Methods run sequentially. These are all the test methods that have dependencies or dependents. These methods will be run in a specific order.
Methods run in no particular order. These are all the methods that don't belong in the first category. The order in which these test methods are run is random and can vary from one run to the next (although by default, TestNG will try to group test methods by class).
In order to give you more control on the methods that belong to the second category, TestNG defines the following interface:

```java 
public interface IMethodInterceptor {
   
  List<IMethodInstance> intercept(List<IMethodInstance> methods, ITestContext context);
 
}
```

The list of methods passed in parameters are all the methods that can be run in any order. Your intercept method is expected to return a similar list of IMethodInstance, which can be either of the following:

* The same list you received in parameter but in a different order.
* A smaller list of IMethodInstance objects.
* A bigger list of IMethodInstance objects.

Once you have defined your interceptor, you pass it to TestNG as a listener. For example:

```
java -classpath "testng-jdk15.jar:test/build" org.testng.TestNG -listener test.methodinterceptors.NullMethodInterceptor
   -testclass test.methodinterceptors.FooTest
```

For the equivalent ant syntax, see the listeners attribute in the [ant documentation](ant.md).

For example, here is a Method Interceptor that will reorder the methods so that test methods that belong to the group "fast" are always run first:

```java 
public List<IMethodInstance> intercept(List<IMethodInstance> methods, ITestContext context) {
  List<IMethodInstance> result = new ArrayList<IMethodInstance>();
  for (IMethodInstance m : methods) {
    Test test = m.getMethod().getConstructorOrMethod().getAnnotation(Test.class);
    Set<String> groups = new HashSet<String>();
    for (String group : test.groups()) {
      groups.add(group);
    }
    if (groups.contains("fast")) {
      result.add(0, m);
    }
    else {
      result.add(m);
    }
  }
  return result;
}
``` 

### TestNG Listeners

There are several interfaces that allow you to modify TestNG's behavior. These interfaces are broadly called "TestNG Listeners". Here are a few listeners:

* IAnnotationTransformer ([doc](#annotation-transformers), [javadoc](https://jitpack.io/com/github/cbeust/testng/master/javadoc/org/testng/IAnnotationTransformer.html))
* IAnnotationTransformer2 ([doc](#annotation-transformers), [javadoc](https://jitpack.io/com/github/cbeust/testng/master/javadoc/org/testng/IAnnotationTransformer2.html))
* IHookable ([doc](#overriding-test-methods), [javadoc](https://jitpack.io/com/github/cbeust/testng/master/javadoc/org/testng/IHookable.html))
* IInvokedMethodListener ([doc](#method-interceptors), [javadoc](https://jitpack.io/com/github/cbeust/testng/master/javadoc/org/testng/IInvokedMethodListener.html))
* IMethodInterceptor ([doc](#method-interceptors)), [javadoc](https://jitpack.io/com/github/cbeust/testng/master/javadoc/org/testng/IMethodInterceptor.html))
* IReporter ([doc](logging.md#logging-reporters), [javadoc](https://jitpack.io/com/github/cbeust/testng/master/javadoc/org/testng/IReporter.html))
* ISuiteListener (doc, [javadoc](https://jitpack.io/com/github/cbeust/testng/master/javadoc/org/testng/ISuiteListener.html))
* ITestListener ([doc](logging.md#logging-listners), [javadoc](https://jitpack.io/com/github/cbeust/testng/master/javadoc/org/testng/ITestListener.html))

When you implement one of these interfaces, you can let TestNG know about it with either of the following ways:

* Using -listener on the [command line](runningtestng.md).
* Using &lt;listeners&gt; with [ant](ant.md).
* Using &lt;listeners&gt; in your testng.xml file.
* Using the @Listeners annotation on any of your test classes.
* Using ServiceLoader.

#### Specifying listeners with testng.xml or in Java

Here is how you can define listeners in your testng.xml file:
```xml
<suite>
 
  <listeners>
    <listener class-name="com.example.MyListener" />
    <listener class-name="com.example.MyMethodInterceptor" />
  </listeners>
 
...
``` 

Or if you prefer to define these listeners in Java:
```java
@Listeners({ com.example.MyListener.class, com.example.MyMethodInterceptor.class })
public class MyTest {
  // ...
}
``` 

The @Listeners annotation can contain any class that extends org.testng.ITestNGListener except IAnnotationTransformer and IAnnotationTransformer2. The reason is that these listeners need to be known very early in the process so that TestNG can use them to rewrite your annotations, therefore you need to specify these listeners in your testng.xml file.

> Note that the @Listeners annotation will apply to your entire suite file, just as if you had specified it in a testng.xml file. If you want to restrict its scope (for example, only running on the current class), the code in your listener could first check the test method that's about to run and decide what to do then. Here's how it can be done.

First define a new custom annotation that can be used to specify this restriction:
```java 
@Retention(RetentionPolicy.RUNTIME)
@Target ({ElementType.TYPE})
public @interface DisableListener {}
``` 

Add an edit check as below within your regular listeners:

```java 
public void beforeInvocation(IInvokedMethod iInvokedMethod, ITestResult iTestResult) {
  ConstructorOrMethod consOrMethod =iInvokedMethod.getTestMethod().getConstructorOrMethod();
  DisableListener disable = consOrMethod.getMethod().getDeclaringClass().getAnnotation(DisableListener.class);
  if (disable != null) {
    return;
  }
  // else resume your normal operations
}
``` 
Annotate test classes wherein the listener is not to be invoked:
```java 
@DisableListener
@Listeners({ com.example.MyListener.class, com.example.MyMethodInterceptor.class })
public class MyTest {
  // ...
}
``` 
#### Specifying listeners with ServiceLoader

Finally, the JDK offers a very elegant mechanism to specify implementations of interfaces on the class path via the [ServiceLoader](http://download.oracle.com/javase/6/docs/api/java/util/ServiceLoader.html) class.

With ServiceLoader, all you need to do is create a jar file that contains your listener(s) and a few configuration files, put that jar file on the classpath when you run TestNG and TestNG will automatically find them.

Here is a concrete example of how it works.

Let's start by creating a listener (any TestNG listener should work):

```java 
package test.tmp;
 
public class TmpSuiteListener implements ISuiteListener {
  @Override
  public void onFinish(ISuite suite) {
    System.out.println("Finishing");
  }
 
  @Override
  public void onStart(ISuite suite) {
    System.out.println("Starting");
  }
}
``` 
Compile this file, then create a file at the location META-INF/services/org.testng.ITestNGListener, which will name the implementation(s) you want for this interface.

You should end up with the following directory structure, with only two files:

``` 
$ tree
|____META-INF
| |____services
| | |____org.testng.ITestNGListener
|____test
| |____tmp
| | |____TmpSuiteListener.class
$ cat META-INF/services/org.testng.ITestNGListener
test.tmp.TmpSuiteListener

``` 
Create a jar of this directory:
``` 
$ jar cvf ../sl.jar .
added manifest
ignoring entry META-INF/
adding: META-INF/services/(in = 0) (out= 0)(stored 0%)
adding: META-INF/services/org.testng.ITestNGListener(in = 26) (out= 28)(deflated -7%)
adding: test/(in = 0) (out= 0)(stored 0%)
adding: test/tmp/(in = 0) (out= 0)(stored 0%)
adding: test/tmp/TmpSuiteListener.class(in = 849) (out= 470)(deflated 44%)

``` 

Next, put this jar file on your classpath when you invoke TestNG:

``` 
$ java -classpath sl.jar:testng.jar org.testng.TestNG testng-single.yaml
Starting
f2 11 2
PASSED: f2("2")
Finishing
``` 
This mechanism allows you to apply the same set of listeners to an entire organization just by adding a jar file to the classpath, instead of asking every single developer to remember to specify these listeners in their testng.xml file.

###  Dependency injection

TestNG supports two different kinds of dependency injection: native (performed by TestNG itself) and external (performed by a dependency injection framework such as Guice).

#### Native dependency injection

TestNG lets you declare additional parameters in your methods. When this happens, TestNG will automatically fill these parameters with the right value. 

Dependency injection can be used in the following places:

* Any @Before method or @Test method can declare a parameter of type ITestContext.
* Any @AfterMethod method can declare a parameter of type ITestResult, which will reflect the result of the test method that was just run.
* Any @Before and @After methods (except @BeforeSuite and @AfterSuite) can declare a parameter of type XmlTest, which contain the current &lt;test&gt; tag.
* Any @BeforeMethod (and @AfterMethod) can declare a parameter of type java.lang.reflect.Method. This parameter will receive the test method that will be called once this @BeforeMethod finishes (or after the method as run for @AfterMethod).
* Any @BeforeMethod can declare a parameter of type Object[]. This parameter will receive the list of parameters that are about to be fed to the upcoming test method, which could be either injected by TestNG, such as java.lang.reflect.Method or come from a @DataProvider.
* Any @DataProvider can declare a parameter of type ITestContext or java.lang.reflect.Method. The latter parameter will receive the test method that is about to be invoked.

You can turn off injection with the @NoInjection annotation:

```java 
public class NoInjectionTest {
 
  @DataProvider(name = "provider")
  public Object[][] provide() throws Exception {
      return new Object[][] { { CC.class.getMethod("f") } };
  }
 
  @Test(dataProvider = "provider")
  public void withoutInjection(@NoInjection Method m) {
      Assert.assertEquals(m.getName(), "f");
  }
 
  @Test(dataProvider = "provider")
  public void withInjection(Method m) {
      Assert.assertEquals(m.getName(), "withInjection");
  }
}

``` 

The below table summarises the parameter types that can be natively injected for the various TestNG annotations:

Annotation	| ITestContext | 	 XmlTest |	 Method | 	 Object[] |	 ITestResult 
---|---|---|---|---|---
BeforeSuite	| Yes |	No | No |	No |	No
BeforeTest | Yes|	Yes|	No |	No| 	No
BeforeGroups |	Yes |	Yes	| No | No |	No
BeforeClass |	Yes |	Yes |	No |	No |	No
BeforeMethod |	Yes |	Yes |	Yes |	Yes| 	Yes
Test	|Yes |	No|	No	| No |	No 
DataProvider| Yes |	No |	Yes |	No |	No
AfterMethod	| Yes |	Yes |	Yes |	Yes |	Yes
AfterClass |Yes	|Yes |	No |	No|	No
AfterGroups	|Yes |	Yes |	No|	No 	No
AfterTest	| Yes	|Yes |	No |	No| 	No
AfterSuite	| Yes |	No |	No |	No |	No


#### Guice dependency injection

If you use Guice, TestNG gives you an easy way to inject your test objects with a Guice module:
```java 
@Guice(modules = GuiceExampleModule.class)
public class GuiceTest extends SimpleBaseTest {
 
  @Inject
  ISingleton m_singleton;
 
  @Test
  public void singletonShouldWork() {
    m_singleton.doSomething();
  }
 
}
``` 

In this example, GuiceExampleModule is expected to bind the interface ISingleton to some concrete class:

```java 
public class GuiceExampleModule implements Module {
 
  @Override
  public void configure(Binder binder) {
    binder.bind(ISingleton.class).to(ExampleSingleton.class).in(Singleton.class);
  }
 
}
``` 

If you need more flexibility in specifying which modules should be used to instantiate your test classes, you can specify a module factory:

```java 
@Guice(moduleFactory = ModuleFactory.class)
public class GuiceModuleFactoryTest {
 
  @Inject
  ISingleton m_singleton;
 
  @Test
  public void singletonShouldWork() {
    m_singleton.doSomething();
  }
}
``` 

The module factory needs to implement the interface [IModuleFactory](https://jitpack.io/com/github/cbeust/testng/master/javadoc/org/testng/IModuleFactory.html):
```java 
public interface IModuleFactory {
 /**
   * @param context The current test context
   * @param testClass The test class
   *
   * @return The Guice module that should be used to get an instance of this
   * test class.
   */
  Module createModule(ITestContext context, Class<?> testClass);
}
``` 

Your factory will be passed an instance of the test context and the test class that TestNG needs to instantiate. Your createModule method should return a Guice Module that will know how to instantiate this test class. You can use the test context to find out more information about your environment, such as parameters specified in testng.xml, etc... You will get even more flexibility and Guice power with parent-module and guice-stage suite parameters. guice-stage allow you to chose the Stage used to create the parent injector. The default one is DEVELOPMENT. Other allowed values are PRODUCTION and TOOL. 

Here is how you can define parent-module in your test.xml file:
```xml
<suite parent-module="com.example.SuiteParenModule" guice-stage="PRODUCTION">
</suite>
``` 
TestNG will create this module only once for given suite. Will also use this module for obtaining instances of test specific Guice modules and module factories, then will create child injector for each test class. With such approach you can declare all common bindings in parent-module also you can inject binding declared in parent-module in module and module factory. 

Here is an example of this functionality:

```java 
package com.example;
 
public class ParentModule extends AbstractModule {
  @Override
  protected void conigure() {
    bind(MyService.class).toProvider(MyServiceProvider.class);
    bind(MyContext.class).to(MyContextImpl.class).in(Singleton.class);
  }
}
package com.example;
 
public class TestModule extends AbstractModule {
  private final MyContext myContext;
 
  @Inject
  TestModule(MyContext myContext) {
    this.myContext = myContext
  }
   
  @Override
  protected void configure() {
    bind(MySession.class).toInstance(myContext.getSession());
  }
}
``` 
```xml 
<suite parent-module="com.example.ParentModule">
</suite>
``` 

```java 
package com.example;
 
@Test
@Guice(modules = TestModule.class)
public class TestClass {
  @Inject
  MyService myService;
  @Inject
  MySession mySession;
   
  public void testServiceWithSession() {
    myService.serve(mySession);
  }
}
``` 

As you see ParentModule declares binding for MyService and MyContext classes. Then MyContext is injected using constructor injection into TestModule class, which also declare binding for MySession. Then parent-module in test XML file is set to ParentModule class, this enables injection in TestModule. Later in TestClass you see two injections: * MyService - binding taken from ParentModule * MySession - binding taken from TestModule This configuration ensures you that all tests in this suite will be run with same session instance, the MyContextImpl object is only created once per suite, this give you possibility to configure common environment [state](https://github.com/google/guice/wiki/Bootstrap) for all tests in suite.


###  Listening to method invocations
The listener [IInvokedMethodListener](https://jitpack.io/com/github/cbeust/testng/master/javadoc/org/testng/IInvokedMethodListener.html) allows you to be notified whenever TestNG is about to invoke a test (annotated with @Test) or configuration (annotated with any of the @Before or @After annotation) method. 

You need to implement the following interface:
```java 
public interface IInvokedMethodListener extends ITestNGListener {
  void beforeInvocation(IInvokedMethod method, ITestResult testResult);
  void afterInvocation(IInvokedMethod method, ITestResult testResult);
}
``` 
and declare it as a listener, as explained in the section about [TestNG listeners](#testng-listeners).

### Overriding test methods

TestNG allows you to override and possibly skip the invocation of test methods. One example of where this is useful is if you need to your test methods with a specific security manager. You achieve this by providing a listener that implements IHookable.

Here is an example with JAAS:

```java 
public class MyHook implements IHookable {
  public void run(final IHookCallBack icb, ITestResult testResult) {
    // Preferably initialized in a @Configuration method
    mySubject = authenticateWithJAAs();
    
    Subject.doAs(mySubject, new PrivilegedExceptionAction() {
      public Object run() {
        icb.callback(testResult);
      }
    };
  }
}
``` 
### Altering suites (or) tests

Sometimes you may need to just want to alter a suite (or) a test tag in a suite xml in runtime without having to change the contents of a suite file.

A classic example for this would be to try and leverage your existing suite file and try using it for simulating a load test on your "Application under test". At the minimum you would end up duplicating the contents of your &lt;test&gt; tag multiple times and create a new suite xml file and work with. But this doesn't seem to scale a lot.

TestNG allows you to alter a suite (or) a test tag in your suite xml file at runtime via listeners. You achieve this by providing a listener that implements [IAlterSuiteListener](https://jitpack.io/com/github/cbeust/testng/master/javadoc/org/testng/IAlterSuiteListener.html). Please refer to Listeners section to learn about listeners.

Here is an example that shows how the suite name is getting altered in runtime:

```java 
public class AlterSuiteNameListener implements IAlterSuiteListener {
 
    @Override
    public void alter(List<XmlSuite> suites) {
        XmlSuite suite = suites.get(0);
        suite.setName(getClass().getSimpleName());
    }
}
``` 

This listener can only be added with either of the following ways:

* Through the &lt;listeners&gt; tag in the suite xml file.
* Through a [Service Loader](#specifying-listeners-with-serviceloader)

This listener cannot be added to execution using the @Listeners annotation.
