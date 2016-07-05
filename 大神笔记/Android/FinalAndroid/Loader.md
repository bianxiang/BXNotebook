在Android 3.0引入。加载器帮助异步加载 Activity 或 Fragment 的数据。加载器功能特性：

- 在每个 Activity 和 Fragment 中都可以使用。
- 异步加载数据。
- 监控数据源，当数据源发生变化时递送新数据。
- 因配置改变被重新创建后，自动重新连接到上一次的游标。因此不需要查询查询数据。

## Loader API 总结

涉及的类和接口：

- `LoaderManager`。抽象类，与 Activity 或 Fragment 关联。负责关联一个或多个 `Loader` 实例。This helps an application manage longer-running operations in conjunction with the Activity or Fragment lifecycle; 一般与 `CursorLoader` 连用。应用可以编写自己的loaders，加载其他类型的数据。一个 Activity 或 Fragment 只有一个`LoaderManager`。但一个 `LoaderManager`可以管理多个加载器。
- `LoaderManager.LoaderCallbacks`。客户端利用此回调接口。与 `LoaderManager` 交互。例如，在 `onCreateLoader()` 回调中创建一个新的加载器。
- `Loader`。抽象类。进行异步数据加载。加载器的基类。常用的子类是`CursorLoader`。可以自己编写子类。当加载器活动时（active），它们需要负责监控数据源，并在数据改变时递送新结果。
- `AsyncTaskLoader`。抽象加载器类。利用一个 `AsyncTask` 完成任务。
- `CursorLoader`。`AsyncTaskLoader`的子类。查询 `ContentResolver`，返回一个 `Cursor`。This class implements the Loader protocol in a standard way for querying cursors. 此加载器是从 `ContentProvider` 加载数据的最好方式。

## 在应用中使用加载器

### 启动一个加载器

一个 Activity 或 Fragment 只有一个 `LoaderManager`。

一般在活动的`onCreate()`方法，或Fragment的`onActivityCreated()`方法中初始化加载器：

```java
// Prepare the loader. Either re-connect with an existing one,
// or start a new one.
getLoaderManager().initLoader(0, null, this);
```

`initLoader()`方法的参数：

- 一个唯一标识，标识加载器。在本例是中是`0`。
- 在构造期提供给加载器的参数。可选。
- `LoaderManager.LoaderCallbacks`实现。

The `initLoader()` call ensures that a loader is initialized and active. It has two possible outcomes:

- 如果标识指定的加载器已存在，重用之前的。
- 如果标识指定的加载器不存在，`initLoader()`触发`LoaderManager.LoaderCallbacks.onCreateLoader()`。在该方法中实例化并返回一个新的加载器。

If at the point of this call the caller is in its started state, and the requested loader already exists and has generated its data, then the system calls `onLoadFinished()` immediately (during initLoader()), so you must be prepared for this to happen.

