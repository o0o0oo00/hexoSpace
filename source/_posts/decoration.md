---
title: RecyclerView.ItemDecoration() 的用法与技巧
date: 2018-12-18 18:06:32
category: 技术
tags: android
---
## ItemDecoration

As we all know but how to use

#### 主要明白其中**重写的三个方法**
分别是：  
`getItemOffsets`只负责规定绘制区域  
`onDrawOver`负责绘制，显示在Item视图之上  
`onDraw`负责绘制，显示在Item视图之下  

<!--more-->

```
No.1
通过设置outRect 来实现Item分割线的绘制范围

override fun getItemOffsets(outRect: Rect, view: View, parent: RecyclerView, state: RecyclerView.State?) {
    super.getItemOffsets(outRect, view, parent, state)
    outRect.set(Rect(0, margin, 0, 0))
}

```
设置Item的偏移量 也就是绘制的分割线范围[在这里](https://juejin.im/post/59099fe844d904006942a983)  


#### 参数解析

```
* @param outRect Rect to receive the output.
* @param view    The child view to decorate
* @param parent  RecyclerView this ItemDecoration is decorating
* @param state   The current state of RecyclerView.
```
#### 如果我想指定分割线在第一个Item的上面绘制，第二个Item的下面绘制，其他的Item不绘制

```
override fun getItemOffsets(outRect: Rect, view: View, parent: RecyclerView, state: RecyclerView.State?) {
	super.getItemOffsets(outRect, view, parent, state)
	val childAdapterPosition = parent.getChildAdapterPosition(view)
	when (childAdapterPosition) {
	    0 -> outRect.set(Rect(0, margin, 0, 0))
	    1 -> outRect.set(Rect(0, 0, 0, margin))
	    else -> outRect.set(Rect())
	}
}
```

**注意!!**，如果想在这个方法中通过`parent.childCount`来获取到子view的数量，是**不可以**的，我觉得是因为addView的时候就已经走到了`getItemOffsets `这个方法，所以结果就是**第一次**走到这个方法的时候`parent.childCount`就是**1**，**第二次**走到这个方法的时候`parent.childCount`就是**2**。


```
No.2 会被Item视图覆盖掉

override fun onDraw(c: Canvas?, parent: RecyclerView?, state: RecyclerView.State?) {
    super.onDraw(c, parent, state)
}
```
 
#### 参数解析 下同
```
* @param c Canvas to draw into
* @param parent RecyclerView this ItemDecoration is drawing into
* @param state The current state of RecyclerView
```

这样就**仅仅是**在第二个Item上画了一个底部分割线 

```
No.3 将覆盖Item视图

override fun onDrawOver(canvas: Canvas?, parent: RecyclerView?, state: RecyclerView.State?) {
    if (parent == null || null == canvas) {
        return
    }
    val index = parent.childCount - 2
    val child = parent.getChildAt(index)
    val left = 0f
    val right = child.right.toFloat()
    val top = (child.bottom - offset).toFloat()
    val bottom = child.bottom.toFloat()
    val rect = RectF(left, top, right, bottom)
    canvas.drawRect(rect, paint.apply { color = color })
}
```