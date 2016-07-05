[toc]

# 有趣特性

## 空安全

```kotlin
// 这里不能通过编译. Artist 不能是null
var notNullArtist: Artist = null

// Artist 可以是 null
var artist: Artist? = null

// 无法编译, artist可能是null，我们需要进行处理
artist.print()

// 只要在artist != null时才会打印
artist?.print()

// 智能转换. 如果我们在之前进行了空检查，则不需要使用安全调用操作符调用
if (artist != null) {
  artist.print()
}

// 只有在确保artist不是null的情况下才能这么调用，否则它会抛出异常
artist!!.print()

// 使用Elvis操作符来给定一个在是null的情况下的替代值
val name = artist?.name ?: "empty"
```

## 扩展方法

我们可以给任何类添加函数。它比那些我们项目中典型的工具类更加具有可读性。举个例子，我们可以给fragment增加一个显示toast的函数：

```kotlin
fun Fragment.toast(message: CharSequence, duration: Int = Toast.LENGTH_SHORT) { 
    Toast.makeText(getActivity(), message, duration).show()
}
```
我们现在可以这么做：

```kotlin
fragment.toast("Hello world!")
```

## 函数式支持（Lambdas）

每次我们去声明一个点击所触发的事件，可以只需要定义我们需要做些什么，而不是需要去创建一个新的Listener呢？我们确实可以这么做，这个（或者其它更多我们感兴趣的事件）我们需要感谢lambda的使用：

```kotlin
view.setOnClickListener { toast("Hello world!") }
```

# 示例

示例应用是一个天气应用。

在 Android Studio 中创建一个项目。

## 配置Gradle

首先，你需要如下修改父 `build.gradle`：

```groovy
buildscript {
    ext.support_version = '23.0.1'
    ext.kotlin_version = '0.13.1514'
    ext.anko_version = '0.7'
    repositories {
        jcenter()
        dependencies {
            classpath 'com.android.tools.build:gradle:1.2.3'
            classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"
        }
    }
}
allprojects {
    repositories {
        jcenter() 
    }
}
```

我们会增加 `Kotlin` 库，`Anko` 库和 `Kotlin Android Extensions plugin` 到 dependencies。

```groovy
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
android {
    ...
}

dependencies {
    compile "com.android.support:appcompat-v7:$support_version"
    compile "org.jetbrains.kotlin:kotlin-stdlib:$kotlin_version"
    compile "org.jetbrains.anko:anko-sdk15:$anko_version"
    compile "org.jetbrains.anko:anko-support-v4:$anko_version"
}

buildscript {
    repositories {
jcenter() }
    dependencies {
      classpath "org.jetbrains.kotlin:kotlin-android-extensions:$kotlin_version"
    } 
}
```

Anko库需要几个依赖。第一个是指所支持的最小的SDK。不能比你在 `build.gradle` 中定义的最小SDK更高，这点很重要。第二增加了额外的 `support-v4` 库的功能，这是导入 `appcompat-v7` 库时隐含增加的。

## 把 MainActivity 转换成 Kotlin 代码

Kotlin plugin 包含了一个有趣的特性，它能把Java代码转成Kotlin代码。正如任何自动化那样，结果不会被很完美地优化，但是在你第一天能够使用Kotlin语言开始编写代码之前，它还是提供了很多的帮助。

所以我们在 `MainActivity.java` 类中使用它。打开文件，然后选择 `Code` -> `Convert Java File to Kotlin File`。

## 测试是否一切就绪

我们想再将编写一些代码来测试Kotlin Android Extensions是否在工作。我现在还不会对这些代码做解释，但是我想要确保它们在你的环境中是正常运行的。这可能是配置中最难的一部分。

首先，打开`activity_main.xml`，然后设置TextView的id：

```xml
<TextView
    android:id="@+id/message"
    android:text="@string/hello_world"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"/>
```

然后，手动在Activity中增加一个import语句（不要担心你现在对这个还不太理解）。

```kotlin
import kotlinx.android.synthetic.activity_main.*
```

