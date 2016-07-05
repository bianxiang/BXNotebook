[toc]

https://docs.gradle.org/current/userguide/userguide.html

## 1 介绍

Gradle provides:

- Very powerful support for multi-project builds.
- Very powerful dependency management (based on Apache Ivy).
- Full support for your existing Maven or Ivy repository infrastructure.
- Support for transitive dependency management without the need for remote repositories or pom.xml and ivy.xml files.
- Ant tasks and builds as first class citizens.
- Groovy build scripts.
- A rich domain model for describing your build.

This user guide, like Gradle itself, is under very active development. Some parts of Gradle aren't documented as completely as they need to be. Some of the content presented won't be entirely clear or will assume that you know more about Gradle than you do. We need your help to improve this user guide. You can find out more about contributing to the documentation at the Gradle web site.

Throughout the user guide, you will find some diagrams that represent dependency relationships between Gradle tasks. These use something analogous to the UML dependency notation, which renders an arrow from one task to the task that the first task depends on.

## 2 概述

Here is a list of some of Gradle's features.

**Declarative builds and build-by-convention**

At the heart of Gradle lies a rich extensible Domain Specific Language (DSL) based on Groovy. Gradle pushes declarative builds to the next level by providing declarative language elements that you can assemble as you like. Those elements also provide build-by-convention support for Java, Groovy, OSGi, Web and Scala projects. Even more, this declarative language is extensible. Add your own new language elements or enhance the existing ones, thus providing concise, maintainable and comprehensible builds.

**Language for dependency based programming**

The declarative language lies on top of a general purpose task graph, which you can fully leverage in your builds. It provides utmost flexibility to adapt Gradle to your unique needs.

**Structure your build**

The suppleness and richness of Gradle finally allows you to apply common design principles to your build. For example, it is very easy to compose your build from reusable pieces of build logic. Inline stuff where unnecessary indirections would be inappropriate. Don't be forced to tear apart what belongs together (e.g. in your project hierarchy). Avoid smells like shotgun changes or divergent change that turn your build into a maintenance nightmare. At last you can create a well structured, easily maintained, comprehensible build.

**Deep API**

From being a pleasure to be used embedded to its many hooks over the whole lifecycle of build execution, Gradle allows you to monitor and customize its configuration and execution behavior to its very core.

**Gradle scales**

Gradle scales very well. It significantly increases your productivity, from simple single project builds up to huge enterprise multi-project builds. This is true for structuring the build. With the state-of-art incremental build function, this is also true for tackling the performance pain many large enterprise builds suffer from.

**Multi-project builds**

Gradle's support for multi-project build is outstanding. Project dependencies are first class citizens. We allow you to model the project relationships in a multi-project build as they really are for your problem domain. Gradle follows your layout not vice versa.

Gradle provides partial builds. If you build a single subproject Gradle takes care of building all the subprojects that subproject depends on. You can also choose to rebuild the subprojects that depend on a particular subproject. Together with incremental builds this is a big time saver for larger builds.

**Many ways to manage your dependencies**

Different teams prefer different ways to manage their external dependencies. Gradle provides convenient support for any strategy. From transitive dependency management with remote Maven and Ivy repositories to jars or directories on the local file system.

**Gradle is the first build integration tool**

Ant tasks are first class citizens. Even more interesting, Ant projects are first class citizens as well. Gradle provides a deep import for any Ant project, turning Ant targets into native Gradle tasks at runtime. You can depend on them from Gradle, you can enhance them from Gradle, you can even declare dependencies on Gradle tasks in your build.xml. The same integration is provided for properties, paths, etc ...

Gradle fully supports your existing Maven or Ivy repository infrastructure for publishing and retrieving dependencies. Gradle also provides a converter for turning a Maven pom.xml into a Gradle script. Runtime imports of Maven projects will come soon.

**Ease of migration**

Gradle can adapt to any structure you have. Therefore you can always develop your Gradle build in the same branch where your production build lives and both can evolve in parallel. We usually recommend to write tests that make sure that the produced artifacts are similar. That way migration is as less disruptive and as reliable as possible. This is following the best-practices for refactoring by applying baby steps.

**Groovy**

