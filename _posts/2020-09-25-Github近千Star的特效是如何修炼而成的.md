---
layout:     post
title:      强行进阶之打造Github近千Star且登顶Trending榜的无敌特效
subtitle:   本文将从0到1带你一步步打造Github上近千Star的无敌炫酷特效
date:       2020-09-19
author:     MLX
header-img: img/android_bg.jpg
catalog: 	 true
tags:
    - Android
    - 自定义View
    - Github
typora-root-url: ..
---

[带你实现女朋友欲罢不能的网易云音乐宇宙尘埃特效](https://juejin.im/post/6871049441546567688)

这是自定义View系列的上一篇文章，上一篇文章是自定义View的，本篇文章是自定义ViewGroup的。

## 前言

这是本篇特效的Github地址：https://github.com/MlxChange/WaveDisPlay

我先承认我吹牛了，标题是近千Star(虽然只有700)，且登顶Trending榜(当时排Kotlin分类第三名)。但是我仍然有一颗上进的心，吹牛是无罪的！

大家且先放下手中40m的大刀，让我先跑39米，看看我这次能给大家整个什么花活再决定是否砍我。

还是同样的道理，我知道没图是骗不了人的，先放图，看看今天实现的效果是否真的那么炫酷。

![](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/161bd77df5c546c295132f986c37b372~tplv-k3u1fbpfcp-zoom-1.image)

怎么样是不是还算有那么一丢丢炫酷的感觉？？

这个效果是一个自定义ViewGroup，相信有很多小伙伴平常写自定义View还是较多的，但是自定义ViewGroup就比较少了。主要啊，自定义ViewGroup考虑的东西太多了，又是什么测量，又是什么布局，有的时候还得考虑滑动冲突，真的是要多麻烦有多麻烦。

不过，本篇就带你一点点分析效果入手，到实现效果，再到处理这些麻烦事，由浅入深的让你也能轻松学会自定义ViewGroup。妈妈再也不用担心我不会自定义ViewGroup了。

另外，大家也可以看看我上一篇文章，对自定义View掌握不够熟练的可以考虑我的实现方式，就是一点点实现，然后再慢慢修改，增加效果。

话不多说，放码过来。

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/5c7aa6cb9861402a80dd8f5284217af0~tplv-k3u1fbpfcp-zoom-1.image)

## 特效分析

由于是自定义ViewGroup，效果还是蛮复杂的。我刚开始看到这个原效果图的时候，我内心是拒绝的，不能说你UI设计那么炫酷，根本不考虑我们程序员的辛苦把，坚决不做这种东西！

于是我和UI打了一架，谁赢了听谁的。现在看到实现的效果，我只想说 **真香**。

![](/img/emoji/真香.jpg)

好了，我不皮了。我们认真来看一下效果把。

首先，这个效果可以看到有很多的页面，就像列表一样，记得之前看过一个效果就是探探那种效果，左滑喜欢，右滑不喜欢和这个类似。所以我起初想的是看能不能从自定义LayoutManager或者自定义一个View继承于RecyclerView入手，后来我还是放弃了，因为它们不太容易实现预览到下一个界面这种效果，最终决定了自定义一个ViewGroup去实现。

这个效果我觉得最特殊的在于能够通过拖拽看到下一个界面的内容，就像拉窗帘一样，并且划过中心之后，能够自动的滑动到另一侧，并且有反弹效果，然后显示下一个页面。当下一个页面完全显示的时候，此时拖拽按钮自动生成，再次预览下下个页面。而且按钮也能按回去，按回去以后反方向就会生成一个新的按钮，显示上一页的内容。

综上所述，我总结了几点：

1. 是一个自定义ViewGroup
2. 子View堆叠显示，就像列表一样可以自定义子View的内容，并且能预览到下一页或者上一页的内容。
3. 有一个拖拽的按钮，在往外拉的时候按钮也在不断的变大，缩回去的时候按钮在不断变小，并且是一个很圆滑的效果
4. 在完全滑动到另一侧的时候，还会反弹几下

其实我觉得其他的都比较好实现，比较麻烦的就是这个拖拽按钮，我决定从拖拽按钮入手。

## 拖拽按钮

虽然讲的是自定义ViewGroup，但是做自定义View不能一上来就开干，我先写个自定义View，先看看这个拖拽按钮能不能实现，如果这个都不能实现的话，那后面的就找个借口甩锅给UI不是就不需要写了吗。

![](/img/emoji/机智如我.jpg)

