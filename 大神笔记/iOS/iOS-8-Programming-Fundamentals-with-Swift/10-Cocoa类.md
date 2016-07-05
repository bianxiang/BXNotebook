[toc]

## 10 Cocoa 类

Cocoa is a big framework, subdivided into many smaller frameworks, and it takes time and experience to become reasonably familiar with it.

Cocoa API 多数是 Objective-C。Cocoa 自身包含大量 Objective-C 的类，它们都是根类 `NSObject` 的子类。

本章介绍 Cocoa 类的结构。It discusses how Cocoa is conceptually organized, in terms of its underlying Objective-C features, 然后调研一些最常遇到的 Cocoa 工具类，concluding with a discussion of the Cocoa root class and its features, which are inherited by all Cocoa classes.

### 10.1 子类

例如，加入你想在界面上防止一条水平线。但 Cocoa 目前没有此控件。你得自己创建 `UIView` 的子类。

1、In our Empty Window example project, choose File → New → File and specify iOS → Source → Cocoa Touch Class, and in particular a subclass of `UIView`. Call the class `MyHorizLine`. Xcode creates MyHorizLine.swift. Make sure it’s part of the app target.
2、In MyHorizLine.swift, replace the contents of the class declaration with this (without further explanation):

```
	required init(coder aDecoder: NSCoder) {
		super.init(coder:aDecoder)
		self.backgroundColor = UIColor.clearColor()
	}
    override func drawRect(rect: CGRect) {
        let c = UIGraphicsGetCurrentContext()
        CGContextMoveToPoint(c, 0, 0)
        CGContextAddLineToPoint(c, self.bounds.size.width, 0)
        CGContextStrokePath(c)
    }
```

3、Edit the storyboard. Find UIView in the Object library (it is called simply “View”), and drag it into the View object in the canvas. You may resize it to be less tall.

4、With the UIView that you just dragged into the canvas still selected, use the Identity inspector to change its class to `MyHorizLine`.

构建运行测试。

上面的例子中，我们没有调用 super；因为 UIView 的 `drawRect:` 什么也没做。但有时你可能要创建 UIView 内建的子类的子类，修改其默认的绘制。例如 `UILabel` 文档里提到两个方法 `drawTextInRect:` 和 `textRectForBounds:limitedToNumberOfLines:`，这两个方法不是给我们调用的，而是在绘制自己时由 Cocoa 调用的；thus, we can subclass UILabel and implement these methods in our subclass to modify how a particular type of label draws itself.

Here’s an example from one of my own apps, in which I subclass UILabel to make a label that draws its own rectangular border and has its content inset somewhat from that border, by overriding `drawTextInRect:`. As the documentation tells us: “In your overridden method, you can configure the current context further and then invoke super to do the actual drawing [of the text].” Let’s try it:

1、In the Empty Window project, make a new class file, a UILabel subclass; call the class `MyBoundedLabel`.

2、In MyBoundedLabel.swift, insert this code into the body of the class declaration:

```
override func drawTextInRect(rect: CGRect) {
    let context = UIGraphicsGetCurrentContext()
    CGContextStrokeRect(context, CGRectInset(self.bounds, 1.0, 1.0))
    super.drawTextInRect(CGRectInset(rect, 5.0, 5.0))
}
```

3、Edit the storyboard, add a UILabel to the interface, and change its class in the Identity inspector to `MyBoundedLabel`.

实际上，除非显式被要求，否则不要创建 Cocoa 类的子类。有些类不被允许（文档中禁止）。

很少用到子类的一个原因是很多内建类利用代理（第11章）来定制他们的行为。例如，我们不会创建 UIApplication 的子类，仅为了响应“应用完成启动”事件，因为我们可以用代理 `(application:didFinishLaunchingWithOptions:)`。因此，模板一般会自动为我们创建一个 `AppDelegate` 类，它不是 `UIApplication` 的子类，而是 `UIApplicationDelegate` 协议的子类。

但如果你要定制应用基础的事件消息行为，就需要创建 UIApplication 的子类，覆盖 `sendEvent:` 方法。The documentation has a special “Subclassing Notes” section that tells you this, and also tells you, rightly, that doing so would be “very rare.” (See Chapter 6 on how to ensure that your UIApplication subclass is instantiated as the app launches.)

### 10.2 Categories 和 Extensions

A category is an Objective-C language feature that allows code to reach right into an existing class and inject additional methods. 它对应于 Swift 的扩展。Swift 头自己大量使用了扩展，不仅用于组织 Swift 自己的代码，也用于修改 Cocoa 类。In the same way, Cocoa uses categories to organize its own classes.

> Objective-C categories have names, and you may see references to these names in the headers, the documentation, and so forth. However, the names are effectively meaningless, so don’t worry about them.

#### Swift 如何使用扩展

查看 Swift 头你会发现很多 native 对象类型声明，包含一个初始声明和一组扩展。例如，在声明了泛型结构 `Array<T>`，后面紧跟着多个扩展。其中部分采纳了协议。

#### 你如何使用扩展

扩展可以对Swift三中对象类型使用，但子类只能用于类类型。

Also, as I explain there, I often use extensions in the same way as the Swift headers do, organizing my code for a single object type into multiple extensions simply for clarity.

#### How Cocoa Uses Categories

Cocoa uses categories as an organizational tool very much as Swift uses extensions. The declaration of a class will often be divided by functionality into multiple categories, and these will often appear in separate header files.

A good example is `NSString`. NSString is defined as part of the Foundation framework, and its basic methods are declared in `NSString.h`. Here we find that NSString itself, aside from its initializers, has just two methods, `length` and `characterAtIndex:`, because these are regarded as the minimum functionality that a string needs in order to be a string.
Additional NSString methods — those that create a string, deal with a string’s encoding, split a string, search in a string, and so on — are clumped into categories. These are shown in the Swift translation of the header as extensions. So, for example, after the declaration for the String class itself, we find this in the Swift translation of the header:

```
extension NSString {
    func getCharacters(buffer: UnsafeMutablePointer<unichar>, range aRange: NSRange)
    func substringFromIndex(from: Int) -> String
    	// ...
    }
```

That, as it turns out, is actually Swift’s translation of this Objective-C code:

