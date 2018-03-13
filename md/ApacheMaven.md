<pre>
注意事項：
	1. 本文內容整理至 <a href="https://maven.apache.org">Apache Maven</a>
	2. 內容僅供參考
</pre>

## Catalog

[TOC]

## What is Apache Maven

The result is a tool that can now be used for building and managing any Java-based project. We hope that we have created something that will make the day-to-day work of Java developers easier and generally help with the comprehension of any Java-based project.

## Installing Apache Maven

The installation of Apache Maven is a simple process of extracting the archive and adding the `bin` folder with the `mvn` command to the `PATH`.

Detailed steps are:

* Ensure `JAVA_HOME` environment variable is set and points to your JDK installation
* Extract distribution archive in any directory

```
$ unzip apache-maven-3.3.9-bin.zip
```

or

```
$ tar xzvf apache-maven-3.3.9-bin.tar.gz
```

Alternatively use your preferred archive extraction tool.

* Add the `bin` directory of the created directory `apache-maven-3.3.9` to the `PATH` environment variable
* Confirm with `mvn -v` in a new shell. The result should look similar to

```
Apache Maven 3.3.3 (7994120775791599e205a5524ec3e0dfe41d4a06; 2015-04-22T04:57:37-07:00)
Maven home: /opt/apache-maven-3.3.3
Java version: 1.8.0_45, vendor: Oracle Corporation
Java home: /Library/Java/JavaVirtualMachines/jdk1.8.0_45.jdk/Contents/Home/jre
Default locale: en_US, platform encoding: UTF-8
OS name: "mac os x", version: "10.8.5", arch: "x86_64", family: "mac"
```

## What is Archetype?

In short, Archetype is a Maven project templating toolkit. An archetype is defined as an original pattern or model from which all other things of the same kind are made. The name fits as we are trying to provide a system that provides a consistent means of generating Maven projects. Archetype will help authors create Maven project templates for users, and provides users with the means to generate parameterized versions of those project templates.

Using archetypes provides a great way to enable developers quickly in a way consistent with best practices employed by your project or organization. Within the Maven project, we use archetypes to try and get our users up and running as quickly as possible by providing a sample project that demonstrates many of the features of Maven, while introducing new users to the best practices employed by Maven. In a matter of seconds, a new user can have a working Maven project to use as a jumping board for investigating more of the features in Maven. We have also tried to make the Archetype mechanism additive, and by that we mean allowing portions of a project to be captured in an archetype so that pieces or aspects of a project can be added to existing projects. A good example of this is the Maven site archetype. If, for example, you have used the quick start archetype to generate a working project, you can then quickly create a site for that project by using the site archetype within that existing project. You can do anything like this with archetypes.

You may want to standardize J2EE development within your organization, so you may want to provide archetypes for EJBs, or WARs, or for your web services. Once these archetypes are created and deployed in your organization's repository, they are available for use by all developers within your organization.

**Using an Archetype**

To create a new project based on an Archetype, you need to call mvn `archetype:generate` goal, like the following:

```
$ mvn archetype:generate
```