Gradle's build scripts are written in Groovy, not XML. But unlike other approaches this is not for simply exposing the raw scripting power of a dynamic language. That would just lead to a very difficult to maintain build. The whole design of Gradle is oriented towards being used as a language, not as a rigid framework. And Groovy is our glue that allows you to tell your individual story with the abstractions Gradle (or you) provide. Gradle provides some standard stories but they are not privileged in any form. This is for us a major distinguishing feature compared to other declarative build systems. Our Groovy support is not just sugar coating. The whole Gradle API is fully Groovy-ized. Adding Groovy results in an enjoyable and productive experience.

**The Gradle wrapper**

The Gradle Wrapper allows you to execute Gradle builds on machines where Gradle is not installed. This is useful for example for some continuous integration servers. It is also useful for an open source project to keep the barrier low for building it. The wrapper is also very interesting for the enterprise. It is a zero administration approach for the client machines. It also enforces the usage of a particular Gradle version thus minimizing support issues.

**Free and open source**

Gradle is an open source project, and is licensed under the ASL.

## 3 教程

第4到第10章是教程。

## 4 安装 Gradle

Gradle 需要 JDK 6 及以上。Gradle 自带 Groovy，不需要安装。

将 `GRADLE_HOME/bin` 添加到环境变量。

输入 `gradle -v` 测试。

JVM options for running Gradle can be set via environment variables. You can use either `GRADLE_OPTS` or `JAVA_OPTS`, or both. `JAVA_OPTS` is by convention an environment variable shared by many Java applications. A typical use case would be to set the HTTP proxy in `JAVA_OPTS` and the memory options in `GRADLE_OPTS`. Those variables can also be set at the beginning of the gradle or gradlew script.

Note that it's not currently possible to set JVM options for Gradle on the command line.

## （未）5 Troubleshooting

This chapter is currently a work in progress.

## 6 构建脚本基础

### 6.1 工程和任务

每个 Gradle 构建由一个或多个工程组成。工程的含义取决于构建的内容。例如，工程可以是一个 JAR 或一个 Web 工程。工程不必是可构建的。工程师一件被做的事情，如部署应用。

每个工程由一个或多个任务组成。任务表示一项原子性的工作。

### 6.2 Hello world

`gradle` 寻找当前目录下的 `build.gradle` 文件。`build.gradle` 文件称为一个构建脚本，或构建“配置”脚本。构建脚本定义一个工程和它的任务。`build.gradle` 示例：

```
task hello {
    doLast {
        println 'Hello world!'
    }
}
```

执行：`gradle -q hello`。

上述任务中包含一项行动（action）。行动是一个闭包，包含一些 Groovy 代码。

`-q` 抑制 Gradle 的日志信息，只显示任务的输出。

### 6.3 任务定义的快捷方式

```
task hello << {
    println 'Hello world!'
}
```

### 6.4 构建脚本是代码

例子：

```
task upper << {
    String someString = 'mY_nAmE'
    println "Original: " + someString
    println "Upper case: " + someString.toUpperCase()
}
```

例二：

```
task count << {
    4.times { print "$it " }
}
```

### 6.5 任务依赖

例子：

```
task hello << {
    println 'Hello world!'
}
task intro(dependsOn: hello) << {
    println "I'm Gradle"
}
```

```
task taskX(dependsOn: 'taskY') << {
    println 'taskX'
}
task taskY << {
    println 'taskY'
}
```

`taskX` 到 `taskY` 的依赖，在 `taskY` 定义前声明，因此要加引号。

Please notice that you can't use shortcut notation (see Section 6.8, “Shortcut notations”) when referring to a task that is not yet defined.

### 6.6 动态任务

利用 Groovy 动态创建任务。

```
4.times { counter ->
    task "task$counter" << {
        println "I'm task number $counter"
    }
}
```

执行 `gradle -q task1`

### 6.7 操纵已存在的任务

例如，在运行时向任务添加依赖。

```
4.times { counter ->
    task "task$counter" << {
        println "I'm task number $counter"
    }
}

task0.dependsOn task2, task3
```

或向已存在的任务添加行为：

```
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
```

Output of `gradle -q hello`

    Hello Venus
    Hello Earth
    Hello Mars
    Hello Jupiter