在`onCreate`中，你现在可以直接得到并访问这个 `TextView` 了。

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    message.text = "Hello Kotlin!"
}
```

# 编写你的第一个类

## 创建一个layout

控制显示天气预报列表的我们使用 `RecyclerView`，所以你需要在 `build.gradle` 中增加一个新的依赖：

```groovy
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile "com.android.support:appcompat-v7:$support_version" 
    compile "com.android.support:recyclerview-v7:$support_version" ...
}
```

然后，`activity_main.xml` 如下：

```xml
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
             android:layout_width="match_parent"
             android:layout_height="match_parent">
    <android.support.v7.widget.RecyclerView
        android:id="@+id/forecast_list"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</FrameLayout>
```

在 `Mainactivity.kt` 中删除掉之前用来测试的能正常运行的所有代码（现在应该会提示错误）。暂且我们使用老的 `findViewByid()` 的方式：

```kotlin
val forecastList = findViewById(R.id.forecast_list) as RecyclerView
forecastList.layoutManager = LinearLayoutManager(this)
```

## The Recycler Adapter

我们同样需要一个 `RecyclerView` 的Adapter。之前我在我博客中讨论过 `RecyclerView`（http://antonioleiva.com/recyclerview/），如果你以前没有使用过，它可能会有所帮助。

`RecyclerView` 中所使用到的布局现在只需要一个 `TextView`，我会手动去创建这个简单的文本列表。增加一个名为 `ForecastListAdapter.kt` 的Kotlin文件，包括如下代码：

```kotlin
public class ForecastListAdapter(val items: List<String>) :
        RecyclerView.Adapter<ForecastListAdapter.ViewHolder>() {
        
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ForecastListAdapter.ViewHolder? {
        return ViewHolder(TextView(parent.context))
        
    override fun onBindViewHolder(holder: ForecastListAdapter.ViewHolder, position: Int) {
        holder.textView.text = items.get(position)
    }
    
    override fun getItemCount(): Int = items.size()
    
    class ViewHolder(val textView: TextView) : RecyclerView.ViewHolder(textView)
}

public class ForecastListAdapter(val items: List<String>) :
        RecyclerView.Adapter<ForecastListAdapter.ViewHolder>() {
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ForecastListAdapter.ViewHolder? {
        return ViewHolder(TextView(parent.context))
    }

    override fun onBindViewHolder(holder: ForecastListAdapter.ViewHolder, position: Int) {
        holder.textView.text = items[position]
    }

    override fun getItemCount(): Int = items.size

    class ViewHolder(val textView: TextView) : RecyclerView.ViewHolder(textView)
}
```

回到 `MainActivity`，现在简单地创建一系列的String放入List中，然后使用创建分配Adapter实利。

```kotlin
private val items = listOf(
    "Mon 6/23 - Sunny - 31/17",
    "Tue 6/24 - Foggy - 21/8",
    "Wed 6/25 - Cloudy - 22/17",
    "Thurs 6/26 - Rainy - 18/11",
    "Fri 6/27 - Foggy - 21/10",
    "Sat 6/28 - TRAPPED IN WEATHERSTATION - 23/18",
    "Sun 6/29 - Sunny - 20/7"
    )
    
override fun onCreate(savedInstanceState: Bundle?) {
    ...
    val forecastList = findViewById(R.id.forecast_list) as RecyclerView
    forecastList.layoutManager = LinearLayoutManager(this) 
    forecastList.adapter = ForecastListAdapter(items)
}
    
```

# Anko是什么？

Anko是JetBrains开发的一个强大的库。它主要的目的是用来替代以前XML的方式来使用代码生成UI布局。这是一个很有趣的特性，我推荐你可以围观下，但是我在这个项目中暂时不使用它。对于我（可能是由于多年的UI绘制经验）来说使用XML更容易一些，但是你会喜欢那种方式的。

然而，这个不是我们能在这个库中找到的唯一一个功能。Anko包含了很多的非常有帮助的函数和属性来避免让你写很多的模版代码。我们将会通过本书见到很多例子，但是你应该快速地认识到这个库帮你解决了什么样的问题。

尽管Anko是非常有帮助的，但是我建议你要理解这个背后到底做了什么。你可以在任何时候使用“ctrl + 点击”或者“cmd + 点击”的方式跳转到Anko的源代码。Anko的实现方式对学习大部分的Kotlin语言都是非常有帮助的。

在之前，让我们来使用Anko来简化一些代码。就像你将看到的，任何时候你使用了Anko库中的某些东西，它们都会以属性名、方法等方式被导入。这是因为Anko使用了*扩展函数*在Android框架中增加了一些新的功能。我们将会在以后看到扩展函数是什么，怎么去编写它。

在 `MainActivity:onCreate`，一个Anko扩展函数可以被用来简化获取一个RecyclerView：

```kotlin
val forecastList: RecyclerView = find(R.id.forecast_list)
```

我们现在还不能使用库中更多的东西，但是Anko能帮助我们简化代码，比如，实例化Intent，Activity之间的跳转，Fragment的创建，数据库的访问，Alert的创建……我们将会在实现这个App的过程中学习到很多有趣的例子。

举个例子，我们可以创建一个toast函数，这个函数不需要传入任何context，它可以被任何Context或者它的子类调用，比如Activity或者Service：

```kotlin
fun Context.toast(message: CharSequence, duration: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(this, message, duration).show()
}
```

这个方法可以在Activity内部直接调用：

```kotlin
toast("Hello world!")
toast("Hello world!", Toast.LENGTH_LONG)
```

当然，Anko已经包括了自己的toast扩展函数，跟上面这个很相似。Anko提供了一些针对`CharSequence`和`resource`的函数，还有两个不同的toast和longToast方法：

```kotlin
toast("Hello world!")
longToast(R.id.hello_world)
```

# 执行一个请求

而且，如你所见，Kotlin提供了一些扩展函数来让请求变得更简单。首先，我们要创建一个新的Request类：

```kotlin
public class Request(val url: String) {
    public fun run() {
        val forecastJsonStr = URL(url).readText()
        Log.d(javaClass.simpleName, forecastJsonStr)
    }
}
```

我们的请求很简单地接收一个url，然后读取结果并在logcat上打印json。实现非常简单，因为我们使用`readText`，这是Kotlin标准库中的扩展函数。这个方法不推荐结果很大的响应，但是在我们这个例子中已经足够好了。

为了可以执行请求，App必须要有Internet权限。所以需要在`AndroidManifest.xml`中添加：：

```xml
<uses-permission android:name="android.permission.INTERNE
```

# 在主线程以外执行请求

Anko提供了非常简单的DSL来处理异步任务，它满足大部分的需求。它提供了一个基本的 `async` 函数用于在其它线程执行代码，也可以选择通过调用 `uiThread` 的方式回到主线程。在子线程中执行请求如下这么简单：

```kotlin
async {
    Request(url).run()
    uiThread { longToast("Request performed") }
}
```

`uiThread ` 如果被一个 `Activity` 调用，那么如果 `activity.isFinishing()` 返回 `true`，则`uiThread` 不会执行，这样就不会在 Activity 销毁的时候遇到奔溃的情况了。

假如你想使用 `Future` 来工作，`async` 返回一个 Java `Future`。而且如果你需要一个返回结果的`Future`，你可以使用 `asyncResult`。

# 复制一个数据类

如果我们使用不可修改的对象，就像我们之前讲过的，假如我们需要修改这个对象状态，必须要创建一个新的一个或者多个属性被修改的实例。这个任务是非常重复且不简洁的。

举个例子，如果我们需要修改 `Forecast` 中的 `temperature`，我们可以这么做：

```kotlin
val f1 = Forecast(Date(), 27.5f, "Shiny day")
val f2 = f1.copy(temperature = 30f)
```

如上，我们拷贝了第一个forecast对象然后只修改了`temperature`的属性而没有修改这个对象的其它状态。

# 转换json到数据类

当我们使用Gson来解析json到我们的类中，这些属性的名字必须要与json中的名字一样，或者可以指定一个`serialised name`（序列化名称）。一个好的实践是，大部分的软件结构中会根据我们app中布局来解耦成不同的模型。所以我喜欢使用声明简化这些类，因为我会在app其它部分使用它之前解析这些类。属性名称与json结果中的名字是完全一样的。
以下是最后的代码：

```kotlin
public class ForecastRequest(val zipCode: String) {
    companion object {
        private val APP_ID = "15646a06818f61f7b8d7823ca833e1ce"
        private val URL = "http://api.openweathermap.org/data/2.5/" +
                "forecast/daily?mode=json&units=metric&cnt=7"
        private val COMPLETE_URL = "$URL&APPID=$APP_ID&q="
	}
	
    public fun execute(): ForecastResult {
        val forecastJsonStr = URL(COMPLETE_URL + zipCode).readText()
        return Gson().fromJson(forecastJsonStr, ForecastResult::class.java)
	}
}

```

记得在 `build.gradle` 中增加你需要的Gson依赖：

```
compile "com.google.code.gson:gson:<version>"
```

# 扩展函数中的操作符

我们不需要去扩展我们自己的类，但是我需要去使用扩展函数扩展我们已经存在的类来让第三方的库能提供更多的操作。几个例子，我们可以去像访问List的方式去访问`ViewGroup`的view：

```kotlin
operator fun ViewGroup.get(position: Int): View = getChildAt(position)
```

现在真的可以非常简单地从一个`ViewGroup`中通过position得到一个view：

```kotlin
val container: ViewGroup = find(R.id.container)
val view = container[2]
```

如果你使用了上面这些代码，`parent.ctx` 不会被编译成功。Anko提供了大量的扩展函数来让Android编程更简单。举个例子，activitys、fragments以及其它包含了 `ctx` 这个属性，通过 `ctx` 这个属性来返回context，但是在View中缺少这个属性。所以我们要创建一个新的名叫 `ViewExtensions.kt` 文件来代替`ui.utils`，然后增加这个扩展属性：

```kotlin
public val View.ctx: Context
    get() = context
```

# 简化setOnClickListener()

我们用Android中非常典型的例子去解释它是怎么工作的：`View.setOnClickListener()`方法。如果我们想用Java的方式去增加点击事件的回调，我首先要编写一个`OnClickListener`接口：

```java
public interface OnClickListener {
    void onClick(View v);
}
```

然后我们要编写一个匿名内部类去实现这个接口：

```java
view.setOnClickListener(new OnClickListener(){
	@Override
	public void onClick(View v) {
		Toast.makeText(v.getContext(), "Click", Toast.LENGTH_SHORT).show();
	}
})
```

我们将把上面的代码转换成Kotlin（使用了Anko的toast函数）：

```kotlin
view.setOnClickListener(object : OnClickListener {
	override fun onClick(v: View) {
		toast("Click")
	}
}
```

很幸运的是，Kotlin允许Java库的一些优化，Interface中包含单个函数可以被替代为一个函数。如果我们这么去定义了，它会正常执行：

```kotlin
fun setOnClickListener(listener: (View) -> Unit)
```

一个lambda表达式通过参数的形式被定义在箭头的左边（被圆括号包围），然后在箭头的右边返回结果值。在这个例子中，我们接收一个View，然后返回一个Unit（没有东西）。所以根据这种思想，我们可以把前面的代码简化成这样：

```kotlin
view.setOnClickListener({ view -> toast("Click")})
```

这是非常棒的简化！当我们定义了一个方法，我们必须使用大括号包围，然后在箭头的左边指定参数，在箭头的右边返回函数执行的结果。如果左边的参数没有使用到，我们甚至可以省略左边的参数：

```kotlin
view.setOnClickListener({ toast("Click") })
```

如果这个函数的最后一个参数是一个函数，我们可以把这个函数移动到圆括号外面：

```kotlin
view.setOnClickListener() { toast("Click") }
```

并且，最后，如果这个函数只有一个参数，我们可以省略这个圆括号：

```kotlin
view.setOnClickListener { toast("Click") }
```

比原始的Java代码简短了5倍多，并且更加容易理解它所做的事情。非常让人影响深刻。Anko给我们提供了简单（基于名字上的）的版本，正如我之前展示给你看的那样它是由扩展函数来实现的：

```kotlin
view.onClick { toast("Click") }
```

# 扩展语言

多亏这些改变，我们可以去创建自己的`builder`和代码块。我们已经在使用一些有趣的函数，比如`with`。如下简单的实现：

```kotlin
inline fun <T> with(t: T, body: T.() -> Unit) { t.body() }
```

这个函数接收一个`T`类型的对象和一个被作为扩展函数的函数。它的实现仅仅是让这个对象去执行这个函数。因为第二个参数是一个函数，所以我们可以把它放在圆括号外面，所以我们可以创建一个代码块，在这这个代码块中我们可以使用`this`和直接访问所有的public的方法和属性：

```kotlin
with(forecast) {
	Picasso.with(itemView.ctx).load(iconUrl).into(iconView)
	dateView.text = date
	descriptionView.text = description
	maxTemperatureView.text = "${high.toString()}"
	minTemperatureView.text = "${low.toString()}"
	itemView.onClick { itemClick(forecast) }
}
```

内联函数与普通的函数有点不同。一个内联函数会在编译的时候被替换掉，而不是真正的方法调用。
另一个例子：我们可以创建代码块只提供`Lollipop`或者更高版本来执行：

```kotlin
inline fun supportsLollipop(code: () -> Unit) {
	if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
		code()
	}
}
```

它只是检查版本，然后如果满足条件则去执行。现在我们可以这么做：

```kotlin
supportsLollipop {
	window.setStatusBarColor(Color.BLACK)
}
```

举个例子，Anko也是基于这个思想来实现`Android Layout`的`DSL`化。你也可以查看`Kotlin reference`中[`使用DSL来编写HTML`]的一个例子。

[`使用DSL来编写HTML`]: http://kotlinlang.org/docs/reference/type-safe-builders.html

# Kotlin Android Extensions

另一个Kotlin团队研发的可以让开发更简单的插件是`Kotlin Android Extensions`。当前仅仅包括了view的绑定。这个插件自动创建了很多的属性来让我们直接访问XML中的view。这种方式不需要你在开始使用之前明确地从布局中去找到这些views。

这些属性的名字就是来自对应view的id，所以我们取id的时候要十分小心，因为它们将会是我们类中非常重要的一部分。这些属性的类型也是来自XML中的，所以我们不需要去进行额外的类型转换。

`Kotlin Android Extensions`的一个优点是它不需要在我们的代码中依赖其它额外的库。它仅仅由插件组层，需要时用于生成工作所需的代码，只需要依赖于Kotlin的标准库。

那它背后是怎么工作的？该插件会代替任何属性调用函数，比如获取到view并具有缓存功能，以免每次属性被调用都会去重新获取这个view。需要注意的是这个缓存装置只会在`Activity`或者`Fragment`中才有效。如果它是在一个扩展函数中增加的，这个缓存就会被跳过，因为它可以被用在`Activity`中但是插件不能被修改，所以不需要再去增加一个缓存功能。

# 怎么去使用Kotlin Android Extensions

如果你还记得，现在项目已经准备好去使用Kotlin Android Extensions。当我们创建这个项目，我们就已经在`build.gradle`中增加了这个依赖：

```groovy
buldscript{
	repositories {
		jcenter()
	}
	dependencies {
		classpath "org.jetbrains.kotlin:kotlin-android-extensions:$kotlin_version"
	}
}
```

唯一一件需要这个插件做的事情是在类中增加一个特定的"手工"`import`来使用这个功能。我们有两个方法来使用它：

## `Activities`或者`Fragments`的`Android Extensions`

这是最典型的使用方式。它们可以作为`activity`或`fragment`的属性是可以被访问的。属性的名字就是XML中对应view的id。

我们需要使用的`import`语句以`kotlin.android.synthetic`开头，然后加上我们要绑定到Activity的布局XML的名字：

```kotlin
import kotlinx.android.synthetic.activity_main.*
```

此后，我们就可以在`setContentView`被调用后访问这些view。新的Android Studio版本中可以通过使用`include`标签在Activity默认布局中增加内嵌的布局。很重要的一点是，针对这些布局，我们也需要增加手工的import：

```kotlin
import kotlinx.android.synthetic.activity_main.*
import kotlinx.android.synthetic.content_main.*
```

## `Views`的`Android Extensions`

前面说的使用还是有局限性的，因为可能有很多代码需要访问XML中的view。比如，一个自定义view或者一个adapter。举个例子，绑定一个xml中的view到另一个view。唯一不同的就是需要`import`：

```kotlin
import kotlinx.android.synthetic.view_item.view.*
```

如果我们需要一个adapter，比如，我们现在要从inflater的View中访问属性：

```kotlin
view.textView.text = "Hello"
```

# Applicaton单例化

按照我们在Java中一样创建一个单例最简单的方式：

```kotlin
class App : Application() {
    companion object {
        private var instance: Application? = null
	    fun instance() = instance!!
    }
    override fun onCreate() {
        super.onCreate()
        instance = this
	}
}
```

为了这个Application实例被使用，要记得在`AndroidManifest.xml`中增加这个`App`：

```xml
<application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:theme="@style/AppTheme"
    android:name=".ui.App">
    ...
</application>
```

Android有一个问题，就是我们不能去控制很多类的构造函数。比如，我们不能初始化一个非null属性，因为它的值需要在构造函数中去定义。所以我们需要一个可null的变量，和一个返回非null值的函数。我们知道我们一直都有一个`App`实例，但是在它调用`onCreate`之前我们不能去操作任何事情，所以我们为了安全性，我们假设`instance()`函数将会总是返回一个非null的`app`实例。

但是这个方案看起来有点不自然。我们需要定义个一个属性（已经有了getter和setter），然后通过一个函数来返回那个属性。我们有其他方法去达到相似的效果么？是的，我们可以通过委托这个属性的值给另外一个类。这个就是我们知道的`委托属性`。

# 委托属性

我们可能需要一个属性具有一些相同的行为，使用 `lazy` 或者 `observable` 可以被很有趣地实现重用。而不是一次又一次地去声明那些相同的代码，Kotlin提供了一个委托属性到一个类的方法。这就是我们知道的 `委托属性`。

当我们使用属性的 `get` 或者 `set` 的时候，属性委托的 `getValue` 和 `setValue` 就会被调用。

属性委托的结构如下：

```kotlin
class Delegate<T> : ReadWriteProperty<Any?, T> {
	fun getValue(thisRef: Any?, property: KProperty<*>): T {
		return ...
	}
	
	fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
		...
	}
}
```

这个T是委托属性的类型。`getValue` 函数接收一个类的引用和一个属性的元数据。`setValue` 函数又接收了一个被设置的值。如果这个属性是不可修改（val），就会只有一个 `getValue` 函数。

下面展示属性委托是怎么设置的：

```kotlin
class Example {
	var p: String by Delegate()
}
```

它使用了 `by` 这个关键字来指定一个委托对象。

# 标准委托

在Kotlin的标准库中有一系列的标准委托。它们包括了大部分有用的委托，但是我们也可以创建我们自己的委托。

## Lazy

它包含一个lambda，当第一次执行 `getValue` 的时候这个lambda会被调用，所以这个属性可以被延迟初始化。之后的调用都只会返回同一个值。这是非常有趣的特性， =当我们在它们第一次真正调用之前不是必须需要它们的时候。我们可以节省内存，在这些属性真正需要前不进行初始化。

```kotlin
class App : Application() {
    val database: SQLiteOpenHelper by lazy {
		MyDatabaseHelper(applicationContext)
	}

