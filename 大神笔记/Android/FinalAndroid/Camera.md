[toc]

## 摄像头

### 简单拍照

#### 请求摄像头的权限

To advertise that your application depends on having a camera, put a `<uses-feature>` tag in your manifest file:

```xml
    <manifest ... >
        <uses-feature android:name="android.hardware.camera"
                      android:required="true" />
        ...
    </manifest>
```

若摄像头不是必需的，设置`android:required`为`false`。在运行时通过`hasSystemFeature(PackageManager.FEATURE_CAMERA)`检查摄像头是否可用。

#### 利用 Camera App 拍照

```java
    static final int REQUEST_IMAGE_CAPTURE = 1;
    private void dispatchTakePictureIntent() {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
            startActivityForResult(takePictureIntent, REQUEST_IMAGE_CAPTURE);
        }
    }
```

注意，`startActivityForResult()`调用前首先检查了`resolveActivity()`。否则若摄像应用不存在，你的应用将崩溃。

#### 获取缩略图

Android的 Camera 应用，将照片的**缩略图**放在`onActivityResult()`收到的`Intent`中。

```java
	@Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == REQUEST_IMAGE_CAPTURE && resultCode == RESULT_OK) {
            Bundle extras = data.getExtras();
            Bitmap imageBitmap = (Bitmap) extras.get("data");
            mImageView.setImageBitmap(imageBitmap);
        }
    }
```

#### 保存全尺寸的图片

若提供一个文件，Android Camera 可以将全尺寸的图片存入文件。

一般来说，用户拍摄的所有照片应该放在设备上的公有外边存储，以期被所有应用访问。合适的位置可以通过`getExternalStoragePublicDirectory()`获取，传入`DIRECTORY_PICTURES`。读写这个应用需要权限 `READ_EXTERNAL_STORAGE` 和 `WRITE_EXTERNAL_STORAGE`。

下面这段代码用于获取存储文件：

```java
    String mCurrentPhotoPath;
    private File createImageFile() throws IOException {
        // Create an image file name
        String timeStamp = new SimpleDateFormat("yyyyMMdd_HHmmss").format(new Date());
        String imageFileName = "JPEG_" + timeStamp + "_";
        File storageDir = Environment.getExternalStoragePublicDirectory(
                Environment.DIRECTORY_PICTURES);
        File image = File.createTempFile(
            imageFileName,  /* prefix */
            ".jpg",         /* suffix */
            storageDir      /* directory */
        );

        // Save a file: path for use with ACTION_VIEW intents
        mCurrentPhotoPath = "file:" + image.getAbsolutePath();
        return image;
    }
```

调用 Camera 时，指定图片的输出文件：

```java
    static final int REQUEST_TAKE_PHOTO = 1;
    private void dispatchTakePictureIntent() {
        Intent takePictureIntent = new Intent(MediaStore.ACTION_IMAGE_CAPTURE);
        if (takePictureIntent.resolveActivity(getPackageManager()) != null) {
            // Create the File where the photo should go
            File photoFile = null;
            try {
                photoFile = createImageFile();
            } catch (IOException ex) {
                // Error occurred while creating the File
                ...
            }
            if (photoFile != null) {
                takePictureIntent.putExtra(MediaStore.EXTRA_OUTPUT,
                        Uri.fromFile(photoFile));
                startActivityForResult(takePictureIntent, REQUEST_TAKE_PHOTO);
            }
        }
    }
```

#### 将照片添加到 Gallery

让用户可以访问到这张图片的最简单方式，是将其添加到系统的**Media Provider**。

> 注意：若你将图片存储在`getExternalFilesDir()`等目录，媒体扫描不能扫到它，因为这些目录是你的应用私有的。

The following example method demonstrates how to invoke the system's media scanner to add your photo to the **Media Provider**'s database, making it available in the Android Gallery application and to other apps.

```java
    private void galleryAddPic() {
        Intent mediaScanIntent = new Intent(Intent.ACTION_MEDIA_SCANNER_SCAN_FILE);
        File f = new File(mCurrentPhotoPath);
        Uri contentUri = Uri.fromFile(f);
        mediaScanIntent.setData(contentUri);
        this.sendBroadcast(mediaScanIntent);
    }
```

