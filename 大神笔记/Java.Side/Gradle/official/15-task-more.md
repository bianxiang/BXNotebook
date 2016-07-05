## 15. 深入任务

### 15.1. 定义任务

第6章展示了使用关键字风格定义任务的方式。还有其他几种风格。

	task(hello) << {
	    println "hello"
	}
	
	task(copy, type: Copy) {
	    from(file('srcDir'))
	    into(buildDir)
	}

可以用字符串做任务名：

	task('hello') <<
	{
	    println "hello"
	}
	
	task('copy', type: Copy) {
	    from(file('srcDir'))
	    into(buildDir)
	}

另一种语法：

	tasks.create(name: 'hello') << {
	    println "hello"
	}
	
	tasks.create(name: 'copy', type: Copy) {
	    from(file('srcDir'))
	    into(buildDir)
	}

Here we add tasks to the tasks collection. Have a look at TaskContainer for more variations of the `create()` method.

### 15.2. 定位任务

You often need to locate the tasks that you have defined in the build file, for example, to configure them or use them for dependencies. There are a number of ways of doing this. 首先，每个任务都是工程的一个属性，用任务名做属性名：

	task hello
	
	println hello.name
	println project.hello.name

或者通过`tasks`集合：

	task hello
	
	println tasks.hello.name
	println tasks['hello'].name

使用`tasks.getByPath()`方法，传入一个任务名、一个相对路，或一个绝对路径。

	project(':projectA') {
	    task hello
	}
	
	task hello
	
	println tasks.getByPath('hello').path
	println tasks.getByPath(':hello').path
	println tasks.getByPath('projectA:hello').path
	println tasks.getByPath(':projectA:hello').path

### 15.3. 配置任务

例子，Gradle提供的Copy任务。任务可以通过它自身提供的API配置。

	task myCopy(type: Copy)

配置1：

	Copy myCopy = task(myCopy, type: Copy)
	myCopy.from 'resources'
	myCopy.into 'target'
	myCopy.include('**/*.txt', '**/*.xml', '**/*.properties')

另一种写法：

	task myCopy(type: Copy)
	
	myCopy {
	   from 'resources'
	   into 'target'
	   include('**/*.txt', '**/*.xml', '**/*.properties')
	}

第三行`myCopy`仅是`tasks.getByName()`的缩写。It is important to note that if you pass a closure to the getByName() method, this closure is applied to configure the task, not when the task executes.

或者，定义任务时提供一个配置闭包：

	task copy(type: Copy) {
	   from 'resources'
	   into 'target'
	   include('**/*.txt', '**/*.xml', '**/*.properties')
	}

### 15.4. 向任务添加依赖

There are several ways you can define the dependencies of a task. In Section 6.5, “Task dependencies” you were introduced to defining dependencies using task names. Task names can refer to tasks in the same project as the task, or to tasks in other projects. To refer to a task in another project, you prefix the name of the task with the path of the project it belongs to. Below is an example which adds a dependency from `projectA:taskX` to `projectB:taskY`:

	project('projectA') {
	    task taskX(dependsOn: ':projectB:taskY') << {
	        println 'taskX'
	    }
	}
	
	project('projectB') {
	    task taskY << {
	        println 'taskY'
	    }
	}


Instead of using a task name, you can define a dependency using a `Task` object, as shown in this example:

	task taskX << {
	    println 'taskX'
	}
	
	task taskY << {
	    println 'taskY'
	}
	
	taskX.dependsOn taskY

For more advanced uses, you can define a task dependency using a closure. When evaluated, the closure is passed the task whose dependencies are being calculated. The closure should return a single `Task` or collection of `Task` objects, which are then treated as dependencies of the task. The following example adds a dependency from `taskX` to all the tasks in the project whose name starts with lib:

	task taskX << {
	    println 'taskX'
	}
	
	taskX.dependsOn {
	    tasks.findAll { task -> task.name.startsWith('lib') }
	}
	
	task lib1 << {
	    println 'lib1'
	}
	
	task lib2 << {
	    println 'lib2'
	}
	
	task notALib << {
	    println 'notALib'
	}

For more information about task dependencies, see the Task API. 

### （未）15.5. Ordering tasks ###

### 15.6. 添加任务的描述

执行任务时会显示描述。

	task copy(type: Copy) {
	   description 'Copies the resource directory to the target directory.'
	   from 'resources'
	   into 'target'
	   include('**/*.txt', '**/*.xml', '**/*.properties')
	}

### 15.7. 替换任务

For example if you want to exchange a task added by the Java plugin with a custom task of a different type. You can achieve this with:

	task copy(type: Copy)
	
	task copy(overwrite: true) << {
	    println('I am the new one.')
	}

### （未）15.8. Skipping tasks ###

### （未）15.9. Skipping tasks that are up-to-date ###

### （未）15.10. Task rules ###

### （未) 15.11. Finalizer tasks ###