Please refer to [Archetype Plugin page](https://maven.apache.org/archetype/maven-archetype-plugin/usage.html).

**Usage**

Calling `archetype:generate` the plugin will first ask to choose the archetype from the internal catalog. Just enter the number of the archetype.

It then asks you to enter the values for the groupId, the artifactId and the version of the project to create and the base package for the sources.

It then asks for confirmation of the configuration and performs the creation of the project.

In the following example, we selected the quickstart archetype (default value, which is 15 here but may vary depending on the repository content) and set `groupId` to `com.company`, `artifactId` to `project`, `version` to `1.0`.

**Maven Archetypes**

| Archetype ArtifactIds | Description |
|-----------------------|-------------|
| maven-archetype-archetype | An archetype to generate a sample archetype. |
| maven-archetype-j2ee-simple | An archetype to generate a simplifed sample J2EE application. |
| maven-archetype-mojo (deprecated) | Deprecated in favour of maven-archetype-plugin, which has a better name. |
| maven-archetype-plugin | An archetype to generate a sample Maven plugin. |
| maven-archetype-plugin-site | An archetype to generate a sample Maven plugin site. |
| maven-archetype-portlet | An archetype to generate a sample JSR-268 Portlet. |
| maven-archetype-quickstart | An archetype to generate a sample Maven project. |
| maven-archetype-simple | An archetype to generate a simple Maven project. |
maven-archetype-site | An archetype to generate a sample Maven site which demonstrates some of the supported document types like APT,  | 
| maven-archetype-site-simple | An archetype to generate a sample Maven site. |
| maven-archetype-webapp | An archetype to generate a sample Maven Webapp project. |

## Creating a Project with Maven

You will need somewhere for your project to reside, create a directory somewhere and start a shell in that directory. On your command line, execute the following Maven goal:

```
$ mvn archetype:generate \
-DarchetypeGroupId=org.apache.maven.archetypes \
-DarchetypeArtifactId=maven-archetype-quickstart \
-DgroupId=com.mycompany.app \
-DartifactId=my-app \
-Dversion=1.0-SNAPSHOT
```

If you have just installed Maven, it may take a while on the first run. This is because Maven is downloading the most recent artifacts (plugin jars and other files) into your local repository. You may also need to execute the command a couple of times before it succeeds. This is because the remote server may time out before your downloads are complete. Don't worry, there are ways to fix that.

You will notice that the generate goal created a directory with the same name given as the artifactId. Change into that directory.

```
$ cd my-app
```

Under this directory you will notice the following standard project structure.

```
my-app
|-- pom.xml
`-- src
    |-- main
    |   `-- java
    |       `-- com
    |           `-- mycompany
    |               `-- app
    |                   `-- App.java
    `-- test
        `-- java
            `-- com
                `-- mycompany
                    `-- app
                        `-- AppTest.java
```

The `src/main/java` directory contains the project source code, the `src/test/java` directory contains the test source, and the `pom.xml` file is the project's Project Object Model, or POM.

**The POM**

The `pom.xml` file is the core of a project's configuration in Maven. It is a single configuration file that contains the majority of information required to build a project in just the way you want. The POM is huge and can be daunting in its complexity, but it is not necessary to understand all of the intricacies just yet to use it effectively. This project's POM is:

```
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
  <modelVersion>4.0.0</modelVersion>

  <groupId>com.mycompany.app</groupId>
  <artifactId>my-app</artifactId>
  <version>1.0-SNAPSHOT</version>
  <packaging>jar</packaging>

  <name>my-app</name>
  <url>http://maven.apache.org</url>

  <properties>
    <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
  </properties>

  <dependencies>
    <dependency>
      <groupId>junit</groupId>
      <artifactId>junit</artifactId>
      <version>3.8.1</version>
      <scope>test</scope>
    </dependency>
  </dependencies>
</project>
```

## Building a Project with Maven

The vast majority of Maven-built projects can be built with the following command:

```
$ mvn clean package
```

This command tells Maven to build all the modules, and to install it in the local repository. The local repository is created in your home directory (or alternative location that you created it), and is the location that all downloaded binaries and the projects you built are stored.

That's it! If you look in the `target` subdirectory, you should find the build output and the final library or application that was being built.

**Note:** Some projects have multiple modules, so the library or application you are looking for may be in a module subdirectory.

While this will build most projects and Maven encourages this standard convention, builds can be customisable. If this does not suffice, please consult the project's documentation.

You may test the newly compiled and packaged JAR with the following command:

```
$ java -cp target/my-app-1.0-SNAPSHOT.jar com.mycompany.app.App
```

Which will print the quintessential:

```
Hello World!
```

## Standard Directory Layout

Having a common directory layout would allow for users familiar with one Maven project to immediately feel at home in another Maven project. The advantages are analogous to adopting a site-wide look-and-feel.

The next section documents the directory layout expected by Maven and the directory layout created by Maven. Please try to conform to this structure as much as possible; however, if you can't these settings can be overridden via the project descriptor.

| Directory | Description |
|-----------|-------------|
| `src/main/java` | Application/Library sources |
| `src/main/resources` | Application/Library resources |
| `src/main/filters` | Resource filter files |
| `src/main/webapp` | Web application sources |
| `src/test/java` | Test sources |
| `src/test/resources` | Test resources |
| `src/test/filters` | Test resource filter files |
| `src/it` | Integration Tests (primarily for plugins) |
| `src/assembly` | Assembly descriptors |
| `src/site` | Site |
| `LICENSE.txt` | Project's license |
| `NOTICE.txt` | Notices and attributions required by libraries that the project depends on |
| `README.txt` | Project's readme |

At the top level files descriptive of the project: a `pom.xml` file (and any properties, `maven.xml` or `build.xml` if using Ant). In addition, there are textual documents meant for the user to be able to read immediately on receiving the source: `README.txt`, `LICENSE.txt`, etc.

There are just two subdirectories of this structure: `src` and `target`. The only other directories that would be expected here are metadata like `CVS` or `.svn`, and any subprojects in a multiproject build (each of which would be laid out as above).

The `target` directory is used to house all output of the build.

The `src` directory contains all of the source material for building the project, its site and so on. It contains a subdirectory for each type: `main` for the main build artifact, test for the unit `test` code and resources, `site` and so on.

Within artifact producing source directories (ie. `main` and `test`), there is one directory for the language `java` (under which the normal package hierarchy exists), and one for `resources` (the structure which is copied to the target classpath given the default resource definition).

If there are other contributing sources to the artifact build, they would be under other subdirectories: for example `src/main/antlr` would contain Antlr grammar definition files.

## What is a POM?

A Project Object Model or POM is the fundamental unit of work in Maven. It is an XML file that contains information about the project and configuration details used by Maven to build the project. It contains default values for most projects. Examples for this is the build directory, which is `target`; the source directory, which is `src/main/java`; the test source directory, which is `src/test/java`; and so on.

The POM was renamed from `project.xml` in Maven 1 to `pom.xml` in Maven 2. Instead of having a `maven.xml` file that contains the goals that can be executed, the goals or plugins are now configured in the `pom.xml`. When executing a task or goal, Maven looks for the POM in the current directory. It reads the POM, gets the needed configuration information, then executes the goal.

Some of the configuration that can be specified in the POM are the project dependencies, the plugins or goals that can be executed, the build profiles, and so on. Other information such as the project version, description, developers, mailing lists and such can also be specified.

## What is Build Lifecycle?

**Build Lifecycle Basics**

Maven is based around the central concept of a build lifecycle. What this means is that the process for building and distributing a particular artifact (project) is clearly defined.

For the person building a project, this means that it is only necessary to learn a small set of commands to build any Maven project, and the POM will ensure they get the results they desired.

There are three built-in build lifecycles: default, clean and site. The `default` lifecycle handles your project deployment, the `clean` lifecycle handles project cleaning, while the `site` lifecycle handles the creation of your project's site documentation.

**A Build Lifecycle is Made Up of Phases**

Each of these build lifecycles is defined by a different list of build phases, wherein a build phase represents a stage in the lifecycle.

For example, the default lifecycle comprises of the following phases (for a complete list of the lifecycle phases, refer to the Lifecycle Reference):

* `validate` - validate the project is correct and all necessary information is available
* `compile` - compile the source code of the project
* `test` - test the compiled source code using a suitable unit testing framework. These tests should not require the code be packaged or deployed
* `package` - take the compiled code and package it in its distributable format, such as a JAR.
* `integration-test` - process and deploy the package if necessary into an environment where integration tests can be run
* `verify` - run any checks to verify the package is valid and meets quality criteria
* `install` - install the package into the local repository, for use as a dependency in other projects locally
* `deploy` - done in an integration or release environment, copies the final package to the remote repository for sharing with other developers and projects.

These lifecycle phases (plus the other lifecycle phases not shown here) are executed sequentially to complete the default lifecycle. Given the lifecycle phases above, this means that when the default lifecycle is used, Maven will first validate the project, then will try to compile the sources, run those against the tests, package the binaries (e.g. jar), run integration tests against that package, verify the package, install the verifed package to the local repository, then deploy the installed package in a specified environment.

## Running Apache Maven

The syntax for running Maven is as follows:

```
mvn [options] [<goal(s)>] [<phase(s)>]
```

All available options are documented in the built in help that you can access with

```
$ mvn -h
```

The typical invocation for building a Maven project uses a Maven life cycle phase. E.g.

```
$ mvn package
```

The built in life cycles and their phases are in order are:

* clean - pre-clean, clean, post-clean
* default - validate, initialize, generate-sources, process-sources, generate-resources, process-resources, compile, process-classes, generate-test-sources, process-test-sources, generate-test-resources, process-test-resources, test-compile, process-test-classes, test, prepare-package, package, pre-integration-test, integration-test, post-integration-test, verify, install, deploy
* site - pre-site, site, post-site, site-deploy

A fresh build of a project generating all packaged outputs and the documentation site and deploying it to a repository manager could be done with

```
$ mvn clean deploy site-deploy
```

Just creating the package and installing it in the local repository for re-use from other projects can be done with

```
$ mvn clean install
```

This is the most common build invocation for a Maven project.

When not working with a project, and in some other use cases, you might want to invoke a specific task implemented by a part of Maven - this is called a goal of a plugin. E.g.:

```
$ mvn archetype:generate
```

or

```
$ mvn checkstyle:check
```

## Feature Summary

The following are the key features of Maven in a nutshell:

* Simple project setup that follows best practices - get a new project or module started in seconds
* Consistent usage across all projects means no ramp up time for new developers coming onto a project
* Superior dependency management including automatic updating, dependency closures (also known as transitive dependencies)
* Able to easily work with multiple projects at the same time
* A large and growing repository of libraries and metadata to use out of the box, and arrangements in place with the largest Open Source projects for real-time availability of their latest releases
* Extensible, with the ability to easily write plugins in Java or scripting languages
* Instant access to new features with little or no extra configuration
* Ant tasks for dependency management and deployment outside of Maven
* Model based builds: Maven is able to build any number of projects into predefined output types such as a JAR, WAR, or distribution based on metadata about the project, without the need to do any scripting in most cases.
* Coherent site of project information: Using the same metadata as for the build process, Maven is able to generate a web site or PDF including any documentation you care to add, and adds to that standard reports about the state of development of the project. Examples of this information can be seen at the bottom of the left-hand navigation of this site under the "Project Information" and "Project Reports" submenus.
* Release management and distribution publication: Without much additional configuration, Maven will integrate with your source control system such as CVS and manage the release of a project based on a certain tag. It can also publish this to a distribution location for use by other projects. Maven is able to publish individual outputs such as a JAR, an archive including other dependencies and documentation, or as a source distribution.
* Dependency management: Maven encourages the use of a central repository of JARs and other dependencies. Maven comes with a mechanism that your project's clients can use to download any JARs required for building your project from a central JAR repository much like Perl's CPAN. This allows users of Maven to reuse JARs across projects and encourages communication between projects to ensure that backward compatibility issues are dealt with.



