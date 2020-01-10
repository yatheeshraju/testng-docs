# Getting Started 


I started TestNG out of frustration for some JUnit deficiencies which I have documented on my weblog [here](https://beust.com/weblog/2004/08/25/testsetup-and-evil-static-methods/) and [here](https://beust.com/weblog/2004/02/08/junit-pain/) Reading these entries might give you a better idea of the goal I am trying to achieve with TestNG.  You can also check out a quick overview of the [main features](http://www.beust.com/weblog/archives/000176.html) and an [article](https://beust.com/weblog/2004/08/18/using-annotation-inheritance-for-testing/) describing a very concrete example where the combined use of several TestNG's features provides for a very intuitive and maintainable testing design.

Here is a very simple test:

```java
package example1;
 
import org.testng.annotations.*;
 
public class SimpleTest {
 
 @BeforeClass
 public void setUp() {
   // code that will be invoked when this test is instantiated
 }
 
 @Test(groups = { "fast" })
 public void aFastTest() {
   System.out.println("Fast test");
 }
 
 @Test(groups = { "slow" })
 public void aSlowTest() {
    System.out.println("Slow test");
 }
 
}
```

The method setUp() will be invoked after the test class has been built and before any test method is run.  In this example, we will be running the group fast, so aFastTest() will be invoked while aSlowTest() will be skipped.

Things to note:

* No need to extend a class or implement an interface.
* Even though the example above uses the JUnit conventions, our methods can be called any name you like, it's the annotations that tell TestNG what they are.
* A test method can belong to one or several groups.

Once you have compiled your test class into the build directory, you can invoke your test with the command line, an ant task (shown below) or an XML file:

```xml 
<project default="test">
 
 <path id="cp">
   <pathelement location="lib/testng-testng-5.13.1.jar"/>
   <pathelement location="build"/>
 </path>
 
 <taskdef name="testng" classpathref="cp"
          classname="org.testng.TestNGAntTask" />
 
 <target name="test">
   <testng classpathref="cp" groups="fast">
     <classfileset dir="build" includes="example1/*.class"/>
   </testng>
 </target>
 
</project>
```

Use ant to invoke it:

```
c:> ant
Buildfile: build.xml
 
test:
[testng] Fast test
[testng] ===============================================
[testng] Suite for Command line test
[testng] Total tests run: 1, Failures: 0, Skips: 0
[testng] ===============================================
 
 
BUILD SUCCESSFUL
Total time: 4 seconds

```
Then you can browse the result of your tests:

``` 
start test-output\index.html (on Windows)
```

