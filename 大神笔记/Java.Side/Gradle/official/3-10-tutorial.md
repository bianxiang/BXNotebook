[toc]

## 3. 教程

第4到第10章是教程。

## 4. 安装

Gradle自带Groovy，因此Groovy不是必需的。

只需要将`GRADLE_HOME/bin`添加到`PATH`。

## 5. Troubleshooting

## 6. 构建脚本基础

### 6.1. 工程和任务

Gradle中一切都建立在两个概念上：工程和任务。

每个Gradle构建由一个或多个工程组成。 工程不一定是一个需要被构建的东西。工程可以表示要做的事情，如部署应用到产品环境。Don't worry if this seems a little vague for now. Gradle's build-by-convention support adds a more concrete definition for what a project is.

每个工程由一个或多个任务组成。 A task represents some atomic piece of work which a build performs. This might be compiling some classes, creating a JAR, generating javadoc, or publishing some archives to a repository.

### 6.2. Hello world

运行`gradle`命令开始一次构建。`gradle`命令在当前目录寻找一个`build.gradle`文件，这个文件被称做构建脚本，但严格讲它是一个构建配置脚本。构建脚本定义一个工程和它的任务。

创建一个`build.gradle`：

	task hello {
	    doLast {
	        println 'Hello world!'
	    }
	}

运行`gradle -q hello`

> 我们的多数例子都会加`-q`选项。This suppresses Gradle's log messages, so that only the output of the tasks is shown.

上面的脚本定义一个任务，向任务添加了一个操作（action）。这个操作是一个简单的闭包。

Gradle的任务与Ant的目标等价。

### 6.3. 任务定义的快捷方式

	task hello << {
	    println 'Hello world!'
	}

### 6.4. 构建脚本是代码

Gradle脚本可以充分利用Groovy。例子：

	task upper << {
	    String someString = 'mY_nAmE'
	    println "Original: " + someString 
	    println "Upper case: " + someString.toUpperCase()
	}

或

	task count << {
	    4.times { print "$it " }
	}

### 6.5. 任务依赖

	task hello << {
	    println 'Hello world!'
	}
	task intro(dependsOn: hello) << {
	    println "I'm Gradle"
	}

运行：`gradle -q intro`

To add a dependency, the corresponding task does not need to exist.

	task taskX(dependsOn: 'taskY') << {
	    println 'taskX'
	}
	task taskY << {
	    println 'taskY'
	}

运行：`gradle -q taskX`

The dependency of taskX to taskY is declared before taskY is defined. This is very important for multi-project builds. Task dependencies are discussed in more detail in Section 15.4, “Adding dependencies to a task”.

Please notice that you can't use shortcut notation (see Section 6.8, “Shortcut notations”) when referring to a task that is not yet defined.

### 6.6. 动态任务

用Groovy动态创建任务：

	4.times { counter ->
	    task "task$counter" << {
	        println "I'm task number $counter"
	    }
	}

运行：`gradle -q task1`

### 6.7. 操纵存在的任务

创建任务后，可以继续修改它。如为其添加依赖：

	4.times { counter ->
	    task "task$counter" << {
	        println "I'm task number $counter"
	    }
	}
	task0.dependsOn task2, task3

运行：

	> gradle -q task0
	I'm task number 2
	I'm task number 3
	I'm task number 0

或向任务增加新行为：

	task hello << {
	    println 'Hello Earth'
	}
	hello.doFirst {
	    println 'Hello Venus'
	}
	hello.doLast {
	    println 'Hello Mars'
	}
	hello << {
	    println 'Hello Jupiter'
	}

运行：

	> gradle -q hello
	Hello Venus
	Hello Earth
	Hello Mars
	Hello Jupiter

`doFirst`和`doLast`可以多次调用，分别向任务的操作列表的开头和结尾添加任务。当任务执行时，列表中的操作按顺序执行。`<<`运算符是`doLast`的别名。

### 6.8. Shortcut notations

As you might have noticed in the previous examples, there is a convenient notation for accessing an existing task. 任务可以当构建脚本的属性被访问。

	task hello << {
	    println 'Hello world!'
	}
	hello.doLast {
	    println "Greetings from the $hello.name task."
	}

This enables very readable code, especially when using the out of the box tasks provided by the plugins (e.g. compile).

### 6.9. 任务的额外属性

可以向任务添加自定义的属性。To add a property named myProperty, set `ext`.myProperty to an initial value.

	task myTask {
	    ext.myProperty = "myValue"
	}
	
	task printTaskProperties << {
	    println myTask.myProperty
	}