`doFirst` 或 `doLast` 可以被调用多次。`<<` 是 `doLast` 的别名。

### 6.8 Shortcut notations

As you might have noticed in the previous examples, there is a convenient notation for accessing an existing task. 每个任务都会成为构建脚本的一个属性：

```
task hello << {
    println 'Hello world!'
}
hello.doLast {
    println "Greetings from the $hello.name task."
}
```

Output of gradle -q hello

    Hello world!
    Greetings from the hello task.

### 6.9 任务的其他属性

可以向任务添加自定义属性。To add a property named `myProperty`, set `ext.myProperty` to an initial value. From that point on, the property can be read and set like a predefined task property.

```
task myTask {
    ext.myProperty = "myValue"
}

task printTaskProperties << {
    println myTask.myProperty
}
```

Extra properties aren't limited to tasks. You can read more about them in Section 13.4.2, “Extra properties”.

### 6.10 Using Ant Tasks

Ant tasks are first-class citizens in Gradle. Gradle provides excellent integration for Ant tasks by simply relying on Groovy. Groovy is shipped with the fantastic AntBuilder. Using Ant tasks from Gradle is as convenient and more powerful than using Ant tasks from a build.xml file. From the example below, you can learn how to execute Ant tasks and how to access Ant properties:

Example 6.13. Using AntBuilder to execute ant.loadfile target

```
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
```

Output of `gradle -q loadfile`

     *** agile.manifesto.txt ***
    Individuals and interactions over processes and tools
    Working software over comprehensive documentation
    Customer collaboration  over contract negotiation
    Responding to change over following a plan
     *** gradle.manifesto.txt ***
	Make the impossible possible, make the possible easy and make the easy elegant.
	(inspired by Moshe Feldenkrais)

There is lots more you can do with Ant in your build scripts. You can find out more in Chapter 16, Using Ant from Gradle.

### 6.11 使用方法

在构建脚本中使用方法：
```
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
```

Output of `gradle -q loadfile`

    I'm fond of agile.manifesto.txt
    I'm fond of gradle.manifesto.txt

方法甚至可以在子工程中使用。

We have devoted a whole chapter to this. See Chapter 62, Organizing Build Logic.

### 6.12 默认任务

可以定义一个或多个默认任务。

```
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
```

现在运行 `gradle -q` 等价于运行 `gradle clean run`。在一个多工程构建中，每个子工程可以有自己的默认任务。若子工程未指定默认任务，使用父任务的默认任务。

### 6.13 Configure by DAG

Gradle 有一个配置阶段和一个执行阶段（详见第 58 章）。配置阶段后，Gradle 知道了所有要执行的任务。Gradle offers you a hook to make use of this information. A use-case for this would be to check if the `release` task is among the tasks to be executed. Depending on this, you can assign different values to some variables.

In the following example, execution of the distribution and release tasks results in different value of the version variable.

Example 6.16. Different outcomes of build depending on chosen tasks

```
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
```

`whenReady` 在 `release` 任务实际执行前执行。

## 7 Java 快速开始

### 7.1 Java 插件

插件是对 Gradle 的扩展，它们对工程进行配置，添加一些预定义的任务等等。Gradle 自带一些插件，包括 Java 插件。该插件可以编译源码、进行单元测试，打包成 JAR。

Java 插件是基于“约定”的。即，插件定义了一些默认值，如 Java 源文件的默认位置。如果你遵循约定，不需要做太多配置。但如果不能遵循约定，Gradle 允许你定制构建。

### 7.2 简单的Java工程

使用 Java 插件：

```
apply plugin: 'java'
```

> Note: The code for this example can be found at samples/java/quickstart in the ‘-all’ distribution of Gradle.

定义一个 Java 工程只需要上面一步。

`gradle tasks` 可以列出一个工程的任务。

源代码应放在 `src/main/java`。测试代码放在 `src/test/java`。 `src/main/resources` 下的文件会被当做资源放入 JAR 文件。`src/test/resources` 会在测试时放在类路径下。所有的输出文件都放在 `build` 目录，JAR 文件放在 `build/libs` 目录下。

#### 7.2.1 构建工程

运行 `gradle build`，Gradle 会编译、测试代码，创建 JAR。

