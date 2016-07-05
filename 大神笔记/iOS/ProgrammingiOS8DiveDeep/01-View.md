[toc]

# 1 Views

一个视图（view）是 `UIView` 及其子类的对象；它知道如何在界面上的一块矩形区域内绘制自己。视图也是一个 responder（`UIView` 是 `UIResponder` 的子类）。视图层级也是 responder chain 的基础（尽管不完全相同）。视图可以来自一个 nib，或在你的代码中创建；两种方法没有好坏：按需使用。

## 1.1 Window

视图层级的根部是应用的窗口。它是 `UIWindow` 的实例（本身也是 `UIView` 子类）。应用只能有一个主窗口。它在启动时创建，永远不会被销毁或替代。它占据整个屏幕。

应用可以在一个外部屏幕显示视图，此时你要创建另外一个 `UIWindow`；但本章假设设备只有一个屏幕、一个窗口。强调：一个屏幕只需要一个 `UIWindow`。网上教程可能谈到显式创建第二个 `UIWindow`，用以在应用的主界面上面显示内容；这种提法是错的。让要内容显示在界面上方，应该添加一个视图，而不是另一个窗口。

窗口必须填满设备的屏幕；实现填满的方式是，在实例化窗口时，将窗口的 frame 设为 screen 的bounds。（后面会讲 frame 和 bounds 的概念。）当你使用主故事板时，`UIApplicationMain` 函数会在应用启动时自动帮你设置。但如果应用没有故事板，你必须自己设置窗口 frame 大小，在应用非常早期阶段：

```swift
let w = UIWindow(frame: UIScreen.mainScreen().bounds)
```

窗口需要在应用的整个生命周期内存在，为此，app delegate 类有一个 `window` 属性。当应用启动时，`UIApplicationMain` 函数实例化 app delegate 类，并保留住它，永不释放。然后，窗口对象会赋给 app delegate 对象的 `window` 属性，因此它也是持久的。

一般不要手工向主窗口添加任何视图。你应该获取一个视图控制器，将其赋给主窗口的 `rootViewController` 属性。如果使用故事板它会在背后自动帮你完成。根视图控制器的主视图将成为主窗口的唯一直接子视图。

要让视图可见，必须先让窗口成为应用的 key 窗口。这是通过调用 `UIWindow` 的实例方法  `makeKeyAndVisible` 来完成的。

下面总结主窗口的创建、配置和显示。有两种情况要考虑：

**应用有主故事板**

若应用有主故事板（由 Info.plist 的键 “Main storyboard file base name” (UIMainStoryboardFile) 指定），则 `UIApplicationMain` 实例化 `UIWindow`，设置其 frame，将其赋给 app delegate 的 `window` 属性。实例化故事板的初始视图控制器，并将其赋给窗口对象的 `rootViewController` 属性。这些都发生在 app delegate 的 `application:didFinishLaunchingWithOptions:` 方法被调用前（附录A）。最后 `UIApplicationMain` 调用 `makeKeyAndVisible` 显示应用的界面。此步将自动导致根视图控制器获取它的主视图（一般从一个 nib 加载），将其作为窗口的根视图。这几步发生在 `application:didFinishLaunchingWithOptions:` 调用后。

**应用没有主故事板**

若没有主故事板，一般需要在代码中创建、配置窗口，No Xcode 6 app template lacks a main storyboard, but if you start with, say, the Single View Application template, you can experiment as follows:

1. 编辑目标。In the General pane, select “Main” in the Main Interface field and delete it (and press Tab to set this change).
2. 删除 Main.storyboard 和 ViewController.swift。
3. 清空 AppDelegate.swift 的内容。

然后编辑 AppDelegate.swift 文件。编译运行。We didn’t set a root view controller, so you will also see a warning about that in the console (“Application windows are expected to have a root view controller at the end of application launch”); I’ll explain in a moment what to do about that.

Example 1-1. An App Delegate class with no storyboard

```swift
import UIKit
@UIApplicationMain
class AppDelegate : UIResponder, UIApplicationDelegate {
    var window : UIWindow?
    func application(application: UIApplication, didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
        self.window = UIWindow(frame:UIScreen.mainScreen().bounds)
        self.window!.backgroundColor = UIColor.whiteColor()
        self.window!.makeKeyAndVisible()
        return true
    }
}
```

几乎不需要创建 `UIWindow` 的子类。若必须，则：

**若应用有故事板**

应用启动时，`UIApplicationMain` 先实例化 app delegate，然后取它的 `window` 属性。若它的值为 `nil`，`UIApplicationMain` 实例化 `UIWindow`，将窗口对象赋给 app delegate 的`window` 属性。但如果 `window` 属性不为 `nil`，`UIApplicationMain` 就使用它作为应用的主窗口。因此，要让你的类作为主窗口的类，直接设置 app delegate 的 `window` 属性，

```swift
lazy var window : UIWindow = {
	return MyWindow(frame: UIScreen.mainScreen().bounds)
}()
```

**应用没有故事板**

见前文，你可以直接在代码里设置 `self.window` 属性，

```swift
// ...
self.window = MyWindow(frame:UIScreen.mainScreen().bounds)
// ...
```

一旦应用运行起来，有多种方式获得到窗口的引用：

1、如果一个 `UIView` 在界面中，可以通过它的 `window` 属性取到窗口。通过 `UIView` 的 `window` 属性可以确定它是否嵌入到了某个窗口中：若未，它的 `window` 属性是 `nil`，此时该 `UIView` 对用户不可见。

2、app delegate 对象通过它的 `window` 属性引用窗口。app delegate 可以通过 shared application 的 `delegate` 属性获得：

```swift
let w = UIApplication.sharedApplication().delegate!.window!!
```

If you prefer something less generic (and requiring less extreme unwrapping of Optionals), cast the delegate explicitly to your app delegate class:

```swift
let w = (UIApplication.sharedApplication().delegate as AppDelegate).window
```

3、The shared application maintains a reference to the window through its `keyWindow` property:

```swift
let w = UIApplication.sharedApplication().keyWindow
```

但此引用不稳定，因为系统可能会创建临时窗口并将其作为应用的 key 窗口。

## 1.2 动手实验

为了验证本书的知识，你可能需要一个工程。下面提供两种简单的方式。

一种方式是从 Single View Application 模板开始。它给你一个主故事板，包含一个场景，包含一个视图控制器，包含一个主视图。