	override fun onCreate() {
	    super.onCreate()
	    val db = database.writableDatabase
    }
}
```

在这个例子中，database并没有被真正初始化，直到第一次调用`onCreate`时。在那之后，我们才确保applicationContext存在，并且已经准备好可以被使用了。`lazy`操作符是线程安全的。

如果你不担心多线程问题或者想提高更多的性能，你也可以使用 `lazy(LazyThreadSafeMode.NONE){ ... }`。


## Observable

这个委托会帮我们监测我们希望观察的属性的变化。当被观察属性的`set`方法被调用的时候，它就会自动执行我们指定的lambda表达式。所以一旦该属性被赋了新的值，我们就会接收到被委托的属性、旧值和新值。

```kotlin
class ViewModel(val db: MyDatabase) {
	var myProperty by Delegates.observable("") {
	    d, old, new ->
	    db.saveChanges(this, new)
	}
}
```

这个例子展示了，一些我们需要关心的ViewMode，每次值被修改了，就会保存它们到数据库。


## Vetoable

这是一个特殊的 `observable`，它让你决定是否这个值需要被保存。它可以被用于在真正保存之前进行一些条件判断。

```kotlin
var positiveNumber = Delegates.vetoable(0) {
    d, old, new ->
	new >= 0
}
```

上面这个委托只允许在新的值是正数的时候执行保存。在lambda中，最后一行表示返回值。你不需要使用return关键字（实质上不能被编译）。

## Not Null

有时候我们需要在某些地方初始化这个属性，但是我们不能在构造函数中确定，或者我们不能在构造函数中做任何事情。第二种情况在Android中很常见：在Activity、fragment、service、receivers……无论如何，一个非抽象的属性在构造函数执行完之前需要被赋值。为了给这些属性赋值，我们无法让它一直等待到我们希望给它赋值的时候。我们至少有两种选择方案。

第一种就是使用可null类型并且赋值为null，直到我们有了真正想赋的值。但是我们就需要在每个地方不管是否是null都要去检查。如果我们确定这个属性在任何我们使用的时候都不会是null，这可能会使得我们要编写一些必要的代码了。

第二种选择是使用 `notNull` 委托。它会含有一个可null的变量并会在我们设置这个属性的时候分配一个真实的值。如果这个值在被获取之前已经被分配了，它就会抛出一个异常。

这个在单例App这个例子中很有用：

```kotlin
class App : Application() {
    companion object {
        var instance: App by Delegates.notNull()
	}
	