Output of `gradle build`

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

其他任务：

- **clean**：删除 `build` 目录。
- **assemble**：编译并打包，但不运行单元测试。**其他插件会增加该任务的内容。**例如如果使用 `War` 插件，该任务会构建 WAR 包。
- **check**：编译和测试代码。Other plugins add more checks to this task. For example, if you use the **checkstyle** plugin, this task will also run Checkstyle against your source code.

#### 7.2.2 外部依赖

仓库用于获取工程的依赖，或用于发布工程的产品。例如，使用 Maven 中央仓库：

```
repositories {
    mavenCentral()
}
```

声明一些依赖：

```
dependencies {
    compile group: 'commons-collections', name: 'commons-collections', version: '3.2'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```

#### 7.2.3 定制工程

Java 插件向工程添加了一些属性。改变这些属性的默认值可以定制。如：

```
sourceCompatibility = 1.5
version = '1.0'
jar {
    manifest {
        attributes 'Implementation-Title': 'Gradle Quickstart',
                   'Implementation-Version': version
    }
}
```

`gradle properties` 可以列出一个工程的所有属性。

可以利用前面章节介绍的技术定制 Java 插件添加的任务。可以设置任务的属性，向任务添加行为，改变任务的依赖，甚至替换一个任务等。In our sample, we will configure the `test` task, which is of type Test, to add a system property when the tests are executed:

```
test {
    systemProperties 'property': 'value'
}
```

#### 7.2.4 发布 JAR 文件

例子，发布到本地的仓库：

```
uploadArchives {
    repositories {
       flatDir {
           dirs 'repos'
       }
    }
}
```

To publish the JAR file, run `gradle uploadArchives`.

#### 7.2.5 创建 Eclipse 工程

首先，先添加插件：

```
apply plugin: 'eclipse'
```

然后执行 `gradle eclipse` 命令产生 Eclipse 的工程文件。More information about the eclipse task can be found in Chapter 38, The Eclipse Plugins.

### 7.3 多工程构建

下面是工程的布局。

    multiproject/
      api/
      services/webservice/
      shared/
      services/shared/

这里有4个工程。工程 **api** 产生 JAR 包。工程 **webservice** 是一个 Web 应用。工程 **shared** 的代码供 api 和 webservice 使用。工程 **services/shared** 依赖 **shared** 工程。

#### 7.3.1 定义多工程构建

多工程构建需要一个设置文件。设置文件放在源码树的根目录，指明包含的工程。名字必须是 **settings.gradle**。例子：
```
include "shared", "api", "services:webservice", "services:shared"
```

You can find out more about the settings file in Chapter 59, Multi-project Builds.

#### 7.3.2 通用配置

对于多工程构建，有一些配置是所有工程共用的。在这里的例子中，我们将这些共用配置定义在根工程中，使用一种称为“配置注入”的技术。这里，根工程就像容器，`subprojects` 方法遍历该容器的每个元素（即每个子工程），并注入特定的配置。This way we can easily define the manifest content for all archives, and some common dependencies:

build.gradle

```
subprojects {
    apply plugin: 'java'
    apply plugin: 'eclipse-wtp'

    repositories {
       mavenCentral()
    }

    dependencies {
        testCompile 'junit:junit:4.12'
    }

    version = '1.0'

    jar {
        manifest.attributes provider: 'gradle'
    }
}
```

So, you can compile, test, and JAR all the projects by running `gradle build` from the root project directory.

Also note that these plugins are only applied within the subprojects section, not at the root level, so the root build will not expect to find Java source files in the root project, only in the subprojects.

#### 7.3.3 工程之间的依赖

子工程间可以有依赖。例如 **api** 工程依赖 **shared** 工程。

api/build.gradle

```
dependencies {
    compile project(':shared')
}
```

See Section 59.7.1, “Disabling the build of dependency projects” for how to disable this functionality.

#### 7.3.4 创建分发

We also add a distribution, that gets shipped to the client:

Example 7.14. Multi-project build - distribution file

api/build.gradle

```
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
```

## 8 依赖管理基础

### 8.2 声明你的依赖

```
apply plugin: 'java'

repositories {
    mavenCentral()
}

dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
    testCompile group: 'junit', name: 'junit', version: '4.+'
}
```