通过代码向界面添加视图，最简单的方法是在控制器的 `viewDidLoad` 方法中。如：

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    let mainview = self.view
    let v = UIView(frame:CGRectMake(100,100,50,50))
    v.backgroundColor = UIColor.redColor() // small red square
    mainview.addSubview(v) // add it to main view
}
```

第二种方式，创建没有主故事板的空应用。见上节开头。视图完全通过代码创建。

设置窗口的 `rootViewController`，可以在 app delegate 的`application:didFinishLaunchingWithOptions:` 方法中。如：

```swift
func application(application: UIApplication,
	didFinishLaunchingWithOptions launchOptions: [NSObject: AnyObject]?) -> Bool {
    self.window = UIWindow(frame: UIScreen.mainScreen().bounds)
    self.window!.rootViewController = UIViewController() // *
    // and now we can add subviews
    let mainview = self.window!.rootViewController!.view
    let v = UIView(frame:CGRectMake(100, 100, 50, 50))
    v.backgroundColor = UIColor.redColor() // small red square
    mainview.addSubview(v) // add it to main view
    // and the rest is as before...
    self.window!.backgroundColor = UIColor.whiteColor()
    self.window!.makeKeyAndVisible()
    return true
}
```

## 1.3 子视图和父视图

iOS 中，一个子视图的部分或全部可以在父视图的外面，一个视图可以压另一个视图。

> To study your app’s view hierarchy at runtime while paused in the debugger, choose Debug → View Debugging → Capture View Hierarchy (new in Xcode 6). I’ll talk more about this feature later in this chapter.

视图层级决定视图绘制的顺序。对于兄弟视图，若有重叠，后画的位于先画的上面。父视图先于它的子视图绘制，因此若有重叠，父视图位于子视图下方。

在 nib 编辑器中，可以通过菜单 Editor → Arrange 下面的菜单项（Send to Front, Send to Back, Send Forward, Send Backward）调整顺序。通过代码调整兄弟顺序的方法见后文。

视图层级的其他影响：

- 视图的透明度被子视图继承。
- 视图可以限制说，子视图超过它的边界的部分都隐藏。通过视图的 `clipsToBounds` 属性设置。
- 父视图“拥有”子视图，从内存管理的角度，父视图负责管理子视图生命（子视图被移除或父视图自身被移除时）。
- 视图大小改变，子视图自动调整大小。

每个 `UIView` 有一个 `superview` 属性（一个 `UIView`）和一个 `subviews` 属性（一个 `UIView` 的数组，按从后到前的顺序）。

`isDescendantOfView:` 用于判断一个视图是否另一个的后代视图（任意深度）。

当你需要到一个视图的引用时，一般通过 outlet。或者每个视图有一个数字标签（`tag` 属性）。在视图层级中任意比它高的视图，可以通过 `viewWithTag:` 方法获取它。标签唯一性由开发者自己保证。

`addSubview:` 添加子视图；`removeFromSuperview` 将视图从父视图中删除。记住，从父视图移除一个子视图将释放它。若你打算重用这个子视图，需要自己 **retain** 它：如赋给一个属性。

上述修改会引发事件。处理这些事件需要创建子类，覆盖一下方法：

- didAddSubview:, willRemoveSubview:
- didMoveToSuperview, willMoveToSuperview:
- didMoveToWindow, willMoveToWindow:

`addSubview:` 方法，子视图将作为最后一个子视图，最后绘制，显示在最前面。子视图是有索引的，从0开始。还有一些方法，可以用于在特定索引插入子视图，或在某个视图之前或之后插入，交换相邻视图，将子视图向前或向后移动：

- insertSubview:atIndex:
- insertSubview:belowSubview:, insertSubview:aboveSubview:
- exchangeSubviewAtIndex:withSubviewAtIndex:
- bringSubviewToFront:, sendSubviewToBack:

没有一次性删除没有子视图的方法。However, a view’s `subviews` array is an immutable copy of the internal list of subviews，因此可以边遍历边删除：

```swift
for v in myView.subviews as [UIView] {
	v.removeFromSuperview()
}
```

或更紧凑的写法：

```
(myView.subviews as [UIView]).map{ $0.removeFromSuperview() }
```

## 1.4 可见性与透明度

让视图不可见，设置 `hidden` 属性为 true。隐藏的视图不会再收到事件。

设置视图背景色，通过 `backgroundColor` 属性。背景为 nil 是透明的。

视图透明度，设置 `alpha` 属性。1.0 表示不透明，0.0 表示全透明。也影响子视图：若父视图的 alpha 是 0.5，则子视图的透明度不会超过 0.5。完全透明的视图（或接近透明），相对于 `hidden` 为 true 的视图：一般也不能被触摸。

视图的 `opaque` 属性则完全是另一回事。修改它不会影响视图的外观。该属性是对绘制系统的提示。若视图完全填满它的 bounds，用不透明的材料，且 `alpha` 为 1.0。此时通过设置 `opaque` 为 true，告诉系统该事实，可以让绘制可以更高效。否则你应该讲 `opaque` 设为 false。`opaque` 不会随 `backgroundColor` 或 `alpha` 改变。正确设置完全靠开发人员；其默认值，很奇怪，是 `true`。

## 1.5 Frame

视图的 `frame` 属性是一个 CGRect；表示视图的矩形在父视图中的位置（按父视图的坐标系统）。父视图的坐标系统默认原点在左上角。

修改 `frame` 将修改视图的位置和大小。通过代码创建视图时，利用 `UIView` 的初始化器 `init(frame:)` 同时指定 `frame`，因为其默认值，`(0.0,0.0,0.0,0.0)` 一般不是你想要的 —— 忘记在创建视图时指定 frame 是一个常见的错误。frame 的默认大小，零，将导致视图不可见。若你想要采纳视图的表示准大小，如一个 `UIButton`，可以调用 `sizeToFit` 方法。

```swift
let v1 = UIView(frame:CGRectMake(113, 111, 132, 194))
v1.backgroundColor = UIColor(red: 1, green: 0.4, blue: 1, alpha: 1)
let v2 = UIView(frame:CGRectMake(41, 56, 132, 194))
v2.backgroundColor = UIColor(red: 0.5, green: 1, blue: 0, alpha: 1)
let v3 = UIView(frame:CGRectMake(43, 197, 160, 230))
v3.backgroundColor = UIColor(red: 1, green: 0, blue: 0, alpha: 1)
mainview.addSubview(v1)
v1.addSubview(v2)
mainview.addSubview(v3)
```

## 1.6 Bounds 和中心

若想让子视图的四边相对于父视图缩进10点。可以利用 `CGRectInset` 和 Swift `CGRect` 的方法 `rectByInsetting`，它们从一个特定的矩形，缩进指定距离，产生另一个矩形。我们将产生的矩形作为子视图的 `frame`。我们需要一个描述父视图大小的矩形，它就是视图的 `bounds` 属性。代码如下：

```swift
let v1 = UIView(frame: CGRectMake(113, 111, 132, 194))
v1.backgroundColor = UIColor(red: 1, green: 0.4, blue: 1, alpha: 1)
let v2 = UIView(frame: v1.bounds.rectByInsetting(dx: 10, dy: 10))
v2.backgroundColor = UIColor(red: 0.5, green: 1, blue: 0, alpha: 1)
mainview.addSubview(v1)
v1.addSubview(v2)
```

修改视图的 `bounds` 将修改它的 `frame`。对 `frame` 的修改围绕视图中心。例如：

```swift
let v1 = UIView(frame: CGRectMake(113, 111, 132, 194))
v1.backgroundColor = UIColor(red: 1, green: 0.4, blue: 1, alpha: 1)
let v2 = UIView(frame: v1.bounds.rectByInsetting(dx: 10, dy: 10))
v2.backgroundColor = UIColor(red: 0.5, green: 1, blue: 0, alpha: 1)
mainview.addSubview(v1)
v1.addSubview(v2)
v2.bounds.size.height += 20
v2.bounds.size.width += 20
```

现在子视图将完全覆盖父视图，子视图的 frame 与父视图的 bounds 相等。向 bounds 的宽高各添加20点，引起子视图的 frame 的宽高也增加20点。由于中心不移动，相对于向上下左右各增加10点。

bounds 坐标系的零点在其左上角。修改 bounds 的原点，会修改视图坐标系的原点。子视图相对于父视图的坐标系统定位，因此会影响子视图的定位。例子：

```swift
let v1 = UIView(frame: CGRectMake(113, 111, 132, 194))
v1.backgroundColor = UIColor(red: 1, green: 0.4, blue: 1, alpha: 1)
let v2 = UIView(frame: v1.bounds.rectByInsetting(dx: 10, dy: 10))
v2.backgroundColor = UIColor(red: 0.5, green: 1, blue: 0, alpha: 1)
mainview.addSubview(v1)
v1.addSubview(v2)
v1.bounds.origin.x += 10
v1.bounds.origin.y += 10
```

上述代码把父视图的左上角的坐标由 `(0.0,0.0)`，修改为 `(10.0,10.0)`。父视图的大小或位置未变。但子视图向上、向左移动，即移到了父视图的左上角。修改视图 bounds 原点看上去有点“反方向”，增加父视图原点，但子视图向负方向移动。{{将 bounds 的原点定位到(10, 10)，即视图原点定位到(10, 10)，即子视图的左上角定位到(10, 10)}}

修改视图的 frame 大小也会返过来影响 bounds 大小。修改视图 bounds 不影响视图的 `center`。该属性与 `frame` 属性一样，表示视图在父视图中的位置，使用的是父视图的坐标系。但 `center` 是 bounds 的中心，the point derived from the bounds like this:

```swift
let c = CGPointMake(theView.bounds.midX, theView.bounds.midY)
```

视图的中心因此是这样一个点，建立视图的 bounds 和父视图 bounds 的位置关系。修改视图的 bounds 不改变它的中心；修改视图的中心也不改变它的 bounds。视图的 bounds 和 center 是独立的，分别描述子视图的大小和相对于父视图的位置。因此 `frame` 是多余的！实际上，`frame` 属性仅是 `center` 和 `bounds` 的方便表达。多数情况下你会用 `frame`，如用 `init(frame:)` 创建视图。当你修改 `frame` 时， bounds 和 center 会随之修改。修改 bounds 或 center，frame 也会随之修改。不过，定位和设置视图大小的最稳定的方式是通过 `bounds` 和 `center`，而不是 `frame`；有些情况下 frame 是无意义的（或表现的很奇怪），但 bounds 和 center 总是没问题。

每个视图都有自己的坐不协调，通过它的 bounds 表达。视图的坐标系统与父视图的坐标系统有着明确的关系，通过它的 `center` 属性表达。窗口中所有视图都是这样，so it is possible to convert between the coordinates of any two views in the same window. Convenience methods are supplied to perform this conversion both for a `CGPoint` and for a CGRect:

- convertPoint:fromView:, convertPoint:toView:
- convertRect:fromView:, convertRect:toView:

若第二个参数为 `nil` 则相对于窗口。例如，若 `v2` 是 `v1` 的子视图，则将 `v2` 放在 `v1` 中心可以写成：

```swift
v2.center = v1.convertPoint(v1.center, fromView: v1.superview)
```

> 若通过 `center` 设置视图的位置，且视图的高度或宽度不是整数（或在单精度屏幕上，不是偶数），则视图的对齐会有问题：its point values in one or both dimensions are located between the screen pixels. This can cause the view to be displayed incorrectly; for example, if the view contains text, the text may be blurry. You can detect this situation in the Simulator by checking Debug → Color Misaligned Images. A simple solution is to set the view’s frame, after positioning it, to the `CGRectIntegral` of its frame, or (in Swift) to call integerize on the view’s frame.

## 1.7 窗口坐标系和屏幕坐标系

设备的屏幕没有 frame，但有 bounds。主窗口没有父视图，但它的 frame 相对于屏幕的 bounds：

```swift
let w = UIWindow(frame: UIScreen.mainScreen().bounds)
```

因此，窗口坐标是屏幕坐标。

iOS 7 及之前，屏幕的坐标系是不变的，不管设备的朝向和应用的旋转。但 iOS 8 修改了这类坐标系统：当应用旋转以补偿设备的旋转，屏幕（和窗口）是旋转后的额。在竖屏下，屏幕和窗口的 bounds 是高大于宽；但在横屏模式下，宽度大于高度。

这项修改可能破坏已有代码。

若你想使用设备坐标，不管屏幕如何旋转，iOS 8 引入了 `UICoordinateSpace` 协议，提供 `bounds` 属性。`UIView` 采纳了 `UICoordinateSpace` 协议。此外还有两个属性也采纳了该协议：

**UIScreen 的 `coordinateSpace` 属性**：该坐标系随着屏幕旋转而旋转；它的 (0.0,0.0) 位于**应用的**左上角。

**UIScreen 的 `fixedCoordinateSpace` 属性**：This coordinate space is invariant, meaning that its top left represents the physical top left of the device qua physical device; its (0.0,0.0) point thus might be in any corner (from the user’s perspective).

To help you convert between coordinate spaces, `UICoordinateSpace` also provides four methods parallel to the coordinate-conversion methods I listed in the previous section:

- convertPoint:fromCoordinateSpace:, convertPoint:toCoordinateSpace:
- convertRect:fromCoordinateSpace:, convertRect:toCoordinateSpace:

So, for example, suppose we have a `UIView v` in our interface, and we wish to learn its position in fixed device coordinates. We could do it like this:

```swift
let r = v.convertRect(v.frame, toCoordinateSpace:
	UIScreen.mainScreen().fixedCoordinateSpace)
