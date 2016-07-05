[toc]

## 11 使用 Gradle 命令行

### 11.1 执行多个任务

一次执行可以执行多个任务，如 `gradle compile test`。任务按列出的顺序执行。会考虑依赖。任务只会执行一次。So `gradle test test` is exactly the same as `gradle test`.

### 11.2 排除任务

You can exclude a task from being executed using the `-x` command-line option and providing the name of the task to exclude. 如 `gradle dist -x test`。

### 11.3 发生错误后不停止构建

默认，任务失败将导致构建中止和失败。In order to discover as many failures as possible in a single build execution, you can use the `--continue` option.

When executed with `--continue`, Gradle will execute every task to be executed where all of the dependencies for that task completed without failure, instead of stopping as soon as the first failure is encountered. Each of the encountered failures will be reported at the end of the build.

If a task fails, any subsequent tasks that were depending on it will not be executed, as it is not safe to do so. For example, tests will not run if there is a compilation failure in the code under test; because the test task will depend on the compilation task (either directly or indirectly).

### 11.4 任务名缩写

指定任务名时，不必写全，只要写足够多的字母。如执行 `dist` 任务，可能可以写成 `gradle d`。

You can also abbreviate each word in a camel case task name. For example, you can execute task `compileTest` by running `gradle compTest` or even `gradle cT`

You can also use these abbreviations with the `-x` command-line option.

### 11.5 选择执行哪个构建

默认在当前目录寻找构建文件。`-b` 选择可以指定构建文件。If you use `-b` option then `settings.gradle` file is not used. Example:

subdir/myproject.gradle

```
task hello << {
    println "using build file '$buildFile.name' in '$buildFile.parentFile.name'."
}
```

Output of `gradle -q -b subdir/myproject.gradle hello`

    using build file 'myproject.gradle' in 'subdir'.

`-p` 选项可以指定工程目录。对于多工程构建，应使用 `-p` 选项，不能用 `-b`。

Output of` gradle -q -p subdir hello`

	using build file 'build.gradle' in 'subdir'.

### 11.6 了解构建的相关信息

Gradle 提供一些内建的任务专门用于显示构建的细节。如构建的结构、依赖关系。

In addition to the built-in tasks shown below, you can also use the **project report plugin** to add tasks to your project which will generate these reports.

#### 11.6.1 列出工程

`gradle projects` 列出给定工程的所有子工程。You can provide a description for a project by setting the `description` property:

```
description = 'The shared API for the application'
```

#### 11.6.2 列出任务

`gradle tasks` 列出工程的主要（main）任务。This report shows the default tasks for the project, if any, and a description for each task.

By default, this report shows only those tasks which have been assigned to a task group. You can do this by setting the `group` property for the task. You can also set the `description` property, to provide a description to be included in the report.

```
dists {
    description = 'Builds the distribution'
    group = 'build'
}
```

You can obtain more information in the task listing using the `--all` option. With this option, the task report lists all tasks in the project, grouped by main task, and the dependencies for each task.

#### 11.6.3 显示任务的用法详情

Running `gradle help --task someTask` gives you detailed information about a specific task or multiple tasks matching the given task name in your multiproject build. This information includes the full task path, the task type, possible command-line options and the description of the given task.

#### 11.6.4 列出工程的依赖

Running `gradle dependencies` gives you a list of the dependencies of the selected project, broken down by configuration. For each configuration, the direct and transitive dependencies of that configuration are shown in a tree. Below is an example of this report:

```
gradle -q dependencies api:dependencies webapp:dependencies
```

Since a dependency report can get large, it can be useful to restrict the report to a particular configuration. This is achieved with the optional `--configuration` parameter:

```
gradle -q api:dependencies --configuration testCompile
```

#### 11.6.5 Getting the insight into a particular dependency

Running `gradle dependencyInsight` gives you an insight into a particular dependency (or dependencies) that match specified input. Below is an example of this report:

Output of `gradle -q webapp:dependencyInsight --dependency groovy --configuration compile`

    org.codehaus.groovy:groovy-all:2.4.4
    \--- project :api
         \--- compile

This task is extremely useful for investigating the dependency resolution, finding out where certain dependencies are coming from and why certain versions are selected. For more information please see the `DependencyInsightReportTask` class in the API documentation.

The built-in `dependencyInsight` task is a part of the 'Help' tasks group. The task needs to configured with the dependency and the configuration. The report looks for the dependencies that match the specified dependency spec in the specified configuration. If Java related plugin is applied, the `dependencyInsight` task is pre-configured with 'compile' configuration because typically it's the compile dependencies we are interested in. You should specify the dependency you are interested in via the command line '--dependency' option. If you don't like the defaults you may select the configuration via '--configuration' option. For more information see the `DependencyInsightReportTask` class in the API documentation.

#### 11.6.6 列出工程属性

Running `gradle properties` gives you a list of the properties of the selected project. This is a snippet from the output:

Output of `gradle -q api:properties`

    ------------------------------------------------------------
    Project :api - The shared API for the application
    ------------------------------------------------------------

    allprojects: [project ':api']
    ant: org.gradle.api.internal.project.DefaultAntBuilder@12345
    antBuilderFactory: org.gradle.api.internal.project.DefaultAntBuilderFactory@12345
    artifacts: org.gradle.api.internal.artifacts.dsl.DefaultArtifactHandler_Decorated@12345
    asDynamicObject: org.gradle.api.internal.ExtensibleDynamicObject@12345
    baseClassLoaderScope: org.gradle.api.internal.initialization.DefaultClassLoaderScope@12345
    buildDir: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build
    buildFile: /home/user/gradle/samples/userguide/tutorial/projectReports/api/build.gradle

#### 11.6.7 Profiling a build

The `--profile` command line option will record some useful timing information while your build is running and write a report to the `build/reports/profile` directory. The report will be named using the time when the build was run.

This report lists summary times and details for both the configuration phase and task execution. The times for configuration and task execution are sorted with the most expensive operations first. The task execution results also indicate if any tasks were skipped (and the reason) or if tasks that were not skipped did no work.

Builds which utilize a buildSrc directory will generate a second profile report for buildSrc in the buildSrc/build directory.

### 11.7 Dry Run

Sometimes you are interested in which tasks are executed in which order for a given set of tasks specified on the command line, but you don't want the tasks to be executed. You can use the `-m` option for this.

