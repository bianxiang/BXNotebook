## 11. 使用Gradle命令行

### 11.1. 执行多个任务

可以一次执行多个任务。只要依次列出，如`gradle compile test`。Gradle会按顺序执行，并考虑任务依赖关系。每个任务只会被执行一次。

例如下面定义四个任务：

	task compile << {
	    println 'compiling source'
	}
	
	task compileTest(dependsOn: compile) << {
	    println 'compiling unit tests'
	}
	
	task test(dependsOn: [compile, compileTest]) << {
	    println 'running unit tests'
	}
	
	task dist(dependsOn: [compile, test]) << {
	    println 'building the distribution'
	}

运行`gradle dist test`，`compile`只会被执行一次。

Because each task is executed once only, executing `gradle test test` is exactly the same as executing `gradle test`.

### 11.2. 排除任务

若想排除任务，在`-x`命令行选项后跟要排除的任务名。还是上面的例子，执行：

	> gradle dist -x test
	:compile
	compiling source
	:dist
	building the distribution
	
	BUILD SUCCESSFUL
	
	Total time: 1 secs
	
尽管`test`是`dist`的依赖，但它未执行。You will also notice that the test task's dependencies, such as compileTest are not executed either. Those dependencies of test that are required by another task, such as compile, are still executed.

### 11.3. 遇到错误继续构建

如果任务失败，默认Gradle会停止执行，构建失败。为了在一次构建中尽量发现更多的错误，可以使用`--continue`选项。

When executed with `--continue`, Gradle will execute every task to be executed where all of the dependencies for that task completed without failure, instead of stopping as soon as the first failure is encountered. Each of the encountered failures will be reported at the end of the build.

若任务失败，则所有后续依赖它的任务都不会执行。


### 11.4. 任务名缩写

在命令行上，任务名不必是全面，只要能唯一识别一个任务。例如要执行`dist`任务，只需要使用`gradle d`。


You can also abbreviate each word in a camel case task name. For example, you can execute task `compileTest` by running `gradle compTest` or even `gradle cT`。

You can also use these abbreviations with the `-x` command-line option.

### 11.5. Selecting which build to execute

When you run the gradle command, it looks for a build file in the current directory. You can use the `-b` option to select another build file. If you use `-b` option then **settings.gradle** file is not used. Example:

subdir/myproject.gradle

	task hello << {
	    println "using build file '$buildFile.name' in '$buildFile.parentFile.name'."
	}

执行：

	> gradle -q -b subdir/myproject.gradle hello
	using build file 'myproject.gradle' in 'subdir'.

Alternatively, you can use the `-p` option to specify the project directory to use. For multi-project builds you should use `-p` option instead of `-b` option.

	> gradle -q -p subdir hello
	using build file 'build.gradle' in 'subdir'.

### 11.6. 获得关于构建的信息

Gradle提供了一些内建的任务用于显示构建的细节。

除了内建的任务，还可以使用**project report plugin**。

#### 11.6.1. 列出工程

运行`gradle projects`，给出当前工程的子工程，按层级显示。例如：

	> gradle -q projects
	------------------------------------------------------------
	Root project
	------------------------------------------------------------
	
	Root project 'projectReports'
	+--- Project ':api' - The shared API for the application
	\--- Project ':webapp' - The Web application implementation
	
	To see a list of the tasks of a project, run gradle <project-path>:tasks
	For example, try running gradle :api:tasks

上面显示的描述信息可以通过工程的`description`属性指定：

build.gradle

	description = 'The shared API for the application'

#### 11.6.2. 列出任务

Running `gradle tasks` gives you a list of the **main** tasks of the selected project. This report shows the default tasks for the project, if any, and a description for each task. Below is an example of this report:

	> gradle -q tasks
	------------------------------------------------------------
	All tasks runnable from root project
	------------------------------------------------------------
	
	Default tasks: dists
	
	Build tasks
	-----------
	clean - Deletes the build directory (build)
	dists - Builds the distribution
	libs - Builds the JAR
	
	Build Setup tasks
	-----------------
	init - Initializes a new Gradle build. [incubating]
	wrapper - Generates Gradle wrapper files. [incubating]
	
	Help tasks
	----------
	dependencies - Displays all dependencies declared in root project 'projectReports'.
	dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
	help - Displays a help message
	projects - Displays the sub-projects of root project 'projectReports'.
	properties - Displays the properties of root project 'projectReports'.
	tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).
	
	To see all tasks and more detail, run with --all.

By default, this report shows only those tasks which have been assigned to a **task group**. You can do this by setting the `group` property for the task. You can also set the `description` property, to provide a description to be included in the report.

build.gradle

	dists {
	    description = 'Builds the distribution'
	    group = 'build'
	}