```

It doesn’t actually matter to what view or coordinate space we send the `convertRect:toCoordinateSpace:` message; the result will be the same.

Occasions where you need such information, however, will be rare. Everything takes place within your root view controller’s main view, and the bounds of that view, which are adjusted for you automatically when the app rotates to compensate for a change in device orientation, are the highest coordinate system that will normally interest you.

## 1.8 Transform

视图的 `transform` 属性，例如可以改变视图的大小，但不影响它的 `bounds` 和 `center`。

`transform` 值是一个 `CGAffineTransform`，一个结构，表示一个 3×3 变换矩阵中的6个值（剩下三个值是常量）。For the details, which are quite simple really, see the “Transforms” chapter of Apple’s Quartz 2D Programming Guide, especially the section called “The Math Behind the Matrices.” 但实际你不需要懂背后的数学，有一些便利的方法，`CGAffineTransformMake...`，用于创建基本的变换：旋转、缩放、平移。第四类变换，skewing 或 shearing，没有便利方法。

视图默认的变换矩阵是 `CGAffineTransformIdentity`，the identity transform。即什么也不变。任何变换都围绕视图的中心，which is held constant。

```swift
let v1 = UIView(frame:CGRectMake(113, 111, 132, 194))
v1.backgroundColor = UIColor(red: 1, green: 0.4, blue: 1, alpha: 1)
let v2 = UIView(frame:v1.bounds.rectByInsetting(dx: 10, dy: 10))
v2.backgroundColor = UIColor(red: 0.5, green: 1, blue: 0, alpha: 1)
mainview.addSubview(v1)
v1.addSubview(v2)
v1.transform = CGAffineTransformMakeRotation(45 * CGFloat(M_PI)/180.0)
```

变换后，视图的 `bounds` 属性不变；内部的坐标系统不改变，因此子视图相对于父视图不变。**但视图的 frame 不再有效**。规则：若视图的 `transform` 不是 identity transform，就不要设置 frame；also, automatic resizing of a subview, discussed later in this chapter, requires that the superview’s `transform` be the identity transform.

Suppose, instead of `CGAffineTransformMakeRotation`, we call `CGAffineTransformMakeScale`, like this:

```swift
v1.transform = CGAffineTransformMakeScale(1.8, 1)
```

可以施加多个变换。There are convenience functions for applying one transform to another. 它们的名字中没有 `Make`。施加的顺序是有影响的。To demonstrate the difference, I’ll start with a subview that exactly overlaps its superview:

```swift
let v1 = UIView(frame:CGRectMake(20, 111, 132, 194))
v1.backgroundColor = UIColor(red: 1, green: 0.4, blue: 1, alpha: 1)
let v2 = UIView(frame:v1.bounds)
v2.backgroundColor = UIColor(red: 0.5, green: 1, blue: 0, alpha: 1)
mainview.addSubview(v1)
v1.addSubview(v2)
```

Then I’ll apply two successive transforms to the subview, leaving the superview to show where the subview was originally. In this example, I translate and then rotate (Figure 1-8):

```swift
v2.transform = CGAffineTransformMakeTranslation(100, 0)
v2.transform = CGAffineTransformRotate(v2.transform, 45 * CGFloat(M_PI)/180.0)
```

In this example, I rotate and then translate (Figure 1-9):

```swiftv2.transform = CGAffineTransformMakeRotation(45 * CGFloat(M_PI)/180.0)v2.transform = CGAffineTransformTranslate(v2.transform, 100, 0)
```

`CGAffineTransformConcat` 使用矩阵乘法连接两个变换。这里顺序也是重要的。The order is the opposite of the order when using convenience functions for applying one transform to another. For example, this gives the same result as Figure 1-9:

```swiftlet r = CGAffineTransformMakeRotation(45 * CGFloat(M_PI)/180.0)let t = CGAffineTransformMakeTranslation(100, 0)v2.transform = CGAffineTransformConcat(t, r) // not r,t
```

To remove a transform from a combination of transforms, apply its inverse. A convenience function lets you obtain the inverse of a given affine transform. Again, order matters. In this example, I rotate the subview and shift it to its “right,” and then remove the rotation (Figure 1-10):

```swiftlet r = CGAffineTransformMakeRotation(45 * CGFloat(M_PI)/180.0)let t = CGAffineTransformMakeTranslation(100, 0)v2.transform = CGAffineTransformConcat(t, r)v2.transform = CGAffineTransformConcat(CGAffineTransformInvert(r), v2.transform)
```

Finally, as there are no convenience methods for creating a skew (shear) transform, I’ll illustrate by creating one manually, without further explanation (Figure 1-11):

```swiftv1.transform = CGAffineTransformMake(1, 0, -0.2, 1, 0, 0)
```
In iOS 7 and before, the `transform` property lay at the heart of an iOS app’s ability to rotate its interface: the window’s `frame` and `bounds` were invariant, locked to the screen, and an app’s interface rotated to compensate for a change in device orientation by applying a rotation transform to the root view, so that its `(0.0,0.0)` point moved to what the user now saw as the top left of the view.

但在 iOS 8 中不是这样。The screen’s coordinate space is effectively rotated, but a coordinate space doesn’t have a `transform` property, so the rotation transform applied to that coordinate space is fictitious: you can work out what has happened, if you really want to, by comparing the screen’s `coordinateSpace` with its `fixedCoordinateSpace`, but none of the views in the story — neither the window, nor the root view, nor any of its subviews — receives a rotation transform when the app’s interface rotates. If you had code, inherited from iOS 7 or before, that relied on the assumption that a rotated app’s root view had a rotation transform, that code will break when compiled and run against iOS 8.

Instead, iOS 8 expects you to concentrate on the dimensions of the window, the root view, and so forth. And this doesn’t mean their absolute dimensions (though you might have reason to consider these), but their dimensions relative to an iPad. This dimensional relationship is embodied in a set of **size classes** which are vended by a view’s `traitCollection` property as a `UITraitCollection` object. I’ll discuss trait collections and size classes further in the next section.

One purpose of this innovation in iOS 8 is so that you can treat app rotation as effectively nothing more than a change in the interface’s proportions: when the app rotates, the long dimension (of the root view, the window, and the screen’s coordinate space bounds) becomes its short dimension and vice versa. This, after all, is what your interface needs to take into account in order to keep working when the app rotates.
Consider, for example, a subview of the root view, located at the bottom right of the screen when the device is in portrait orientation. If the root view’s bounds width and bounds height are effectively swapped, then that poor old subview will now be outside the bounds height, and therefore off the screen — unless your app responds in some way to this change to reposition it. Such a response is called layout, a subject that will occupy most of the rest of this chapter. The point, however, is that what you’re responding to, in iOS 8, is just a change in the window’s proportions; the fact that this change stems from rotation of the app’s interface is virtually irrelevant.

## 1.9 特点集（Trait Collections）和尺寸类（Size Classes）

特点集（Trait collections）是 iOS 8 一项主要创新。Every view in the interface, from the window on down, as well as any view controller whose view is part of the interface, inherits from the environment the value of its `traitCollection` property, which it has by virtue of implementing the `UITraitEnvironment` protocol. The `traitCollection` is a `UITraitCollection`, a value class consisting of four properties:

**displayScale**：从当前屏幕继承的缩放，对于单、双精度值分别是 1 和 2。对于 iPhone 6 Plus 是3.。（该属性与 `UIScreen` 的 `scale` 属性相等）。

**userInterfaceIdiom**：`UserInterfaceIdiom` 枚举，`.Phone` 或 `.Pad`，表示当前运行的设备类型。（等于 `UIDevice` 的 `userInterfaceIdiom` 属性）。

**horizontalSizeClass, verticalSizeClass**：`UIUserInterfaceSizeClass` 枚举值， `.Regular` 或 `.Compact`。它们称为 **size classes**. size classes 组合起来有以下含义：

1. 垂直和水平 size classes 都是 `.Regular`。当前运行在 iPad 上。
2. 垂直 size class 是 `.Regular`，但水平 size class 是 `.Compact`。当前运行在 iPhone，竖屏。
3. 垂直和水平 size classes 都是 `.Compact`。运行在 iPhone（iPhone 6 Plus 除外），横屏。
4. 垂直 size class 是 `.Compact`，但水平 size class 是 `.Regular`，运行在 iPhone 6 Plus，横屏。

> To be iPhone 6 Plus–native, your app must either designate a .xib or .storyboard file as its launch screen or contain a Retina HD 5.5 launch image in the asset catalog. 否则 iPhone 6 Plus 上的 size class 将与其他 iPhone 相同 （且你的应用会被放大）。

The trait collection properties might not change during the lifetime of an app, but they still might differ, and thus be of interest to your code, from one run of an app to another. In particular, if you write a universal app, one that runs natively on different device types (iPhone and iPad), you will probably want your interface to differ depending on which device type we’re running on; trait collections are the iOS 8 way to detect that.

Moreover, some trait collection properties can change during the lifetime of an app. 比如 iphone 上的 size classes 会随着应用朝向的变化而变化。

Thus, the environment’s trait collection is considered to have changed on two main occasions:

- The interface is assembled initially.
- The app rotates on an iPhone.

At such moments, the `traitCollectionDidChange:` message is propagated down the hierarchy of `UITraitEnvironments` (meaning primarily, for our purposes, view controllers and views); the old trait collection is provided as the parameter, and the new trait collection can be retrieved as `self.traitCollection`.

> Perhaps you are now saying to yourself: “Wait, there aren’t enough size classes!” You’re absolutely right. Apple has decided that size classes should not differentiate between an iPad in portrait orientation and an iPad in landscape orientation. This seems an odd design decision (especially since iPad apps whose interface changes radically between landscape and portrait are standard, as I’ll illustrate when discussing split views in Chapter 9); I can’t explain it.

It is also possible to create a trait collection yourself. (It may not be immediately obvious why this would be a useful thing to do; I’ll give an example in the next chapter.) Oddly, however, you can’t set any trait collection properties directly; instead, you form a trait collection through an initializer that determines just one property, and if you want to add further property settings, you have to combine trait collections by calling `init(traitsFromCollections:)`. For example:

```swift
let tcdisp = UITraitCollection(displayScale: 2.0)
let tcphone = UITraitCollection(userInterfaceIdiom: .Phone)
let tcreg = UITraitCollection(verticalSizeClass: .Regular)
let tc = UITraitCollection(traitsFromCollections: [tcdisp, tcphone, tcreg])
```

When combining trait collections with `init(traitsFromCollections:)`, an ordered intersection is performed. If two trait collections are combined, and one sets a property and the other doesn’t (the property isn’t set or its value isn’t yet known), the one that sets the property wins; if they both set the property, the winner is the trait collection that appears later in the array.

Similarly, if you create a trait collection and you don’t specify a property, this means that the value for that property is to be inherited if the trait collection finds itself in the inheritance hierarchy. (You cannot, however, insert a trait collection directly into the inheritance hierarchy simply by setting a view’s trait collection; traitCollection isn’t a settable property. Instead, you’ll use a UIViewController method,
`setOverrideTraitCollection:forChildViewController:`; view controllers are the subject of Chapter 6.)

To compare trait collections, call `containsTraitsInCollection:`. This returns true if the value of every specified property of the second trait collection (the argument) matches that of the first trait collection (the target of the message).

## 1.10 布局

三种布局方式：

- **手工布局**：当父视图大小变化时，它的 `layoutSubviews` 方法会被调用；因此若要手工布局，创建子类，覆盖 `layoutSubviews`。
- **Autoresizing**：Autoresizing 是 iOS 6 之前的自动布局方式。A subview will respond to its superview’s being resized, in accordance with the rules prescribed by the subview’s `autoresizingMask` property value.
- **Autolayout**：从 iOS 6 开始引入，布局依靠视图的约束。A constraint (an instance of `NSLayoutConstraint`) is a full-fledged object with numeric values describing some aspect of the size or position of a view, often in terms of some other view; it is much more sophisticated, descriptive, and powerful than the `autoresizingMask`. 可以对一个视图施加多个约束。Autolayout is implemented behind the scenes in `layoutSubviews`; in effect, constraints allow you to write sophisticated `layoutSubviews` functionality without code.

上面三种策略可以组合使用。很少需要手工布局。Autoresizing 是默认使用的策略，除非你设置父视图的 `autoresizesSubviews` 属性为 `false`，或除非视图使用 autolayout。自动布局可以在局部使用；使用 autolayout 的视图可以与使用 autoresizing 的视图放在一起。

使用 autolayout 的主要地方是 nib 文件。Xcode 5 和 6，所有的 .storyboard 和 .xib  文件默认启用 autolayout。要验证这一点，选中文件，显示 File inspector，找到选项 “Use Auto Layout”。

通过代码创建和添加的视图，默认使用 autoresizing，而非 autolayout。

### 1.10.1 Autoresizing

Autoresizing is a matter of conceptually assigning a subview “springs and struts.” 弹簧（spring）可以拉伸；但撑杆（strut）不行。Springs and struts can be assigned internally or externally, horizontally or vertically. Thus you can specify (using internal springs and struts) whether and how the view can be resized, and (using external springs and struts) whether and how the view can be repositioned. For example:

- 子视图要始终居中父视图，但要随着父视图调整大小。它需要外部的撑杆和内部的弹簧。如果不要随着父视图调整大小，则需要外部的弹簧和内部的撑杆。
- 一个确认按钮，放在父视图的右下角。使用内部的撑杆，在外部的右边的下边使用撑杆，在上边和左边使用弹簧。
- 一个文本框放在父视图的顶部。宽度与父视图相同。在外部使用撑杆，底部除外，使用弹簧；内部垂直使用撑杆，水平使用弹簧。

在代码中，弹簧和撑杆的组合通过视图的 `autoresizingMask` 属性设置。它是一个位掩码，因此你可以通过“二进制或”组合它们。`UIViewAutoresizing` 的某种成员，表示弹簧；whatever isn’t specified is a strut. 默认是 `.None`，表示所有都是撑杆：但实际上无法做到所有都是撑杆，因为当父视图大小改变后，总有什么需要改变；现实中， `.None` 相当于 `.FlexibleRightMargin` 和 `.FlexibleBottomMargin` 的组合。

> In debugging, when you log a UIView to the console, its `autoresizingMask` is reported using the word “autoresize” and a list of the springs. The margins are LM, RM, TM, and BM; the internal dimensions are W and H. For example, `autoresize = LM+TM` means that what’s flexible is the left and top margins; `autoresize = W+BM` means that what’s flexible is the width and the bottom margin.

例子。让一个视图在上面拉伸，一个在右下角固定，

```swift
let v1 = UIView(frame:CGRectMake(100, 111, 132, 194))
v1.backgroundColor = UIColor(red: 1, green: 0.4, blue: 1, alpha: 1)
let v2 = UIView(frame:CGRectMake(0, 0, 132, 10))
v2.backgroundColor = UIColor(red: 0.5, green: 1, blue: 0, alpha: 1)
let v3 = UIView(frame:CGRectMake(v1.bounds.width-20, v1.bounds.height-20, 20, 20))
v3.backgroundColor = UIColor(red: 1, green: 0, blue: 0, alpha: 1)
mainview.addSubview(v1)
v1.addSubview(v2)
v1.addSubview(v3)

