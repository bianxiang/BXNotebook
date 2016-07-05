[toc]

### （未）Animating Views Using Scenes and Transitions

Android includes the **transitions** framework, which enables you to easily animate changes between two view hierarchies. The framework includes built-in animations for common effects and lets you create custom animations and transition lifecycle callbacks.

> Note: For Android versions earlier than 4.4.2 (API level 19) but greater than or equal to Android 4.0 (API level 14), use the `animateLayoutChanges` attribute to animate layouts.

#### The Transitions Framework

This framework applies one or more animations to all the views in the hierarchies as it changes between them.
The framework has the following features:

- Group-level animations：Applies one or more animation effects to all of the views in a view hierarchy.
- Transition-based animation：Runs animations based on the changes between starting and ending view property values.
- 内建的动画：预定的常见特效，如淡入和移动。
- Resource file support：Loads view hierarchies and built-in animations from layout resource files.
- 生命周期回调：Defines callbacks that provide finer control over the animation and hierarchy change process.

**概述**

The framework animates each view by changing one or more of its property values over time between the initial or starting view hierarchy and the final or ending view hierarchy.

The transitions framework works in parallel with view hierarchies and animations. The purpose of the framework is to store the state of view hierarchies, change between these hierarchies in order to modify the appearance of the device screen, and animate the change by storing and applying animation definitions.

The transitions framework provides abstractions for scenes, transitions, and transition managers. These are described in detail in the following sections. To use the framework, you create scenes for the view hierarchies in your app that you plan to change between. Next, you create a transition for each animation you want to use. To start the animation between two view hierarchies, you use a **transition manager** specifying the transition to use and the ending scene. This procedure is described in detail in the remaining lessons in this class.

**Scenes**

场景存储视图层级的状态，包括其中所有视图及其属性值。Storing the view hierarchy state in a scene enables you to transition into that state from another scene. The framework provides the `Scene` class to represent a scene.

The transitions framework lets you create scenes from layout resource files or from ViewGroup objects in your code. Creating a scene in your code is useful if you generated a view hierarchy dynamically or if you are modifying it at runtime.

In most cases, you do not create a starting scene explicitly. If you have applied a transition, the framework uses the previous ending scene as the starting scene for any subsequent transitions. If you have not applied a transition, the framework collects information about the views from the current state of the screen.

A scene can also define its own actions that run when you make a scene change. For example, this feature is useful for cleaning up view settings after you transition to a scene.

In addition to the view hierarchy and its property values, a scene also stores a reference to the parent of the view hierarchy. This root view is called a **scene root**. Changes to the scene and animations that affect the scene occur within the scene root.

**Transitions**

In the transitions framework, animations create a series of frames that depict a change between the view hierarchies in the starting and ending scenes. Information about the animation is stored in a `Transition` object. To run the animation, you apply the transition using a `TransitionManager` instance. The framework can transition between two different scenes or transition to a different state for the current scene.

The framework includes a set of built-in transitions for commonly-used animation effects, such as fading and resizing views. You can also define your own custom transitions to create an animation effect using the APIs in the animations framework. The transitions framework also enables you to combine different animation effects in a transition set that contains a group of individual built-in or custom transitions.

The transition lifecycle is similar to the activity lifecycle, and it represents the transition states that the framework monitors between the start and the completion of an animation. At important lifecycle states, the framework invokes callback methods that you can implement to make adjustments to your user interface at different phases of the transition.

**限制**

This section lists some known limitations of the transitions framework:

- Animations applied to a SurfaceView may not appear correctly. SurfaceView instances are updated from a non-UI thread, so the updates may be out of sync with the animations of other views.
- Some specific transition types may not produce the desired animation effect when applied to a `TextureView`.
- Classes that extend `AdapterView`, such as ListView, manage their child views in ways that are incompatible with the transitions framework. If you try to animate a view based on `AdapterView`, the device display may hang.
- If you try to resize a `TextView` with an animation, the text will pop to a new location before the object has completely resized. To avoid this problem, do not animate the resizing of views that contain text.

#### Creating a Scene

场景存储视图层级的状态，包括其中所有视图及其属性值。The transitions framework can run animations between a starting and an ending scene. The starting scene is often determined automatically from the current state of the user interface. For the ending scene, the framework enables you to create a scene from a layout resource file or from a group of views in your code.

This lesson shows you how to create scenes in your app and how to define scene actions. The next lesson shows you how to transition between two scenes.

Note: The framework can animate changes in a single view hierarchy without using scenes, as described in Apply a Transition Without Scenes. However, understanding this lesson is essential to work with transitions.

**Create a Scene From a Layout Resource**