#### Decode a Scaled Image

Managing multiple full-sized images can be tricky with limited memory. If you find your application running out of memory after displaying just a few images, you can dramatically reduce the amount of dynamic heap used **by expanding the JPEG into a memory array that's already scaled to match the size of the destination view**. The following example method demonstrates this technique.

```
    private void setPic() {
        // Get the dimensions of the View
        int targetW = mImageView.getWidth();
        int targetH = mImageView.getHeight();

        // Get the dimensions of the bitmap
        BitmapFactory.Options bmOptions = new BitmapFactory.Options();
        bmOptions.inJustDecodeBounds = true;
        BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
        int photoW = bmOptions.outWidth;
        int photoH = bmOptions.outHeight;

        // Determine how much to scale down the image
        int scaleFactor = Math.min(photoW/targetW, photoH/targetH);

        // Decode the image file into a Bitmap sized to fill the View
        bmOptions.inJustDecodeBounds = false;
        bmOptions.inSampleSize = scaleFactor;
        bmOptions.inPurgeable = true;

        Bitmap bitmap = BitmapFactory.decodeFile(mCurrentPhotoPath, bmOptions);
        mImageView.setImageBitmap(bitmap);
    }
```

### 简单录像

与拍照一样，首先要请求摄像头的权限。

#### 利用 Camera 应用录制视频

```java
    static final int REQUEST_VIDEO_CAPTURE = 1;
    private void dispatchTakeVideoIntent() {
        Intent takeVideoIntent = new Intent(MediaStore.ACTION_VIDEO_CAPTURE);
        if (takeVideoIntent.resolveActivity(getPackageManager()) != null) {
            startActivityForResult(takeVideoIntent, REQUEST_VIDEO_CAPTURE);
        }
    }
```

注意，`startActivityForResult()`调用前首先检查了`resolveActivity()`。否则若摄像应用不存在，你的应用将崩溃。

#### 查看视频

The Android Camera application returns the video in the Intent delivered to `onActivityResult()` as a Uri pointing to the video location in storage. The following code retrieves this video and displays it in a VideoView.

```java
    @Override
    protected void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (requestCode == REQUEST_VIDEO_CAPTURE && resultCode == RESULT_OK) {
            Uri videoUri = intent.getData();
            mVideoView.setVideoURI(videoUri);
        }
    }
```

### 控制摄像头

本节介绍如何用框架API直接控制摄像头硬件。

#### 打开 Camera 对象

推荐在`onCreate()`中打开新线程，在`onResume()`中，在新线程内打开摄像头。

若摄像头正被其他应用使用，`Camera.open()`会抛出异常，因此要在外面包围try-catch。

```java
    private boolean safeCameraOpen(int id) {
        boolean qOpened = false;
        try {
            releaseCameraAndPreview();
            mCamera = Camera.open(id);
            qOpened = (mCamera != null);
        } catch (Exception e) {
            Log.e(getString(R.string.app_name), "failed to open Camera");
            e.printStackTrace();
        }
        return qOpened;
    }

    private void releaseCameraAndPreview() {
        mPreview.setCamera(null);
        if (mCamera != null) {
            mCamera.release();
            mCamera = null;
        }
    }
```

#### 创建摄像头预览

利用`SurfaceView`绘制摄像头的捕获。

预览器要实现`android.view.SurfaceHolder.Callback`接口，用于爸图像数据从摄像头硬件传给应用。

```java
    class Preview extends ViewGroup implements SurfaceHolder.Callback {
        SurfaceView mSurfaceView;
        SurfaceHolder mHolder;
        Preview(Context context) {
            super(context);

            mSurfaceView = new SurfaceView(context);
            addView(mSurfaceView);

            // Install a SurfaceHolder.Callback so we get notified when the
            // underlying surface is created and destroyed.
            mHolder = mSurfaceView.getHolder();
            mHolder.addCallback(this);
            mHolder.setType(SurfaceHolder.SURFACE_TYPE_PUSH_BUFFERS);
        }
    ...
    }
```

The preview class must be passed to the Camera object before the live image preview can be started, as shown below.