```
@interface NSString (NSStringExtensionMethods)
- (void)getCharacters:(unichar *)buffer range:(NSRange)aRange;
- (NSString *)substringFromIndex:(NSUInteger)from;
// ...
@end
```

That notation — the keyword `@interface`, followed by a class name, followed by another name in parentheses — is an Objective-C **category**. Moreover, although the declarations for some of Cocoa’s `NSString` categories appear in this same file, NSString.h, many of them appear elsewhere. For example:

- A string may serve as a file pathname, so we also find a category on NSString in `NSPathUtilities.h`, where methods and properties such as `pathComponents` are declared for splitting a pathname string into its constituents and the like.
- In `NSURL.h`, which is primarily devoted to declaring the NSURL class (and its categories), there’s also another NSString category, declaring methods for dealing with percent-escaping in a URL string, such as `stringByAddingPercentEscapesUsingEncoding`.
- Off in a completely different framework (UIKit), `NSStringDrawing.h` adds two further NSString categories, with methods like `drawAtPoint:` having to do with drawing a string in a graphics context.

### 10.3 协议

Objective-C 也有协议。因为 Objective-C 对象类型只有类，因此所有 Objective-C 协议对 Swift 都是类协议。相反，Swift 中只有标记 `@objc` 的协议可以被 Objective-C 使用。

Cocoa 大量使用协议。

#### Informal Protocols

An informal protocol isn’t really a protocol at all; it’s just a way of providing the compiler with a method signature so that it will allow a message to be sent without complaining.

There are two complementary ways to implement an informal protocol. One is to define a category on `NSObject`; this makes any object eligible to receive the messages listed in the category. The other is to define a protocol to which no class formally conforms; instead, messages listed in the protocol are sent only to objects typed as `id` (AnyObject), thus suppressing any possible objections from the compiler.

These techniques were widespread before protocols could declare methods as optional; 现在已经基本不需要了。In iOS 8, very few informal protocols remain — but they do exist. For example, `NSKeyValueCoding` (discussed later in this chapter) is an informal protocol; you may see the term `NSKeyValueCoding` in the documentation and elsewhere, but there isn’t actually any NSKeyValueCoding type: it’s a **category** on NSObject.

#### Optional Methods

Objective-C 协议和 Swift 的 `@objc` 协议，可以有可选成员。

The answer is that Objective-C is both dynamic and introspective. Objective-C 能够询问对象是否能处理某个消息。关键方法是 `NSObject` 的 `respondsToSelector:` 方法，which takes a selector parameter and returns a Bool. (A selector is basically a method name expressed independently of any method call; see Appendix A.)

Demonstrating `respondsToSelector:` in Swift is generally a little tricky, because it’s hard to get Swift to throw away its strict type checking long enough to let us send an object a message to which it might not respond. In this artificial example, I start by defining, at top level, two classes: one that derives from `NSObject`, because otherwise we can’t send `respondsToSelector:` to it, and another to declare the message that I want to send conditionally:

```
class MyClass : NSObject {
}
class MyOtherClass {
	@objc func woohoo() {}
}
```

Now I can say this:

```
let mc = MyClass()
if mc.respondsToSelector("woohoo") {
	(mc as AnyObject).woohoo()
}
```

Note the cast of `mc` to `AnyObject`. This causes Swift to abandon its strict type checking (see Suppressing type checking); we can now send this object any message that Swift knows about, provided it is susceptible to Objective-C introspection — that’s why I marked my woohoo declaration as `@objc` to start with. As you know, Swift provides a
shorthand for sending a message conditionally — append a question mark to the name of the message:

```
let mc = MyClass()
(mc as AnyObject).woohoo?()
```

两种方法完全等价；后者是前者的语法糖。使用`?`，Swift 会帮我们调用 `respondsToSelector:`。

That explains also how optional protocol members work. It is no coincidence that Swift treats optional protocol members like AnyObject members. Here’s the example I gave in Chapter 4:

```
@objc protocol Flier {
    optional var song : String {get}
    optional func sing()
}
```

When you call `sing?()` on an object typed as a Flier, `respondsToSelector:` is called behind the scenes to determine whether this call is safe. You wouldn’t want to send a message optionally, or call respondsToSelector: explicitly, before sending just any old message, because it isn’t generally necessary except with optional methods, and it slows things down a little. But Cocoa does in fact call `respondsToSelector:` on your objects as a matter of course. To see that this is true, implement `respondsToSelector:` on AppDelegate in our Empty Window project and instrument it with logging:

```
override func respondsToSelector(aSelector: Selector) -> Bool {
    println(aSelector)
    return super.respondsToSelector(aSelector)
}
```

The output on my machine, as the Empty Window app launches, includes the following (I’m omitting private methods and multiple calls to the same method):

    application:handleOpenURL:
    application:openURL:sourceApplication:annotation:
    applicationDidReceiveMemoryWarning:
    applicationWillTerminate:
    applicationSignificantTimeChange:
    application:willChangeStatusBarOrientation:duration:
    application:didChangeStatusBarOrientation:
    application:willChangeStatusBarFrame:
    application:didChangeStatusBarFrame:
    application:deviceAccelerated:
    application:deviceChangedOrientation:
    applicationDidBecomeActive:
    applicationWillResignActive:
    applicationDidEnterBackground:
    applicationWillEnterForeground:
    applicationWillSuspend:
    application:didResumeWithOptions:
    application:handleWatchKitExtensionRequest:reply:
    application:shouldSaveApplicationState:
    application:supportedInterfaceOrientationsForWindow:
    application:performFetchWithCompletionHandler:
    application:didReceiveRemoteNotification:fetchCompletionHandler:
    application:willFinishLaunchingWithOptions:
    application:didFinishLaunchingWithOptions:

That’s Cocoa, checking to see which of the optional `UIApplicationDelegate` protocol methods (including a couple of undocumented methods) are actually implemented by our `AppDelegate` instance — which, because it is the `UIApplication` object’s delegate and formally conforms to the `UIApplicationDelegate` protocol, has explicitly agreed that it might be willing to respond to any of those messages. The entire delegate pattern (Chapter 11) depends upon this technique. Observe the policy followed here by Cocoa: it checks all the optional protocol methods once, when it first meets the object in question, and presumably stores the results; thus, the app is slowed a tiny bit by this one-time initial bombardment of `respondsToSelector:` calls, but now Cocoa knows all the answers and won’t have to perform any of these same checks on the same object later.