v2.autoresizingMask = .FlexibleWidth
v3.autoresizingMask = .FlexibleTopMargin | .FlexibleLeftMargin
```

You might modify the previous example to pin the size of v1 to the size of the root view, and then run the app and rotate the device. Thus you might initially configure v1 like this:

```
v1.frame = mainview.bounds
v1.autoresizingMask = .FlexibleHeight | .FlexibleWidth
```

Autoresizing 只能描述子视图与父视图的关系；Before autolayout, the way to achieve more sophisticated goals of that sort was to combine **autoresizing** with manual layout in `layoutSubviews`. Autoresizing 发生在 `layoutSubviews` 之前，so your `layoutSubviews` code is free to come marching in and tidy up whatever autoresizing didn’t get quite right.

### 1.10.2 Autolayout

每个视图都可以选择启用或不启用 Autolayout，选择启用的方式有：

- 通过代码向视图添加自动布局约束。参与的视图将启用自动布局。
- Your app loads a nib for which “Use Auto Layout” is checked. nib 实例化的每个视图都使用自动布局。
- A view in the interface, which would be an instance of a custom UIView subclass of yours, returns true from the class method `requiresConstraintBasedLayout`. That view uses autolayout. The reason for this third approach to opting in to autolayout is that you might need autolayout to be switched on in order to add autolayout constraints in code. A common place to create constraints in code is in a view’s `updateConstraints` implementation (discussed later in this chapter). However, if autolayout isn’t switched on, `updateConstraints` won’t be called. So `requiresConstraintBasedLayout` provides a way of switching it on.

一个视图用自动布局，但它一旁相邻的视图可以不用；父视图可以用自动布局，但它的一个或全部子视图不用。However, autolayout is implemented through the superview chain, 因此一个视图使用了自动布局，则它的父视图都会使用；如果其中一个是视图控制器的主视图（一般是这样），视图控制器会收到自动布局的相关事件。

> 一个 nib 中的界面不能只部分使用布局：要么全用要么全不用。若想部分使用，分拆成多个小 nibs，在运行时加载和组合。

#### 约束

一个约束是一个 `NSLayoutConstraint` 对象；描述视图的绝对宽度或高度，或描述视图的一个属性与另一个视图的一个属性的关系。两个视图不需要相邻，不一定是父子关系，只要它们具有相同的祖先就行。

下面是 `NSLayoutConstraint` 的一些主要属性：

**firstItem, firstAttribute, secondItem, secondAttribute**：约束涉及的两个视图及相应属性（ `NSLayoutAttribute` ）。若约束描述视图的绝对宽高，`secondItem` 为 nil，`secondAttribute` 为 `.NotAnAttribute`。 `NSLayoutAttribute` 有以下值：

    .Left, .Right
    .Top, .Bottom
    .Leading, .Trailing
    .Width, .Height
    .CenterX, .CenterY
    .Baseline, .FirstBaseline

`.FirstBaseline` 主要用于多行标签；`.Baseline` 是最后一个基线。

**multiplier, constant**：These numbers will be applied to the second attribute’s value to determine the first attribute’s value. The multiplier is multiplied by the second attribute’s value; the constant is added to that product. The first attribute is set to the result. If you’re describing a view’s width or height absolutely, the multiplier will be 1 and the constant will be the width or height value.

**relation**：An `NSLayoutRelation` stating how the two attribute values are to be related to one another, as modified by the multiplier and the constant. This is the operator that goes in the spot where I put the equal sign in the equation in the preceding paragraph. It might be an equal sign (`.Equal`), but inequalities are also permitted (`.LessThanOrEqual`, `.GreaterThanOrEqual`).

**priority**：优先级范围从 1000（必须）到1。某些标准的行为具有标准优先级。约束的优先级决定它施加的顺序。允许约束相冲突，只要它们的优先级不同。

一个约束属于某个视图。一个视图可以有多个约束：`UIView` 有一个 `constraints` 属性和几个实例方法：

    addConstraint:, addConstraints:
    removeConstraint:, removeConstraints:

约束应该添加到哪个视图：参与约束的两个视图的最近的祖先。或可能是两个视图中的一个。例如，若约束描述了视图的绝对宽度，则约束添加到该视图；如果设定了相对于父视图的偏移，约束添加到父视图；若对齐两个兄弟视图的上边缘，则约束添加到它们的父视图。

iOS 8 开始，约束有一个布尔属性 `active`，and constraints can be activated or deactivated together with `NSLayoutConstraint` class methods `activateConstraints:` and `deactivateConstraints:`. Unfortunately, these features are currently undocumented; but it appears that activating a constraint is equivalent to adding it automatically to the correct view, and thus these are ways to add and remove constraints with the focus of attention on the constraint rather than on the view.

`NSLayoutConstraint` 属性是只读的，`priority` 和 `constant` 除外。要修改其他的属性，只能删除约束后重新添加。

#### Autoresizing 约束

自动布局会涉及到未采用自动布局的视图。运行时会负责处理：将视图的 `frame` 和 `autoresizingMask` 设置转换为约束。结果是一组隐式约束，类 `NSAutoresizingMaskLayoutConstraint` 的实例。

例如，加入我有一个 `UILabel`，frame 是 (20.0,20.0,42.0,22.0)，`autoresizingMask` 是 `.None`。If this label were suddenly to come under autolayout, then its superview would acquire four implicit constraints setting its width and height at 42 and 22 and its center X and center Y at 41 and 31.

仅当视图的 `translatesAutoresizingMaskIntoConstraints` 属性为 true 时才发生转换。That is, in fact, the default if the view came into existence either in code or by instantiation from a nib where “Use Auto Layout” is not checked. The assumption is that if a view came into existence in either of those ways, you want its `frame` and `autoresizingMask` to act as its constraints if it becomes involved in autolayout.

That’s a sensible rule, but it means that if you intend to apply any explicit constraints of your own to such a view, you’ll probably want to remember to turn off this automatic behavior by setting the view’s `translatesAutoresizingMaskIntoConstraints` property to false. If you don’t, you’re going to end up with both implicit constraints and explicit constraints affecting this view, and it’s unlikely that you would want that. Typically, that sort of situation will result in a conflict between constraints, as I’ll explain a little later; indeed, what usually happens to me is that I don’t remember to set the view’s `translatesAutoresizingMaskIntoConstraints` property to false, and am reminded to do so only when I do get a conflict between constraints.

> For some obscure reason, `translatesAutoresizingMaskIntoConstraints` is not a directly settable property in Swift; to set it, you have to call a view’s `setTranslatesAutoresizingMaskIntoConstraints:` method.

#### 在代码中创建约束

I’ll generate the same views and subviews and layout behavior as in Figures 1-12 and 1-13, but using constraints. Observe that I don’t bother to assign the subviews v2 and v3 explicit frames as I create them, because constraints will take care of positioning them, and that I remember (for once) to set their `translatesAutoresizingMaskIntoConstraints` properties to `false`:

```swift
let v1 = UIView(frame:CGRectMake(100, 111, 132, 194))
v1.backgroundColor = UIColor(red: 1, green: 0.4, blue: 1, alpha: 1)
let v2 = UIView()
v2.backgroundColor = UIColor(red: 0.5, green: 1, blue: 0, alpha: 1)
let v3 = UIView()
v3.backgroundColor = UIColor(red: 1, green: 0, blue: 0, alpha: 1)
mainview.addSubview(v1)
v1.addSubview(v2)
v1.addSubview(v3)
v2.setTranslatesAutoresizingMaskIntoConstraints(false)
v3.setTranslatesAutoresizingMaskIntoConstraints(false)
v1.addConstraint(NSLayoutConstraint(item: v2,
        attribute: .Left,
        relatedBy: .Equal,
        toItem: v1,
        attribute: .Left,
        multiplier: 1, constant: 0))
