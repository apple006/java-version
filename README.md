# OSGL Version

[![APL v2](https://img.shields.io/badge/license-Apache%202-blue.svg)](http://www.apache.org/licenses/LICENSE-2.0.html) 
[![Maven Central](https://img.shields.io/maven-central/v/org.osgl/osgl-version.svg)](http://search.maven.org/#search%7Cga%7C1%7Cosgl-version)
[![Build Status](https://travis-ci.org/osglworks/java-version.svg?branch=master)](https://travis-ci.org/osglworks/java-version)
[![codecov](https://codecov.io/gh/osglworks/java-version/branch/master/graph/badge.svg)](https://codecov.io/gh/osglworks/java-version)
[![Javadocs](http://www.javadoc.io/badge/org.osgl/osgl-version.svg?color=red)](http://www.javadoc.io/doc/org.osgl/osgl-version)

A simple tool to prepare version information when building application/library plus access version information at runtime

## Installation

Add the following dependency into your `pom.xml` file:

```xml
<dependency>
    <groupId>org.osgl</groupId>
    <artifactId>osgl-version</artifactId>
    <version>${osgl-version.version}</version>
</dependency>
```

## Prepare version info

For library/app author, you need to prepare your version info so that Version tool can generate version at runtime. 

Let's say your library/app package name is `org.mrcool.swissknife`, you need to add a file named `.version` into `src/resources/org/mrcool/swissknife` dir, the file content should be:

```properties
# artifact is optional, if not provided the package name will be used
artifact=<delivery-name>

# version is mandatory, if not provided then UNKNOWN version will be returned
version=<the project version>

# build number is optional, if not provided then empty string will be used
build=<SCM build number, e.g. git hash>
``` 
As a good practice you can rely on maven's resource filtering to automatically generate the version and build number for you, in which case your `.version` would look like:

```properties
artifact=${project.artifactId}
version=${project.version}
## build number is optional
build=${buildNumber}
```

Where the `${project.artifactId}` and `${project.version}` comes from standard maven environment, while `${buildNumber}` comes from the [buildnumber maven plugin](http://www.mojohaus.org/buildnumber-maven-plugin/).

**Note** to use maven environment variables in your `.version` file, you must enable filter in your resource plugin:

```xml
<resources>
  <resource>
    <directory>src/main/resources</directory>
    <filtering>true</filtering>
    <includes>
      <include>**/.version</include>
    </includes>
   </resource>
</resources>
```

## Get version info at runtime

Once you have prepared your `.version` file and packaged it into the jar file, the user of the jar file can always access the version information through simple API call:

```java
Version version1 = Version.of(org.mrcool.swissknife.SwissKnife.class);
System.out.println(version1.getPackage()); // print `org.mrcool.swissknife`
System.out.println(version1.getArtifactId()); // print `swissknife`
System.out.println(version1.getProjectVersion()); // print `1.0`
System.out.println(version1.getBuildNumber()); // print `ebf1`
System.out.println(version1.getVersion()); // print `r1.0-ebf1`
System.out.println(version1); // print `swissknife-r1.0-ebf1`

// Another method to get Version info
Version version2 = Version.of("org.mrcool.swissknife.db");

// If a certain library's version is SNAPSHOT, e.g. 1.0-SNAPSHOT, 
// then the version tag is decorated with `v` instead of `r`:
System.out.println(version2.getProjectVersion()); // print `1.0-SNAPSHOT`
System.out.println(version2.getBuildNumber()); // print `51b9`
System.out.println(version2.getVersion()); // print `v1.0-SNAPSHOT-51b9`
System.out.println(version2); // print `swissknife-v1.0-SNAPSHOT-ebf1`
```

**Tips** If app or library needs to decide it's own version there are shortcut way for that:

```java
package com.myproj;

public class Foo {
    public Version version() {
        // normal way to get Foo's version:
        return Version.of(Foo.class);
    }
    
    public Version version2() {
        // easy way (but more runtime cost) to get Foo's version
        return Version.get();
    }
}
```

Initially Version tool will hit the resource file to load the version info, once it is loaded, the tool will cache the loaded version instance with the package name so that next time it won't hit any I/O operation for the same package name.
