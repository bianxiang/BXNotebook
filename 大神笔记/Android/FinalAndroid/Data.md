# 数据存储

## SharedPreferences

若只有少量键值对要保存，[使用SharedPreferences](http://developer.android.com/reference/android/content/SharedPreferences.html)。SharedPreferences对象指向一个键值对文件，提供简单的读写API。SharedPreferences使用文件由框架管理，可以私有或共享。This data will persist across user sessions (even if your application is killed).

> SharedPreferences APIs用于读写键值对，不要与Preference APIs混淆（后者是帮助构建设置的UI）。

### 获取对SharedPreferences的引用

利用下面两个方法，创建新的共享偏好文件，或访问已存在的：

- `getPreferences()`：如果活动只需要有一个SharedPreferences文件，使用该方法。该方法返回属于活动的、默认的共享偏好文件，因此不要提供文件名。  
```SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);```
- `getSharedPreferences()`：可以从任何Context调用，传入文件名。

例如，从Fragment访问：

	Context context = getActivity();
	SharedPreferences sharedPref = context.getSharedPreferences(
	        getString(R.string.preference_file_key), Context.MODE_PRIVATE);

文件名要全局唯一（uniquely identifiable to your app），如 `com.example.myapp.PREFERENCE_FILE_KEY`。

> 如果共享创建创建时配置`MODE_WORLD_READABLE`或`MODE_WORLD_WRITEABLE`，则任何知道文件标识符的app都能访问你的数据。

### 写SharedPreferences

先调用SharedPreferences的`edit()`，创建`SharedPreferences.Editor`。最后调用`commit()`。

	SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
	SharedPreferences.Editor editor = sharedPref.edit();
	editor.putInt(getString(R.string.saved_high_score), newHighScore);
	editor.commit();

### 读SharedPreferences

	SharedPreferences sharedPref = getActivity().getPreferences(Context.MODE_PRIVATE);
	int defaultValue = getResources().getInteger(R.string.saved_high_score_default);
	long highScore = sharedPref.getInt(getString(R.string.saved_high_score), defaultValue);

### 例子

	public class Calc extends Activity {
	    public static final String PREFS_NAME = "MyPrefsFile";
	
	    @Override
	    protected void onCreate(Bundle state){
	       super.onCreate(state);
	       . . .
	
	       // Restore preferences
	       SharedPreferences settings = getSharedPreferences(PREFS_NAME, 0);
	       boolean silent = settings.getBoolean("silentMode", false);
	       setSilent(silent);
	    }
	
	    @Override
	    protected void onStop(){
	       super.onStop();
	
	      // We need an Editor object to make preference changes.
	      // All objects are from android.context.Context
	      SharedPreferences settings = getSharedPreferences(PREFS_NAME, 0);
	      SharedPreferences.Editor editor = settings.edit();
	      editor.putBoolean("silentMode", mSilentMode);
	
	      // Commit the edits!
	      editor.commit();
	    }
	}


