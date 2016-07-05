Android 3.0 (API level 11)开始，Android设备不再要求必须有Menu按钮。于是，Android apps应该从传统的六项菜单，转换到使用action bar。

尽管菜单项的设计改变了，定义一组操作和选项的语义仍基于[Menu](http://developer.android.com/reference/android/view/Menu.html) APIs。下面介绍如何常见三种基础的菜单：

- **Options menu和action bar**  
options menu是收集activity菜单项的主要集合。用于放置影响全局App的功能，如搜索、设置。  
在Android 2.3及之前的版本，可以利用Menu按钮呼出options menu。  
在Android 3.0及之后，items from the options menu are presented by the action bar as a combination of on-screen action items and overflow options.从Android 3.0开始，在某些设备上可能不存在菜单键。
- **上下文菜单和contextual action mode**  
用户长按一个元素后呼出的浮动菜单是context menu。  
在Android 3.0及之后，you should instead use the contextual action mode to enable actions on selected content. This mode displays action items that affect the selected content in a bar at the top of the screen and allows the user to select multiple items.
- **弹出（Popup）菜单**  
弹出菜单垂直显示一组项，锚定到调用菜单的View上。It's good for providing an overflow of actions that relate to specific content or to provide options for a second part of a command. Actions in a popup menu should not directly affect the corresponding content—that's what contextual actions are for. Rather, the popup menu is for extended actions that relate to regions of content in your activity.

## 在XML中定义菜单

利用XML创建菜单，可以为不同设备创建不同菜单。

菜单XML放入`res/menu/`目录。

- `<menu>` 定义一个菜单，作为XML文件的根节点，容纳`<item>`和`<group>`。
- `<item>` 创建一个`MenuItem`，表示一个菜单项。可以包含嵌套的`<menu>`以创建子菜单。
- `<group>` 可选的、不可见的`<item>`容器。It allows you to categorize menu items so they share properties such as active state and visibility.

例子：`game_menu.xml`

	<?xml version="1.0" encoding="utf-8"?>
	<menu xmlns:android="http://schemas.android.com/apk/res/android">
	    <item android:id="@+id/new_game"
	          android:icon="@drawable/ic_new_game"
	          android:title="@string/new_game"
	          android:showAsAction="ifRoom"/>
	    <item android:id="@+id/help"
	          android:icon="@drawable/ic_help"
	          android:title="@string/help" />
	</menu>

`<item>`支持以下特性：

- `android:id` 
- `android:icon`：引用一个drawable
- `android:title`：引用一个字符串
- `android:showAsAction`：指定何时及如何显示在action bar上。

更多特性参见[Menu Resource](http://developer.android.com/guide/topics/resources/menu-resource.html)。

在`<item>`中嵌套`<menu>`实现子菜单：

	<?xml version="1.0" encoding="utf-8"?>
	<menu xmlns:android="http://schemas.android.com/apk/res/android">
	    <item android:id="@+id/file"
	          android:title="@string/file" >
	        <!-- "file" submenu -->
	        <menu>
	            <item android:id="@+id/create_new"
	                  android:title="@string/create_new" />
	            <item android:id="@+id/open"
	                  android:title="@string/open" />
	        </menu>
	    </item>
	</menu>

重启菜单使用`MenuInflater.inflate()`。

## 创建Options Menu

options menu出现在屏幕上的位置取决于开发应用的版本：

- 为Android 2.3.x (API level 10)和之前的开发应用，当用户按下Menu键后，options menu出现在屏幕底部。When opened, the first visible portion is the icon menu, which holds up to six menu items. 如果菜单项超过6个，Android将第6项和后面的放入overflow menu，点击*More*可以显示。
- 为Android 3.0 (API level 11)或更高版本开发，options menu中的项出现在action bar。默认所有项都在action overflow（按下overflow图标或按下设备上的Menu按键都会显示）。为了快速访问重要的actions，你可以将一些项提升到action bar上：设置 `<item>` 的`android:showAsAction="ifRoom"`。

![](options_menu.png)

Figure 1. Options menu in the Browser, on Android 2.3.

![](actionbar.png)

Figure 2. Action bar from the Honeycomb Gallery app, showing navigation tabs and a camera action item (plus the action overflow button).

> Note: Even if you're not developing for Android 3.0 or higher, you can build your own action bar layout for a similar effect. For an example of how you can support older versions of Android with an action bar, see the [Action Bar Compatibility](http://developer.android.com/resources/samples/ActionBarCompat/index.html) sample.

You can declare items for the options menu from either your Activity subclass or a Fragment subclass. 如果Activity和Fragment都声明了options menu，它们将合并到UI。显示Android的items，然后是每个Fragment的（按Fragment插入Activity顺序）。可以通过`<item>`的`android:orderInCategory`特性调整顺序。

要为Activity定义options menu，覆盖`onCreateOptionsMenu()`（Fragment有自己的`onCreateOptionsMenu()`）。在该方法中，重启定义在XML中的菜单到`Menu`对象。

	@Override
	public boolean onCreateOptionsMenu(Menu menu) {
	    MenuInflater inflater = getMenuInflater();
	    inflater.inflate(R.menu.game_menu, menu);
	    return true;
	}

可以功过`add()`添加菜单项，通过`findItem()`获取一个`MenuItem`。

If you've developed your application for Android 2.3.x and lower, 系统在用户首次打开菜单时调用`onCreateOptionsMenu()`创建options menu。If you've developed for Android 3.0 and higher, 系统在活动启动时创建`onCreateOptionsMenu()`，并将它们显示到action bar中。

### 创建点击事件

当用户选择options menu（或action bar）中的一项，系统调用活动的`onOptionsItemSelected()`方法。传入选中的`MenuItem`。识别此项可以调用`getItemId()`，返回菜单项的`android:id`。

	@Override
	public boolean onOptionsItemSelected(MenuItem item) {
	    // Handle item selection
	    switch (item.getItemId()) {
	        case R.id.new_game:
	            newGame();
	            return true;
	        case R.id.help:
	            showHelp();
	            return true;
	        default:
	            return super.onOptionsItemSelected(item);
	    }
	}

成功处理后，返回true。如果不想处理，调用父类的`onOptionsItemSelected()`（默认实现返回false）。

如果活动有Fragment，系统先调用Activity的`onOptionsItemSelected()`，然后依次（按添加顺序）调用Fragment的，直到有方法返回true。

> Tip: Android 3.0，可以在XML中ton故宫`android:onClick`定义点击行为。特性值是活动中的一个方法。该方法只接受一个`MenuItem`做参数。

> Tip: 如果你的一个应用，有多个活动包含相同的options menu，考虑创建一个Activity基类，只实现`onCreateOptionsMenu()`和`onOptionsItemSelected()`方法，然后让其余活动继承。This way, you can manage one set of code for handling menu actions and each descendant class inherits the menu behaviors. If you want to add menu items to one of the descendant activities, override onCreateOptionsMenu() in that activity. Call super.onCreateOptionsMenu(menu) so the original menu items are created, then add new menu items with menu.add(). You can also override the super class's behavior for individual menu items.

### 在运行时创建菜单项

系统调用`onCreateOptionsMenu()`后，它会保留Menu实例，不会再调用`onCreateOptionsMenu()`，unless the menu is invalidated for some reason。However, you should use `onCreateOptionsMenu()` only to create the initial menu state and not to make changes during the activity lifecycle.

如果想在运行时根据活动状态改变options menu，可以在`onPrepareOptionsMenu()`中做。向该方法传入当前已存在的`Menu`对象，你可以对它进行修改，添加、删除或禁用菜单项。Fragments也提供`onPrepareOptionsMenu()`回调。

在Android 2.3.x及之前，每次用户打开（通过Menu按钮）options menu时，系统调用`onPrepareOptionsMenu()`。

在Android 3.0及之后，options menu总是存在的（位于action bar）。因此当需要改变时，需要调用` invalidateOptionsMenu()`，触发系统调用`onPrepareOptionsMenu()`。

> Note: You should never change items in the options menu based on the View currently in focus. When in touch mode (when the user is not using a trackball or d-pad), views cannot take focus, so you should never use focus as the basis for modifying items in the options menu. If you want to provide menu items that are context-sensitive to a View, 使用上下文菜单。

## （未）创建上下文菜单

## 创建弹出菜单

A [PopupMenu](http://developer.android.com/reference/android/widget/PopupMenu.html) is a modal menu anchored to a View. It appears below the anchor view if there is room, or above the view otherwise. It's useful for:

- Providing an overflow-style menu for actions that relate to specific content (such as Gmail's email headers, shown in figure 4).
- Providing a second part of a command sentence (such as a button marked "Add" that produces a popup menu with different "Add" options).
- Providing a drop-down similar to Spinner that does not retain a persistent selection.

> Note: This is not the same as a context menu, which is generally for actions that affect selected content. For actions that affect selected content, use the contextual action mode or floating context menu.

![](popupmenu.png)

Figure 4. A popup menu in the Gmail app, anchored to the overflow button at the top-right.

> Note: PopupMenu is available with API level 11 and higher.

如果菜单定义在XML中，显示弹出菜单的方法是：

- 利用构造器实例化`PopupMenu`，which takes the current application Context and the View to which the menu should be anchored.
- 利用`MenuInflater`充气菜单资源，到`PopupMenu.getMenu()`返回的`Menu`对象. On API level 14 and above, you can use PopupMenu.inflate() instead.
- 调用PopupMenu.show()

例如点击下面的按钮弹出菜单：

	<ImageButton
	    android:layout_width="wrap_content" 
	    android:layout_height="wrap_content" 
	    android:src="@drawable/ic_overflow_holo_dark"
	    android:contentDescription="@string/descr_overflow_button"
	    android:onClick="showPopup" />

	public void showPopup(View v) {
	    PopupMenu popup = new PopupMenu(this, v);
	    MenuInflater inflater = popup.getMenuInflater();
	    inflater.inflate(R.menu.actions, popup.getMenu());
	    popup.show();
	}

The menu is dismissed when the user selects an item or touches outside the menu area. 可以通过`PopupMenu.OnDismissListener`监听解散（dismiss）事件。

### 处理点击事件

处理用户选择菜单项的事件，实现`PopupMenu.OnMenuItemClickListener`接口，通过`PopupMenu`的`setOnMenuItemclickListener()`方法注册。

例如：

	public void showMenu(View v) {
	    PopupMenu popup = new PopupMenu(this, v);
	
	    // This activity implements OnMenuItemClickListener
	    popup.setOnMenuItemClickListener(this);
	    popup.inflate(R.menu.actions);
	    popup.show();
	}

	@Override
	public boolean onMenuItemClick(MenuItem item) {
	    switch (item.getItemId()) {
	        case R.id.archive:
	            archive(item);
	            return true;
	        case R.id.delete:
	            delete(item);
	            return true;
	        default:
	            return false;
	    }
	}

## （未）Creating Menu Groups

