### 10.4 一些 Foundation 类

The Foundation classes of Cocoa provide basic data types and utilities that will form the basis of your communication with Cocoa. For more information, start with Apple’s list of the Foundation classes in the Foundation Framework Reference.

#### 常用结构和常量

NSRange 是一个 C 结构。它有两个整数字段，`location` 和 `length`。
Cocoa 提供一些便利方法处理 `NSRange`。`NSMakeRange` 通过两个整数创建 `NSRange`。Swift 将 `NSRange` 桥接为一个 Swift 结构。二者可以互转（当Range的位置类型是 `Int`）：Swift 向 `NSRange` 添加了一个初始化器，取一个 Swift 的 `Range`；一个 `toRange` 方法。

`NSNotFound` 是一个常量整数，表示目标未找到。如：

```
let arr = ["hey"] as NSArray
let ix = arr.indexOfObject("ho")
if ix == NSNotFound {
	println("it wasn't found")
}
```

Swift 的全局函数 `find`，返回一个包裹 Int 的 Optional，则此时找不到返回 nil。

Swift 会做一些自动的桥接。如 NSString 的 `rangeOfString:` 方法，在 Cocoa 中返回一个 `NSRange`；Swift 重新配置它返回一个 Swift `Range` (of String.Index)，包裹在 Optional；若找不到返回 nil（当 NSRange 的 `location` 是 `NSNotFound`）：

```
let s = "howdy"
let r = s.rangeOfString("ha") // nil; an Optional wrapping a Swift Range
```

若你想得到的就是 `NSRange`，则应该先把 Swift 的 String 强转为 `NSString`：

```
let s = "howdy" as NSString
let r = s.rangeOfString("ha") // an NSRange
if r.location == NSNotFound {
	println("it wasn't found")
}
```

#### NSString 及相关

NSString and Swift String are bridged to one another, passing a Swift String to Cocoa where an NSString is expected, calling Cocoa NSString methods on a Swift String, and so forth. For example:

```
let s = "hello"
let s2 = s.capitalizedString
```

In that code, s is a Swift String and s2 is a Swift String, but the `capitalizedString` property actually belongs to Cocoa. In the course of that code, a Swift String has been bridged to NSString and passed to Cocoa, which has processed it to get the capitalized string; the capitalized string is an `NSString`, but it has been bridged back to a Swift String.

In some cases, you will need to cross the bridge yourself by casting explicitly. The problem typically arises where you need to provide an index on a string. For example:

```
let s = "hello"
let s2 = s.substringToIndex(4) // compile error
```

The problem here is that the bridging is in your way. Swift has no objection to your calling the `substringToIndex:` method on a Swift String, but then the index value must be a `String.Index`, which is rather tricky to construct (see Chapter 3):

```
let s2 = s.substringToIndex(advance(s.startIndex, 4))
```

If you don’t want to talk that way, you must cast the String to an `NSString` beforehand; now you are in Cocoa’s world, where string indexes are integers:

```
let s2 = (s as NSString).substringToIndex(4)
```

As I explained in Chapter 3, however, those two calls are not equivalent: they can give different answers! The reason is that String and NSString have fundamentally different notions of what constitutes an element of a string (see The String–NSString Element Mismatch). A String is a Swift sequence, and must resolve its elements into characters, which means that it must walk the string, coalescing any combining codepoints; an NSString behaves as if it were an array of UTF16 codepoints. On the Swift side, each increment in a `String.Index` corresponds to a true character, but access by index or range requires walking the string; on the Cocoa side, access by index or range is extremely fast, but might not correspond to character boundaries. (See the “Characters and Grapheme Clusters” chapter of Apple’s String Programming Guide.)

Another important difference between a Swift String and a Cocoa NSString is that an `NSString` is immutable. This means that, with NSString, you can do things such as obtain a new string based on the first — as `capitalizedString` and `substringToIndex:` do — but you can’t change the string in place. To do that, you need another class, a subclass of
`NSString`, `NSMutableString`. `NSMutableString` has many useful methods, and you’ll probably want to take advantage of them; but Swift String isn’t bridged to `NSMutableString`, so you can’t get from String to `NSMutableString` merely by casting. To obtain an `NSMutableString`, you’ll have to make one. The simplest way is with NSMutableString’s initializer `init(string:)`, which expects an NSString — meaning that
you can pass a Swift String. 但反过来，`NSMutableString` 可以强转为 Swift `String`：

```
let s = "hello"
let ms = NSMutableString(string:s)
ms.deleteCharactersInRange(NSMakeRange(ms.length-1,1))
let s2 = (ms as String) + "ion" // now s2 is a Swift String
```

As I said in Chapter 3, native Swift String methods are thin on the ground. All the real string-processing power lives over on the Cocoa side of the bridge. So you’re going to be crossing that bridge a lot! And this will not be only for the power of the `NSString` and `NSMutableString` classes. Many other useful classes are associated with them.

For example, suppose you want to search a string for some substring. All the best ways come from Cocoa:

- An `NSString` can be searched using various `rangeOfString:...` methods, with numerous options such as ignoring diacriticals, ignoring case, starting at the end, and insisting that the substring occupy the start or end of the searched string.
- Perhaps you don’t know exactly what you’re looking for: you need to describe it structurally. `NSScanner` lets you walk through a string looking for pieces that fit certain criteria; for example, with NSScanner (and NSCharacterSet) you can skip past everything in a string that precedes a number and then extract the number.
- By specifying the option `.RegularExpressionSearch`, you can search using a regular expression. Regular expressions are also supported as a separate class, `NSRegularExpression`, which in turn uses `NSTextCheckingResult` to describe match results.
- More sophisticated automated textual analysis is supported by some additional classes, such as `NSDataDetector`, an `NSRegularExpression` subclass that efficiently finds certain types of string expression such as a URL or a phone number, and `NSLinguisticTagger`, which actually attempts to analyze text into its grammatical parts of speech.

In this example, our goal is to replace all occurrences of the word “hell” with the word “heaven.” We don’t want to replace mere occurrences of the substring “hell” — for example, “hello” should be left intact. Thus our search needs some intelligence as to what constitutes a word boundary. That sounds like a job for a regular expression. Swift doesn’t have regular expressions, so everything has to be done by Cocoa:

```
var s = NSMutableString(string:"hello world, go to hell")
let r = NSRegularExpression(pattern: "\\bhell\\b", options: .CaseInsensitive, error: nil)!
r.replaceMatchesInString(s, options: nil, range: NSMakeRange(0,s.length), withTemplate: "heaven")
```

NSString also has convenience utilities for working with a file path string, and is often used in conjunction with `NSURL`, which is another Foundation class worth looking into. In addition, NSString — like some other classes discussed in this section — provides methods for writing out to a file’s contents or reading in a file’s contents; the file can be
specified either as an NSString file path or as an NSURL.

An NSString carries no font and size information. Interface objects that display strings (such as `UILabel`) have a `font` property that is a `UIFont`; but this determines the single font and size in which the string will display. If you want styled text — where different runs of text have different style attributes (size, font, color, and so forth) — you need to use `NSAttributedString`, along with its supporting classes `NSMutableAttributedString`, `NSParagraphStyle`, and `NSMutableParagraphStyle`. These allow you to style text and paragraphs easily in sophisticated ways. The built-in interface objects that display text can display an `NSAttributedString`.

String drawing in a graphics context can be performed with methods provided through the `NSStringDrawing` category on `NSString` (see the String UIKit Additions Reference) and on `NSAttributedString` (see the NSAttributedString UIKit Additions Reference).

#### NSDate 及相关

An NSDate is a date and time, represented internally as a number of seconds (`NSTimeInterval`) since some reference date. Calling NSDate’s initializer `init()` — i.e., saying `NSDate()` — gives you a date object for the current date and time. Date operations involving anything simpler than a count of seconds will involve the use of `NSDateComponents`, and conversions between `NSDate` and `NSDateComponents` require that you pass through an `NSCalendar`. Here’s an example of constructing a date based on its calendrical values:

```
let greg = NSCalendar(calendarIdentifier:NSGregorianCalendar)!
let comp = NSDateComponents()
comp.year = 2015
comp.month = 8
comp.day = 10
comp.hour = 15
let d = greg.dateFromComponents(comp) // Optional wrapping NSDate
```

Similarly, `NSDateComponents` provides the correct way to do date arithmetic. Here’s how to add one month to a given date:

```
let d = NSDate() // or whatever
let comp = NSDateComponents()
comp.month = 1
let greg = NSCalendar(calendarIdentifier:NSGregorianCalendar)!
let d2 = greg.dateByAddingComponents(comp, toDate:d, options:nil)
```

You will also likely be concerned with dates represented as strings. If you don’t take explicit charge of a date’s string representation, it is represented by a string whose format may surprise you. For example, if you simply println an `NSDate`, you are shown the date in the GMT timezone, which can be confusing if that isn’t where you live. A simple solution is to call `descriptionWithLocale:`; the locale comprises the user’s current time zone, language, region format, and calendar settings:

```
println(d)
// 2015-02-02 16:36:41 +0000
println(d.descriptionWithLocale(NSLocale.currentLocale()))
// Optional("Monday, February 2, 2015 at 8:36:41 AM Pacific Standard Time")
```

For exact creation and parsing of date strings, use NSDateFormatter, which uses a format string similar to NSLog (and NSString’s stringWithFormat:). In this example, we surrender completely to the user’s locale by generating an NSDateFormatter’s format with dateFormatFromTemplate:options:locale: and the current locale. The “template” is a string listing the date components to be used, but their order, punctuation, and language are left up to the locale:

```
let df = NSDateFormatter()
let format = NSDateFormatter.dateFormatFromTemplate("dMMMMyyyyhmmaz", options:0, locale:NSLocale.currentLocale())
df.dateFormat = format
let s = df.stringFromDate(NSDate()) // just now
```

The result is the date shown in the user’s time zone and language, using the correct linguistic conventions. It involves a combination of region format and language, which are two separate settings. Thus:

On my device, the result might be “February 2, 2015, 8:36 AM PST.” If I change my device’s region to France, it might be “2 February 2015 8:36 am GMT-8.” If I also change my device’s language to French, it might be “2 février 2015 8:36 AM UTC−8.”

#### NSNumber

An NSNumber is an object that wraps a numeric value. The wrapped value can be any standard Objective-C numeric type (including BOOL, the Objective-C equivalent of Swift Bool). It comes as a surprise to Swift users that `NSNumber` is needed. Objective-C 中普通数字不是对象（是标量，见附录A），因此不能用于期望对象的地方。`NSNumber` 负责数字和对象的转换。

Swift does its best to shield you from having to deal directly with NSNumber. It bridges Swift numeric types to Objective-C in two different ways:

- 若期望一个普通数字，则Swift数字桥接到一个普通数字（标量）。
- 若期望一个对象，Swift数字桥接到 NSNumber。

例子：

```
let ud = NSUserDefaults.standardUserDefaults()
ud.setInteger(0, forKey: "Score")
ud.setObject(0, forKey: "Score")
```

The second and third lines look alike, but Swift treats the literal 0 differently: In the second line, `setInteger:forKey:` expects an integer (a scalar) as its first parameter, so Swift turns the Int struct value 0 into an ordinary Objective-C number. In the third line, `setObject:forKey:` expects an object as its first parameter, so Swift turns the Int struct value 0 into an NSNumber.

Naturally, if you need to cross the bridge explicitly, you can. You can cast a Swift number to an NSNumber: `let n = 0 as NSNumber`

For more control over what numeric type an NSNumber will wrap, you can call one of NSNumber’s initializers: `let n = NSNumber(float:0)`

Coming back from Objective-C to Swift, a value will typically arrive as an `AnyObject` and you will have to cast down. NSNumber comes with properties for accessing the wrapped value by its numeric type. Recall this example from Chapter 5, involving an NSNumber extracted as a value from an NSNotification’s userInfo dictionary:

```
if let ui = n.userInfo {
    if let prog = ui["progress"] as? NSNumber {
    	self.progress = prog.doubleValue
    }
}
```

If an AnyObject is actually an NSNumber, you can cast all the way down to a Swift numeric type. Thus, the same example can be rewritten like this, without explicitly mentioning NSNumber:

```
if let ui = n.userInfo {
    if let prog = ui["progress"] as? Double {
    	self.progress = prog
    }
}
```