Extra properties aren't limited to tasks. You can read more about them in Section 13.4.2, “Extra properties”.

### 6.10. Using Ant Tasks

Ant tasks are first-class citizens in Gradle. Gradle provides excellent integration for Ant tasks by simply relying on Groovy. Groovy is shipped with the fantastic **AntBuilder**. Using Ant tasks from Gradle is as convenient and more powerful than using Ant tasks from a build.xml file. From the example below, you can learn how to execute ant tasks and how to access ant properties:

	task loadfile << {
	    def files = file('../antLoadfileResources').listFiles().sort()
	    files.each { File file ->
	        if (file.isFile()) {
	            ant.loadfile(srcFile: file, property: file.name)
	            println " *** $file.name ***"
	            println "${ant.properties[file.name]}"
	        }
	    }
	}

运行：

	> gradle -q loadfile
	*** agile.manifesto.txt ***
	Individuals and interactions over processes and tools
	Working software over comprehensive documentation
	Customer collaboration  over contract negotiation
	Responding to change over following a plan
	 *** gradle.manifesto.txt ***
	Make the impossible possible, make the possible easy and make the easy elegant.
	(inspired by Moshe Feldenkrais)

### 6.11. 使用方法

组织构建逻辑的第一步是抽象出方法：

	task checksum << {
	    fileList('../antLoadfileResources').each {File file ->
	        ant.checksum(file: file, property: "cs_$file.name")
	        println "$file.name Checksum: ${ant.properties["cs_$file.name"]}"
	    }
	}
	
	task loadfile << {
	    fileList('../antLoadfileResources').each {File file ->
	        ant.loadfile(srcFile: file, property: file.name)
	        println "I'm fond of $file.name"
	    }
	}
	
	File[] fileList(String dir) {
	    file(dir).listFiles({file -> file.isFile() } as FileFilter).sort()
	}

后面可以看到，在一个多工程构建中，这些方法可以被子工程使用。 If your build logic becomes more complex, Gradle offers you other very convenient ways to organize it. We have devoted a whole chapter to this. See Chapter 59, Organizing Build Logic.

### 6.12. 默认任务

可以定义一个或多个任务：

	defaultTasks 'clean', 'run'
	
	task clean << {
	    println 'Default Cleaning!'
	}
	
	task run << {
	    println 'Default Running!'
	}
	
	task other << {
	    println "I'm not a default task!"
	}

运行：

	> gradle -q
	Default Cleaning!
	Default Running!

在过过程构建中，每个子工程可以有自己的默认任务。若子工程未指定默认任务，将使用父工程的默认任务。

### 6.13. Configure by DAG

As we later describe in full detail (see Chapter 55, The Build Lifecycle), Gradle包含一个配置阶段和一个执行阶段。配置阶段后，Gradle知道所有要执行的任务。 Gradle offers you a hook to make use of this information. A use-case for this would be to check if the release task is among the tasks to be executed. Depending on this, you can assign different values to some variables.

In the following example, execution of the `distribution` and `release` tasks results in different value of the version variable.

	task distribution << {
	    println "We build the zip with version=$version"
	}
	
	task release(dependsOn: 'distribution') << {
	    println 'We release now'
	}
	
	gradle.taskGraph.whenReady {taskGraph ->
	    if (taskGraph.hasTask(release)) {
	        version = '1.0'
	    } else {
	        version = '1.0-SNAPSHOT'
	    }
	}

Output of `gradle -q distribution`

	> gradle -q distribution
	We build the zip with version=1.0-SNAPSHOT

Output of `gradle -q release`

	> gradle -q release
	We build the zip with version=1.0
	We release now

The important thing is that whenReady affects the release task before the release task is executed. This works even when the release task is not the primary task (i.e., the task passed to the gradle command).

## 7. Java 快速入门

### 7.1. Java插件

A plugin is an extension to Gradle which configures your project in some way, 一般会添加一些预定义的任务。 Gradle自带很多插件，编写自己的插件也很简单。

Java插件是基于约定的。即插件对工程的很多方面都定义了默认值，如源文件放在哪。如果你的工程遵循约定，一般不需要要构建脚本里写态度。若你的工程不遵循约定，Gradle允许你定制。实际上，因为对Java工程的支持实现为一个插件，你甚至不需要使用这个插件来构建你的Java工程。

### 7.2. 一个简单的Java工程

> The code for this example can be found at `samples/java/quickstart` which is in both the binary and source distributions of Gradle.

向构建脚本添加：

	apply plugin: 'java'

This is all you need to define a Java project. 

> 可以利用`gradle tasks`列出工程的所有任务。

