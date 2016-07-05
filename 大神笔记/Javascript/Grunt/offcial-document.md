[toc]

## 官方文档

http://gruntjs.com/getting-started

## 入门

Grunt 和 Grunt 的插件通过 npm 安装。

### 安装CLI

全局安装Grunt的command line interface (CLI)。

	npm install -g grunt-cli

Note that installing **grunt-cli** does not install the Grunt task runner! The job of the Grunt CLI is simple: run the version of Grunt which has been installed next to a **Gruntfile**. This allows multiple versions of Grunt to be installed on the same machine simultaneously.

Each time grunt is run, it looks for a locally installed Grunt using node's require() system. Because of this, you can run grunt from any subfolder in your project.

Installed Grunt tasks can be listed by running `grunt --help`.

### 准备一个新的Grunt工程

向工程添加两个文件：package.json 和 Gruntfile。Gruntfile可能是Gruntfile.js或Gruntfile.coffee，用于配置和定义任务，加载Grunt插件。

package.json的例子：

    {
      "name": "my-project-name",
      "version": "0.1.0",
      "devDependencies": {
        "grunt": "~0.4.5",
        "grunt-contrib-jshint": "~0.10.0",
        "grunt-contrib-nodeunit": "~0.4.1",
        "grunt-contrib-uglify": "~0.5.0"
      }
    }

向已存在的package.json添加Grunt和它的插件，最简单的方法是`npm install <module> --save-dev`。如：

	npm install grunt --save-dev

### Gruntfile

Gruntfile.js 或 Gruntfile.coffee 是一个有效的Javascript或CoffeeScript文件，位于package.json旁边。

一个Gruntfile文件由以下部分组成：

- "wrapper"函数
- 工程和任务配置
- 加载Grunt插件和任务
- 定制任务

#### 样例Gruntfile文件

The **grunt-contrib-uglify** plugin's uglify task is configured to minify a source file and generate a banner comment dynamically using that metadata. When grunt is run on the command line, the uglify task will be run by default.

    module.exports = function(grunt) {

      // Project configuration.
      grunt.initConfig({
        pkg: grunt.file.readJSON('package.json'),
        uglify: {
          options: {
            banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
          },
          build: {
            src: 'src/<%= pkg.name %>.js',
            dest: 'build/<%= pkg.name %>.min.js'
          }
        }
      });

      // Load the plugin that provides the "uglify" task.
      grunt.loadNpmTasks('grunt-contrib-uglify');

      // Default task(s).
      grunt.registerTask('default', ['uglify']);

    };

####  "wrapper"函数

所有的Grunt代码需要包裹在一个函数中：

    module.exports = function(grunt) {
      // Do grunt-related things in here
    };

#### 工程和任务配置

多数Grunt任务需要配置，这些配置就是传给`grunt.initConfig`方法的对象。

可以在配置对象中存储任何数据（只要与任务需要的数据不冲突，否则会被忽略）。由于这是一个有效的Javascript文件，可以使用任何Javascript功能，函数，计算等。

与多数任务一样，**grunt-contrib-uglify**插件的`uglify`期望配置对象中有一个同名属性。

#### 加载Grunt插件和任务

常见的任务以插件的形式提供。记得插件使用前要通过`npm install`安装。在Gruntfile企业一个插件：

    // Load the plugin that provides the "uglify" task.
    grunt.loadNpmTasks('grunt-contrib-uglify');

#### 定制任务

通过定义`default`任务，可以配置Grunt默认运行一个或多个任务。在命令行上运行`grunt`不指定任何任务时运行默认任务。

如果你需要的任务没有Grunt插件提供，可以在Gruntfile中就地定义。例如：

    module.exports = function(grunt) {

      // A very basic default task.
      grunt.registerTask('default', 'Log some stuff.', function() {
        grunt.log.write('Logging some stuff...').ok();
      });

    };

Custom project-specific tasks don't need to be defined in the Gruntfile; they may be defined in external .js files and loaded via the `grunt.loadTasks` method.

