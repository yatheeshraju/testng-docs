# Download 

### Current Release Version

#### Maven

```xml
<repositories>
  <repository>
    <id>jcenter</id>
    <name>bintray</name>
    <url>http://jcenter.bintray.com</url>
  </repository>
</repositories>
 
<dependency>
  <groupId>org.testng</groupId>
  <artifactId>testng</artifactId>
  <version>6.10</version>
  <scope>test</scope>
</dependency>
```

#### Gradle

```javascript 
repositories {
    jcenter()
}
 
dependencies {
    testCompile 'org.testng:testng:6.10'
}
```

### Snapshots

TestNG automatically uploads snapshots to Sonatype which you can access by adding the following repository:

```
repositories {
    maven {
        url 'https://oss.sonatype.org/content/repositories/snapshots'
    }
}
```

### Eclipse plug-in

Java 1.7+ is required for running the TestNG for Eclipse plugin.

Eclipse 4.2 and above is required. Eclipse 3.x is NOT supported any more, please update your Eclipse to 4.2 or above.

You can use either the [Eclipse Marketplace](http://marketplace.eclipse.org/content/testng-eclipse) or the update site:

##### Install via Eclipse Marketplace

Go to the [TestNG page on the Eclipse Market](https://marketplace.eclipse.org/content/testng-eclipse) Place and drag the icon called "Install" onto your workspace.

##### Install from update site

* Select Help / Install New Software...
* Enter the update site URL in "Work with:" field:
* Update site for release: [http://dl.bintray.com/testng-team/testng-eclipse-release/](http://dl.bintray.com/testng-team/testng-eclipse-release/).
* Make sure the check box next to URL is checked and click Next.
* Eclipse will then guide you through the process.

You can also install older versions of the plug-ins here. Note that the URL's on this page are update sites as well, not direct download links.

#### Build TestNG from source code

TestNG is also hosted on [GitHub](https://github.com/cbeust/testng), where you can download the source and build the distribution yourself:

```
$ git clone git://github.com/cbeust/testng.git
$ cd testng
$ ./build-with-gradle
```

You will then find the jar file in the target directory

#### Build the TestNG Eclipse Plugin from source code

TestNG Eclipse Plugin is hosted on [GitHub](https://github.com/cbeust/testng-eclipse), you can download the source code and [build by ourself](https://github.com/cbeust/testng-eclipse/blob/master/README.md#building).