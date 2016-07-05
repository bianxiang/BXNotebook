## SurfaceView

SurfaceView继承自View。与View不同的是，View更新必须在UI线程，但SurfaceView更新画面可以在自定义线程。｛｛但需要开发者自己保证线程安全？｝｝

继承`SurfaceView`时一般还要实现`SurfaceHolder.Callback`接口｛｛于是可以`getHolder().addCallback(this)`｝｝。绘制界面的方法是`onDraw`。每调用一次该方法，就重绘一帧画面。`SurfaceHolder.Callback`接口中定义了3个生命周期回调，用于初始化资源、影响SurfaceView的改变、在销毁时释放资源。