### 8.3 依赖配置

In Gradle dependencies are grouped into configurations. A configuration is simply a named set of dependencies. We will refer to them as dependency configurations. You can use them to declare the external dependencies of your project. As we will see later, they are also used to declare the publications of your project.

**Java** 插件定义了几个标准配置。These configurations represent the classpaths that the Java plugin uses. Some are listed below, and you can find more details in Table 23.5, “Java plugin - dependency configurations”.

- **compile**：编译产品所需要的依赖。
- **runtime**：在运行时所需要的依赖。默认包含编译时的依赖。
- **testCompile**：The dependencies required to compile the test source of the project. By default, also includes the compiled production classes and the compile time dependencies.
- **testRuntime**：The dependencies required to run the tests. By default, also includes the compile, runtime and test compile dependencies.

Various plugins add further standard configurations. You can also define your own custom configurations to use in your build. Please see Section 52.3, “Dependency configurations” for the details of defining and customizing dependency configurations.

### 8.4 外部依赖

你可以声明各种类型的依赖。其中一种是外部依赖。外部依赖指在外部构建，存放在仓库（如 Maven，或本地文件系统）。

To define an external dependency, you add it to a dependency configuration:

```
dependencies {
    compile group: 'org.hibernate', name: 'hibernate-core', version: '3.6.7.Final'
}
```

外部依赖靠 `group`、 `name` 和 `version` 识别。Depending on which kind of repository you are using, `group` and `version` may be optional.

声明外部依赖的简写方式是： `“group:name:version”`。

```
dependencies {
    compile 'org.hibernate:hibernate-core:3.6.7.Final'
}
```

To find out more about defining and working with dependencies, have a look at Section 52.4, “How to declare your dependencies”.

### 8.5 仓库

Gradle 在仓库中寻找外部依赖的文件。Gradle understands several different repository formats, such as Maven and Ivy, 支持几种访问仓库的方式，如本地文件系统或 HTTP。

Gradle 默认未定义任何仓库。需要我们自己定义。如使用 Maven 中央仓库。

```
repositories {
    mavenCentral()
}
```

或其他 Maven 仓库：

```
repositories {
    maven {
        url "http://repo.mycompany.com/maven2"
    }
}
```

Or a remote Ivy repository:

```
repositories {
    ivy {
        url "http://repo.mycompany.com/repo"
    }
}
```

仓库可以在本地。This works for both Maven and Ivy repositories.

```
repositories {
    ivy {
        // URL can refer to a local directory
        url "../local-repo"
    }
}
```

一个工程可以有多个仓库。Gradle 会按指定的顺序在各个仓库中寻找依赖，找到后即停止。

To find out more about defining and working with repositories, have a look at Section 52.6, “Repositories”.

### 8.6 发布成品

Dependency configurations are also used to publish files. We call these files publication artifacts, or usually just artifacts.

The plugins do a pretty good job of defining the artifacts of a project, so you usually don't need to do anything special to tell Gradle what needs to be published. However, you do need to tell Gradle where to publish the artifacts. You do this by attaching repositories to the `uploadArchives` task. Here's an example of publishing to a remote Ivy repository:

```
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
```

Now, when you run `gradle uploadArchives`, Gradle will build and upload your Jar. Gradle will also generate and upload an ivy.xml as well.

You can also publish to Maven repositories. The syntax is slightly different. Note that you also need to apply the **Maven** plugin in order to publish to a Maven repository. when this is in place, Gradle will generate and upload a pom.xml.

```
apply plugin: 'maven'

uploadArchives {
    repositories {
        mavenDeployer {
            repository(url: "file://localhost/tmp/myRepo/")
        }
    }
}
```

To find out more about publication, have a look at Chapter 53, Publishing artifacts.

## （未）9 Groovy Quickstart

要构建一个 Groovy 工程，需要一个 **Groovy** 插件。This plugin extends the Java plugin to add Groovy compilation capabilities to your project. Your project can contain Groovy source code, Java source code, or a mix of the two. In every other respect, a Groovy project is identical to a Java project, which we have already seen in Chapter 7, Java Quickstart.

## （未）10 Web Application Quickstart