	override fun onCreate() {
        super.onCreate()
        instance = this
	}
}
```

## 从Map中映射值

另外一种属性委托方式就是，属性的值会从一个map中获取value，属性的名字对应这个map中的key。这个委托可以让我们做一些很强大的事情，因为我们可以很简单地从一个动态地map中创建一个对象实例。如果我们import `kotlin.properties.getValue`，我们可以从构造函数映射到`val`属性来得到一个不可修改的map。如果我们想去修改map和属性，我们也可以import `kotlin.properties.setValue`。类需要一个`MutableMap`作为构造函数的参数。

想象我们从一个Json中加载了一个配置类，然后分配它们的key和value到一个map中。我们可以仅仅通过传入一个map的构造函数来创建一个实例：

```kotlin
import kotlin.properties.getValue

class Configuration(map: Map<String, Any?>) {
    val width: Int by map
    val height: Int by map
    val dp: Int by map
    val deviceName: String by map
}
```

作为一个参考，这里我展示下对于这个类怎么去创建一个必须要的map：

```kotlin
conf = Configuration(mapOf(
    "width" to 1080,
    "height" to 720,
    "dp" to 240,
    "deviceName" to "mydevice"
))
```

# 怎么去创建一个自定义的委托

先来说说我们要实现什么，举个例子，我们创建一个`notNull`的委托，它只能被赋值一次，如果第二次赋值，它就会抛异常。

Kotlin库提供了几个接口，我们自己的委托必须要实现：`ReadOnlyProperty`和`ReadWriteProperty`。具体取决于我们被委托的对象是`val`还是`var`。

我们要做的第一件事就是创建一个类然后继承`ReadWriteProperty`：

```kotlin
private class NotNullSingleValueVar<T>() : ReadWriteProperty<Any?, T> {

