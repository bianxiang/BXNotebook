## 13. 编写构建脚本

### 13.1. Gradle构建语言

Gradle的DSL基于Groovy。

A build script can contain any Groovy language element. Gradle assumes that each build script is encoded using UTF-8.

### 13.2. The Project API

In the tutorial in Chapter 7, Java Quickstart we used, for example, the `apply()` method. Where does this method come from? We said earlier that the build script defines a project in Gradle. 对构建中的每个工程，Gradle创建一个`Project`类型的对象，and associates this Project object with the build script. 当构建脚本执行时，它会配置这个Project对象：

- 构建脚本中的调用的方法，若不是在构建脚本中定义的，则代理给`Project`对象。
- 属性也是一样

例如，访问`Project`对象的`name`属性：

build.gradle

	println name
	println project.name

运行:

	> gradle -q check
	projectApi
	projectApi

Only if you define a property or a method which has the same name as a member of the Project object, you need to use the project property.

#### 13.2.1. 标准工程属性

`Project`对象提供了一些标准属性，下面是部分：

|Name       |Type      |Default Value|
|-----------|----------|-------------|
|project    |Project   |The Project instance|
|name 	    |String    |工程目录的名字|
|path 	    |String    |工程的绝对路径|
|description|String    |A description for the project.|
|projectDir |File 	   |包含构建脚本的目录|
|buildDir 	|File 	   |projectDir/build|
|group 	    |Object    |unspecified|
|version 	|Object    |未指定|
|ant 	    |AntBuilder|An AntBuilder instance|


### 13.3. The Script API

When Gradle executes a script, it compiles the script into a class which implements `Script`. This means that all of the properties and methods declared by the Script interface are available in your script.

### 13.4. 声明变量

在构建脚本中可以声明两种变量：局部变量和extra properties。

#### 13.4.1. 局部变量

局部变量通过`def`声明。They are only visible in the scope where they have been declared. Local variables are a feature of the underlying Groovy language.

build.gradle

	def dest = "dest"
	
	task copy(type: Copy) {
	    from "source"
	    into dest
	}

#### 13.4.2. Extra properties

All enhanced objects in Gradle's domain model can hold extra user-defined properties. This includes, but is not limited to, projects, tasks, and source sets. Extra properties can be added, read and set via the owning object's `ext` property. Alternatively, an ext block can be used to add multiple properties at once.

build.gradle

	apply plugin: "java"
	
	ext {
	    springVersion = "3.1.0.RELEASE"
	    emailNotification = "build@master.org"
	}
	
	sourceSets.all { ext.purpose = null }
	
	sourceSets {
	    main {
	        purpose = "production"
	    }
	    test {
	        purpose = "test"
	    }
	    plugin {
	        purpose = "production"
	    }
	}
	
	task printProperties << {
	    println springVersion
	    println emailNotification
	    sourceSets.matching { it.purpose == "production" }.each { println it.name }
	}

上面的例子中，一个ext block向project对象添加了两个属性。Additionally, a property named purpose is added to each source set by setting `ext.purpose` to null (null is a permissible value). Once the properties have been added, they can be read and set like predefined properties.

By requiring special syntax for adding a property, Gradle can fail fast when an attempt is made to set a (predefined or extra) property but the property is misspelled or does not exist. Extra properties can be accessed from anywhere their owning object can be accessed, giving them a wider scope than local variables. Extra properties on a project are visible from its subprojects.

For further details on extra properties and their API, see ExtraPropertiesExtension.

### 13.5. Groovy基础