## 配置任务

### 任务配置与目标

任务运行时，Grunt会寻找同名的配置。每个任务可以有多个目标。例如，`concat`任务有目标`foo`和`bar`。

    grunt.initConfig({
      concat: {
        foo: {
          // concat task "foo" target options and files go here.
        },
        bar: {
          // concat task "bar" target options and files go here.
        },
      },
      uglify: {
        bar: {
          // uglify task "bar" target options and files go here.
        },
      },
    });

运行时可以指定到目标一级，如`grunt concat:foo`，则Grunt只会处理特定目标的配置。而运行`grunt concat`会遍历所有目标，依次处理每一个。

如果任务通过`grunt.task.renameTask`被改名，Grunt会利用新任务名找配置。

### 选项

在任务配置中，一个`options`属性可以用于覆盖内建的默认值。每个目标可以有`options`属性专属于这个目标。目标级别的`options`覆盖任务级别的`options`。

`options`对象是可选的，不需要可以省略。

    grunt.initConfig({
      concat: {
        options: {
          // Task-level options may go here, overriding task defaults.
        },
        foo: {
          options: {
            // "foo" target options may go here, overriding task-level options.
          },
        },
        bar: {
          // No options specified; this target will use task-level options.
        },
      },
    });

### Files

多数任务都要操纵文件，为此Grunt对“声明任务可以操纵的文件”做了抽象。There are several ways to define **src-dest (source-destination) file mappings**, offering varying degrees of verbosity and control. Any multi task will understand all the following formats, so choose whichever format best meets your needs.

All files formats support `src` and `dest` but the "Compact" and "Files Array" formats support a few additional properties:

