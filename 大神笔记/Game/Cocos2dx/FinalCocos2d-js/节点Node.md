[toc]

## 节点 Node

`cc.Node`。 `cc.Sprite`、`cc.Layer` 的父类都是 `cc.Node`。

### 重要操作

创建节点：

```js
var childNode = new cc.Node();
```

添加子节点。第二个参数是Z轴，第三个参数是标签。

```js
node.addChild(childNode, 0, 123);
```

根据标签查找子节点：

```js
node.getChildByTag(123);
```

通过标签删除所有子节点，并停止该子节点上的一切动作：

```js
node.removeChildByTag(123, true);
```

删除 `childNode` 子节点，并停止该子节点上的一切动作：

```js
node.removeChild(childNode, true);
```

删除节点的所有子节点，并停止这些子节点的所有动作：

```js
node.removeAllChildrenWithCleanup(true);
```

从父节点删除自己，并停止所有该节点上的动作。

```js
node.removeFromParentAndCleanup(true);
```

### 重要属性

位置 `position` 和锚点 `anchorPoint`。

```js
node.setPosition(size.width/2, 0);
node.setAnchorPoint(cc.p(1.0, 1.0));
```

等效的3.0风格写法：

```js
node.x=size.width/2;
node.y=0;
node.anchorX = 1.0;
node.anchorY = 1.0;
```

### 定时器相关

下面的 `cc.Node` 一些与定时相关的方法。

调用 `node.scheduleUpdate();` 方法，则定时器会每帧调用节点的 `update(dt)` 方法。
调用 `node.unscheduleUpdate();` 方法，停止 `update(dt)` 方法的调度。

`node.schedule(callback, interval, repeat, delay);`，分别制定回调函数、时间间隔、重复次数和延迟时间。
如果想实现60帧的调度，`interval` 传 `1.0/60`。
`repeat` 传值 `cc.REPEAT_FOREVER` 表示无限循环。
时间单位为秒。

`node.unschedule(callback);` 取消制定回调函数。

`node.unscheduleAllCallbacks();` 取消节点的所有调度。

`node.scheduleOnce(callback, delay);` 延时后执行一次回调。

### 生命周期

一些生命周期函数：

- `onEnter`
- `onEnterTransitionDidFinish`：进入且过度动画结束
- `onExit`
- `onExitTransitionDidStart`：退出且开始动画刚开始

这些生命周期函数对于节点的子类，特别是场景、层、精灵来说非常重要。


