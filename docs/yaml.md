# YAML

TestNG supports YAML as an alternate way of specifying your suite file. For example, the following XML file:

```xml
<suite name="SingleSuite" verbose="2" thread-count="4">
 
  <parameter name="n" value="42" />
 
  <test name="Regression2">
    <groups>
      <run>
        <exclude name="broken" />
      </run>
    </groups>
 
    <classes>
      <class name="test.listeners.ResultEndMillisTest" />
    </classes>
  </test>
</suite>
```
and here is its YAML version:

```yaml
name: SingleSuite
threadCount: 4
parameters: { n: 42 }
 
tests:
  - name: Regression2
    parameters: { count: 10 }
    excludedGroups: [ broken ]
    classes:
      - test.listeners.ResultEndMillisTest

``` 

Here is [TestNG's own suite ](https://github.com/cbeust/testng/blob/master/src/test/resources/testng.xml)file, and its [YAML counterpart](https://github.com/cbeust/testng/blob/master/src/test/resources/testng.yaml).
You might find the YAML file format easier to read and to maintain. YAML files are also recognized by the TestNG Eclipse plug-in. You can find more information about YAML and TestNG in this blog post.

!!! info 
    TestNG by default does not bring in the YAML related library into your classpath. So depending upon your build system (Gradle/Maven) you need to add an explicit reference to YAML library in your build file.
    
For e.g, If you were using Maven, you would need to add a dependency as below into your pom.xml file:

```xml
<dependency>
  <groupid>org.yaml</groupid>
  <artifactid>snakeyaml</artifactid>
  <version>1.23</version>
</dependency>
```
Or if you were using Gradle, you would add a dependency as below into your build.gradle file:
```javascript
compile group: 'org.yaml', name: 'snakeyaml', version: '1.23'
```