我先看看这个拖拽按钮是怎么实现的。

先从左边的按钮开始看，毕竟屏幕的原点在左上角嘛。

首先它距离左边有一小点距离，从上到下就是一条线，只是中间有个圆不圆，尖不尖的突起。这个突起我先不管它，我先画出这个线。因为是测试，我创建一个TestView

```kotlin
class TestView  @JvmOverloads constructor(
    context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0
) : View(context, attrs, defStyleAttr) {


}
```

然后是画线。我们不妨先思考一下，如何画这个线。在效果图中虽然确实是一条线，但是它左边也有内容显示，所以我觉得更像是一个细长的矩形，只不过这个细长的矩形中出了一个叛徒(突起)。所以画线变成了画矩形，而且由于这个叛徒的存在，简单的canvas画矩形显然是无法在后续的矩形上添加这个叛徒的，所以得用那个另外一个方式画矩形，那么是什么呢？

没错，就是Path，Path一样可以达到同样的效果。所以开干。

### 画矩形

定义一个画笔和Path

```kotlin
var path= Path()
var paint =Paint()

init {
    paint.color=Color.RED //为了方便辨识，我们定义红色
    paint.style=Paint.Style.STROKE
}
```

矩形怎么画呢？其实和我们在纸上画一模一样，从原点画个线到右边，然后在画直线到下面，再往左画一个直线，最后和原点闭合。如此一个矩形就出来了。灵魂画师就是我本人了。

![](/img/waveRect.png)

这个画法其实Path很好的帮我们实现，并且我们为了方便，还是定义一个中心点centerX和centerY

```kotlin
override fun onDraw(canvas: Canvas) {
    super.onDraw(canvas)
    path.lineTo(100f,0f)//100只是个测试距离，画上图步骤1
    path.lineTo(100f,centerY*2)//画步骤2
    path.lineTo(0f,centerY*2)//画步骤3
    path.close()//闭合，也就是画步骤4
    canvas.drawPath(path,paint)
}

override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
    super.onSizeChanged(w, h, oldw, oldh)
    centerX= (w/2).toFloat()//定义屏幕中心X值
    centerY= (h/2).toFloat()//定义屏幕中心Y值
}
```

那我们看一下效果吧

<img src="/img/waveImg1.jpg" style="zoom: 25%;" />

emmm看起来有点意思了，不过状态栏是什么鬼？啊 这个不要管他啦，就是透明状态栏而已啦。

### 画突起

那我们现在考虑一下如何画这个突起呢？

其实突起嘛，在纸上画应该是这样的

<img src="/img/画突起1.png" style="zoom:50%;" />

你看如果我们在纸上画还是很简单的，右边的线那里只需要多个突起即可。多了三步，看看代码怎么实现吧

```kotlin
override fun onDraw(canvas: Canvas) {
    super.onDraw(canvas)
    path.lineTo(100f,0f)//上图第一步
    path.lineTo(100f,centerY-200)//上图第二步
    path.lineTo(180f,centerY)//第三步
    path.lineTo(100f,centerY+200)//第四步
    path.lineTo(100f,centerY*2)//第五步
    path.lineTo(0f,centerY*2)//第六步
    path.close()//第七步
    canvas.drawPath(path,paint)
}
```

效果如何呢？

<img src="/img/waveImg2.jpg" style="zoom:25%;" />

看起来有点意思了哦~不过人家那个效果看起来很圆滑，你这个尖尖的，有点不大一样啊。

这位兄台，你说的很有道理，不过圆的我不会画啊。咋办呢？就只有尖尖才能维持得了生活这样子，里面的老哥个个都是人才说话又好听，啊等等不对，走错片场了。

圆形是什么呢？是曲线，没错吧？在计算机中我们该如何画曲线呢？我们看看有没有小伙伴知道啊，我等你十分钟

啊，十分钟过去了，没人知道我就说答案了哦

没错，就是贝塞尔曲线！

![](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97617b523b534dbf83b3b10790288a83~tplv-k3u1fbpfcp-zoom-1.image)

什么，你没听过贝塞尔曲线？老哥，你out了。

贝塞尔曲线是计算机中模拟曲线的一种算法，通过方程balabalabla，省略一堆定义.....

