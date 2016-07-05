[toc]

## 31 实现页面（UIPageViewController）

UIPageViewController 实现翻页效果。

`UIPageViewController` 类突出了视图控制器与容器控制器的区别。视图控制器负责管理UI视图（一般以故事板或XIB中的视图层级的形式）。容器控制器，用于管理多个视图控制器，一般提供视图间切换的机制。

UIPageViewController 类使得用户可以通过翻页手势在多个视图间导航。每一页实际是一个视图控制器，根据数据源按需创建。

**UIPageViewController DataSource**

数据源要实现 `UIPageViewControllerDataSource` 协议，至少实现以下两个方法：

- `viewControllerAfterViewController` - This method is passed a view controller representing the currently displayed page and is required to return the view controller corresponding to the next page in the paging sequence.
- `viewControllerBeforeViewController` - This method is passed the view controller representing the currently displayed page and is required to return the view controller corresponding to the previous page in the paging sequence.

一些配置选项控制 `UIPageViewController` 的外观和行为：

可以水平或垂直翻页。例如，垂直翻页像翻挂历。These options are configured using the following constants:

- UIPageViewControllerNavigationOrientation.Horizontal
- UIPageViewControllerNavigationOrientation.Vertical

配置书脊的位置。取决于翻页方向。一般取 `UIPageViewControllerSpineLocation.Min`，表示书脊在页面左边或上面。Similarly, the `UIPageViewControllerSpineLocation.Max` setting will position the spine at the right or bottom edge of the display. 若要同时显示两页，要使用  `UIPageViewControllerSpineLocationMid`。

The view controller may also be configured to treat pages as being double sided via the `doubleSided` property. Note that when using `UIPageViewControllerSpineLocationMid` spine location it will be necessary to provide the page controller with two view controllers (one for the left hand page and one for the right) for each page turn. Similarly, when using either the min or max spine location together with the double sided setting, view controllers for both the front and back of the current page will be required for each page.

**The UIPageViewController Delegate Protocol**

In addition to a data source, instances of the `UIPageViewController` class may also be assigned a delegate which, in turn, may implement the following delegate methods:

- spineLocationForInterface - The purpose of this delegate method is to allow the spine location to be changed in the event that the device is rotated by the user. An application might, for example, switch to `UIPageViewControllerSpineLocationMid` layout when the device is placed in a landscape orientation. The method is passed the new orientation and must return a corresponding spine location value. Before doing so it may, for example, also set up two view controllers if a switch is being made to a UIPageViewControllerSpineLocationMid spine location.
- `transitionComplete` - This method is called after the user initiates a page transition via a screen based gesture. The success or otherwise of the transition may be identified through the implementation of a completion handler.

## 32. UIPageViewController 应用的例子

创建新工程。但模板**不**选 Page-based Application。它是一个12月日历。它复杂。

我们仍用 Single View Application 模板。产品名 PageApp。

示例应用用 UIWebView 显示一些HTML。先创建**内容的**视图控制器。新建文件，父类 `UIViewController`，类名 `ContentViewController`。不要XIB。

Select the ContentViewController.swift file from the project navigator panel and add a reference to the data object:

```
import UIKit
class ContentViewController: UIViewController {
    var dataObject: AnyObject?
    ...
```

Next, select the Main.storyboard file and drag and drop a new View Controller object from the Object Library to the storyboard canvas. Display the Identity Inspector and change the Class setting to `ContentViewController`. In the Identity section beneath the Class setting, specify a Storyboard ID of `contentView`.

Drag and drop a Web View object from the Object Library to the `ContentViewController` view in the storyboard canvas and size and position it so that it fills the entire view. With the Web View object selected in the canvas, use the Auto Layout Pin menu to configure Spacing to nearest neighbor constraints on all four sides of the view with the Constrain to margins option switched off.

Select the Web View object in the storyboard panel, display the Assistant Editor panel and verify that the editor is displaying the contents of the ContentViewController.swift file. Ctrl-click on the web view object and drag to a position just below the class declaration line in the Assistant Editor. Release the line and in the resulting connection dialog establish an outlet connection named `webView`.