v1.addConstraint(NSLayoutConstraint(item: v2,
        attribute: .Right,
        relatedBy: .Equal,
        toItem: v1,
        attribute: .Right,
        multiplier: 1, constant: 0))
v1.addConstraint(NSLayoutConstraint(item: v2,
        attribute: .Top,
        relatedBy: .Equal,
        toItem: v1,
        attribute: .Top,
        multiplier: 1, constant: 0))
v2.addConstraint(NSLayoutConstraint(item: v2,
        attribute: .Height,
        relatedBy: .Equal,
        toItem: nil,
        attribute: .NotAnAttribute,
        multiplier: 1, constant: 10))
v3.addConstraint(NSLayoutConstraint(item: v3,
        attribute: .Width,
        relatedBy: .Equal,
        toItem: nil,
        attribute: .NotAnAttribute,
        multiplier: 1, constant: 20))
v3.addConstraint(NSLayoutConstraint(item: v3,
        attribute: .Height,
        relatedBy: .Equal,
        toItem: nil,
        attribute: .NotAnAttribute,
        multiplier: 1, constant: 20))
v1.addConstraint(NSLayoutConstraint(item: v3,
        attribute: .Right,
        relatedBy: .Equal,
        toItem: v1,
        attribute: .Right,
        multiplier: 1, constant: 0))
