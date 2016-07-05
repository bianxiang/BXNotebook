Android的核心 Dalvik virtual machine (VM) 不是纯粹的Java，它提供的与传统的 Java SDK 也不完全相同。

## 使用脚本语言

This includes incorporating your own scripting language into your application, something that is expressly prohibited on some other devices.

一种Java脚本语言是 BeanShell。BeanShell gives you Java-compatible syntax with implicit typing and no compilation required.

要使用 BeanShell 脚本，需要将 BeanShell interpreter 的JAR放入`libs/`。不幸的是 2.0b4 的 JAR 不能在 Android 0.9 及之后的 SDKs 使用，或许是由于编译JAR的编译器的问题。Instead, you should probably check out the source code from Apache Subversion and execute `ant jarcore` to build it, and then copy the resulting JAR (in BeanShell’s `dist/` directory) to your own project’s `libs/`. Or, just use the BeanShell JAR that accompanies the source code for this book in the Java/AndShell project.

From there, using BeanShell on Android is no different from using BeanShell in any other Java environment:

1. 创建 BeanShell `Interpreter` 类的实例。
2. 通过`Interpreter#set()`设置全局选项。
3. Call `Interpreter#eval()` to run the script and, optionally, get the result of the last statement.

```java
public class MainActivity extends Activity {

	private Interpreter i=new Interpreter();

	public void go(View v) {
		EditText script=(EditText)findViewById(R.id.script);
		String src=script.getText().toString();
		try {
			i.set("context", MainActivity.this);
			i.eval(src);
		} catch (bsh.EvalError e) {
			AlertDialog.Builder builder = new AlertDialog.Builder(MainActivity.this);
			builder.setTitle("Exception!").setMessage(e.toString()).setPositiveButton("OK", null).show();
		}
	}
}
```

And now, some caveats:

- 不是所有的脚本语言都能工作。For example, those that implement their own form of just-in-time (JIT) compilation, generating Java bytecodes on-the-fly, would probably need to be augmented to generate Dalvik VM bytecodes instead of those for stock Java implementations. Simpler languages that execute from parsed scripts, calling Java reflection APIs to call back into compiled classes, will likely work better. 即便这样，部分特性——依赖于Java API是Dalvik所没有的——无法工作。For example, there could be stuff hidden inside BeanShell or the add-on JARs that does not work on today’s Android.
- Scripting languages without JIT will inevitably be slower than compiled Dalvik applications. Slower may mean users experience sluggishness. Slower definitely means more battery life is consumed for the same amount of work. So, building a whole Android application in BeanShell, simply because you feel it is easier to program in, may cause your users to be unhappy.
- Scripting languages that expose the whole Java API, like BeanShell, can pretty much do anything the underlying Android security model allows. So, if your application has the READ_CONTACTS permission, expect any BeanShell scripts your application runs to have the same permission.
- 脚本语言的解析器一般较大。

Additionally, Scripting Layer for Android (SL4A), described at http://code.google.com/p/android-scripting/, allows you to write scripts in a wide range of scripting languages, beyond BeanShell, such as the following: Perl, Python, JRuby, Lua, JavaScript (implemented via Rhino, the Mozilla JavaScript interpreter written in Java), PHP.

These scripts are not full-fledged applications, though the SL4A team is working on allowing you to turn them into APK files complete with basic UIs. For on-device development, SL4A is a fine choice. Notable projects developed with SL4A include the Nexus One sensor logging payload. If you’re interested in further SL4A reading and development, an excellent book on the topic is **Pro Android Python with SL4A**, by Paul Ferrill (Apress, 2011).


