## 21. 创建

所有实际的功能都是插件提供的。插件添加新任务（如`JavaCompile`）、领域对象（如`SourceSet`）、约定（如Java源代码在**src/main/java**），扩展核心对象或其他插件的对象。

### 21.1. 应用插件

插件通过`Project.apply()`方法应用到工程。

	apply plugin: 'java'

插件都有一个短名，如`‘java’`。或者使用另一种写法：

	apply plugin: org.gradle.api.plugins.JavaPlugin

Thanks to Gradle's default imports (see Appendix E, Existing IDE Support and how to cope without it) you could also write:

	apply plugin: JavaPlugin

应用插件是等幂的。即若插件已被应用，再被应用会被忽略。

插件只不过是实现`Plugin`接口的对象。

For more on writing your own plugins, see Chapter 58, Writing Custom Plugins.

### 21.2. 插件做什么


- 向工程添加任务，如`compile`、`test`
- Pre-configure added tasks with useful defaults.
- Add dependency configurations to the project (see Section 8.3, “Dependency configurations”).
- Add new properties and methods to existing type via extensions.

Example 21.4. Tasks added by a plugin

	apply plugin: 'java'
	
	task show << {
	    println relativePath(compileJava.destinationDir)
	    println relativePath(processResources.destinationDir)
	}

`compileJava`和`processResources`是Java创建添加的两个任务。

### 21.3. 约定

Plugins can pre-configure the project in smart ways to support convention-over-configuration. Gradle provides mechanisms and sophisticated support and it's a key ingredient in powerful-yet-concise build scripts.

We saw in the example above that the Java plugins adds a task named `compileJava` that has a property named `destinationDir` (that configures where the compiled Java source should be placed). The Java plugin defaults this property to point to **build/classes/main** in the project directory. This is an example of convention-over-configuration via a reasonable default.

We can change this property simply by giving it a new value.

	apply plugin: 'java'
	
	compileJava.destinationDir = file("$buildDir/output/classes")
	
	task show << {
	    println relativePath(compileJava.destinationDir)
	}

However, the `compileJava` task is likely to not be the only task that needs to know where the class files are.

The Java plugin adds the concept of source sets (see SourceSet) to describe the aspects of a set of source, one aspect being where the class files should be written to when it is compiled. The Java plugin maps the `destinationDir` property of the `compileJava` task to this aspect of the source set.

We can change where the class files are written via the source set.

	apply plugin: 'java'
	
	sourceSets.main.output.classesDir = file("$buildDir/output/classes")
	
	task show << {
	    println relativePath(compileJava.destinationDir)
	}


In the example above, we applied the Java plugin which, among other things, did the following:

- Added a new domain object type: SourceSet
- Configured a main source set with default (i.e. conventional) values for properties
- Configured supporting tasks to use these properties to perform work

All of this happened during the apply plugin: "java" step. In the example above, we changed the desired location of the class files after this conventional configuration had been performed. Notice by the output with the example that the value for compileJava.destinationDir also changed to reflect the configuration change.

Consider the case where another task is to consume the classes files. If this task is configured to use the value from sourceSets.main.output.classesDir, then changing it in this location will update both the compileJava task and this other consumer task whenever it is changed.

This ability to configure properties of objects to reflect the value of another object's task at all times (i.e. even when it changes) is known as “convention mapping”. It allows Gradle to provide conciseness through convention-over-configuration and sensible defaults yet not require complete reconfiguration if a conventional default needs to be changed. Without this, in the example above, we would have had to reconfigure every object that needs to work with the class files.