v1.addConstraint(NSLayoutConstraint(item: v3,
        attribute: .Bottom,
        relatedBy: .Equal,
        toItem: v1,
        attribute: .Bottom,
        multiplier: 1, constant: 0))
```

> Once you are using explicit constraints to position and size a view, do not set its frame (or bounds and center) subsequently; use constraints alone. Otherwise, when layoutSubviews is called, the view will jump back to where its constraints position it. (The exception is that you may set a view’s frame if you are in layoutSubviews, as I’ll explain later.)

#### （未）格式语言

通过格式语言，一次可以创建多个约束。例如：`"V:|[v2(10)]"`。其中，`V:` 表示垂直方向；另一个选择是 `H:`，它也是默认值，因此可省略。A view’s name appears in square brackets, and a pipe (|) signifies the superview, so here we’re portraying v2’s top edge as butting up against its superview’s top edge. Numeric dimensions appear in parentheses, and a numeric dimension
accompanying a view’s name sets that dimension of that view, so here we’re also setting v2’s height to 10.

To use a visual format, you have to provide a dictionary mapping the string name of each view mentioned to the actual view. For example, the dictionary accompanying the preceding expression might be `["v2":v2]`. So here’s another way of expressing of the preceding code example, using
the visual format shorthand throughout:

```swift
let d = ["v2":v2,"v3":v3]
v1.addConstraints(
	NSLayoutConstraint.constraintsWithVisualFormat(
		"H:|[v2]|", options: nil, metrics: nil, views: d))
