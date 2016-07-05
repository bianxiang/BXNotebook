[toc]

# [iOS9] 5 触摸

`UIView` 是一个 `UIResponder` 的子类。`UIResponder` 还有其他子类，但它们都不可见。用户看到的是视图，用户触摸的也是视图。(The user actually sees layers, but a layer is not a `UIResponder` and is not involved with touches.)

触摸由一个 `UITouch` 对象表示。该对象包裹在一个 `UIEvent` 对象中发送给应用。由应用负责将其发送给合适的 `UIView`。

实际上多数情况下你不需要直接处理 `UIEvent` 和 `UITouch`。多数内建的视图会处理这些底层事件，在更高的层面调用你的代码。

## 5.1 触摸事件与试图

从屏幕上没有任何触摸开始，随着一个或多个指头的触摸，到下一次屏幕上再没有任何触摸，构成一个“多点触摸序列”。

在多点触摸的过程中，每当有手指发生变化，都会通知应用。每次通知是一个 `UIEvent`。一个多点触摸序列中发送的事件是同一个 `UIEvent` 对象。

每个 `UIEvent` 包含一个或多个 `UITouch` 对象。一个 `UITouch` 对应一个手指。在一个多点触摸序列中，一个手指被同一个 `UITouch` 对象表示，直到手指离开屏幕。

系统仅在手指变换时报告。对于一个给定的 `UITouch` 对象，只可能发生四件事，它们称为“触摸阶段”，通过 `phase` 属性表示（`UITouchPhase`）：

- `.Began`：手指第一次触摸屏幕；新创建一个 `UITouch`；该阶段是第一个阶段，只会发生一次。
- `.Moved`：手指在屏幕上移动。
- `.Stationary`：手指在屏幕上没有移动。当其他手指移动，导致需要触发 `UIEvent` 事件。而这根手指未移动时，这根手指也要上报，但阶段是未移动。
- `.Ended`：手指离开屏幕。只发生一次。`UITouch` 对象会被销毁。

实际上还有一个可能的阶段：

- `.Cancelled`：系统放弃一个多点触摸序列，因为它被其他事物中断，如用户点击了 Home 键或锁屏键；接入电话。gesture recognizer 也可能让触摸取消。

`UITouch` 对象创建时，应用决定哪个 `UIView` 与之关联，将其设为触摸对象的 `view` 属性。该视图与此触摸对象保持关联，直到手指离开屏幕。

一个 `UIEvent` 可能被发送到多个视图。若该视图与一个 `UITouch` 关联，则该 `UITouch` 属于这个 `UIEvent`。

若一个 `UIEvent` 与一个视图相关的所有 `UITouch` 都处于 `.Stationary` 阶段，该 `UIEvent` 不会发送给该 `UIView`。

## 5.2 接收事件

一个 `UIResponder`（`UIView`）有四个方法对应 `UITouch` 的四个阶段。通过这四个方法将一个 `UIEvent` 传递给视图：

- `touchesBegan:withEvent:`
- `touchesMoved:withEvent:`
- `touchesEnded:withEvent:`
- `touchesCancelled:withEvent:`

这些方法的参数包括：

- 相关的触摸：与视图关联的触摸，且阶段与相应方法名匹配。若有多个触摸，They arrive as a Set。若只有一个，若你只关心其中一个，可以通过 `first` 获取。
- `UIEvent` 对象。可以通过 `allTouches` 获取到所有触摸，包括不与当前视图关联或不处于方法指定阶段的触摸。You can call `touchesForView:` or `touchesForWindow:` to ask for the set of touches associated with a particular view or window.

`UITouch` 的属性和方法：

- `locationInView:`、 `previousLocationInView:`：当前和之前的触摸位置，坐标相对于给定视图。The view you’ll be interested in will often be `self` or `self.superview`; supply `nil` to get the location with respect to the window. The previous location will be of interest only if the phase is `.Moved`.
- `timestamp`：When the touch last changed. A touch is timestamped when it is created (`.Began`) and each time it moves (`.Moved`). There can be a delay between the occurrence of a physical touch and the delivery of the corresponding `UITouch`, so to learn about the timing of touches, consult the timestamp, not the clock.
- `tapCount`：If two touches are in roughly the same place in quick succession, and the first one is brief, the second one may be characterized as a repeat of the first. They are different touch objects, but the second will be assigned a `tapCount` one larger than the previous one. The default is 1, so if (for example) a touch’s tapCount is 3, then this is the third tap in quick succession in roughly the same spot.
- `view`：与触摸关联的视图。
- `majorRadius`, `majorRadiusTolerance`：Respectively, the radius of the touch (approximately half its size) and the uncertainty of that measurement, in points.

Here are some additional `UIEvent` properties:- `type`：This will be `UIEventType.Touches`. There are other event types, but you’re not going to receive any of them this way.
- `timestamp`：When the event occurred.

## 5.3 限制触摸

触摸事件可以在应用级别关闭：通过 `UIApplication` 的 `beginIgnoringInteractionEvents`。常见关闭原因是在做动画。This call should be balanced by `endIgnoringInteractionEvents`. Pairs can be nested, in which case interactivity won’t be restored until the **outermost** `endIgnoringInteractionEvents` has been reached.
`UIView` 的一些属性会限制触摸：

