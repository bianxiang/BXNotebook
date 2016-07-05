[toc]

## Canvas接口绘制方法

## Canvas的save和restore
onDraw方法会传入一个Canvas对象，它是你用来绘制控件视觉界面的画布。

在onDraw方法里，我们经常会看到调用save和restore方法，它们到底是干什么用的呢？

- save：用来保存Canvas的状态。save之后，可以调用Canvas的平移、放缩、旋转、错切、裁剪等操作。
- restore：用来恢复Canvas之前保存的状态。防止save后对Canvas执行的操作对后续的绘制有影响。

save和restore要配对使用（restore可以比save少，但不能多），如果restore调用次数比save多，会引发Error。save和restore之间，往往夹杂的是对Canvas的特殊操作。

例如：我们先想在画布上绘制一个右向的三角箭头，当然，我们可以直接绘制，另外，我们也可以先把画布旋转90°，画一个向上的箭头，然后再旋转回来（**这种旋转操作对于画圆周上的标记非常有用**）。

## 技巧

画圆周上的东西，旋转画布。例如Path画形状文字。

![](img/canvas_trick_rotate.jpg)

```java
    Path path = new Path();
    path.addArc(new RectF(0,0,150,150), -180, 180);
    Paint citePaint = new Paint(paint);
    citePaint.setTextSize(14);
    citePaint.setStrokeWidth(1);
    canvas.drawTextOnPath("http://www.android777.com", path, 28, 0, citePaint);

    for(int i=0 ; i <count ; i++){
        if(i%5 == 0){
            canvas.drawLine(0f, y, 0, y+12f, paint);
            canvas.drawText(String.valueOf(i/5+1), -4f, y+25f, tmpPaint);
        } else {
            canvas.drawLine(0f, y, 0f, y +5f, tmpPaint);
        }
        canvas.rotate(360/count,0f,0f); //旋转画纸
    }
```