v1.addConstraints(
	NSLayoutConstraint.constraintsWithVisualFormat(
		"V:|[v2(10)]", options: nil, metrics: nil, views: d))
v1.addConstraints(
	NSLayoutConstraint.constraintsWithVisualFormat(
    	"H:[v3(20)]|", options: nil, metrics: nil, views: d))
v1.addConstraints(
    NSLayoutConstraint.constraintsWithVisualFormat(
    	"V:[v3(20)]|", options: nil, metrics: nil, views: d))
```

Here are some further things to know when generating constraints with the visual format syntax:

#### （未）Constraints as objects

Although the examples so far have involved creating constraints and adding them directly to the interface — and then forgetting about them — it is frequently useful to form constraints and keep them on hand for future use (typically in a property). A common use case is where you intend, at some future time, to change the interface in some radical way, such as by inserting or removing a view; you’ll probably find it convenient to keep multiple sets of constraints on hand, each set being appropriate to a particular configuration of the interface. It is then trivial to swap constraints out of and into the interface along with views that they affect.

In this example, we have prepared two properties, `constraintsWith` and `constraintsWithout`, initialized as empty arrays of `NSLayoutConstraint`:

```
var constraintsWith = [NSLayoutConstraint]()
var constraintsWithout = [NSLayoutConstraint]()
```

We create within our main view (mainview) three views, v1, v2, and v3, which are red, yellow, and blue rectangles respectively. We keep strong references (as properties) to all three views. For some reason, we will later want to remove and insert the yellow view (v2) dynamically as the app runs, moving the blue view to where the yellow view was when the yellow view is absent (Figure 1-14). So we create two sets of constraints, one describing the positions of v1, v2, and v3 when all three are present, the other describing the positions of v1 and v3 when v2 is absent. We start with v2 present, so it is the first set of constraints that we initially hand over to our main view:

#### Guides 和 margins

在 iOS 中，界面顶部和顶部常常被一条通栏占据（状态栏、导航栏、工具栏、标签栏）。你的布局一般占据通栏之间的部分。iOS 6 和之前，视图控制器会自动调整视图大小让视图适合中间的区域。但 iOS 7 和 iOS 8，视图可以在通栏后方，延伸到屏幕边界。这些通栏有的会动态的显示和因此，或改变高度；例如 iOS 8 的默认行为是，在横屏模式下，iPhone 应用的通知栏隐藏，导航栏在竖屏模式下比在横屏模式下更高。Therefore, you need something else to anchor your vertical constraints to — something that will move vertically to reflect the location of the bars. 否则界面会有时正确显示有时有问题。For example, consider a view whose top is constrained to the top of the view controller’s main view, which is its superview:

```swift
mainview.addConstraints(
    NSLayoutConstraint.constraintsWithVisualFormat(
    	"V:|-0-[v]", options: nil, metrics: nil, views: ["v":v])
    )