An NSNumber object is just a wrapper and no more. 它不能直接参与数值计算；它不是一个数字。它包裹一个数字。若需要数字，要从 NSNumber 中抽出。

An `NSNumber` subclass, `NSDecimalNumber`, on the other hand, can be used in calculations, thanks to a bunch of arithmetic methods:

```
let dec1 = NSDecimalNumber(float: 4.0)
let dec2 = NSDecimalNumber(float: 5.0)
let sum = dec1.decimalNumberByAdding(dec2) // 9.0
```

`NSDecimalNumber` is useful particularly for rounding, because there’s a handy way to specify the desired rounding behavior. Underlying `NSDecimalNumber` is the `NSDecimal` struct (it is an `NSDecimalNumber`’s `decimalValue`). `NSDecimal` comes with C functions that are faster than `NSDecimalNumber` methods.

#### NSValue

`NSValue` 是 `NSNumber` 的父类。用于包裹非数值的 C 值，如C的结构。

Convenience methods provided through the NSValueUIGeometryExtensions category on `NSValue` (see the NSValue UIKit Additions Reference) allow easy wrapping and unwrapping of CGPoint, CGSize, CGRect, CGAffineTransform, UIEdgeInsets, and UIOffset; additional categories allow easy wrapping and unwrapping of NSRange, CATransform3D, CMTime, CMTimeMapping, CMTimeRange, MKCoordinate, and MKCoordinateSpan. You are unlikely to need to store any other kind of C value in an NSValue, but you can if you need to.

Swift will not magically bridge any of these C struct types to or from an NSValue. You must manage them explicitly, exactly as you would do if your code were written in Objective-C. In this example from my own code, we use Core Animation to animate the movement of a button in the interface from one position to another; the button’s starting and ending positions are each expressed as a CGPoint, but an animation’s fromValue and toValue must be objects. `CGPoint` 不是 Objective-C 对象，因此必须被包裹进 `NSValue`：

```
let ba = CABasicAnimation(keyPath:"position")
ba.duration = 10
ba.fromValue = NSValue(CGPoint:self.oldButtonCenter)
ba.toValue = NSValue(CGPoint:goal)
self.button.layer.addAnimation(ba, forKey:nil)
```

Similarly, you can make an array of CGPoint in Swift, because CGPoint becomes a Swift object type (a Swift struct), and a Swift Array can have elements of any type; 但你不能将这样的数组传给 Objective-C，因为 Objective-C 的 NSArray 只能容纳对象。Thus you must wrap the CGPoints in NSValue objects first. This is another animation example, where I set the values array (an NSArray) of a keyframe animation by turning an array of CGPoints into an array of NSValues:

```
anim.values = [oldP,p1,p2,newP].map{NSValue(CGPoint:$0)}
```

#### NSData

NSData is a general sequence of bytes; basically, it’s just a buffer, a chunk of memory. It is immutable; the mutable version is its subclass `NSMutableData`.

In practice, NSData tends to arise in two main ways:

- When downloading data from the Internet. For example, `NSURLConnection` and `NSURLSession` supply whatever they retrieve from the Internet as NSData. Transforming it from there into (let’s say) a string, specifying the correct encoding, would then be up to you.
- When storing an object as a file or in user preferences. For example, you can’t store a UIColor value directly into user preferences. So if the user has made a color choice and you need to save it, you transform the UIColor into an NSData (using `NSKeyedArchiver`) and save that:

```
let ud = NSUserDefaults.standardUserDefaults()
let c = UIColor.blueColor()
let cdata = NSKeyedArchiver.archivedDataWithRootObject(c)
ud.setObject(cdata, forKey: "myColor")
```

#### 相等与比较

In Swift, the equality and comparison operators can be overridden for an object type that adopts Equatable and Comparable (Operators). 但 Objective-C 运算符无此功能，它们只能用于标量。

To permit determination of whether two objects are “equal” — whatever that may mean for this object type — an Objective-C class must implement `isEqual:`, which is inherited from `NSObject`. Swift will help out by treating `NSObject` as `Equatable` and by permitting the use of the `==` operator, implicitly converting it to an `isEqual:` call. Thus, if a class does implement `isEqual:`, ordinary Swift comparison will work. For example:

```
let n1 = NSNumber(integer:1)
let n2 = NSNumber(integer:2)
let n3 = NSNumber(integer:3)
let ok = n2 == 2 // true
let ok2 = n2 == NSNumber(integer:2) // true
let ix = find([n1,n2,n3], 2) // Optional wrapping 1
```

There are two parts to this apparent magic:

- The numbers are being wrapped in NSNumber objects for us.
- The `==` operator (also used behind the scenes by the `find` function) is being converted to an `isEqual:` call.

NSNumber implements `isEqual:` to compare two NSNumber objects by comparing the numeric values that they wrap; therefore the equality comparisons all work correctly.

If an NSObject subclass doesn’t implement `isEqual:`, it inherits NSObject’s implementation, which compares the two objects for identity (like Swift’s `===` operator). For example, these two Dog objects can be compared with the `==` operator, even though Dog does not adopt Equatable, because they derive from NSObject — but Dog doesn’t implement isEqual:, so == defaults to NSObject’s identity comparison:

```
class Dog : NSObject {
    var name : String
    init(_ name:String) {self.name = name}
}
let d1 = Dog("Fido")
let d2 = Dog("Fido")
let ok = d1 == d2 // false
```

A number of classes that implement `isEqual:` also implement more specific and efficient tests. The usual Objective-C way to determine whether two NSNumber objects are equal (in the sense of wrapping identical numbers) is by calling `isEqualToNumber:`. Similarly, NSString has `isEqualToString:`, NSDate has `isEqualToDate:`, and so forth. However, these classes do also implement isEqual:, so I don’t think there’s any reason not to use the Swift `==` operator.

Similarly, in Objective-C it is up to individual classes to supply ordered comparison methods. The standard method is called `compare:`, and returns one of three `NSComparisonResult` cases:

- `.OrderedAscending`：The receiver is less than the argument.
- `.OrderedSame`：The receiver is equal to the argument.
- `.OrderedDescending`：The receiver is greater than the argument.

Swift comparison operators (< and so forth) do not magically call `compare:` for you. You can’t compare two NSNumber values directly:

```
let n1 = NSNumber(integer:1)
let n2 = NSNumber(integer:2)
let ok = n1 < n2 // compile error
```

You will typically fall back on calling `compare:` yourself, exactly as in Objective-C:

```
let n1 = NSNumber(integer:1)
let n2 = NSNumber(integer:2)
let ok = n1.compare(n2) == .OrderedAscending // true
```

#### NSIndexSet

NSIndexSet represents a collection of unique whole numbers; its purpose is to express element numbers of an **ordered** collection, such as an `NSArray`. Thus, for instance, to retrieve multiple objects simultaneously from an array, you specify the desired indexes as an `NSIndexSet`. It is also used with other things that are array-like; for example, you pass an `NSIndexSet` to a `UITableView` to indicate what sections to insert or delete.

To take a specific example, let’s say you want to speak of elements 1, 2, 3, 4, 8, 9, and 10 of an `NSArray`. `NSIndexSet` expresses this notion in some compact implementation that can be readily queried. The actual implementation is opaque, but you can imagine that this `NSIndexSet` might consist of two `NSRange` structs, {1,4} and {8,3}, and `NSIndexSet`’s methods actually invite you to think of an NSIndexSet as composed of ranges.

An `NSIndexSet` is immutable; its mutable subclass is `NSMutableIndexSet`. You can form a simple `NSIndexSet` consisting of just one contiguous range directly, by passing an NSRange to `indexSetWithIndexesInRange:`; but to form a more complex index set you’ll need to use `NSMutableIndexSet` so that you can append additional ranges:

```
var arr = ["zero", "one", "two", "three", "four", "five"]
arr.extend(["six", "seven", "eight", "nine", "ten"])
let ixs = NSMutableIndexSet()
ixs.addIndexesInRange(NSRange(1…4))
ixs.addIndexesInRange(NSRange(8…10))
let arr2 = (arr as NSArray).objectsAtIndexes(ixs)
```

To walk through (enumerate) the index values specified by an NSIndexSet, you can use for…in; alternatively, you can walk through an `NSIndexSet`’s indexes or ranges by calling `enumerateIndexesUsingBlock:` or `enumerateRangesUsingBlock:` or their variants.

#### NSArray 和 NSMutableArray

NSArray 与 Swift Array 相互桥接。一些限制：

- NSArray 元素只能是 Objective-C 对象。即只能是类实例，或桥接到 Objective-C 类类型的 Swift 类型的实例。
- NSArray 不维护元素的类型信息。与 Swift 不同的是，可以容纳不同类型的元素。因此在Swift 中收到的来自 Objective-C 的 NSArray 都是 `[AnyObject]`。

For a full discussion of how to bridge back and forth between Swift Array and Objective-C NSArray, implicitly and by casting, see Swift Array and Objective-C NSArray.

An NSArray’s length is its `count`, and a particular object can be obtained by index number using `objectAtIndex:`.

因为 NSArray 实现了 `objectAtIndexedSubscript:`，因此可以通过下标访问。该方法是 Objective-C 与 Swift 下标 getter 的等价。In fact, by a kind of trickery, when you examine the NSArray header file translated into Swift, this method is shown as a subscript declaration! Thus, the Objective-C version of the header file shows this declaration:

```
	- (id)objectAtIndexedSubscript:(NSUInteger)idx;
```

But the Swift version of the same header file shows this:

```
	subscript (idx: Int) -> AnyObject { get }
```

You can seek an object within an array with `indexOfObject:` or `indexOfObjectIdenticalTo:`; the former’s idea of equality is to call `isEqual:`, whereas the latter uses object identity (like Swift’s `===`). As I mentioned earlier, if the object is not found in the array, the result is `NSNotFound`.

与 Swift Array 不同的是，NSArray 是不可变的。（元素自身可以修改，但不能增删替换元素）。To do those things while staying in the Objective-C world, you
can derive a new array consisting of the original array plus or minus some objects, or use `NSArray`’s subclass, `NSMutableArray`. Swift Array is not bridged to NSMutableArray; if you want an NSMutableArray, you must create it. The simplest way is with the `NSMutableArray` initializers, `init()` or `init(array:)`.

Once you have an NSMutableArray, you can call methods such as NSMutableArray’s `addObject:` and `replaceObjectAtIndex:withObject:`. You can also assign into an `NSMutableArray` using subscripting. Again, this is because `NSMutableArray` implements a special method, `setObject:atIndexedSubscript:`; Swift recognizes this as equivalent
to a subscript setter. Coming back the other way, you cannot cast directly from `NSMutableArray` to a Swift Array of any type other than `[AnyObject]`; the usual approach is to cast up from `NSMutableArray` to `NSArray` and then down to a specific type of Swift Array:

```
let marr = NSMutableArray()
marr.addObject(1) // an NSNumber
marr.addObject(2) // an NSNumber
let arr = marr as NSArray as! [Int]
```

Cocoa provides ways to search or filter an array using a block. You can also derive a sorted version of an array, supplying the sorting rules in various ways, or if it’s a mutable array, you can sort it directly. You might prefer to perform those kinds of operation in the Swift Array world, but it can be useful to know how to do them the Cocoa way. For example:

```
let pep = ["Manny", "Moe", "Jack"] as NSArray
let ems = pep.objectsAtIndexes(
	pep.indexesOfObjectsPassingTest {
		obj, idx, stop in
		return (obj as! NSString).rangeOfString(
        	"m", options:.CaseInsensitiveSearch
        ).location == 0
	}
) // ["Manny", "Moe"]
```

#### NSDictionary 和 NSMutableDictionary

`NSDictionary` 与 Swift Dictionary 桥接。但有一些限制：

- NSDictionary keys will usually be an NSString, but they don’t have to be; the formal requirement is that they be Objective-C objects whose classes adopt `NSCopying`. NSDictionary uses hashing under the hood, but it does not formally require hashability, because `NSObject` itself is hashable. Therefore, the Swift equivalent is to see NSDictionary keys as a form of `NSObject`, which Swift extends to adopt Hashable.
- NSDictionary values must be Objective-C objects. NSDictionary has no information as to the type of its keys and values. Unlike Swift, different keys can be of different types, and different values can be of different types. Therefore, any NSDictionary arriving from Objective-C into Swift is typed as `[NSObject:AnyObject]`.

