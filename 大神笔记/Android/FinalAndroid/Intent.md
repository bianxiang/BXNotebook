[toc]

## Intent

### 显示应用选择器

若一个操作可以被多个应用处理，且用户很可能每次会选择不同的应用——如分享——此时，最好显式的弹出一个选择对话框，强迫用户么次都要选择执行操作的应用。

```java
    Intent intent = new Intent(Intent.ACTION_SEND);
    ...
    // This says something like "Share this photo with"
    String title = getResources().getString(R.string.chooser_title);
    // Create intent to show chooser
    Intent chooser = Intent.createChooser(intent, title);

    // 检查intent至少可以解析成一个活动！
    if (intent.resolveActivity(getPackageManager()) != null) {
        startActivity(chooser);
    }
```