```

在横屏模式下没有问题。但在竖屏模式下，状态栏会显示，会压在界面上面。

`UIViewController` 提供和维护了两个不可见的视图，**top layout guide** 和 **bottom layout guide**，它们是主视图的子视图。子视图的上面的约束应相对于 top layout guide 的底边，子视图的底边应相对于 bottom layout guide 的顶边。top layout guide 的底边会自动匹配上部通栏的最底边，或主视图的顶部（如果没有顶部通栏）；bottom layout guide 类似。

在代码中可以通过 `UIViewController` 的属性 `topLayoutGuide` 和 `bottomLayoutGuide` 访问这两个布局指导。

```swift
mainview.addConstraints(
    NSLayoutConstraint.constraintsWithVisualFormat(
    	"V:[tlg]-0-[v]", options: nil, metrics: nil,
    	views: ["tlg":self.topLayoutGuide, "v":v])
    )
```

And here’s the same thing without a visual format string:

```swift
mainview.addConstraint(
    NSLayoutConstraint(item: v,
        attribute: .Top,
        relatedBy: .Equal,
        toItem: self.topLayoutGuide,
        attribute: .Bottom,
        multiplier: 1, constant: 0)
)
```

iOS 8 开始，视图可以有 margins。视图的 `layoutMargins` 属性是一个 `UIEdgeInsets`， expressing the minimum standard distance of subviews from the edge of this view as superview. A visual format string that pins a subview’s edge to its superview’s edge, expressed as a pipe character (|) and a hyphen with no explicit distance value, will cause the subview to butt up against the superview’s margin. iOS 7 及之前，该标准的最小间距是 20；在 iOS 8，it is up to the individual superview。视图控制器的主视图，默认上下是0，左右是16；其他视图，四边都是8。

Thus, for example, here’s a view that’s butting up against its superview’sleft margin:

```swiftmainview.addConstraints(	NSLayoutConstraint.constraintsWithVisualFormat(	"H:|-[v]", options: nil, metrics: nil, views: ["v":v]))
```
When using the full constraint-creation syntax, you pin a view with respect to another view’s margins using an additional set of `NSLayoutAttribute` values that’s new in iOS 8:
	.LeftMargin, .RightMargin	.TopMargin, .BottomMargin	.LeadingMargin, .TrailingMargin	.CenterXWithinMargins, .CenterYWithinMarginsSo here’s the same view placed against its superview’s left margin, without using a visual format string:

```swift
mainview.addConstraint(	NSLayoutConstraint(item: v,		attribute: .Left,		relatedBy: .Equal,		toItem: mainview,		attribute: .LeftMargin,		multiplier: 1, constant: 0))
```

An additional UIView property, `preservesSuperviewLayoutMargins`, if true, causes a view to adopt as its `layoutMargins` the intersection of its own and its superview’s `layoutMargins`. For example, consider the following:

```swiftlet v = UIView()v.backgroundColor = UIColor.redColor()v.setTranslatesAutoresizingMaskIntoConstraints(false)mainview.addSubview(v)mainview.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat(
	"H:|-(0)-[v]-(0)-|", options: nil, metrics: nil, views: ["v":v]))mainview.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat(	"V:|-(0)-[v]-(0)-|", options: nil, metrics: nil, views: ["v":v]))v.preservesSuperviewLayoutMargins = true
```
The view v has, by default, layout margins {8, 8, 8, 8}. Its superview, mainview, is the view controller’s main view, and has, by default, layout margins {0, 16, 0, 16}. The subview v exactly covers its superview mainview: their edges match. Normally this would have no effect on v’s layout margins, but because we have set its preservesSuperviewLayoutMargins to true, the part of mainview’s layout margins that extends further inwards than v’s layout margins is adopted by v as its own, so that v’s layout margins are now actually {8, 16, 8, 16}. We can see the effect of this if we subsequently give v a subview pinned to its layout margins:

```swiftlet v1 = UIView()v1.backgroundColor = UIColor.greenColor()v1.setTranslatesAutoresizingMaskIntoConstraints(false)v.addSubview(v1)v.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat(	"H:|-[v1]-|", options: nil, metrics: nil, views: ["v1":v1]))v.addConstraints(NSLayoutConstraint.constraintsWithVisualFormat(	"V:|-[v1]-|", options: nil, metrics: nil, views: ["v1":v1]))
```
The green subview v1 is inset 16 points at the left and right from its red superview v, and 8 points at the top and bottom.

#### （未）Mistakes with constraints

#### （未）Intrinsic content size

### （未）1.10.3 Configuring Layout in the Nib

### （未）1.10.4 View Debugging, Previewing, and Designing

### （未）1.10.5 Events Related to Layout