See Swift Dictionary and Objective-C NSDictionary for a full discussion of how to bridge back and forth between Swift Dictionary and Objective-C NSDictionary, including casting.

An NSDictionary is immutable; its mutable subclass is `NSMutableDictionary`. Swift Dictionary is not bridged to NSMutableDictionary; you can most easily make an NSMutableDictionary with an initializer, `init()` or `init(dictionary:)`.

The keys of an NSDictionary are distinct (using `isEqual:` for comparison). If you add a key–value pair to an `NSMutableDictionary`, then if that key is not already present, the pair is simply added, but if the key is already present, then the corresponding value is replaced. This is parallel to the behavior of Swift Dictionary.

The fundamental use of an NSDictionary is to request an entry’s value by key (using `objectForKey:`); if no such key exists, the result is `nil`. In Objective-C, nil is not an object, and thus cannot be a value in an `NSDictionary`; the meaning of this response is thus unambiguous. Swift handles this by treating the result of `objectForKey:` as an `AnyObject?` — that is, an Optional wrapping an AnyObject.

Subscripting is possible on an NSDictionary or an NSMutableDictionary for similar reasons to why subscripting is possible on an NSArray or an NSMutableArray. NSDictionary implements `objectForKeyedSubscript:`, and Swift understands this as equivalent to a subscript getter. In addition, NSMutableDictionary implements `setObject:forKeyedSubscript:`, and Swift understands this as equivalent to a subscript setter.

You can get from an NSDictionary a list of keys (allKeys), a list of values (allValues), or a list of keys sorted by value. You can also walk through the key–value pairs together using a block, and you can even filter an `NSDictionary` by a test against its values.

#### NSSet

NSSet 是独特对象的无序结合。独特意味着两个对象 `isEqual:` 不会返回true。可以通过  for…in 遍历。

`NSOrderedSet` 是有序集合。可以通过下标访问（因为它实现了 `objectAtIndexedSubscript:`）。

> Handing an array over to an ordered set uniques the array, meaning that order is maintained but only the first occurrence of an equal object is moved to the set.

An NSSet is immutable. You can derive one NSSet from another by adding or removing elements, or you can use its subclass, `NSMutableSet`. Similarly, `NSOrderedSet` has its mutable counterpart, `NSMutableOrderedSet` (which you can insert into by subscripting, because it implements `setObject:atIndexedSubscript:`).

`NSCountedSet`, a subclass of `NSMutableSet`, is a mutable unordered collection of objects that are not necessarily distinct (this concept is often referred to as a bag). It is implemented as a set plus a count of how many times each element has been added.

In Swift 1.2, `Set` is bridged to `NSSet`, with restrictions similar to `NSArray`. Nothing in Swift is bridged to `NSMutableSet`, `NSCountedSet`, `NSOrderedSet`, or `NSMutableOrderedSet`, but they are easily formed by coercion from a set or an array using an initializer. Coming back the other way, Swift receives an NSSet as a `Set<AnyObject>`; you can cast an NSSet down to a specific type of Swift Set, and thus you can cast an NSMutableSet or NSCountedSet up to NSSet and down to a Swift Set (similar to an NSMutableArray). NSOrderedSet comes with “façade” properties that present it as an array or a set. Because of their special behaviors, however, you are much more likely to leave an NSCountedSet or an NSOrderedSet in its Objective-C form for as long you’re working with it.

#### NSNull

The NSNull class does nothing but supply a pointer to a singleton object, `NSNull()`. This singleton object is used to stand for `nil` in situations where an actual Objective-C object is required and nil is not permitted. For example, you can’t use nil as the value of an element of a collection (such as NSArray, NSSet, or NSDictionary), and Objective-C knows nothing about Swift Optionals, so you’d use `NSNull()` instead.

You can test an object for equality against `NSNull()` using the ordinary equality operator (==), because it falls back on `NSObject`’s `isEqual:`, which is identity comparison. This is a singleton instance, and therefore identity comparison works.

#### 不可变与可变

(Swift doesn’t face the same issue, because its fundamental built-in object types such as String, Array, and Dictionary are structs, and therefore are value types, which cannot be mutated in place; they can be changed only by being replaced, and that is something that can be guarded against or detected through a setter observer.)

You can also use `copy` (produces an immutable copy) and `mutableCopy` (produces a mutable copy), both inherited from `NSObject`; but these are not as convenient because they yield an AnyObject which must then be cast.

These immutable/mutable class pairs are all implemented as class clusters, which means that Cocoa uses a secret class, different from the documented class you work with. You may discover this by peeking under the hood; for example, saying `NSStringFromClass(s.dynamicType)`, where s is an NSString, might yield a mysterious value `"__NSCFString"`. You should not spend any time wondering about this secret class. It is subject to change without notice and is none of your business; you should never have looked at it in the first place.

#### Property Lists

A property list is a string (XML) representation of data. The Foundation classes NSString, NSData, NSArray, and NSDictionary are the only classes that can be converted into a property list. Moreover, an NSArray or NSDictionary can be converted into a property list only if the only classes it collects are these classes, along with NSDate and NSNumber. (This is why, as I mentioned earlier, you must convert a UIColor into an NSData in order to store it in user defaults; the user defaults is a property list.)

属性列表的主要用途是存储数据到文件。是序列化值的一种方式。NSArray and NSDictionary provide convenience methods `writeToFile:atomically:` and `writeToURL:atomically:` that generate property list files given a pathname or file URL, respectively; conversely, they also provide initializers that create an NSArray object or an NSDictionary object based on the property list contents of a given file. For this very reason, you are likely to start with one of these classes when you want to create a property list. (NSString and NSData, with their methods writeToFile:... and writeToURL:..., just write the data out as a file directly, not as a property list.)

When you reconstruct an NSArray or NSDictionary object from a property list file in this way, the collections, string objects, and data objects in the collection are all immutable. If you want them to be mutable, or if you want to convert an instance of one of the other property list classes to a property list, you’ll use the NSPropertyListSerialization class (see the Property List Programming Guide).

### （未）10.5 访问器、属性、键值编码

Objective-C 实例变量与 Swift 实例属性基本一样。但前者一般是私有的，若想对外暴露，一般通过实现访问器方法：getter 和 setter。

getter方法名一般与实例变量名相同。如变量 `myVar` 的 getter 的方法名也为 `myVar`。