You can create a `Scene` instance directly from a layout resource file. Use this technique when the view hierarchy in the file is mostly static. The resulting scene represents the state of the view hierarchy at the time you created the `Scene` instance. If you change the view hierarchy, you have to recreate the scene. The framework creates the scene from the entire view hierarchy in the file; you can not create a scene from part of a layout file.

To create a `Scene` instance from a layout resource file, retrieve the scene root from your layout as a `ViewGroup` instance and then call the `Scene.getSceneForLayout()` method with the scene root and the resource ID of the layout file that contains the view hierarchy for the scene.

**Define Layouts for Scenes**

The code snippets in the rest of this section show you how to create two different scenes with the same scene root element. The snippets also demonstrate that you can load multiple unrelated Scene objects without implying that they are related to each other.

The example consists of the following layout definitions:

The main layout of an activity with a text label and a child layout.
A relative layout for the first scene with two text fields.
A relative layout for the second scene with the same two text fields in different order.
The example is designed so that all of the animation occurs within the child layout of the main layout for the activity. The text label in the main layout remains static.

The main layout for the activity is defined as follows:

res/layout/activity_main.xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/master_layout">
    <TextView
        android:id="@+id/title"
        ...
        android:text="Title"/>
    <FrameLayout
        android:id="@+id/scene_root">
        <include layout="@layout/a_scene" />
    </FrameLayout>
</LinearLayout>
This layout definition contains a text field and a child layout for the scene root. The layout for the first scene is included in the main layout file. This allows the app to display it as part of the initial user interface and also to load it into a scene, since the framework can load only a whole layout file into a scene.

The layout for the first scene is defined as follows:

res/layout/a_scene.xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/scene_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <TextView
        android:id="@+id/text_view1
        android:text="Text Line 1" />
    <TextView
        android:id="@+id/text_view2
        android:text="Text Line 2" />
</RelativeLayout>
The layout for the second scene contains the same two text fields (with the same IDs) placed in a different order and is defined as follows:

res/layout/another_scene.xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:id="@+id/scene_container"
    android:layout_width="match_parent"
    android:layout_height="match_parent" >
    <TextView
        android:id="@+id/text_view2
        android:text="Text Line 2" />
    <TextView
        android:id="@+id/text_view1
        android:text="Text Line 1" />
</RelativeLayout>
Generate Scenes from Layouts

After you create definitions for the two relative layouts, you can obtain an scene for each of them. This enables you to later transition between the two UI configurations. To obtain a scene, you need a reference to the scene root and the layout resource ID.

The following code snippet shows you how to get a reference to the scene root and create two Scene objects from the layout files:

Scene mAScene;
Scene mAnotherScene;

// Create the scene root for the scenes in this app
mSceneRoot = (ViewGroup) findViewById(R.id.scene_root);

// Create the scenes
mAScene = Scene.getSceneForLayout(mSceneRoot, R.layout.a_scene, this);
mAnotherScene =
    Scene.getSceneForLayout(mSceneRoot, R.layout.another_scene, this);
In the app, there are now two Scene objects based on view hierarchies. Both scenes use the scene root defined by the FrameLayout element in res/layout/activity_main.xml.

Create a Scene in Your Code
You can also create a Scene instance in your code from a ViewGroup object. Use this technique when you modify the view hierarchies directly in your code or when you generate them dynamically.

To create a scene from a view hierarchy in your code, use the Scene(sceneRoot, viewHierarchy) constructor. Calling this constructor is equivalent to calling the Scene.getSceneForLayout() method when you have already inflated a layout file.

The following code snippet demonstrates how to create a Scene instance from the scene root element and the view hierarchy for the scene in your code:

Scene mScene;

// Obtain the scene root element
mSceneRoot = (ViewGroup) mSomeLayoutElement;

// Obtain the view hierarchy to add as a child of
// the scene root when this scene is entered
mViewHierarchy = (ViewGroup) someOtherLayoutElement;

// Create a scene
mScene = new Scene(mSceneRoot, mViewHierarchy);
Create Scene Actions
The framework enables you to define custom scene actions that the system runs when entering or exiting a scene. In many cases, defining custom scene actions is not necessary, since the framework animates the change between scenes automatically.

Scene actions are useful for handling these cases:

Animate views that are not in the same hierarchy. You can animate views for both the starting and ending scenes using exit and entry scene actions.
Animate views that the transitions framework cannot animate automatically, such as ListView objects. For more information, see Limitations.
To provide custom scene actions, define your actions as Runnable objects and pass them to the Scene.setExitAction() or Scene.setEnterAction() methods. The framework calls the setExitAction() method on the starting scene before running the transition animation and the setEnterAction() method on the ending scene after running the transition animation.

Note: Do not use scene actions to pass data between views in the starting and ending scenes. For more information, see Defining Transition Lifecycle Callbacks.