简而言之，贝塞尔曲线就是帮我们画曲线的，通过位置点和控制点来确定曲线。如果不会的小伙伴，我建议看看[GcsSloop](https://www.gcssloop.com/#blog)或者扔物线的关于贝塞尔曲线的基础用法。我当初学习的时候也是跟着他们学习的。非常有学习价值~

我们看一下二阶贝塞尔曲线的方法把

```java
/**
 * 从上一个点开始，绘制二阶Bezier曲线
 * (x1,y1)为控制点， (x2,y2)为终点
 */
public void quadTo(float x1, float y1, float x2, float y2) ；
```

那么话说回来，用贝塞尔曲线画，应该怎么画呢？我们继续尝试在纸上画一下试试

![](/img/wave贝塞尔1.png)

贝塞尔曲线就对应着步骤三了.起始点就是A点了，控制点就是B点了，终点那应该就是C点了。

这个点的坐标呢？我们假设步骤一的长度为100，a点到B点在X方向上的偏移是100，B点的Y坐标就是中心坐标的话，B点的坐标就是(100+100,centerY)。然后A到C的距离我们设定为200，这些应该不难理解

我们代码上实践一下

```kotlin
override fun onDraw(canvas: Canvas) {
    super.onDraw(canvas)
    path.lineTo(100f,0f)//步骤1
    path.lineTo(100f,centerY-100)步骤2
    path.quadTo(200f,centerY,100f,centerY+100)//步骤3
    path.lineTo(100f,centerY*2)//步骤4
    path.lineTo(0f,centerY*2)//步骤5
    path.close()//步骤6
    canvas.drawPath(path,paint)
}
```

效果是如何的呢？我们看看

<img src="/img/wave突起2.jpg" style="zoom:25%;" />

emmm好像不是那么辣眼睛了，不过这个突起着实有点突兀。怎么看怎么觉得像是塞了钱进剧组的。

你这么一说，我和效果图一比对，我感觉我做成这样，老板怕不是要立马开除我了。可是效果图那么圆滑的特效是如何做出来的呢？

其实也很简单，二阶贝塞尔曲线不行，那就让它兄弟三阶贝塞尔曲线出马，三阶贝塞尔曲线还不行，就四阶五阶。。。

其实最终效果是个六阶贝塞尔曲线，如下图

<img src="/img/六阶贝塞尔曲线.png" style="zoom:50%;" />

P1点和P8点是位置点，其余全是控制点。红色的线就是生成的曲线，是不是看得出来很圆滑了呢？方案是不是很简单呢？

![](/img/emoji/人话么.jpeg)

啊，大刀不要落下来！我认错了！其实一点不简单。我当初思考，也是思考了半天才想出来。

六阶虽然难弄，但是我们可以分解啊。分解成两个三阶的贝塞尔曲线不就OK了？

我们以P1为起始点，P4为终点，P2,P3就是位置点，我们半个半个画不就很简单了嘛

上半部分就变成了这样：

![](/img/贝塞尔曲线2.png)

那我们来看一下三阶贝塞尔曲线的方法把

```java
/*
 *	x1,y1是第一个控制点的位置，对应P2
 *	x2,y2是第二个控制点的位置, 对应P3
 *	x3,y3是最后一个位置点的位置，对应p4
 */
public void cubicTo(float x1, float y1, float x2, float y2,float x3, float y3)
```

那么他们的坐标是怎么确定的呢？

我们在纸上继续画画试试

![](/img/wave突起3.png)

P1,P4就是位置点，P2,P3就是控制点。那么P2,P3,P4的距离如何去确定呢？

有很多在线生成贝塞尔曲线的网站，你生成四个点，摆的差不多了，看看他们的坐标就能大概知道是多少了。

我之前呢已经去试过了，以P1为(0,0)的话呢,几个点的坐标如下：

p2(0,100)，p3(90,75)，p4(100,150)。由于P4的Y坐标我们已经确定为centerY了，所以P1只能是(100,centerY-150).代码如下：

```kotlin
override fun onDraw(canvas: Canvas) {
    super.onDraw(canvas)
    path.lineTo(100f,0f)//第一步
    path.lineTo(100f,centerY-150)//第二步
    path.cubicTo(100f,centerY-150f+100,100f+90f,centerY-150+75f,100f+100f,centerY)//第三步
    path.lineTo(100f,centerY*2)//第五步
    path.lineTo(0f,centerY*2)//第六步
    path.close()//第七步
    canvas.drawPath(path,paint)
}
```

由于啊，我们只是花了上半部分，所以上图中的第四步没有画，我们主要是想先看看上半部分是不是我们想要的效果，运行一下，效果来咯

<img src="/img/突起上半部分.jpg" style="zoom:25%;" />

看起来上半部分效果确实圆滑了许多，那我依葫芦画瓢，把下半部分也完成。那下半部分的话，就是以P4为起点了

![](/img/贝塞尔2.png)

p1就是上半部分的p4位置，对照上半部分的相对位置，那么此时p2,p3,p4的坐标就很容易确定了

p1(100f+100f,centerY)，p2(100f+90f,centerY+90f)，p3(100f,centerY+75f)，p4(100f,centerY+150f)

这是代码：

```kotlin
override fun onDraw(canvas: Canvas) {
    super.onDraw(canvas)
    path.lineTo(100f,0f)//第一步
    path.lineTo(100f,centerY-150)//第二步
    path.cubicTo(100f,centerY-150f+100,100f+90f,centerY-150+75f,100f+100f,centerY)//第三步
    path.cubicTo(100f+90f,centerY+90f,100f,centerY+75f,100f,centerY+150f)//第四步
    path.lineTo(100f,centerY*2)//第五步
    path.lineTo(0f,centerY*2)//第六步
    path.close()//第七步
    canvas.drawPath(path,paint)
}
```

空口无凭，来看看效果才是硬道理

<img src="/img/突起3.jpg" style="zoom:25%;" />

怎么样，效果是不是很好很圆滑了，再也不是尖尖角了。Mlx，YES！

既然这个效果已经实现了，我们开始下一步，动画！

## 拖拽动画

首先，分析上面的动画，我们可以发现，这个小突起会跟随手指上下移动，并且当手指向右边滑动的时候，突起会变大，那我们先不管左右的情况，先考虑上下滑动的情况。

### 上下动画

在上面画的这个突起中，突起的最高点我们设定的Y值是centerY。突起如果移动，这几个点的相对位置应该是不会变的。唯一改变的就是突起的最高点才对。

那么问题来了，突起的最高点它应该怎么变化？

没错，就是跟随手指上下移动。现在情况就是这样了，突起的最高点也就是上半部分和下半部分的位置点，Y值是centerY，现在应该Y值跟随手指，而其他的相对距离保持不变。

![](/img/wave突起3.png)

还是以这个图为例，就是P4点的Y值改变，P1 P2 P3的X坐标都不会发生改变，Y坐标以P4的坐标为原点。这样的话就能让这个突起只是上下改变了。

也就是说，
$$
P1(100,P4.y-150) ,P2(100,P4.y-50),P3(190,P4.y-75) P4(200,Y)
$$
这里唯一的变量就是Y了，Y既然跟随手指那就是手指按下的地方的Y值咯。怎么记录呢？

嘿嘿嘿，就是`onTouchEvent`方法啦

我们先定义一个变量currentY来记录当前手指的位置，如果没有触摸默认就是屏幕中心

```kotlin
var currentY=0f//记录当前手指触摸位置
override fun onSizeChanged(w: Int, h: Int, oldw: Int, oldh: Int) {
        super.onSizeChanged(w, h, oldw, oldh)
        centerX= (w/2).toFloat()
        centerY= (h/2).toFloat()
        currentY=centerY //默认为屏幕中心
}
```

现在我们需要记录触摸的位置

```kotlin
override fun onTouchEvent(event: MotionEvent): Boolean {
    when(event.action){
        MotionEvent.ACTION_MOVE-> {
            currentY=event.y
            invalidate()//重新绘制界面
        }
    }
    return true
}
```

只做了这些还不够哦，我们画画的地方还没有更改呢，还没有以P4为标准呢。我们现在去改一下。当时是centerY，现在新人换旧人了，应该是currentY了

```kotlin
override fun onDraw(canvas: Canvas) {
    super.onDraw(canvas)
    path.lineTo(100f,0f)
    path.lineTo(100f,currentY-150)
    //画上半部分
    path.cubicTo(100f,currentY-150f+100,100f+90f,currentY-150+75f,100f+100f,currentY)
    //画下半部分
    path.cubicTo(100f+90f,currentY+90f,100f,currentY+75f,100f,currentY+150f)
    path.lineTo(100f,centerY*2)
    path.lineTo(0f,centerY*2)
    path.close()
    canvas.drawPath(path,paint)
}
```

啊哈，让我们高兴得看看效果把~

![](/img/突起1.gif)

????

![](/img/emoji/逗我.png)

兄弟我真不是在逗你玩，咱们出现了一点小的失误。你想啊，path是路径对吧，我们一直在添加路径，可是之前的路径还在啊，也就是说这些路径是叠加的。所以我们需要每次绘制路径之后，删除原来的路径。

就像这样：

```kotlin
override fun onDraw(canvas: Canvas) {
    ...
    canvas.drawPath(path,paint)//画路径
    path.reset()//画完之后删除之前的路径
}
```

如此以来，我们再看一下？

<img src="/img/突起2.gif" style="zoom:50%;" />

这样一来，是不是可以了？简直完美！我没骗你把~

![](/img/emoji/不错.jpg)

### 左右动画

上下的事情处理完了，是不是轮到左右的滑动事件了？

我们再次分析一下效果哈~

我们再次观察效果图，会发现

在往外拖拽的过程中，这个突起小啾啾会变大，但是当突起到达了屏幕中心的时候，是最大的，不会再进一步变大了。

你问我突起变大什么意思？我怀疑你在开车。。。

突起变大，用图的话就很好理解，我们继续在纸上作图

![](/img/左右突起1.png)

一开始突起是这样的，变大以后就是这样的

<img src="/img/左右突起2.png" style="zoom:50%;" />

我们观察一下什么地方发生了变化呢？

可以很明显地观察到，P1到P4的距离发生了改变，我们暂且称P1到P4在Y方向的距离为这个突起的半径把。因为P1到P4的距离是整个突起在Y方向的一半嘛~

那么图中很明显就是这个半径变大了，而且P1到P4在X方向上也变大了，

再用一幅图来表示

![](/img/左右突起3.png)

可以看到有个蓝色的矩形，简单的来说， 突起变大，就是这个蓝色的矩形在不断的变大，这个矩形的长度就是整个突起的长度，这个矩形的宽度就是突起的宽度。当然了，我这里画的不太像，毕竟灵魂画手。

所以说P1,P2,P3,P4的坐标完全可以用这个矩形来表示。我们可以给这个矩形定义长度`dragHeight`和宽度`dragWidth`，长度对应Y方向，宽度对应X方向。

就像是这样：

![](/img/左右突起4.png)

上图矩形的宽度定义为dragWidth，矩形的长度定义为dragHeight。经过多次测试，得到最好的效果后，坐标关系对应如下。而且已经知道，P4的Y坐标就是手指触摸坐标currentY。那么最终坐标如下：
$$
P1(x,currentY-dragHeight) ,  P2(x, (P1.Y+P4.Y)/2 + 12dp), 
$$

$$
P3((x+dragWidth)*0.94 ,  (P1.Y+P4.Y)/2),  P4(x+dragWidth, currentY)
$$

而且，
$$
dragHeight = dragWidth*1.5
$$
看到了吗，`dragHeight`是由`dragWidth`决定的。而四个点的坐标全部由`dragHeight`来决定。

那么`dragWidth`怎么确定呢？聪明的小伙伴应该已经发现了，没错，这个dragWidth就是由手指左右拖动的距离来确定的。可是为了良好的用户体验，不让手指挡住拖动按钮，所以突起的最高点应该在手指按下地方的往左一点点。定义`currentX`为手指触摸点的X值，那么有下面公式：
$$
dragWidth = currentX - 12dp
$$
可能有小伙伴问了，你这些都是啥啥啥，上来整一堆公式，还让不让人愉快的玩耍了。

其实我也么的办法，这些参数啊确实是我调教好的~你也可以直接拿去用啦，如果你觉得不好，你也可以自己手动修改以下参数的啦~

那么既然有了这些参数，我们修改一下之前的代码，首先定义这个矩形的长和宽

```kotlin
var dragWidth=0
var dragHeight=0
```

然后修改一下之前的控制点和数据点的坐标，不过这里直接填写数据是不是很麻烦，根本不知道哪个点是哪个点呢。所以根据我们的图，来定义七个点，分别对应下图中的点1到点8

<img src="/img/六阶贝塞尔曲线.png" style="zoom:50%;" />

照例我们不能再ondraw中定义哦~为了方便大家能够对照图来理解，我直接按图中点来命名了，大家就不要吐槽我的命名了哦。需要注意的是，我们的曲线是两个三阶贝塞尔曲线组成的，所以P4和P5是一个点

```kotlin
private val point1=PointF(0f,0f)
private val point2=PointF(0f,0f)
private val point3=PointF(0f,0f)
private val point4=PointF(0f,0f)
private val point5=PointF(0f,0f)
private val point6=PointF(0f,0f)
private val point7=PointF(0f,0f)
```

所以我们修改一下绘画的代码：