setter方法名以 `set` 开头，如变量 `myVar` 的 setter 名是 `setMyVar:`。

声明属性的语法糖 `@property`，如：

```
@property(nonatomic) CGRect frame;
```

上述声明会产生一个 `frame` 方法和一个 `setFrame:` 方法。

Objective-C 中通过 `@property` 方式声明的属性， Swift 会将其看做 Swift 的实例属性：`var frame: CGRect`。

Objective-C 中你可以直接调用 `setFrame:` 和 `frame` 方法。但在 Swift 中不能：若 Objective-C 声明使用 `@property`，访问器方法对 Swift 不可见。

Apple检查了所有 API，确保所有的访问器方法都有相应的 `@property` 声明，以使 Swift 可以将它们看做属性。但仍有一些访问器方法没有相应属性。如 `UIView` 的两个实例方法：

```
- (BOOL)translatesAutoresizingMaskIntoConstraints;
- (void)setTranslatesAutoresizingMaskIntoConstraints:(BOOL)flag;
```

It looks as if these are accessors for a `translatesAutoresizingMaskIntoConstraints` property; and in Objective-C, you can even pretend that there is such a property. But for some reason there is (as of this writing) no formal `@property` declaration for `translatesAutoresizingMaskIntoConstraints` — and so in Swift what you see are the two accessor methods and no property.

An Objective-C property declaration can include the word `readonly` in the parentheses. This indicates that there is a getter but no setter. So, for example (ignore the other material in the parentheses):

```
@property(nonatomic,readonly,retain) CALayer *layer;
```

Swift will reflect this restriction with `{get}` after the declaration, as if this were a computed read-only property; the compiler will not permit you to assign to such a property:

```
var layer: CALayer { get }
```

#### Swift 访问器

Just as Objective-C properties are actually a shorthand for accessor methods, so Objective-C treats Swift properties as a shorthand for accessor methods — even though no such methods are formally present. 若再 Swift 声明一个属性 `prop`，Objective-C 可以调用 `prop` 方法读取值，调用 `setProp:` 方法设置值。

Swift 中，不要显式书写属性的访问器方法！从 Swift 1.2 开始，编译器将禁止这样做。若需要显式实现访问器方法，使用计算属性。

You can even change the Objective-C names of these accessor methods! To do so, add an `@objc(...)` attribute with the Objective-C name in parentheses. For example:

```
var color : UIColor {
    @objc(hue) get {
        println("someone called the getter")
        return UIColor.redColor()
    }
    @objc(setHue:) set {
    	println("someone called the setter")
    }
}
```

If all you want to do is add functionality to the setter, use a property observer. 例如：

```
class MyView: UIView {
    override var frame : CGRect {
        didSet {
        	println("the frame setter was called: \(super.frame)")
        }
    }
}
```

#### 键值编程

Cocoa 可以动态调用访问器，在运行时以一个字符串访问 Swift 属性，此机制称为 key–value coding (KVC)。(This resembles, and is related to, the ability to use a selector name for introspection with `respondsToSelector:`.) The string name is the key; what is passed to or returned from the accessor is the value. 键值编程的基础是 `NSKeyValueCoding` 协议，一个非正式协议；它实际是注入 NSObject 的一个 category。A Swift class, to be susceptible to key–value coding, must therefore be derived from `NSObject`.

The fundamental key–value coding methods are `valueForKey:` and `setValue:forKey:`. When one of these methods is called on an object, the object is introspected. In simplified terms, first the appropriate **accessor** is sought; if it doesn’t exist, the instance variable is accessed directly. Another useful pair of methods is `dictionaryWithValuesForKeys:` and `setValuesForKeysWithDictionary:`, which allow you to get and set multiple key–value pairs by way of an `NSDictionary` with a single command.

The value in key–value coding must be an Objective-C object — that is, it is typed as `AnyObject`. When calling `valueForKey:`, you’ll receive an Optional wrapping an AnyObject, which you’ll want to cast down safely to its expected type.

A class is key–value coding compliant (or KVC compliant) on a given key if it provides the accessor methods, or possesses the instance variable, required for access via that key. An attempt to access a key for which a class is not key–value coding compliant will cause an exception at runtime. It is useful to be familiar with the message you’ll get when such a crash occurs, so let’s cause it deliberately:

```
let obj = NSObject()
obj.setValue("howdy", forKey:"keyName") // crash
```

The console says: “This class is not key value coding-compliant for the key keyName.” The last word in that error message, despite the lack of quotes, is the key string that caused the trouble. What would it take for that method call not to crash? The class of the object to which it is sent would need to have a setKeyName: setter method (or a keyName or _keyName instance variable). In Swift, as I demonstrated in the previous section, an instance property implies the existence of accessor methods. Thus, we can use key–value coding on an instance of any NSObject subclass that has a declared property, provided the key string is the string name of that property. Let’s try it! Here is such a class:

```
class Dog : NSObject {
	var name : String = ""
}
```

And here’s our test:

```
var d = Dog()
d.setValue("Fido", forKey:"name") // no crash!
println(d.name) // "Fido" - it worked!
```

UIViewController is derived from NSObject, so we can also use key–value coding to test our color accessor methods. Recall that I named them `setHue:` and `hue:`

```
var color : UIColor {
    @objc(hue) get {
    println("someone called the getter")
    	return UIColor.redColor()
    }
    @objc(setHue:) set {
    	println("someone called the setter")
    }
}
```

Now I have a way to call those accessors. Here, I’ll call the getter:

```
let c = self.valueForKey("hue") as? UIColor // "someone called the getter"
println(c) // Optional(UIDeviceRGBColorSpace 1 0 0 1)
```

#### Uses of Key–Value Coding

Key–value coding allows you, in effect, to decide at runtime, based on a string, what
accessor to call. In the simplest case, you’re using a string to access a dynamically
specified property. That’s useful in Objective-C code; but such unfettered introspective
dynamism is contrary to the spirit of Swift, and in translating my own Objective-C code
into Swift I have found myself accomplishing the same effect in other ways.
Here’s an example. In a flashcard app, I have a class Term, representing a Latin word. It
declares many properties. Each card displays one term, with its various properties shown
in different text fields. If the user taps any of three text fields, I want the interface to


### （未）10.6 The Secret Life of NSObject