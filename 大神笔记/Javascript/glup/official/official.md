https://github.com/gulpjs/gulp/tree/master/docs

[toc]

## 入门

1、全局安装

	$ npm install --global gulp

2、作为工程的开发依赖

	$ npm install --save-dev gulp

3、在工程根目录中创建`gulpfile.js`

    var gulp = require('gulp');
    gulp.task('default', function() {
      // place code for your default task here
    });

4、运行

	$ gulp

To run individual tasks, use `gulp <task> <othertask>`.

The gulp community is growing, with new plugins being added daily. See the [main website](http://gulpjs.com/plugins/) for a complete list.

## API文档

### gulp.src(globs[, options])

Emits files matching provided glob or an array of globs. Returns a Node.js Stream of [Vinyl files](https://github.com/wearefractal/vinyl-fs) that can be [piped](http://nodejs.org/api/stream.html#stream_readable_pipe_destination_options) to plugins.

```js
gulp.src('client/templates/*.jade')
  .pipe(jade())
  .pipe(minify())
  .pipe(gulp.dest('build/minified_templates'));
```

`glob` refers to [node-glob syntax](https://github.com/isaacs/node-glob) or it can be a direct file path.

参数`globs`：通配符（glob，一个字符串）或通配符的数组。
参数`options`：一个对象。Options to pass to [node-glob] through [glob-stream].

gulp adds some additional options in addition to the [options supported by node-glob][node-glob documentation] and [glob-stream]:

`options.buffer`
类型：布尔。默认：`true`。
Setting this to `false` will return `file.contents` as a stream and not buffer files. This is useful when working with large files. **Note:** Plugins might not implement support for streams.

`options.read`
类型：布尔。默认：`true`。
Setting this to `false` will return `file.contents` as null and not read the file at all.

`options.base`
类型：字符串。默认：everything before a glob starts (see [glob2base])
E.g., consider `somefile.js` in `client/js/somedir`:

```js
gulp.src('client/js/**/*.js') // Matches 'client/js/somedir/somefile.js' and resolves `base` to `client/js/`
  .pipe(minify())
  .pipe(gulp.dest('build'));  // Writes 'build/somedir/somefile.js'

gulp.src('client/js/**/*.js', { base: 'client' })
  .pipe(minify())
  .pipe(gulp.dest('build'));  // Writes 'build/js/somedir/somefile.js'
```

### gulp.dest(path[, options])

Can be piped to and it will write files. Re-emits all data passed to it so you can pipe to multiple folders. 不存在的文件夹会被自动创建。

```javascript
gulp.src('./client/templates/*.jade')
  .pipe(jade())
  .pipe(gulp.dest('./build/templates'))
  .pipe(minify())
  .pipe(gulp.dest('./build/minified_templates'));
```

The write path is calculated by appending the file relative path to the given destination directory. 相对路径根据文件*base*计算。参见`gulp.src`。

参数`path`：字符串或一个函数。
输出文件夹。Or a function that returns it, the function will be provided a [vinyl File instance](https://github.com/wearefractal/vinyl).

参数`options`：对象。

`options.cwd`
类型：字符串。默认：`process.cwd()`。`cwd` for the output folder, only has an effect if provided output folder is relative.

`options.mode`
类型：字符串。默认：`0777`。Octal permission string specifying mode for any folders that need to be created for output folder.

### gulp.task(name[, deps], fn)

Define a task using [Orchestrator].

```js
gulp.task('somename', function() {
  // Do stuff
});
```

参数`name`：任务的名字。如果该任务要在命令行中运行，不要加空格。

参数`deps`：数组。该任务运行前要执行且完成的任务列表。

```js
gulp.task('mytask', ['array', 'of', 'task', 'names'], function() {
  // Do stuff
});
```

**Note:** Are your tasks running before the dependencies are complete?  Make sure your dependency tasks are correctly using the async run hints: take in a callback or return a promise or event stream.

参数`fn`：执行任务的函数。Generally this takes the form of `gulp.src().pipe(someplugin())`.

如果`fn`满足以下条件，任务是异步的：

1、接受一个回调：

```js
// run a command in a shell
var exec = require('child_process').exec;
gulp.task('jekyll', function(cb) {
  // build Jekyll
  exec('jekyll build', function(err) {
    if (err) return cb(err); // return error
    cb(); // finished task
  });
});
```

2、返回一个流

```js
gulp.task('somename', function() {
  var stream = gulp.src('client/**/*.js')
    .pipe(minify())
    .pipe(gulp.dest('build'));
  return stream;
});
```

3、返回一个promise

```js
var Q = require('q');

gulp.task('somename', function() {
  var deferred = Q.defer();

  // do async stuff
  setTimeout(function() {
    deferred.resolve();
  }, 1);

  return deferred.promise;
});
```

**Note:** By default, tasks run with maximum concurrency -- e.g. it launches all the tasks at once and waits for nothing. If you want to create a series where tasks run in a particular order, you need to do two things:

- give it a hint to tell it when the task is done,
- and give it a hint that a task depends on completion of another.

For these examples, let's presume you have two tasks, "one" and "two" that you specifically want to run in this order:

1. In task "one" you add a hint to tell it when the task is done. Either take in a callback and call it when you're done or return a promise or stream that the engine should wait to resolve or end respectively.
2. In task "two" you add a hint telling the engine that it depends on completion of the first task.

So this example would look like this:

```js
var gulp = require('gulp');

// takes in a callback so the engine knows when it'll be done
gulp.task('one', function(cb) {
    // do stuff -- async or otherwise
    cb(err); // if err is not null and not undefined, the run will stop, and note that it failed
});

// identifies a dependent task must be complete before this one begins
gulp.task('two', ['one'], function() {
    // task 'one' is done now
});

gulp.task('default', ['one', 'two']);
```

### gulp.watch(glob [, opts], tasks) or gulp.watch(glob [, opts, cb])

Watch files and do something when a file changes. This always returns an EventEmitter that emits `change` events.

### gulp.watch(glob[, opts], tasks)

参数`glob`：字符串或数组。一个或多个通配符，要监控哪些文件。

参数`opts`：对象。Options, that are passed to [`gaze`](https://github.com/shama/gaze).

参数：`tasks`：数组。当文件改变时执行的任务，added with `gulp.task()`

```js
var watcher = gulp.watch('js/**/*.js', ['uglify','reload']);
watcher.on('change', function(event) {
  console.log('File ' + event.path + ' was ' + event.type + ', running tasks...');
});
```

### gulp.watch(glob[, opts, cb])

参数`glob`：字符串或数组。一个或多个通配符，要监控哪些文件。

参数`opts`：对象。Options, that are passed to [`gaze`](https://github.com/shama/gaze).

参数：`cb(event)`：一个函数。每次改变后调用该函数。

```js
gulp.watch('js/**/*.js', function(event) {
  console.log('File ' + event.path + ' was ' + event.type + ', running tasks...');
});
```

回调参数`event`包含：

- `event.type`：字符串。事件类型，either `added`, `changed` or `deleted`.
- `event.path`：字符串。触发事件的文件类型。

[node-glob documentation]: https://github.com/isaacs/node-glob#options
[node-glob]: https://github.com/isaacs/node-glob
[glob-stream]: https://github.com/wearefractal/glob-stream
[gulp-if]: https://github.com/robrich/gulp-if
[Orchestrator]: https://github.com/robrich/orchestrator
[glob2base]: https://github.com/wearefractal/glob2base