Gradle期望源文件在`src/main/java`，测试源文件在`src/test/java`。In addition, any files under `src/main/resources` will be included in the JAR file as resources, and any files under `src/test/resources` will be included in the classpath used to run the tests. 输出目录是`build`。Jar文件在`build/libs`。

#### 7.2.1. 构建工程

Java插件向你的工程添加了很多任务。但构建工程只需要用几个。最常用的任务是`build`，会对工程进行完整构建。当你运行`gradle build`，Gradle会编译测试代码，创建JAR包：

	> gradle build
	:compileJava
	:processResources
	:classes
	:jar
	:assemble
	:compileTestJava
	:processTestResources
	:testClasses
	:test
	:check
	:build

	BUILD SUCCESSFUL
	
	Total time: 1 secs

其他任务有：

- `clean`：删除`build`目录和文件。
- `assemble`：编译打包代码。但不运行单元测试。Other plugins add more artifacts to this task. For example, if you use the **War** plugin, this task will also build the WAR file for your project. 
- `check`：编译并测试代码。Other plugins add more checks to this task. For example, if you use the **checkstyle** plugin, this task will also run Checkstyle against your source code. 

#### 7.2.2. 外部依赖

In Gradle, artifacts such as JAR files, are located in a repository. 工程从仓库获取其依赖。工程会向仓库发布构建。例如，使用Maven仓库：

	repositories {
	    mavenCentral()
	}

例子。 Here, we will declare that our production classes have a compile-time dependency on commons collections, and that our test classes have a compile-time dependency on junit:

	dependencies {
	    compile group: 'commons-collections', name: 'commons-collections', version: '3.2'
	    testCompile group: 'junit', name: 'junit', version: '4.+'
	}

You can find out more in Chapter 8, Dependency Management Basics.

#### 7.2.3. 定制工程

Java插件向工程添加了几个属性。这些属性的默认值一般是合适的。若不，改变很容易。例子，改变工程版本，改变我们源码的Java版本。同时向JAR manifest添加一个特性：

	sourceCompatibility = 1.5
	version = '1.0'
	jar {
	    manifest {
	        attributes 'Implementation-Title': 'Gradle Quickstart', 'Implementation-Version': version
	    }
	}

> 利用 `gradle properties` 列出工程的属性。

Java插件添加的任务与我们自己写的任务没什么区别。我们也可以设置它们的属性，添加行为，改变任务的依赖，或替换整个任务。In our sample, we will configure the `test` task, which is of type Test, to add a system property when the tests are executed:

	test {
	    systemProperties 'property': 'value'
	}
	
#### 7.2.4. 发布Jar

Usually the JAR file needs to be published somewhere. To do this, you need to tell Gradle where to publish the JAR file. In Gradle, artifacts such as JAR files are published to repositories. In our sample, we will publish to a local directory. You can also publish to a remote location, or multiple locations.

	uploadArchives {
	    repositories {
	       flatDir {
	           dirs 'repos'
	       }
	    }
	}

运行：`gradle uploadArchives`。

#### 7.2.5. 创建一个Eclipse工程

To import your project into Eclipse, you need to add another plugin to your build file:

	apply plugin: 'eclipse'

Now execute `gradle eclipse` command to generate Eclipse project files. More on Eclipse task can be found in Chapter 38, The Eclipse Plugin.

#### 7.2.6. 总结

下面是一个完整的构建脚本：
	
	apply plugin: 'java'
	apply plugin: 'eclipse'
	
	sourceCompatibility = 1.5
	version = '1.0'
	jar {
	    manifest {
	        attributes 'Implementation-Title': 'Gradle Quickstart', 'Implementation-Version': version
	    }
	}
	
	repositories {
	    mavenCentral()
	}
	
	dependencies {
	    compile group: 'commons-collections', name: 'commons-collections', version: '3.2'
	    testCompile group: 'junit', name: 'junit', version: '4.+'
	}
	
	test {
	    systemProperties 'property': 'value'
	}
	
	uploadArchives {
	    repositories {
	       flatDir {
	           dirs 'repos'
	       }
	    }
	}

### 7.3. 多工程Java构建


下面是工程布局：

	multiproject/
	  api/
	  services/webservice/
	  shared/

> The code for this example can be found at samples/java/multiproject which is in both the binary and source distributions of Gradle.

**api**工程产生一个JAR。**webservice**是一个Web工程。**shared**工程的代码供**api**和**webservice**共享。

#### 7.3.1. 定义一个多工程的构建

需要在源码顶级目录定义一个设置文件。必须叫做**settings.gradle**。

	include "shared", "api", "services:webservice", "services:shared"