You can obtain more information in the task listing using the `--all` option. With this option, the task report lists all tasks in the project, grouped by **main** task, and the dependencies for each task. Here is an example:

	> gradle -q tasks --all
	------------------------------------------------------------
	All tasks runnable from root project
	------------------------------------------------------------
	
	Default tasks: dists
	
	Build tasks
	-----------
	clean - Deletes the build directory (build)
	api:clean - Deletes the build directory (build)
	webapp:clean - Deletes the build directory (build)
	dists - Builds the distribution [api:libs, webapp:libs]
	    docs - Builds the documentation
	api:libs - Builds the JAR
	    api:compile - Compiles the source files
	webapp:libs - Builds the JAR [api:libs]
	    webapp:compile - Compiles the source files
	
	Build Setup tasks
	-----------------
	init - Initializes a new Gradle build. [incubating]
	wrapper - Generates Gradle wrapper files. [incubating]
	
	Help tasks
	----------
	dependencies - Displays all dependencies declared in root project 'projectReports'.
	api:dependencies - Displays all dependencies declared in project ':api'.
	webapp:dependencies - Displays all dependencies declared in project ':webapp'.
	dependencyInsight - Displays the insight into a specific dependency in root project 'projectReports'.
	api:dependencyInsight - Displays the insight into a specific dependency in project ':api'.
	webapp:dependencyInsight - Displays the insight into a specific dependency in project ':webapp'.
	help - Displays a help message
	api:help - Displays a help message
	webapp:help - Displays a help message
	projects - Displays the sub-projects of root project 'projectReports'.
	api:projects - Displays the sub-projects of project ':api'.
	webapp:projects - Displays the sub-projects of project ':webapp'.
	properties - Displays the properties of root project 'projectReports'.
	api:properties - Displays the properties of project ':api'.
	webapp:properties - Displays the properties of project ':webapp'.
	tasks - Displays the tasks runnable from root project 'projectReports' (some of the displayed tasks may belong to subprojects).
	api:tasks - Displays the tasks runnable from project ':api'.
	webapp:tasks - Displays the tasks runnable from project ':webapp'.

#### 11.6.3. 显示任务用法细节

Running `gradle help --task someTask` gives you detailed information about a specific task or multiple tasks matching the given task name in your multiproject build. Below is an example of this detailed information:

	> gradle -q help --task libs
	Detailed task information for libs
	
	Paths
	     :api:libs
	     :webapp:libs
	
	Type
	     Task (org.gradle.api.Task)
	
	Description
	     Builds the JAR

#### 11.6.4. 列出工程依赖

Running `gradle dependencies` gives you a list of the dependencies of the selected project, broken down by configuration. For each configuration, the direct and transitive dependencies of that configuration are shown in a tree. Below is an example of this report:

	> gradle -q dependencies api:dependencies webapp:dependencies
	------------------------------------------------------------
	Root project
	------------------------------------------------------------
	
	No configurations
	
	------------------------------------------------------------
	Project :api - The shared API for the application
	------------------------------------------------------------
	
	compile
	\--- org.codehaus.groovy:groovy-all:2.3.3
	
	testCompile
	\--- junit:junit:4.11
	     \--- org.hamcrest:hamcrest-core:1.3
	
	------------------------------------------------------------
	Project :webapp - The Web application implementation
	------------------------------------------------------------
	
	compile
	+--- project :api
	|    \--- org.codehaus.groovy:groovy-all:2.3.3
	\--- commons-io:commons-io:1.2
	
	testCompile
	No dependencies

Since a dependency report can get large, it can be useful to restrict the report to a particular configuration. This is achieved with the optional `--configuration` parameter:

	> gradle -q api:dependencies --configuration testCompile
	------------------------------------------------------------
	Project :api - The shared API for the application
	------------------------------------------------------------
	
	testCompile
	\--- junit:junit:4.11
	     \--- org.hamcrest:hamcrest-core:1.3 ####

#### 11.6.5. Getting the insight into a particular dependency

Running `gradle dependencyInsight` gives you an insight into a particular dependency (or dependencies) that match specified input. Below is an example of this report:

	> gradle -q webapp:dependencyInsight --dependency groovy --configuration compile
	org.codehaus.groovy:groovy-all:2.3.3
	\--- project :api
	     \--- compile

This task is extremely useful for investigating the dependency resolution, finding out where certain dependencies are coming from and why certain versions are selected. For more information please see DependencyInsightReportTask.

The built-in **dependencyInsight** task is a part of the 'Help' tasks group. The task needs to configured with the dependency and the configuration. The report looks for the dependencies that match the specified dependency spec in the specified configuration. If java related plugin is applied, the **dependencyInsight** task is pre-configured with 'compile' configuration because typically it's the compile dependencies we are interested in. You should specify the dependency you are interested in via the command line '`--dependency`' option. If you don't like the defaults you may select the configuration via '`--configuration`' option. For more information see DependencyInsightReportTask.

#### 11.6.6. 列出工程属性

Running `gradle properties` gives you a list of the properties of the selected project. This is a snippet from the output:

	> gradle -q api:properties
	------------------------------------------------------------
	Project :api - The shared API for the application
	------------------------------------------------------------
	
	allprojects: [project ':api']
	ant: org.gradle.api.internal.project.DefaultAntBuilder@12345
	antBuilderFactory: org.gradle.api.internal.project.DefaultAntBuilderFactory@12345
	artifacts: org.gradle.api.internal.artifacts.dsl.DefaultArtifactHandler@12345
	asDynamicObject: org.gradle.api.internal.ExtensibleDynamicObject@12345
	baseClassLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@12345
	buildDir: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build
	buildFile: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build.gradle

#### 11.6.7. Profiling a build

The `--profile` command line option will record some useful timing information while your build is running and write a report to the **build/reports/profile** directory. The report will be named using the time when the build was run.

This report lists summary times and details for both the configuration phase and task execution. The times for configuration and task execution are sorted with the most expensive operations first. The task execution results also indicate if any tasks were skipped (and the reason) or if tasks that were not skipped did no work.

Builds which utilize a buildSrc directory will generate a second profile report for buildSrc in the buildSrc/build directory.

### 11.7. Dry Run

Sometimes you are interested in which tasks are executed in which order for a given set of tasks specified on the command line, but you don't want the tasks to be executed. You can use the `-m` option for this. For example `gradle -m clean compile` shows you all tasks to be executed as part of the clean and compile tasks. This is complementary to the tasks task, which shows you the tasks which are available for execution.
























