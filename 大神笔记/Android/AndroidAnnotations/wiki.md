[toc]

Enhanced components

@EActivity
@EApplication
@EBean
@EFragment
@EProvider
@EReceiver
@EIntentService
@EService
@EView
@EViewGroup
Injection

@AfterExtras
@AfterInject
@AfterViews
@App
@Bean
@Extra
@FragmentArg
@FragmentById
@FragmentByTag
@FromHtml
@HttpsClient
@NonConfigurationInstance
@RootContext
@SystemService
@ViewById
@ViewsById
Event binding

@TextChange
@AfterTextChange
@BeforeTextChange
@EditorAction
@FocusChange
@CheckedChange
@Touch
@Click
@LongClick
@ItemClick
@ItemLongClick
@ItemSelect
@OptionsItem
@SeekBarProgressChange
@SeekBarTouchStart
@SeekBarTouchStop
@KeyDown
@KeyUp
@KeyLongPress
@KeyMultiple
@PageScrollStateChanged
@PageScrolled
@PageSelected
Threading

@Background
@UiThread
@SupposeBackground
@SupposeUiThread
Misc

@InstanceState
@WindowFeature
@Fullscreen
@CustomTitle
@InjectMenu
@OptionsMenu
@OptionsMenuItem
@OrmLiteDao
@RoboGuice
@Trace
@Transactional
@OnActivityResult
@OnActivityResult.Extra
@HierarchyViewerSupport
@ServiceAction
@Receiver
@Receiver.Extra
@ReceiverAction
@ReceiverAction.Extra
@IgnoredWhenDetached
@IgnoreWhen
@WakeLock
Resource injection

@StringRes
@AnimationRes
@ColorRes
@DimensionPixelOffsetRes
@DimensionPixelSizeRes
@DimensionRes
@BooleanRes
@ColorStateListRes
@DrawableRes
@IntArrayRes
@IntegerRes
@LayoutRes
@MovieRes
@StringArrayRes
@TextArrayRes
@TextRes
@HtmlRes
Rest API

@Rest
@RestService
@Get
@Post
@Put
@Patch
@Delete
@Options
@Head
@Accept
@RequiresHeader
@RequiresCookie
@RequiresCookieInUrl
@RequiresAuthentication
@SetsCookie
@RequiresCookieInUrl
@Path
@Body
@Field
@Part
@Header
@Headers
Typesafe SharedPreferences

@DefaultBoolean
@DefaultFloat
@DefaultInt
@DefaultLong
@DefaultString
@DefaultStringSet
@DefaultRes
@Pref
@SharedPref
Preference API helpers

@PreferenceScreen
@PreferenceHeaders
@PreferenceByKey
@PreferenceChange
@PreferenceClick
@AfterPreferences

## 增强组件

### @EActivity

`@EActivity` 表示一个活动要被 AA 增强。它的值是一个布局id，可以为空。

```java
@EActivity(R.layout.main)
public class MyActivity extends Activity {
}
```

Also see how to start an annotated activity.

### @EApplication 和 @App

https://github.com/excilys/androidannotations/wiki/Enhancing-the-Application-class

You can enhance your Android Application class with the `@EApplication` annotation. You can then start using most AA annotations, except the ones related to views and extras:

```java
@EApplication
public class MyApplication extends Application {
  public void onCreate() {
    super.onCreate();
    initSomeStuff();
  }

  @SystemService
  NotificationManager notificationManager;

  @Bean
  MyEnhancedDatastore datastore;

  @RestService
  MyService myService;

  @OrmLiteDao(helper = DatabaseHelper.class, model = User.class)
  UserDao userDao;

  @Background
  void initSomeStuff() {
    // init some stuff in background
  }
}
```

You can inject the application class using the `@App` annotation:

```java
@EActivity
public class MyActivity extends Activity {
  @App
  MyApplication application;
}
```

It also works for any kind of annotated component, such as `@EBean`:

```java
@EBean
public class MyBean {
  @App
  MyApplication application;
}
```

Since AndroidAnnotations 3.0, the application class must be annotated with `@EApplication`.

### 增强非标准 Android 类

You just need to annotate it with `@EBean`:

```java
@EBean
public class MyClass {
}
```

`@EBean` 注解的类只能有一个构造器。该构造器必须没有参数，或由一个 `Context` 参数。

You can use most AA annotations in an `@EBean` class:

```java
@EBean
public class MyClass {
  @SystemService
  NotificationManager notificationManager;
  @UiThread
  void updateUI() {
  }
}
```

You can use view related annotations (`@View`, `@Click`...) in your `@EBean` class:

```java
@EBean
public class MyClass {
  @ViewById
  TextView myTextView;

  @Click(R.id.myButton)
  void handleButtonClick() {
  }
}
```

Notice that this will only work if the root Android component that depends on MyClass is an activity with a layout that contains the needed views. Otherwise, `myTextView` will be null, and `handleButtonClick()` will never be called.

#### 注入根上下文：`@RootContext`

You can inject the root Android component that depends on your `@EBean` class, using the `@RootContext` annotation. Please notice that it only gets injected if the context has the right type.

```java
@EBean
public class MyClass {

  @RootContext
  Context context;

  // Only injected if the root context is an activity
  @RootContext
  Activity activity;

  // Only injected if the root context is a service
  @RootContext
  Service service;

  // Only injected if the root context is an instance of MyActivity
  @RootContext
  MyActivity myActivity;

}
```

In the MyClass instance referenced by the following activity, the `service` field of MyClass (see above) will be null.

```java
@EActivity
public class MyActivity extends Activity {

  @Bean
  MyClass myClass;

}
```

#### 在依赖注入后执行代码：@AfterInject

When the constructor of your `@EBean` annotated class is called, it's fields have not been injected yet. If you need to execute code at build time, after dependency injection, you should use the `@AfterInject` annotation on some methods.

```java
@EBean
public class MyClass {

  @SystemService
  NotificationManager notificationManager;

  @Bean
  MyOtherClass dependency;

  public MyClass() {
    // notificationManager and dependency are null
  }

  @AfterInject
  public void doSomethingAfterInjection() {
    // notificationManager and dependency are set
  }

}
```

If the parent and child classes have `@AfterViews`, `@AfterInject` or `@AfterExtras` annotated methods with the same name, the generated code will be buggy. See issue #591 for more details.