With the user interface designed, select the ContentViewController.swift file. Each time the user turns a page in the application, the data source methods for a UIPageViewController object are going to create a new instance of our `ContentViewController` class and set the `dataObject` property of that instance to the HTML that is to be displayed on the web view object. As such, the `viewWillAppear` method of ContentViewController needs to assign the value stored in the `dataObject` property to the web view object. To achieve this behavior, add the `viewWillAppear` method to assign the HTML to the web view:

```
import UIKit
class ContentViewController: UIViewController {
    var dataObject: AnyObject?
    @IBOutlet weak var webView: UIWebView!
    ...
    override func viewWillAppear(animated: Bool) {
        super.viewWillAppear(animated)
        webView.loadHTMLString(dataObject as String, baseURL: NSURL(string: ""))
    }
    ...
```

At this point work on the content view controller is complete. The next step is to create the data model for the application.

The data model for the application is going to consist of an array object containing a number of string objects, each configured to contain slightly different HTML content. For the purposes of this example, the data source for the UIPageViewController instance will be the application’s ViewController class.

This class will, therefore, need references to an NSArray and a UIPageViewController object. It will also be necessary to declare this class as implementing the `UIPageViewControllerDataSource` and `UIPageViewControllerDelegate` protocols. Select the ViewController.swift file and add these references as follows:

```
import UIKit
class ViewController: UIViewController, UIPageViewControllerDataSource, UIPageViewControllerDelegate {
    var pageController: UIPageViewController?
    var pageContent = NSArray()
    ...
```

The final step in creating the model is to add a method to the ViewController.swift file to add the HTML strings to the array and then call that method from `viewDidLoad`:

```
import UIKit
class ViewController: UIViewController, UIPageViewControllerDataSource, UIPageViewControllerDelegate {
    ...
    override func viewDidLoad() {
        super.viewDidLoad()
        createContentPages()
    }
    func createContentPages() {
        var pageStrings = [String]()
        for i in 1...11 {
            let contentString = "<html><head></head><body><br><h1>Chapter \(i)</h1>
            <p>This is the page \(i) of content displayed using UIPageViewController in iOS 8.
            </p></body></html>"
            pageStrings.append(contentString)
        }
        pageContent = pageStrings
    }
    ...
```

The application now has a content view controller and a data model from which the content of each page will be extracted by the data source methods. The next logical step, therefore, is to implement those data source methods. As previously outlined in Implementing a Page based iOS 8 Application using UIPageViewController, instances of the UIPageViewController class need a data source. This takes the form of two methods, one of which is required to return the view controller to be displayed after the currently displayed view controller, and the other the view controller to be displayed before the current view controller. Since the ViewController class is going to act as the data source for the page view controller object, these two methods, together with two convenience methods (which we will borrow from the Xcode Page-based Application template) will need to be added to the ViewController.swift file. Begin by adding the two convenience functions:

```
func viewControllerAtIndex(index: Int) -> ContentViewController? {
    if (pageContent.count == 0) || (index >= pageContent.count) {
    	return nil
    }
    let storyBoard = UIStoryboard(name: "Main", bundle: NSBundle.mainBundle())
    let dataViewController = storyBoard.instantiateViewControllerWithIdentifier("contentView") as ContentViewController
    dataViewController.dataObject = pageContent[index]
    return dataViewController
}
func indexOfViewController(viewController: ContentViewController) -> Int {
    if let dataObject: AnyObject = viewController.dataObject {
	    return pageContent.indexOfObject(dataObject)
    } else {
    	return NSNotFound
    }
}
```

The `viewControllerAtIndex` method begins by checking to see if the page being requested is outside the bounds of available pages by checking if the index reference is zero (the user cannot page back beyond the first page) or greater than the number of items in the pageContent array. In the event that the index value is valid, a new instance of the `ContentViewController` class is created and the `dataObject` property set to the contents of the corresponding item in the `pageContent` array of HTML strings. Since the view controller is stored in the storyboard file, the following code is used to get a reference to the storyboard and to create a new `ContentViewController` instance:

```
let storyBoard = UIStoryboard(name: "Main", bundle: NSBundle.mainBundle())
let dataViewController = storyBoard.instantiateViewControllerWithIdentifier("contentView") as ContentViewController
```

The `indexOfViewController` method is passed a viewController object and is expected to return the index value of the controller. It does this by extracting the dataObject property of the view controller and finding the index of the matching element in the pageContent array. All that remains to be implemented as far as the data source is concerned are the two data source protocol methods which, in turn, make use of the two convenience methods to return the view controllers before and after the current view controller:

```
func pageViewController(pageViewController: UIPageViewController, viewControllerBeforeViewController viewController: UIViewController) ->
UIViewController? {
    var index = indexOfViewController(viewController as ContentViewController)
    if (index == 0) || (index == NSNotFound) {
    	return nil
    }
    index--
    return viewControllerAtIndex(index)
}
func pageViewController(pageViewController: UIPageViewController, viewControllerAfterViewController viewController: UIViewController) -> UIViewController? {
    var index = indexOfViewController(viewController as ContentViewController)
    if index == NSNotFound {
	    return nil
    }
    index++
    if index == pageContent.count {
    	return nil
    }
    return viewControllerAtIndex(index)
}
```

With the data source implemented, the next step is to create and initialize an instance of the UIPageViewController class.

**初始化UIPageViewController**

All that remains is to create the UIPageViewController instance and initialize it appropriately. Since this needs to be performed only once per application invocation a suitable location for this code is the viewDidLoad method of the ViewController class. Select the ViewController.swift file and modify the viewDidLoad method so that it reads as follows:

```
override func viewDidLoad() {
    super.viewDidLoad()
    createContentPages()
    pageController = UIPageViewController(transitionStyle: .PageCurl, navigationOrientation: .Horizontal, options: nil)
    pageController?.delegate = self
    pageController?.dataSource = self
    let startingViewController: ContentViewController = viewControllerAtIndex(0)!
    let viewControllers: NSArray = [startingViewController]
    pageController!.setViewControllers(viewControllers, direction: .Forward, animated: false, completion: nil)
    self.addChildViewController(pageController!)
    self.view.addSubview(self.pageController!.view)
    var pageViewRect = self.view.bounds
    pageController!.view.frame = pageViewRect
    pageController!.didMoveToParentViewController(self)
}
```

All the code for the application is now complete. Before compiling and running the application some time needs to be taken to deconstruct and analyze the code in the viewDidLoad method. After constructing the data model with the call to the createContentPage method an instance of the UIPageViewController class is created specifying page curling and horizontal navigation orientation. Since the current class is going to act as the datasource and delegate for the page controller this also needs to be configured:

```
pageController = UIPageViewController(transitionStyle: .PageCurl, navigationOrientation: .Horizontal, options: nil)
pageController?.delegate = self
pageController?.dataSource = self
```

Before the first page can be displayed a view controller must first be created. This can be achieved by calling our viewControllerAtIndex convenience method. Once a content view controller has been returned it needs to be assigned to an array object:

```
let startingViewController: ContentViewController = viewControllerAtIndex(0)!
let viewControllers: NSArray = [startingViewController]
```

Note that only one content view controller is needed because the page controller is configured to display only one, single sided page at a time. Had the page controller been configured for two pages (with a mid-location spine) or for double sided pages it would have been necessary to create two content view controllers at this point and assign both to the array.

With an array containing the content view controller ready, the array needs to be assigned to the view controller with the navigation direction set to forward mode:

```
pageController!.setViewControllers(viewControllers, direction: .Forward,
animated: false, completion: nil)
```

Finally, the standard steps need to be taken to add the page view controller to the current view. Code is also included to ensure that the pages fill the entire screen:

```
self.addChildViewController(self.pageController!)
self.view.addSubview(pageController!.view)
var pageViewRect = self.view.bounds
pageController!.view.frame = pageViewRect
pageController!.didMoveToParentViewController(self)
```