A camera instance and its related preview must be created in a specific order, with the camera object being first. In the snippet below, the process of initializing the camera is encapsulated so that `Camera.startPreview()` is called by the `setCamera()` method, whenever the user does something to change the camera. The preview must also be restarted in the preview class `surfaceChanged()` callback method.

```java
    public void setCamera(Camera camera) {
        if (mCamera == camera) { return; }
        stopPreviewAndFreeCamera();
        mCamera = camera;
        if (mCamera != null) {
            List<Size> localSizes = mCamera.getParameters().getSupportedPreviewSizes();
            mSupportedPreviewSizes = localSizes;
            requestLayout();
            try {
                mCamera.setPreviewDisplay(mHolder);
            } catch (IOException e) {
                e.printStackTrace();
            }
            // Important: Call startPreview() to start updating the preview
            // surface. Preview must be started before you can take a picture.
            mCamera.startPreview();
        }
    }
```

#### 修改摄像头的设置

Camera settings change the way that the camera takes pictures, from the zoom level to exposure compensation. This example changes only the preview size; see the source code of the Camera application for many more.

```java
    public void surfaceChanged(SurfaceHolder holder, int format, int w, int h) {
        // Now that the size is known, set up the camera parameters and begin
        // the preview.
        Camera.Parameters parameters = mCamera.getParameters();
        parameters.setPreviewSize(mPreviewSize.width, mPreviewSize.height);
        requestLayout();
        mCamera.setParameters(parameters);

        // Important: Call startPreview() to start updating the preview surface.
        // Preview must be started before you can take a picture.
        mCamera.startPreview();
    }
```

#### 设置预览的朝向

Most camera applications lock the display into landscape mode because that is the natural orientation of the camera sensor. This setting does not prevent you from taking portrait-mode photos, because the orientation of the device is recorded in the EXIF header. The `setCameraDisplayOrientation()` method lets you change how the preview is displayed without affecting how the image is recorded. However, in Android prior to API level 14, you must stop your preview before changing the orientation and then restart it.

#### 拍照

Use the `Camera.takePicture()` method to take a picture once the preview is started. You can create `Camera.PictureCallback` and `Camera.ShutterCallback` objects and pass them into `Camera.takePicture()`.

If you want to grab images continously, you can create a `Camera.PreviewCallback` that implements `onPreviewFrame()`. For something in between, you can capture only selected preview frames, or set up a delayed action to call `takePicture()`.

#### 重启预览

After a picture is taken, you must restart the preview before the user can take another picture. In this example, the restart is done by overloading the shutter button.

```java
    @Override
    public void onClick(View v) {
        switch(mPreviewState) {
        case K_STATE_FROZEN:
            mCamera.startPreview();
            mPreviewState = K_STATE_PREVIEW;
            break;

        default:
            mCamera.takePicture( null, rawCallback, null);
            mPreviewState = K_STATE_BUSY;
        } // switch
        shutterBtnConfig();
    }
```

#### 停止预览并释放摄像头

Once your application is done using the camera, it's time to clean up. In particular, you must release the Camera object, or you risk crashing other applications, including new instances of your own application.

When should you stop the preview and release the camera? Well, having your preview surface destroyed is a pretty good hint that it’s time to stop the preview and release the camera, as shown in these methods from the Preview class.

```java
    public void surfaceDestroyed(SurfaceHolder holder) {
        // Surface will be destroyed when we return, so stop the preview.
        if (mCamera != null) {
            // Call stopPreview() to stop updating the preview surface.
            mCamera.stopPreview();
        }
    }

    /**
     * When this function returns, mCamera will be null.
     */
    private void stopPreviewAndFreeCamera() {
        if (mCamera != null) {
            // Call stopPreview() to stop updating the preview surface.
            mCamera.stopPreview();

            // Important: Call release() to release the camera for use by other
            // applications. Applications should release the camera immediately
            // during onPause() and re-open() it during onResume()).
            mCamera.release();

            mCamera = null;
        }
    }
```

Earlier in the lesson, this procedure was also part of the `setCamera()` method, so initializing a camera always begins with stopping the preview.