Also, while there is a guaranteed order about when we call @AfterViews, -Inject or -Extras annotated methods, there is no guaranteed order for calling each of the methods with the same `@AfterXXX` annotation (see issue #810).

Details about when the methods with one of those annotations are called you can find [here](https://github.com/excilys/androidannotations/wiki/%40AfterXXX-call-order).

#### 注入定制的类：`@Bean`

To use this enhanced class in another enhanced class or in an enhanced Android component, use `@Bean`:

```java
@EBean
public class MyOtherClass {
  @Bean
  MyClass myClass;
}
```

You always get a new instance when you use @Bean on a field, unless that bean is a Singleton (see Scopes below).

It is worth noting that the generated subclasses are final, which means that you can't extend generated classes. However, you can extends original classes and generated code will be also generated for the child class. This works for every annotations.

For example the following class will also have myOtherClass injected :

```java
@EActivity
public class MyChildActivity extends MyActivity {
}
```
#### Scopes

AndroidAnnotations currently support two scopes:

- the default scope: a new bean instance is created each time a bean must be injected
- the singleton scope: a new instance of the bean is created the first time it is needed. It is then retained and the same instance is always injected.

```java
@EBean(scope = Scope.Singleton)
public class MySingleton {
}
```

Caution

- If your bean has a Singleton scope, it will only keep a reference to the application context (using `context.getApplicationContext()`. That's because it lives longer than any activity / service, so it shouldn't keep a reference to any such Android component, to avoid memory leaks.
- We also disable view injection and view event binding in `@EBean` components with Singleton scope, for the exact same reason (views have a reference to the activity, which may therefore create a leak).

#### 注入实现

If you want to use a dependency in your code through its API (either a superclass or an interface), you can specify the implementation class as the value parameter of the `@Bean` annotation. In latter case you must annotate the implementation class and not the interface.

```java
@EActivity
public class MyActivity extends Activity {
    /* A MyImplementation instance will be injected.
     * MyImplementation must be annotated with @EBean and implement MyInterface.
     */
    @Bean(MyImplementation.class)
    MyInterface myInterface;
}
```

### 增强 Fragments

#### Support for FragmentActivity

Prior to AndroidAnnotations 2.6, there was no support for fragment injection. However, we made sure that at least extending `FragmentActivity` instead of `Activity` didn't break AndroidAnnotations:

```java
@EActivity(R.id.main)
public class DetailsActivity extends FragmentActivity {
}
```

#### Fragment Support

Since AndroidAnnotations 2.6

AndroidAnnotations supports both `android.app.Fragment` and `android.support.v4.app.Fragment`, and automatically uses the right APIs based on the fragment types.

To start using AndroidAnnotations features in a fragment, annotate it with `@EFragment`:

```java
@EFragment
public class MyFragment extends Fragment {
}
```

AndroidAnnotations will generate a fragment subclass with a trailing underscore, e.g. `MyFragment_`. You should use the generated subclass in your xml layouts and when creating new instance fragments:

```
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:orientation="horizontal" >

        <fragment
            android:id="@+id/myFragment"
            android:name="com.company.MyFragment_"
            android:layout_width="fill_parent"
            android:layout_height="fill_parent" />

    </LinearLayout>
```

Programmatically:

```java
MyFragment fragment = new MyFragment_();
```

Or you can use the fluent builder:

```java
MyFragment fragment = MyFragment_.builder().build();
```

You can now use all kind of annotations in your fragment:

```java
@EFragment
public class MyFragment extends Fragment {
    @Bean
    SomeBean someBean;

    @ViewById
    TextView myTextView;

    @App
    MyApplication customApplication;

    @SystemService
    ActivityManager activityManager;

    @OrmLiteDao(helper = DatabaseHelper.class, model = User.class)
    UserDao userDao;

    @Click
    void myButton() {
    }

    @UiThread
    void uiThread() {

    }

    @AfterInject
    void calledAfterInjection() {
    }

    @AfterViews
    void calledAfterViewInjection() {
    }

    @Receiver(actions = "org.androidannotations.ACTION_1")
    protected void onAction1() {

    }
}
```

View injection and event listener binding will only be based on views contained inside the fragment. Note, however, that it's isn't currently the case for `@EBean` injected inside fragments: they have access to the activity views.

#### Fragment Layout

The standard way to associate a view with a fragment is to override `onCreateView()`. You can let AndroidAnnotations handle that for you by setting the value param of the `@EFragment` annotation:

```java
@EFragment(R.layout.my_fragment_layout)
public class MyFragment extends Fragment {
}
```

If you need to override `onCreateView()`, e.g. because you need to access `savedInstanceState`, you can still let AndroidAnnotations handle the layout creation by returning null:

```java
@EFragment(R.layout.my_fragment_layout)
public class MyFragment extends Fragment {
    public View onCreateView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        return null;
    }
}
```

#### 强制布局注入

Since AndroidAnnotations 3.3

We only inject the layout passed as the first annotation parameter if the `onCreateView` method from the annotated class (or its superclass if the method is not overridden) returns null. This is because we do not want to override your custom non-null View returned from `onCreateView`. In some cases still that would be the desired behavior: for example if you extend from the `ListFragment` class, it returns a non-null View from `onCreateView`, and your layout does not get injected which is passed to the @EFragment annotation. To work around this, we added a boolean `forceLayoutInjection` annotation parameter, if it is set to true, the injection happens regardless of the return value of the super `onCreateView` method. It is false by default, so the old behavior takes place if you do not explicitly set `forceLayoutInjection` to true.

```java
@EFragment(R.layout.my_custom_layout)
public class MyFragment extends ListFragment {
  // R.layout.my_custom_layout will be injected
}
```

#### 注入 Fragments

You may inject fragments in classes annotated with @EActivity, @EFragment, @EView, @EViewGroup, @EBean, using `@FragmentById` or `@FragmentByTag`. If you don't specify any value on the annotation, the field name is used.

We recommend using `@FragmentById` rather then `@FragmentByTag`, because no compile time validation is performed for the latter.

Please be aware that `@FragmentById` and `@FragmentByTag` can only inject fragments, not create them, so they must already exist in the activity (either by defining them in the layout or by creating them programmatically in `onCreate()`.

You can inject fragments even if they are not annotated with `@EFragment`.

```java
@EActivity(R.layout.fragments)
public class MyFragmentActivity extends FragmentActivity {
  @FragmentById
  MyFragment myFragment;

  @FragmentById(R.id.myFragment)
  MyFragment myFragment2;

  @FragmentByTag
  MyFragment myFragmentTag;

  @FragmentByTag("myFragmentTag")
  MyFragment myFragmentTag2;
}
```

#### Note on `DialogFragment`

Unfortunetaly you cannot create a new Dialog instance in the onCreateDialog() method if you are using @EFragment. You should just call super.onCreateDialog(), then configure the returned `Dialog` instance. Then you can setup your views in a `@AfterViews` annotated method. Please read [this thread](https://github.com/excilys/androidannotations/issues/486#issuecomment-110672535) for more information

#### @FragmentArg

The `@FragmentArg` annotation indicates that a fragment field should be injected with the corresponding Fragment Argument.

The setter method in the generated builder will always have the same name as the argument. By default, the key used to bind the value is the field name, but you can change it by providing a value to the `@FragmentArg` annotation.

Usage example:

```java
@EFragment
public class MyFragment extends Fragment {
  @FragmentArg("myStringArgument")
  String myMessage;

  @FragmentArg
  String anotherStringArgument;

  @FragmentArg("myDateExtra")
  Date myDateArgumentWithDefaultValue = new Date();
}
```

The fragment builder will hold dedicated methods for these arguments:

```java
MyFragment myFragment = MyFragment_.builder()
  .myMessage("Hello")
  .anotherStringArgument("World")
  .build();
```

### 增强 content providers：@EProvider

You can enhance an Android Content Provider with the `@EProvider` annotation:

```java
@EProvider
public class MyContentProvider extends ContentProvider {
}
```

You can then start using most AA annotations, except the ones related to views and extras:

```java
@EProvider
public class MyContentProvider extends ContentProvider {

  @SystemService
  NotificationManager notificationManager;

  @Bean
  MyEnhancedDatastore datastore;

  @OrmLiteDao(helper = DatabaseHelper.class, model = User.class)
  UserDao userDao;

  @UiThread
  void showToast() {
    Toast.makeText(getContext().getApplicationContext(), "Hello World!", Toast.LENGTH_LONG).show();
  }

  // ...
}
```

### 增强 broadcast receivers：@EReceiver

You can enhance an Android BroadcastReceiver with the `@EReceiver` annotation:

```java
@EReceiver
public class MyReceiver extends BroadcastReceiver {
}
```

You can then start using most AA annotations, except the ones related to views and extras:

```java
@EReceiver
public class MyReceiver extends BroadcastReceiver {

  @SystemService
  NotificationManager notificationManager;

  @Bean
  SomeObject someObject;

}
```

#### @ReceiverAction

Since AndroidAnnotations 3.2

The `@ReceiverAction` annotation allows to simply handle Broadcasts in an enhanced Receiver.

By default the Method name is used to determine the Action to be handled, but you can use the value of `@ReceiverAction` to pass another Action name.

Method annotated with `@ReceiverAction` may have the following parameters :

- An `android.content.Context` which will be the context given in `void onReceive(Context context, Intent intent)`
- An `android.content.Intent` which will be the intent given in `void onReceive(Context context, Intent intent)`
- Any native, `android.os.Parcelable` or `java.io.Serializable` parameters annotated with `@ReceiverAction.Extra` which will be the extra put in the intent. The key of this extra is the value of the annotation `@ReceiverAction.Extra` if set or the name of the parameter.

```java
@EReceiver
public class MyIntentService extends BroadcastReceiver {

    @ReceiverAction("BROADCAST_ACTION_NAME")
    void mySimpleAction(Intent intent) {
        // ...
    }

    @ReceiverAction
    void myAction(@ReceiverAction.Extra String valueString, Context context) {
        // ...
    }

    @ReceiverAction
    void anotherAction(@ReceiverAction.Extra("specialExtraName") String valueString, @ReceiverAction.Extra long valueLong) {
        // ...
    }

    @Override
    public void onReceive(Context context, Intent intent) {
        // empty, will be overridden in generated subclass
    }
}
```

Since AndroidAnnotations 3.3

Note: Since BroadcastReceiver#onReceive is abstract, you have to add an empty implementation. For convenience, we provide the `AbstractBroadcastReceiver` class, which implements that method, so you do not have to do in your actual class if you derive it.

Note: You can now pass multiple actions that should be handled by using the value of `@ReceiverAction`.

```java
@ReceiverAction({"MULTI_BROADCAST_ACTION1", "MULTI_BROADCAST_ACTION2"})
void multiAction(Intent intent) {
    // ...
}
```

#### Data Schemes

With the `dataScheme` parameter you can set one or more dataSchemes that should be handled by the Receiver.

```java
@EReceiver
public class MyIntentService extends BroadcastReceiver {

  @ReceiverAction(actions = android.content.Intent.VIEW, dataSchemes = "http")
  protected void onHttp() {
    // Will be called when an App wants to open a http website but not for https.
  }

  @ReceiverAction(actions = android.content.Intent.VIEW, dataSchemes = {"http", "https"})
  protected void onHttps() {
    // Will be called when an App wants to open a http or https website.
  }
}
```

#### @Receiver

Your activity/fragment/service can be notified of intents using the `@Receiver` annotation, instead of declaring a `BroadcastReceiver`.

```java
@EActivity
public class MyActivity extends Activity {

  @Receiver(actions = "org.androidannotations.ACTION_1")
  protected void onAction1() {

  }

}
```

More information about the [@Receiver annotation](https://github.com/excilys/androidannotations/wiki/Receiving-intents).

Since AndroidAnnotations 3.1

### 增强 IntentService

You can enhance an Android IntentService with the `@EIntentService` annotation to simply handle actions in `@ServiceAction` annotated methods. As for `@EService`, you can then start using most AA annotations, except the ones related to views and extras.

```java
@EIntentService
public class MyIntentService extends IntentService {

    public MyIntentService() {
        super("MyIntentService");
    }

    @ServiceAction
    void mySimpleAction() {
        // ...
    }

    @ServiceAction
    void myAction(String param) {
        // ...
    }

    @Override
    protected void onHandleIntent(Intent intent) {
        // Do nothing here
    }
}
```

You can start an enhanced `IntentService` via the inner Builder :

```java
MyIntentService_.intent(getApplication()) //
    .myAction("test") //
    .start();
```

If you invoke multiple actions on the builder, only the last action will be executed.

Since AndroidAnnotations 3.3

Note: Since `IntentService#onHandleIntent` is abstract, you have to add an empty implementation. For convenience, we provide the `AbstractIntentService` class, which implements that method, so you do not have to do in your actual class if you derive it.

### 增强 Service：@EService

Since AndroidAnnotations 2.4

You can enhance an Android Service with the `@EService` annotation:

```java
@EService
public class MyService extends Service {
}
```

You can then start using most AA annotations, except the ones related to views and extras:

```java
@EService
public class MyService extends IntentService {

  @SystemService
  NotificationManager notificationManager;

  @Bean
  MyEnhancedDatastore datastore;

  @RestService
  MyRestClient myRestClient;

  @OrmLiteDao(helper = DatabaseHelper.class, model = User.class)
  UserDao userDao;

  public MyService() {
      super(MyService.class.getSimpleName());
  }

  @Override
  protected void onHandleIntent(Intent intent) {
    // Do some stuff...

    showToast();
  }

  @UiThread
  void showToast() {
    Toast.makeText(getApplicationContext(), "Hello World!", Toast.LENGTH_LONG).show();
  }
}
```

You can start an enhances service via the inner Builder :

```java
MyService_.intent(getApplication()).start();
```

Since AndroidAnnotations 3.0

The inner builder provide a stop method to stop this enhanced service:

```java
MyService_.intent(getApplication()).stop();
```

### 增强定制 views

`@EView` and `@EViewGroup` are the annotations to use if you want to create custom components.

#### 为什么要用定制的组件

If you notice that you're duplicating some parts of your layouts in different locations in your app, and that you're duplicating and calling the same methods again and again to control these parts, then a custom component can probably make your life a lot easier !

#### Custom Views with @EView

Since AndroidAnnotations 2.4

Just create a new class that extends `View`, and annotate it with `@EView`. You can than start using annotations in this view:

```java
@EView
public class CustomButton extends Button {
    @App
    MyApplication application;
    @StringRes
    String someStringResource;

    public CustomButton(Context context, AttributeSet attrs) {
        super(context, attrs);
    }
}
```

You can then start using it in your layouts (don't forget the `_`):

```
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:orientation="vertical" >

        <com.androidannotations.view.CustomButton_
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

        <!-- ... -->

    </LinearLayout>
```

You can also create it programmatically:

```java
CustomButton button = CustomButton_.build(context);
```

#### Custom ViewGroups with `@EViewGroup`

Since AndroidAnnotations 2.2

How to create it ?

First of all, let's create a layout XML file for this component.

```
    <?xml version="1.0" encoding="utf-8"?>
    <merge xmlns:android="http://schemas.android.com/apk/res/android" >

        <ImageView
            android:id="@+id/image"
            android:layout_alignParentRight="true"
            android:layout_alignBottom="@+id/title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:src="@drawable/check" />

        <TextView
            android:id="@+id/title"
            android:layout_toLeftOf="@+id/image"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:textColor="@android:color/white"
            android:textSize="12pt" />

        <TextView
            android:id="@+id/subtitle"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_below="@+id/title"
            android:textColor="#FFdedede"
            android:textSize="10pt" />

    </merge>
```

Did you know about the `merge` tag ? When this layout will get inflated, the childrens will be added directly to the parent, you'll save a level in the view hierarchy.

As you can see I used some RelativeLayout specific layout attributes (`layout_alignParentRight`, `layout_alignBottom`, `layout_toLeftOf`, etc...), it's because I'm assuming my layout will be inflated in a `RelativeLayout`. And that's indeed what I'll do !

```java
@EViewGroup(R.layout.title_with_subtitle)
public class TitleWithSubtitle extends RelativeLayout {

    @ViewById
    protected TextView title, subtitle;

    public TitleWithSubtitle(Context context, AttributeSet attrs) {
        super(context, attrs);
    }

    public void setTexts(String titleText, String subTitleText) {
        title.setText(titleText);
        subtitle.setText(subTitleText);
    }

}
```

There you are ! Easy, isn't it ?

Now let's see how to use this brand new component.

#### How to use it ?

A custom component can be declared and placed in a layout just as any other View:

```java
    <?xml version="1.0" encoding="utf-8"?>
    <LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:gravity="center"
        android:orientation="vertical" >

        <com.androidannotations.viewgroup.TitleWithSubtitle_
            android:id="@+id/firstTitle"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

        <com.androidannotations.viewgroup.TitleWithSubtitle_
            android:id="@+id/secondTitle"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

        <com.androidannotations.viewgroup.TitleWithSubtitle_
            android:id="@+id/thirdTitle"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />

    </LinearLayout>
```

As always, don't forget the `_` at the end of the component name!
And because I'm using AA, I can just as easily get my custom components injected in my activity and use them!

```java
@EActivity(R.layout.main)
public class Main extends Activity {

    @ViewById
    protected TitleWithSubtitle firstTitle, secondTitle, thirdTitle;

    @AfterViews
    protected void init() {
        firstTitle.setTexts("decouple your code",
                "Hide the component logic from the code using it.");

        secondTitle.setTexts("write once, reuse anywhere",
                "Declare you component in multiple " +
                "places, just as easily as you " +
                "would put a single basic View.");

        thirdTitle.setTexts("Let's get stated!",
                "Let's see how AndroidAnnotations can make it easier!");
    }

}
```

Most AA annotations are available in an `@EViewGroup`, give it a try !

## 注入

### Extras

Since AndroidAnnotations 1.0

#### @Extra

The `@Extra` annotation indicates that an activity field should be injected with the corresponding Extra from the Intent that was used to start the activity.

Usage example:

```java
@EActivity
public class MyActivity extends Activity {

  @Extra("myStringExtra")
  String myMessage;

  @Extra("myDateExtra")
  Date myDateExtraWithDefaultValue = new Date();

}
```

Since AndroidAnnotations 2.6

If you do not provide any value for the `@Extra` annotation, the name of the field will be used.

```java
@EActivity
public class MyActivity extends Activity {
  // The name of the extra will be "myMessage"
  @Extra
  String myMessage;
}
```

Note that you can use the intent builder to pass extra values.

```java
MyActivity_.intent().myMessage("hello").start() ;
```

#### 处理 onNewIntent()

Since AndroidAnnotations 2.6

AndroidAnnotations overrides `setIntent()`, and automatically reinjects the extras based on the given Intent when you call `setIntent()`.

This allows you to automatically reinject the extras by calling `setIntent()` from `onNewIntent()`.

```java
@EActivity
public class MyActivity extends Activity {

    @Extra("myStringExtra")
    String myMessage;

    @Override
    protected void onNewIntent(Intent intent) {
        super.onNewIntent(intent);
        setIntent(intent);
    }
}
```

Since AndroidAnnotations 3.2

When a method annotated with `@AfterExtras` is present in the activity, we override the `onNewIntent()` method to call `setIntent()`.

Since AndroidAnnotations 4.0

WARNING: Starting with Android Annotations 4.0 we no longer override `onNewIntent()` and you have to override it by yourself if needed.

#### 在 extras 注入后执行代码

Since AndroidAnnotations 3.1

If you need to execute code after extras injection, you should use the `@AfterExtras` annotation on some methods.

```java
@EActivity
public class MyClass {

  @Extra
  String someExtra;

  @Extra
  int anotherExtra;

  @AfterExtras
  public void doSomethingAfterExtrasInjection() {
    // someExtra and anotherExtra are set to the value contained in the incoming intent
    // if an intent does not contain one of the extra values the field remains unchanged
  }

}
```


#### 警告

If the parent and child classes have `@AfterViews`, `@AfterInject` or `@AfterExtras` annotated methods with the same name, the generated code will be buggy. See issue #591 for more details.

Also, while there is a guaranteed order about when we call @AfterViews, -Inject or -Extras annotated methods, there is no guaranteed order for calling each of the methods with the same @AfterXXX annotation (see issue #810).

Details about when the methods with one of those annotations are called you can find [here](https://github.com/excilys/androidannotations/wiki/%40AfterXXX-call-order).


### 注入视图

Since AndroidAnnotations 1.0

#### @ViewById

The `@ViewById` annotation indicates that an activity field should be bound with the corresponding View component from the layout. It is the same as calling the `findViewById()` method. The view id can be set in the annotation parameter, ie `@ViewById(R.id.myTextView)`. If the view id is not set, the name of the field will be used. The field must not be private.

Usage example:

```java
@EActivity
public class MyActivity extends Activity {

  // Injects R.id.myEditText
  @ViewById
  EditText myEditText;

  @ViewById(R.id.myTextView)
  TextView textView;
}
```

#### @AfterViews

The `@AfterViews` annotation indicates that a method should be called after the views binding has happened.

When `onCreate()` is called, `@ViewById` fields are not set yet. Therefore, you can use `@AfterViews` on methods to write code that depends on views.

Usage example:

```java
@EActivity(R.layout.main)
public class MyActivity extends Activity {

    @ViewById
    TextView myTextView;

    @AfterViews
    void updateTextWithDate() {
        myTextView.setText("Date: " + new Date());
    }
[...]
```

You can annotate multiple methods with `@AfterViews`. Don't forget that you should not use any view field in `onCreate()`.

Recall that injection is always made as soon as possible. Therefore, it's safe to use any field annotated, e.g., with `@Extra` or `@InstanceState` in `@AfterViews` methods as these tags don't require any view to be set (as `@AfterViews` do). Therefore you can safely assume that such fields will be already initialized with their intended values in methods annotated with `@AfterViews`.

警告

If the parent and child classes have `@AfterViews`, `@AfterInject` or `@AfterExtras` annotated methods with the same name, the generated code will be buggy. See issue #591 for more details.

Also, while there is a guaranteed order about when we call @AfterViews, -Inject or -Extras annotated methods, there is no guaranteed order for calling each of the methods with the same @AfterXXX annotation (see issue #810).

Details about when the methods with one of those annotations are called you can find [here](https://github.com/excilys/androidannotations/wiki/%40AfterXXX-call-order).

#### @ViewsById

Since AndroidAnnotations 3.1

This annotation is similar to `@ViewById`, but it injects a set of Views. It can be used on `java.util.List` or `android.view.View` subtype fields. The annotation value should be an array of R.id.* values. After injection the Views with the given IDs will be available in the List, but only the non-null ones as to avoid adding null checks to the code.

Usage example:

```java
@EActivity
public class MyActivity extends Activity {

    @ViewsById({R.id.myTextView1, R.id.myOtherTextView})
    List<TextView> textViews;

    @AfterViews
    void updateTextWithDate() {
        for (TextView textView : textViews) {
			textView.setText("Date: " + new Date());
        }
    }
}
```

### 注入 HTML

Since AndroidAnnotations 2.2

If you want to inject HTML text in a `TextView` (may it be to format it or because you love HTML), there are two annotations that can help you:

- `@FromHtml`
- `@HtmlRes`

Let's say you have the following string resource:

```
    <?xml version="1.0" encoding="utf-8"?>
    <resources>
        <string name="hello_html"><![CDATA[Hello <b>World</b>!]]></string>
    </resources>
```

#### @HtmlRes

This annotation acts as `@StringRes` (retrieves a String resource) and wraps the result with a call to `HTML.fromHtml()`:

```java
@EActivity
public class MyActivity extends Activity {

  // Injects R.string.hello_html
  @HtmlRes(R.string.hello_html)
  Spanned myHelloString;

  // Also injects R.string.hello_html
  @HtmlRes
  CharSequence helloHtml;

}
```

Note that Spanned implements CharSequence, thus you can use both for a `@HtmlRes`.

#### @FromHtml

This annotation must be used on a TextView already annotated with `@ViewById`. The purpose of this annotation is to set HTML text in a TextView:

```java
@EActivity
public class MyActivity extends Activity {

  @ViewById(R.id.my_text_view)
  @FromHtml(R.string.hello_html)
  TextView textView;

  // Injects R.string.hello_html into the R.id.hello_html view
  @ViewById
  @FromHtml
  TextView helloHtml;

}
```

### （未）简单 HTTPS

### @NonConfigurationInstance

Since AndroidAnnotations 2.5

配置发生改变后，活动可能会被销毁后重建。This behavior is great to reload resources, but you usually need to transfert references to extensive states (loaded bitmaps, network connections, actively running threads...) from the old to new activity instance.

That's what `Activity.onRetainNonConfigurationInstance()` is for (see [RetainingAnObject](http://developer.android.com/guide/topics/resources/runtime-changes.html#RetainingAnObject)). Using this method requires casting from Object, and can be cumbersome when you have multiple objects.

Annotate your activity fields with `@NonConfigurationInstance` to retain instances that are intensive to compute, on configuration changes.

```java
@EActivity
public class MyActivity extends Activity {

  @NonConfigurationInstance
  Bitmap someBitmap;

  @NonConfigurationInstance
  @Bean
  MyBackgroundTask myBackgroundTask;

}
```

Caution: while you can annotate any field, you should never annotate a field that is tied to the Activity, such as a `Drawable`, an `Adapter`, a `View` or any other object that's associated with a `Context`. If you do, it will leak all the views and resources of the original activity instance. Leaking resources means that your application maintains a hold on them and they cannot be garbage-collected, so lots of memory can be lost.

This caution doesn't apply to beans annotated with `@Bean`, because AndroidAnnotations automatically takes care of **rebinding their context**.

### 注入系统服务：@SystemService

The `@SystemService` annotation indicates that an activity field should be injected with the corresponding Android System service.

It is the same as calling the `Context.getSystemService()` method.


```java
@EActivity
public class MyActivity extends Activity {
  @SystemService
  NotificationManager notificationManager;
}
```