You can find out more about the settings file in Chapter 56, Multi-project Builds.


### 7.3.2. 共用的配置

多工程构建中，工程之间一般有共用的配置。In our sample, we will define this common configuration in the root project, using a technique called configuration injection. Here, the root project is like a container and the subprojects method iterates over the elements of this container - the projects in this instance - and injects the specified configuration. This way we can easily define the manifest content for all archives, and some common dependencies:

build.gradle

	subprojects {
	    apply plugin: 'java'
	    apply plugin: 'eclipse-wtp'
	
	    repositories {
	       mavenCentral()
	    }
	
	    dependencies {
	        testCompile 'junit:junit:4.11'
	    }
	
	    version = '1.0'
	
	    jar {
	        manifest.attributes provider: 'gradle'
	    }
	}

Notice that our sample applies the Java plugin to each subproject. So, you can compile, test, and JAR **all** the projects by running `gradle build` from the root project directory.

#### 7.3.3. 工程间的依赖

在**api**的构建文件中我们添加对**shared**工程的依赖。Gradle会确保在编译**api**前先编译**shared**。

api/build.gradle

	dependencies {
	    compile project(':shared')
	}

See Section 56.7.1, “Disabling the build of dependency projects” for how to disable this functionality.

#### 7.3.4. 创建分发

We also add a distribution, that gets shipped to the client:

api/build.gradle

	task dist(type: Zip) {
	    dependsOn spiJar
	    from 'src/dist'
	    into('libs') {
	        from spiJar.archivePath
	        from configurations.runtime
	    }
	}
	
	artifacts {
	   archives dist
	}

## 8. 依赖管理基础


### 8.1. 什么是依赖管理

依赖管理由两部分组成。首先，Gradle需要知道你的工程的依赖。Secondly, Gradle needs to build and upload the things that your project produces. We call these outgoing files the **publications** of the project. Let's look at these two pieces in more detail:

依赖可能需要从远程Maven或Ivy仓库下载，或从本地目录，或从多工程构建中的另一个工程。

依赖本身也有依赖。

### 8.2. 声明你的依赖


build.gradle

	apply plugin: 'java'
	
	repositories {
	    mavenCentral()
	}
	
	dependencies {
	    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
	    testCompile group: 'junit', name: 'junit', version: '4.+'
	}

### 8.3. 依赖配置

In Gradle dependencies are grouped into configurations. 一个配置是一组命名的依赖。称它们是依赖配置。As we will see later, they are also used to declare the publications of your project.

Java插件定义了一组标准的配置。这些配置表示Java插件使用的类路径。下面列出了部分，and you can find more details in Table 23.5, “Java plugin - dependency configurations”.

- **compile**：The dependencies required to compile the production source of the project.
- **runtime**：The dependencies required by the production classes at runtime. By default, also includes the compile time dependencies.
- **testCompile**：The dependencies required to compile the test source of the project. By default, also includes the compiled production classes and the compile time dependencies.
- **testRuntime**：The dependencies required to run the tests. By default, also includes the compile, runtime and test compile dependencies. 

Various plugins add further standard configurations. You can also define your own custom configurations to use in your build. Please see Section 50.3, “Dependency configurations” for the details of defining and customizing dependency configurations.

### 8.4. 外部依赖

There are various types of dependencies that you can declare. 其中一种是外部依赖。This a dependency on some files built outside the current build, and stored in a repository of some kind, such as Maven central, or a corporate Maven or Ivy repository, or a directory in the local file system.

To define an external dependency, you add it to a dependency configuration:

build.gradle

	dependencies {
	    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
	}

外部依赖的标识包括`group`, `name` 和 `version` 三部分。有些仓库不使用`group`和`version`。

声明外部依赖的简便方法是使用字符串：`"group:name:version"`.

build.gradle

	dependencies {
	    compile 'org.hibernate:hibernate-core:3.6.7.Final'
	}

To find out more about defining and working with dependencies, have a look at Section 50.4, “How to declare your dependencies”. 

### 8.5. 仓库

Gradle从仓库中寻找依赖。Gradle understands several different repository formats, such as Maven and Ivy, and several different ways of accessing the repository, such as using the local file system or HTTP.

Gradle默认不定义任何仓库。若想使用外部依赖，需要我们至少定义一个。比如使用Maven中央仓库：

build.gradle

	repositories {
	    mavenCentral()
	}

或者某个远程Maven仓库：

build.gradle

	repositories {
	    maven {
	        url "http://repo.mycompany.com/maven2"
	    }
	}

Or a remote Ivy repository:

build.gradle

	repositories {
	    ivy {
	        url "http://repo.mycompany.com/repo"
	    }
	}

You can also have repositories on the local file system. This works for both Maven and Ivy repositories.

build.gradle
	
	repositories {
	    ivy {
	        // URL can refer to a local directory
	        url "../local-repo"
	    }
	}

一个工程可以有多个仓库。Gradle会依次寻找，找到后便不再寻找。

To find out more about defining and working with repositories, have a look at Section 50.6, “Repositories”.

### 8.6. Publishing artifacts

Dependency configurations are also used to publish files. We call these files publication artifacts, or usually just artifacts.

The plugins do a pretty good job of defining the artifacts of a project, so you usually don't need to do anything special to tell Gradle what needs to be published. However, you do need to tell Gradle where to publish the artifacts. You do this by attaching repositories to the uploadArchives task. Here's an example of publishing to a remote Ivy repository:

build.gradle
	
	uploadArchives {
	    repositories {
	        ivy {
	            credentials {
	                username "username"
	                password "pw"
	            }
	            url "http://repo.mycompany.com"
	        }
	    }
	}

Now, when you run `gradle uploadArchives`, Gradle will build and upload your Jar. Gradle will also generate and upload an ivy.xml as well.

You can also publish to Maven repositories. The syntax is slightly different. Note that you also need to apply the Maven plugin in order to publish to a Maven repository. In this instance, Gradle will generate and upload a **pom.xml**.

build.gradle

	apply plugin: 'maven'
	
	uploadArchives {
	    repositories {
	        mavenDeployer {
	            repository(url: "file://localhost/tmp/myRepo/")
	        }
	    }
	}

To find out more about publication, have a look at Chapter 51, Publishing artifacts.

## 9. Groovy 快速入门

要构建Groovy工程，需要Groovy插件。该插件扩展了Java插件。工程中可以包含Groovy源代码、Java源代码。其他方面，Groovy工程与Java工程类似。

### 9.1. 一个简单的Groovy工程

build.gradle

	apply plugin: 'groovy'

> The code for this example can be found at **samples/groovy/quickstart** which is in both the binary and source distributions of Gradle.

上述代码同时会向工程启用Java插件。Groovy插件扩展了`compile`任务，寻找**src/main/groovy**里的文件，扩展了`compileTest`任务寻找**src/test/groovy**里的测试代码。`compile`任务使用联合编译，于是这个目录下也可以有Java代码。

要编译Groovy，需要声明使用的Groovy的版本和Groovy库的位置。You do this by adding a dependency to the `groovy` configuration. The `compile` configuration inherits this dependency, so the groovy libraries will be included in classpath when compiling Groovy and Java source. 例如，我们使用Maven中央仓库的Groovy 2.2.0：

build.gradle

	repositories {
	    mavenCentral()
	}
	
	dependencies {
	    compile 'org.codehaus.groovy:groovy-all:2.3.3'
	}

下面是完整的构建文件：

build.gradle

	apply plugin: 'eclipse'
	apply plugin: 'groovy'
	
	repositories {
	    mavenCentral()
	}
	
	dependencies {
	    compile 'org.codehaus.groovy:groovy-all:2.3.3'
	    testCompile 'junit:junit:4.11'
	}

运行`gradle build`。

## 10. Web 应用快速入门

> 本章还在编写中！

Gradle为Web开发提供了两个插件：War插件和Jetty插件。War插件扩展了Java插件，构建WAR文件夹。Jetty插件扩展了War插件，可以让应用直接部署到一个嵌入的Jetty容器。

### 10.1. 构建一个WAR文件

To build a WAR file, you apply the War plugin to your project:

	apply plugin: 'war'

> The code for this example can be found at **samples/webApplication/quickstart** which is in both the binary and source distributions of Gradle.

This also applies the Java plugin to your project. Running `gradle build` will compile, test and WAR your project. Gradle will look for the source files to include in the WAR file in **src/main/webapp**. Your compiled classes, and their runtime dependencies are also included in the WAR file.

> **Groovy Web 应用**： 组合使用 War 和 Groovy 插件可以构建基于 Groovy  的Web应用。相关的Groovy库会被添加到最终的WAR文件。

### 10.2. 运行Web应用

To run your web application, you apply the Jetty plugin to your project:

	apply plugin: 'jetty'

This also applies the War plugin to your project. Running `gradle jettyRun` will run your web application in an embedded Jetty web container. Running `gradle jettyRunWar` will build the WAR file, and then run it in an embedded web container.

TODO: which url, configure port, uses source files in place and can edit your files and reload.






