- `userInteractionEnabled`：若设为 false，则视图（与它的所有子视图），不会受到触摸事件。Touches on this view or one of its subviews “fall through” to a view behindit.
- `alpha`：If set to 0.0 (or extremely close to it), this view (along with its subviews) is excluded from receiving touches. Touches on this view or one of its subviews “fall through” to a view behind it.
- `hidden`：If set to true, this view (along with its subviews) is excluded from receiving touches.
- `multipleTouchEnabled`：If set to false, this view never receives more than one touch simultaneously; once it receives a touch, it doesn’t receive any other touches until that first touch has ended.
- `exclusiveTouch`：An exclusiveTouch view receives a touch only if no other views in the same window have touches associated with them; once an exclusiveTouch view has received a touch, then while that touch exists no other view in the same window receives any touches. (This is the only one of these properties that can’t be set in the nib editor.)

## 5.4 拦截触摸

Thanks to gesture recognizers (discussed later in this chapter), in most cases you won’t have to interpret touches at all; you’ll let a gesture recognizer do most of that work. Even so, it is beneficial to be conversant with the nature of touch interpretation; this will help you interact with a gesture recognizer, write your own gesture recognizer, or subclass an existing one. Furthermore, not every touch sequence can be codified through a gesture recognizer; sometimes, directly interpreting touches is the best approach.
To figure out what’s going on as touches are received by a view, your code must essentially function as a kind of state machine. You’ll receive various touches method calls, and your response will partly depend upon what happened previously, so you’ll have to record somehow, such as in instance properties, the information that you’ll need in order to decide what to do when the next touches method is called. Such an architecture can make writing and maintaining touch-analysis code quite tricky.

To illustrate the business of interpreting touches, we’ll start with a view that can be dragged with the user’s finger. For simplicity, I’ll assume that this view receives only a single touch at a time. (This assumption is easy to enforce by setting the view’s `multipleTouchEnabled` to false, which is the default.)The trick to making a view follow the user’s finger is to realize that a view is positioned by its center, which is in superview coordinates, but the user’s finger might not be at the center of the view. So at every stage of the drag we must change the view’s center by the change in the user’s finger position in superview coordinates:

```swift
override func touchesMoved(touches: Set<UITouch>, withEvent e: UIEvent?) {	let t = touches.first!	let loc = t.locationInView(self.superview)	let oldP = t.previousLocationInView(self.superview)	let deltaX = loc.x - oldP.x	let deltaY = loc.y - oldP.y	var c = self.center	c.x += deltaX	c.y += deltaY	self.center = c}
```

Next, let’s add a restriction that the view can be dragged only vertically or horizontally. All we have to do is hold one coordinate steady; but which coordinate? Everything seems to depend on what the user does initially. So we’ll do a one-time test the first time we receive `touchesMoved:withEvent:`. Now we’re maintaining two Bool state properties, `self.decided` and `self.horiz`:

```swift
override func touchesBegan(touches: Set<UITouch>, withEvent e: UIEvent?) {	self.decided = false}override func touchesMoved(touches: Set<UITouch>, withEvent e: UIEvent?) {	let t = touches.first!	if !self.decided {
		self.decided = true
		let then = t.previousLocationInView(self)
		let now = t.locationInView(self)
		let deltaX = fabs(then.x - now.x)
		let deltaY = fabs(then.y - now.y)
		self.horiz = deltaX >= deltaY
	}
	let loc = t.locationInView(self.superview)
	let oldP = t.previousLocationInView(self.superview)
	let deltaX = loc.x - oldP.x
	let deltaY = loc.y - oldP.y
	var c = self.center
	if self.horiz {
		c.x += deltaX
	} else {
		c.y += deltaY
	}
	self.center = c}
```

Another area in which manual touch handling can rapidly prove overwhelming is when it comes to distinguishing between different gestures that the user is to be permitted to perform on a view. Imagine, for example, a view that distinguishes between a finger tapping briefly and a finger remaining down for a longer time. We can’t know how long a tap is until it’s over, so we must wait until then before deciding; once again, this requires maintaining state in a property (`self.time`):

```swiftoverride func touchesBegan(touches: Set<UITouch>, withEvent e: UIEvent?) {	self.time = touches.first!.timestamp}override func touchesEnded(touches: Set<UITouch>, withEvent e: UIEvent?) {	let diff = event.timestamp - self.time	if (diff < 0.4) {		print("short")	} else {		print("long")	}}
```
Distributing our various tasks correctly is tricky. We know when we have a double tap as early as touchesBegan:withEvent:, but we respond to the double tap in touchesEnded:withEvent:. Therefore we use a property (self.single) to communicate between the two. We don’t start our delayed response to a single tap until touchesEnded:withEvent:, because what matters is the time between the taps as a whole, not between the starts of the taps:

```swiftoverride func touchesBegan(touches: Set<UITouch>, withEvent e: UIEvent?) {let ct = touches.first!.tapCountswitch ct {case 2:self.single = falsedefault: break}
}override func touchesEnded(touches: Set<UITouch>, withEvent e: UIEvent?) {let ct = touches.first!.tapCountswitch ct {case 1:self.single = truedelay(0.3) {if self.single { // no second tap intervenedprint("single tap")}}case 2:print("double tap")default: break}}
```