		override fun getValue(thisRef: Any?, property: KProperty<*>): T {
	        throw UnsupportedOperationException()
        }
           
		override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
		}
}
```

这个委托可以作用在任何非null的类型。它接收任何类型的引用，然后像getter和setter那样使用T。现在我们需要去实现这些函数。

- Getter函数 如果已经被初始化，则会返回一个值，否则会抛异常。
- Setter函数 如果仍然是null，则赋值，否则会抛异常。

```kotlin
private class NotNullSingleValueVar<T>() : ReadWriteProperty<Any?, T> {
    private var value: T? = null
    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return value ?: throw IllegalStateException("${desc.name} " +
                "not initialized")
	}
	
	override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
		this.value = if (this.value == null) value
		else throw IllegalStateException("${desc.name} already initialized")
	}
}
```

现在你可以创建一个对象，然后添加函数使用你的委托：

```kotlin
object DelegatesExt {
    fun notNullSingleValue<T>():
            ReadWriteProperty<Any?, T> = NotNullSingleValueVar()
}
```

# 重新实现Application单例化

在这个情景下，委托就可以帮助我们了。我们知道我们的单例不会是null，但是我们不能使用构造函数去初始化属性。所以我们可以使用 `notNull` 委托：

```kotlin
class App : Application() {
    companion object {
        var instance: App by Delegates.notNull()
    }

	override fun onCreate() {
        super.onCreate()
        instance = this
	}
}
```

这种情况下有个问题，我们可以在app的任何地方去修改这个值，因为如果我们使用 `Delegates.notNull()`， 属性必须是var的。但是我们可以使用刚刚创建的委托，这样可以多一点保护。我们只能修改这个值一次：

```kotlin
companion object {
	var instance: App by DelegatesExt.notNullSingleValue()
}
```

尽管，在这个例子中，使用单例可能是最简单的方法，但是我想用代码的形式展示给你怎么去创建一个自定义的委托。