- `filter`：Either a valid [fs.Stats method name](http://nodejs.org/docs/latest/api/fs.html#fs_class_fs_stats) or a function that is passed the matched `src` filepath and returns `true` or `false`.
- `nonull`：If set to true then the operation will include non-matching patterns. Combined with grunt's `--verbose` flag, this option can help debug file path issues.
- `dot`：Allow patterns to match filenames starting with a period, even if the pattern does not explicitly have a period in that spot.
- `matchBase`：If set, patterns without slashes will be matched against the basename of the path if it contains slashes. For example, `a?b` would match the path `/xyz/123/acb`, but not `/xyz/acb/123`.
- `expand`：Process a dynamic src-dest file mapping, see "Building the files object dynamically" for more information.

Other properties will be passed into the underlying libs as matching options. See the [node-glob](https://github.com/isaacs/node-glob) and [minimatch](https://github.com/isaacs/minimatch) documentation for more options.

#### 紧凑格式

这种格式允许每个目标只有一个src-dest文件映射。多用于只读任务，如**grunt-contrib-jshint**，即只需要一个`src`属性，有没有`dest`不重要。This format also supports additional properties per src-dest file mapping.

    grunt.initConfig({
      jshint: {
        foo: {
          src: ['src/aa.js', 'src/aaa.js']
        },
      },
      concat: {
        bar: {
          src: ['src/bb.js', 'src/bbb.js'],
          dest: 'dest/b.js',
        },
      },
    });

#### 文件对象格式

这种格式支持每个目标读个src-dest映射。属性名是目标文件，属性值是源文件（一个或多个）。可以指定多个映射。但映射不能有额外的属性。

    grunt.initConfig({
      concat: {
        foo: {
          files: {
            'dest/a.js': ['src/aa.js', 'src/aaa.js'],
            'dest/a1.js': ['src/aa1.js', 'src/aaa1.js'],
          },
        },
        bar: {
          files: {
            'dest/b.js': ['src/bb.js', 'src/bbb.js'],
            'dest/b1.js': ['src/bb1.js', 'src/bbb1.js'],
          },
        },
      },
    });

#### 文件数组模式

这种格式支持每个目标多个src-dest文件映射，也支持为映射添加额外属性。

    grunt.initConfig({
      concat: {
        foo: {
          files: [
            {src: ['src/aa.js', 'src/aaa.js'], dest: 'dest/a.js'},
            {src: ['src/aa1.js', 'src/aaa1.js'], dest: 'dest/a1.js'},
          ],
        },
        bar: {
          files: [
            {src: ['src/bb.js', 'src/bbb.js'], dest: 'dest/b/', nonull: true},
            {src: ['src/bb1.js', 'src/bbb1.js'], dest: 'dest/b1/', filter: 'isFile'},
          ],
        },
      },
    });

#### 之前的格式

The dest-as-target file format is a holdover from before multi tasks and targets existed, where the destination filepath is actually the target name. Unfortunately, because target names are filepaths, running `grunt task:target` can be awkward. Also, you can't specify target-level options or additional properties per src-dest file mapping.

Consider this format deprecated, and avoid it where possible.

    grunt.initConfig({
      concat: {
        'dest/a.js': ['src/aa.js', 'src/aaa.js'],
        'dest/b.js': ['src/bb.js', 'src/bbb.js'],
      },
    });

#### 定制过滤函数

`filter`属性帮助过滤目标文件。Simply use a valid [fs.Stats](http://nodejs.org/docs/latest/api/fs.html#fs_class_fs_stats) method name. The following will clean only if the pattern matches an actual file:

    grunt.initConfig({
      clean: {
        foo: {
          src: ['tmp/**/*'],
          filter: 'isFile',
        },
      },
    });

Or create your own filter function and return true or false whether the file should be matched. For example the following will only clean folders that are empty:

    grunt.initConfig({
      clean: {
        foo: {
          src: ['tmp/**/*'],
          filter: function(filepath) {
            return (grunt.file.isDir(filepath) && require('fs').readdirSync(filepath).length === 0);
          },
        },
      },
    });

#### 通配符模式

Grunt的通配符支持利用库node-glob和minimatch。

- `*` 匹配任意数量的字符，但不包括`/`。
- `?` 匹配单个字符，但不包括`/`。
- `**` 匹配任意数量字符，包括`/`，as long as it's the only thing in a path part
- `{}` allows for a comma-separated list of "or" expressions
- `!` at the beginning of a pattern will negate the match

Grunt还允许将路径或模式放入数组。模式会按（列出）顺序处理，匹配到的文件加入结果集。`!`前缀的文件从结果集排除。**结果集中文件只会出现一次！**

For example:

    // 指定单个文件
    {src: 'foo/this.js', dest: ...}
    // 文件数组
    {src: ['foo/this.js', 'foo/that.js', 'foo/the-other.js'], dest: ...}
    // 或使用通配符模式
    {src: 'foo/th*.js', dest: ...}

    // 单个通配符模式
    {src: 'foo/{a,b}*.js', dest: ...}
    // Could also be written like this:
    {src: ['foo/a*.js', 'foo/b*.js'], dest: ...}

    // foo/下的所有js文件，按字母顺序！
    {src: ['foo/*.js'], dest: ...}
    // bar.js在最开始，剩下的文件在后面（字母序）
    {src: ['foo/bar.js', 'foo/*.js'], dest: ...}

    // 除bar.js之外的所有文件，按字母序
    {src: ['foo/*.js', '!foo/bar.js'], dest: ...}
    // 所有文件按字母序，bar.js在最后
    {src: ['foo/*.js', '!foo/bar.js', 'foo/bar.js'], dest: ...}

    // Templates may be used in filepaths or glob patterns:
    {src: ['src/<%= basename %>.js'], dest: 'build/<%= basename %>.min.js'}
    // But they may also reference file lists defined elsewhere in the config:
    {src: ['foo/*.js', '<%= jshint.all.src %>'], dest: ...}

For more on glob pattern syntax, see the **node-glob** and **minimatch** documentation.

#### 动态构建文件对象

When you want to process many individual files, a few additional properties may be used to build a files list dynamically. These properties may be specified in both "Compact" and "Files Array" mapping formats.

设置`expand`为`true`以启用下面的选项：

- `cwd`：所有的源文件相对于（但不包括）这个路径
- `src`：模式，相对于`cwd`
- `dest`：目标路径前缀
- `ext`：Replace any existing extension with this value in generated dest paths.
- `extDot`：Used to indicate where the period indicating the extension is located. Can take either 'first' (extension begins after the first period in the file name) or 'last' (extension begins after the last period), and is set by default to 'first' [Added in 0.4.3]
- `flatten`：Remove all path parts from generated dest paths.
- `rename`：This function is called for each matched src file, (after extension renaming and flattening). The dest and matched src path are passed in, and this function must return a new dest value. If the same dest is returned more than once, each src which used it will be added to an array of sources for it.

In the following example, the uglify task will see the same list of src-dest file mappings for both the `static_mappings` and `dynamic_mappings` targets, because Grunt will automatically expand the `dynamic_mappings` files object into 4 individual static src-dest file mappings—assuming 4 files are found—when the task runs.

Any combination of static src-dest and dynamic src-dest file mappings may be specified.

{{
即使使用通配符，也不能动态处理文件添加删除？比如`lib/*`，如果在grunt运行后，向lib下添加文件，不会被grunt捕获到？
}}

    grunt.initConfig({
      uglify: {
        static_mappings: {
          // Because these src-dest file mappings are manually specified, every
          // time a new file is added or removed, the Gruntfile has to be updated.
          files: [
            {src: 'lib/a.js', dest: 'build/a.min.js'},
            {src: 'lib/b.js', dest: 'build/b.min.js'},
            {src: 'lib/subdir/c.js', dest: 'build/subdir/c.min.js'},
            {src: 'lib/subdir/d.js', dest: 'build/subdir/d.min.js'},
          ],
        },
        dynamic_mappings: {
          // Grunt will search for "**/*.js" under "lib/" when the "uglify" task
          // runs and build the appropriate src-dest file mappings then, so you
          // don't need to update the Gruntfile when files are added or removed.
          files: [
            {
              expand: true,     // Enable dynamic expansion.
              cwd: 'lib/',      // Src matches are relative to this path.
              src: ['**/*.js'], // Actual pattern(s) to match.
              dest: 'build/',   // Destination path prefix.
              ext: '.min.js',   // Dest filepaths will have this extension.
              extDot: 'first'   // Extensions in filenames begin after the first dot
            },
          ],
        },
      },
    });

### 模板

Templates specified using `<% %>` delimiters will be automatically expanded when tasks read them from the config. Templates are expanded recursively until no more remain.

The entire config object is the context in which properties are resolved. Additionally, grunt and its methods are available inside templates, eg. `<%= grunt.template.today('yyyy-mm-dd') %>`.

- `<%= prop.subprop %>` Expand to the value of prop.subprop in the config, regardless of type. Templates like this can be used to reference not only string values, but also arrays or other objects.
- `<% %>` Execute arbitrary inline JavaScript code. This is useful with control flow or looping.

Given the sample concat task configuration below, running grunt concat:sample will generate a file named build/abcde.js by concatenating the banner `/* abcde */` with all files matching `foo/*.js` + `bar/*.js` + `baz/*.js`.

    grunt.initConfig({
      concat: {
        sample: {
          options: {
            banner: '/* <%= baz %> */\n',   // '/* abcde */\n'
          },
          src: ['<%= qux %>', 'baz/*.js'],  // [['foo/*.js', 'bar/*.js'], 'baz/*.js']
          dest: 'build/<%= baz %>.js',      // 'build/abcde.js'
        },
      },
      // Arbitrary properties used in task configuration templates.
      foo: 'c',
      bar: 'b<%= foo %>d', // 'bcd'
      baz: 'a<%= bar %>e', // 'abcde'
      qux: ['foo/*.js', 'bar/*.js'],
    });

### 导入外部数据

In the following Gruntfile, project metadata is imported into the Grunt config from a package.json file, and the grunt-contrib-uglify plugin uglify task is configured to minify a source file and generate a banner comment dynamically using that metadata.

Grunt has grunt.file.readJSON and grunt.file.readYAML methods for importing JSON and YAML data.

    grunt.initConfig({
      pkg: grunt.file.readJSON('package.json'),
      uglify: {
        options: {
          banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
        },
        dist: {
          src: 'src/<%= pkg.name %>.js',
          dest: 'dist/<%= pkg.name %>.min.js'
        }
      }
    });

## Sample Gruntfile

下面的例子使用了5个Grunt插件：**grunt-contrib-uglify**、**grunt-contrib-qunit**、**grunt-contrib-concat**、**grunt-contrib-jshint**、**grunt-contrib-watch**。

第一部分是"wrapper"函数：

    module.exports = function(grunt) {
    }

然后是初始化配置对象：

    grunt.initConfig({
    });

从package.json中读取一些属性：

    module.exports = function(grunt) {
      grunt.initConfig({
        pkg: grunt.file.readJSON('package.json')
      });
    };

下面定义任务需要的配置。下面是"concat"任务的配置：

    concat: {
      options: {
        // define a string to put between each file in the concatenated output
        separator: ';'
      },
      dist: {
        // the files to concatenate
        src: ['src/**/*.js'],
        // the location of the resulting JS file
        dest: 'dist/<%= pkg.name %>.js'
      }
    }

下面配置uglify插件，用于压缩Javascript：

    uglify: {
      options: {
        // the banner is inserted at the top of the output
        banner: '/*! <%= pkg.name %> <%= grunt.template.today("dd-mm-yyyy") %> */\n'
      },
      dist: {
        files: {
          'dist/<%= pkg.name %>.min.js': ['<%= concat.dist.dest %>']
        }
      }
    }

The QUnit plugin is really simple to set up. You just need to give it the location of the test runner files, which are the HTML files QUnit runs on.

    qunit: {
      files: ['test/**/*.html']
    },

The JSHint plugin is also very simple to configure:

    jshint: {
      // define the files to lint
      files: ['gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
      // configure JSHint (documented at http://www.jshint.com/docs/)
      options: {
          // more options here if you want to override JSHint defaults
        globals: {
          jQuery: true,
          console: true,
          module: true
        }
      }
    }

JSHint simply takes an array of files and then an object of options. These are all documented on the JSHint site. If you're happy with the JSHint defaults, there's no need to redefine them in the Gruntfile.

Finally we have the watch plugin:

    watch: {
      files: ['<%= jshint.files %>'],
      tasks: ['jshint', 'qunit']
    }

This can be run on the command line with grunt watch. When it detects any of the files specified have changed (here, I just use the same files I told JSHint to check), it will run the tasks you specify, in the order they appear.

Finally, we have to load in the Grunt plugins we need. These should have all been installed through npm.

    grunt.loadNpmTasks('grunt-contrib-uglify');
    grunt.loadNpmTasks('grunt-contrib-jshint');
    grunt.loadNpmTasks('grunt-contrib-qunit');
    grunt.loadNpmTasks('grunt-contrib-watch');
    grunt.loadNpmTasks('grunt-contrib-concat');

And finally set up some tasks. Most important is the default task:

    // this would be run by typing "grunt test" on the command line
    grunt.registerTask('test', ['jshint', 'qunit']);

	// the default task can be run just by typing "grunt" on the command line
	grunt.registerTask('default', ['jshint', 'qunit', 'concat', 'uglify']);

下面是最终的Gruntfile.js：

    module.exports = function(grunt) {

      grunt.initConfig({
        pkg: grunt.file.readJSON('package.json'),
        concat: {
          options: {
            separator: ';'
          },
          dist: {
            src: ['src/**/*.js'],
            dest: 'dist/<%= pkg.name %>.js'
          }
        },
        uglify: {
          options: {
            banner: '/*! <%= pkg.name %> <%= grunt.template.today("dd-mm-yyyy") %> */\n'
          },
          dist: {
            files: {
              'dist/<%= pkg.name %>.min.js': ['<%= concat.dist.dest %>']
            }
          }
        },
        qunit: {
          files: ['test/**/*.html']
        },
        jshint: {
          files: ['Gruntfile.js', 'src/**/*.js', 'test/**/*.js'],
          options: {
            // options here to override JSHint defaults
            globals: {
              jQuery: true,
              console: true,
              module: true,
              document: true
            }
          }
        },
        watch: {
          files: ['<%= jshint.files %>'],
          tasks: ['jshint', 'qunit']
        }
      });

      grunt.loadNpmTasks('grunt-contrib-uglify');
      grunt.loadNpmTasks('grunt-contrib-jshint');
      grunt.loadNpmTasks('grunt-contrib-qunit');
      grunt.loadNpmTasks('grunt-contrib-watch');
      grunt.loadNpmTasks('grunt-contrib-concat');

      grunt.registerTask('test', ['jshint', 'qunit']);

      grunt.registerTask('default', ['jshint', 'qunit', 'concat', 'uglify']);

    };

## 创建任务

If you don't specify a task, but a task named "default" has been defined, that task will run (unsurprisingly) by default.

### 任务别名

If a task list is specified, the new task will be an alias for one or more other tasks. Whenever this "alias task" is run, every specified tasks in `taskList` will be run, in the order specified. The `taskList` argument must be an array of tasks.

	grunt.registerTask(taskName, [description, ] taskList)

This example alias task defines a "default" task whereby the "jshint", "qunit", "concat" and "uglify" tasks are run automatically if Grunt is executed without specifying any tasks:

	grunt.registerTask('default', ['jshint', 'qunit', 'concat', 'uglify']);

Task arguments can be specified as well. In this example, the alias "dist" runs both the "concat" and "uglify" tasks, each with a "dist" argument:

grunt.registerTask('dist', ['concat:dist', 'uglify:dist']);

### Multi Tasks

When a multi task is run, Grunt looks for a property of the same name in the Grunt configuration. Multi-tasks can have multiple configurations, defined using arbitrarily named "targets."

Specifying both a task and target like `grunt concat:foo` or `grunt concat:bar` will process just the specified target's configuration, while running `grunt concat` will iterate over all targets, processing each in turn. Note that if a task has been renamed with `grunt.task.renameTask`, Grunt will look for a property with the new task name in the config object.

Most of the contrib tasks, including the **grunt-contrib-jshint plugin jshint task** and **grunt-contrib-concat plugin concat task** are multi tasks.

	grunt.registerMultiTask(taskName, [description, ] taskFunction)

Given the specified configuration, this example multi task would log `foo: 1,2,3` if Grunt was run via `grunt log:foo`, or it would log `bar: hello world` if Grunt was run via `grunt log:bar`. If Grunt was run as `grunt log` however, it would log `foo: 1,2,3` then `bar: hello` world then `baz: false`.

    grunt.initConfig({
      log: {
        foo: [1, 2, 3],
        bar: 'hello world',
        baz: false
      }
    });

    grunt.registerMultiTask('log', 'Log stuff.', function() {
      grunt.log.writeln(this.target + ': ' + this.data);
    });

### "Basic" Tasks

When a basic task is run, Grunt doesn't look at the configuration or environment—it just runs the specified task function, passing any specified colon-separated arguments in as function arguments.

	grunt.registerTask(taskName, [description, ] taskFunction)

This example task logs `foo, testing 123` if Grunt is run via `grunt foo:testing:123`. If the task is run without arguments as `grunt foo` the task logs `foo`, no args.

    grunt.registerTask('foo', 'A sample task that logs stuff.', function(arg1, arg2) {
      if (arguments.length === 0) {
        grunt.log.writeln(this.name + ", no args");
      } else {
        grunt.log.writeln(this.name + ", " + arg1 + " " + arg2);
      }
    });

### 定制任务

You can go crazy with tasks. If your tasks don't follow the "multi task" structure, use a custom task.

    grunt.registerTask('default', 'My "default" task description.', function() {
      grunt.log.writeln('Currently running the "default" task.');
    });

Inside a task, you can run other tasks.

    grunt.registerTask('foo', 'My "foo" task.', function() {
      // Enqueue "bar" and "baz" tasks, to run after "foo" finishes, in-order.
      grunt.task.run('bar', 'baz');
      // Or:
      grunt.task.run(['bar', 'baz']);
    });

Tasks can be asynchronous.

    grunt.registerTask('asyncfoo', 'My "asyncfoo" task.', function() {
      // Force task into async mode and grab a handle to the "done" function.
      var done = this.async();
      // Run some sync stuff.
      grunt.log.writeln('Processing task...');
      // And some async stuff.
      setTimeout(function() {
        grunt.log.writeln('All done!');
        done();
      }, 1000);
    });

Tasks can access their own name and arguments.

    grunt.registerTask('foo', 'My "foo" task.', function(a, b) {
      grunt.log.writeln(this.name, a, b);
    });

    // Usage:
    // grunt foo foo:bar
    //   logs: "foo", undefined, undefined
    //   logs: "foo", "bar", undefined
    // grunt foo:bar:baz
    //   logs: "foo", "bar", "baz"

Tasks can fail if any errors were logged.

    grunt.registerTask('foo', 'My "foo" task.', function() {
      if (failureOfSomeKind) {
        grunt.log.error('This is an error message.');
      }

      // Fail by returning false if this task had errors
      if (ifErrors) { return false; }

      grunt.log.writeln('This is the success message');
    });

When tasks fail, all subsequent tasks will be aborted unless `--force` was specified.

    grunt.registerTask('foo', 'My "foo" task.', function() {
      // Fail synchronously.
      return false;
    });

    grunt.registerTask('bar', 'My "bar" task.', function() {
      var done = this.async();
      setTimeout(function() {
        // Fail asynchronously.
        done(false);
      }, 1000);
    });

Tasks can be dependent on the successful execution of other tasks. Note that `grunt.task.requires` won't actually RUN the other task(s). It'll just check to see that it has run and not failed.

    grunt.registerTask('foo', 'My "foo" task.', function() {
      return false;
    });

    grunt.registerTask('bar', 'My "bar" task.', function() {
      // Fail task if "foo" task failed or never ran.
      grunt.task.requires('foo');
      // This code executes if the "foo" task ran successfully.
      grunt.log.writeln('Hello, world.');
    });

    // Usage:
    // grunt foo bar
    //   doesn't log, because foo failed.
    // grunt bar
    //   doesn't log, because foo never ran.

Tasks can fail if required configuration properties don't exist.

    grunt.registerTask('foo', 'My "foo" task.', function() {
      // Fail task if "meta.name" config prop is missing
      // Format 1: String
      grunt.config.requires('meta.name');
      // or Format 2: Array
      grunt.config.requires(['meta', 'name']);
      // Log... conditionally.
      grunt.log.writeln('This will only log if meta.name is defined in the config.');
    });

Tasks can access configuration properties.

    grunt.registerTask('foo', 'My "foo" task.', function() {
      // Log the property value. Returns null if the property is undefined.
      grunt.log.writeln('The meta.name property is: ' + grunt.config('meta.name'));
      // Also logs the property value. Returns null if the property is undefined.
      grunt.log.writeln('The meta.name property is: ' + grunt.config(['meta', 'name']));
    });
    Take a look at the contrib tasks for more examples.

### CLI options / environment

Use `process.env` to access the environment variables.

Read more about the available command-line options on the [Using the CLI](http://gruntjs.com/using-the-cli) page.

### Why doesn't my asynchronous task complete?

Chances are this is happening because you have forgotten to call the this.async method to tell Grunt that your task is asynchronous. For simplicity's sake, Grunt uses a synchronous coding style, which can be switched to asynchronous by calling this.async() within the task body.

Note that passing false to the done() function tells Grunt that the task has failed.

For example:

    grunt.registerTask('asyncme', 'My asynchronous task.', function() {
      var done = this.async();
      doSomethingAsync(done);
    });