尽管`initLoader()`方法返回创建的`Loader`。你一般不需要引用它。`LoaderManager`会自动管理其生命周期。`LoaderManager`负责在需要时启动和停止加载，维护加载器的状态。即，你基本不需要与加载器直接交互(though for an example of using loader methods to fine-tune a loader's behavior, see the [LoaderThrottle](http://developer.android.com/resources/samples/ApiDemos/src/com/example/android/apis/app/LoaderThrottle.html) sample)。You most commonly use the `LoaderManager.LoaderCallbacks` methods to intervene in the loading process when particular events occur.

### 重启加载器

使用`initLoader()`时，若指定ID的加载器已存在，会重用原来的。但有时你向放弃旧的重新创建一个。

要弃用旧数据可以使用`restartLoader()`。For example, this implementation of `SearchView.OnQueryTextListener` restarts the loader when the user's query changes. 加载器需要重启才能使用修改后的查询条件：

```java
public boolean onQueryTextChanged(String newText) {
    // Called when the action bar search text has changed. Update
    // the search filter, and restart the loader to do a new query
    // with this filter.
    mCurFilter = !TextUtils.isEmpty(newText) ? newText : null;
    getLoaderManager().restartLoader(0, null, this);
    return true;
}
```

### 使用`LoaderManager`回调

客户端利用回调接口`LoaderManager.LoaderCallbacks`与`LoaderManager`交互。

Loaders, in particular CursorLoader, are expected to retain their data after being stopped. This allows applications to keep their data across the activity or fragment's `onStop()` and `onStart()` methods, so that when users return to an application, they don't have to wait for the data to reload. You use the `LoaderManager.LoaderCallbacks` methods when to know when to create a new loader, and to tell the application when it is time to stop using a loader's data.

`LoaderManager.LoaderCallbacks`包含以下方法：
- `onCreateLoader()`：实例化并返回给定ID的一个新`Loader`。
- `onLoadFinished()`：之前创建的加载器完成加载。
- `onLoaderReset()`：当之前创建的加载器被重置时调用，thus making its data unavailable.

#### `onCreateLoader`

本例中，`onCreateLoader()`回调方法创建一个`CursorLoader`。You must build the `CursorLoader` using its constructor method, which requires the complete set of information needed to perform a query to the `ContentProvider`. Specifically, it needs:
- `uri` — The URI for the content to retrieve.
- `projection` — A list of which columns to return. Passing null will return all columns, which is inefficient.
- `selection` — A filter declaring which rows to return, formatted as an SQL WHERE clause (excluding the WHERE itself). Passing null will return all rows for the given URI.
- `selectionArgs` — You may include `?s` in the selection, which will be replaced by the values from selectionArgs, in the order that they appear in the selection. The values will be bound as Strings.
- `sortOrder` — How to order the rows, formatted as an SQL ORDER BY clause (excluding the ORDER BY itself). Passing null will use the default sort order, which may be unordered.

例子：

```java
	 // If non-null, this is the current filter the user has provided.
	String mCurFilter;
	...
	public Loader<Cursor> onCreateLoader(int id, Bundle args) {
		// 这个例子中只有一个加载器，因此忽略id
		Uri baseUri;
		if (mCurFilter != null) {
			baseUri = Uri.withAppendedPath(Contacts.CONTENT_FILTER_URI,
					  Uri.encode(mCurFilter));
		} else {
			baseUri = Contacts.CONTENT_URI;
		}

		String select = "((" + Contacts.DISPLAY_NAME + " NOTNULL) AND ("
				+ Contacts.HAS_PHONE_NUMBER + "=1) AND ("
				+ Contacts.DISPLAY_NAME + " != '' ))";
		return new CursorLoader(getActivity(), baseUri,
				CONTACTS_SUMMARY_PROJECTION, select, null,
				Contacts.DISPLAY_NAME + " COLLATE LOCALIZED ASC");
	}
```

#### `onLoadFinished`

之前创建的加载器完成加载后调用。This method is guaranteed to be called prior to the release of the last data that was supplied for this loader. At this point you should remove all use of the old data (since it will be released soon), but should not do your own release of the data since its loader owns it and will take care of that.

The loader will release the data once it knows the application is no longer using it. For example, if the data is a cursor from a `CursorLoader`, you should not call `close()` on it yourself. If the cursor is being placed in a `CursorAdapter`, you should use the `swapCursor()` method so that the old Cursor is not closed. For example:
```java
	// This is the Adapter being used to display the list's data.
	SimpleCursorAdapter mAdapter;
	...

	public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
		// Swap the new cursor in.  (The framework will take care of closing the
		// old cursor once we return.)
		mAdapter.swapCursor(data);
	}
```

#### `onLoaderReset`

This method is called when a previously created loader is being reset, thus making its data unavailable. This callback lets you find out when the data is about to be released so you can remove your reference to it.

This implementation calls `swapCursor()` with a value of null:
```java
	// This is the Adapter being used to display the list's data.
	SimpleCursorAdapter mAdapter;
	...

	public void onLoaderReset(Loader<Cursor> loader) {
		// This is called when the last Cursor provided to onLoadFinished()
		// above is about to be closed.  We need to make sure we are no
		// longer using it.
		mAdapter.swapCursor(null);
	}
```

## 例子

As an example, here is the full implementation of a Fragment that displays a ListView containing the results of a query against the contacts content provider. It uses a CursorLoader to manage the query on the provider.

For an application to access a user's contacts, as shown in this example, its manifest must include the permission `READ_CONTACTS`.

```java
	public static class CursorLoaderListFragment extends ListFragment
			implements OnQueryTextListener, LoaderManager.LoaderCallbacks<Cursor> {

		// This is the Adapter being used to display the list's data.
		SimpleCursorAdapter mAdapter;

		// If non-null, this is the current filter the user has provided.
		String mCurFilter;

		@Override public void onActivityCreated(Bundle savedInstanceState) {
			super.onActivityCreated(savedInstanceState);

			// Give some text to display if there is no data.  In a real
			// application this would come from a resource.
			setEmptyText("No phone numbers");

			// We have a menu item to show in action bar.
			setHasOptionsMenu(true);

			// Create an empty adapter we will use to display the loaded data.
			mAdapter = new SimpleCursorAdapter(getActivity(),
					android.R.layout.simple_list_item_2, null,
					new String[] { Contacts.DISPLAY_NAME, Contacts.CONTACT_STATUS },
					new int[] { android.R.id.text1, android.R.id.text2 }, 0);
			setListAdapter(mAdapter);

			// Prepare the loader.  Either re-connect with an existing one,
			// or start a new one.
			getLoaderManager().initLoader(0, null, this);
		}

		@Override public void onCreateOptionsMenu(Menu menu, MenuInflater inflater) {
			// Place an action bar item for searching.
			MenuItem item = menu.add("Search");
			item.setIcon(android.R.drawable.ic_menu_search);
			item.setShowAsAction(MenuItem.SHOW_AS_ACTION_IF_ROOM);
			SearchView sv = new SearchView(getActivity());
			sv.setOnQueryTextListener(this);
			item.setActionView(sv);
		}

		public boolean onQueryTextChange(String newText) {
			// Called when the action bar search text has changed.  Update
			// the search filter, and restart the loader to do a new query
			// with this filter.
			mCurFilter = !TextUtils.isEmpty(newText) ? newText : null;
			getLoaderManager().restartLoader(0, null, this);
			return true;
		}

		@Override public boolean onQueryTextSubmit(String query) {
			// Don't care about this.
			return true;
		}

		@Override public void onListItemClick(ListView l, View v, int position, long id) {
			// Insert desired behavior here.
			Log.i("FragmentComplexList", "Item clicked: " + id);
		}

		// These are the Contacts rows that we will retrieve.
		static final String[] CONTACTS_SUMMARY_PROJECTION = new String[] {
			Contacts._ID,
			Contacts.DISPLAY_NAME,
			Contacts.CONTACT_STATUS,
			Contacts.CONTACT_PRESENCE,
			Contacts.PHOTO_ID,
			Contacts.LOOKUP_KEY,
		};
		public Loader<Cursor> onCreateLoader(int id, Bundle args) {
			Uri baseUri;
			if (mCurFilter != null) {
				baseUri = Uri.withAppendedPath(Contacts.CONTENT_FILTER_URI,
						Uri.encode(mCurFilter));
			} else {
				baseUri = Contacts.CONTENT_URI;
			}

			// Now create and return a CursorLoader that will take care of
			// creating a Cursor for the data being displayed.
			String select = "((" + Contacts.DISPLAY_NAME + " NOTNULL) AND ("
					+ Contacts.HAS_PHONE_NUMBER + "=1) AND ("
					+ Contacts.DISPLAY_NAME + " != '' ))";
			return new CursorLoader(getActivity(), baseUri,
					CONTACTS_SUMMARY_PROJECTION, select, null,
					Contacts.DISPLAY_NAME + " COLLATE LOCALIZED ASC");
		}

		public void onLoadFinished(Loader<Cursor> loader, Cursor data) {
			// Swap the new cursor in.  (The framework will take care of closing the
			// old cursor once we return.)
			mAdapter.swapCursor(data);
		}

		public void onLoaderReset(Loader<Cursor> loader) {
			// This is called when the last Cursor provided to onLoadFinished()
			// above is about to be closed.  We need to make sure we are no
			// longer using it.
			mAdapter.swapCursor(null);
		}
	}
```




