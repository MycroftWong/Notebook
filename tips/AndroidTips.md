# 一些Android小细节

## android:clipToPadding, android:clipChildren的使用与分析

为了实现一些特殊的效果，才会用到这两个属性。

### ViewPager

- 通常可以见到与```ViewPager```配合使用的Tab的定制化，会要求中间的Tab会比较突出，而为了实现这个效果，可以讲Tab容器的高度设置为中间Tab的高度，而其他的Tab的实际高度（或视觉高度）则比较矮，不过这一定程度上会影响到```ViewPager```内容区域的显示或点击效果。

- 那么可以看到经常会有类似于```ViewPager```的左右滑动，一次性会显示多个item的界面。去掉```ViewPager```的一个item会占```ViewPager```整个区域的固定思维，那么应该可以想象到完全可以使用```ViewPager```来实现这样的效果。

- 文章[android:clipToPadding和android:clipChildren](http://www.jcodecraeer.com/a/anzhuokaifa/androidkaifa/2015/0317/2613.html)的例子中，举出一个```ListView```的实际内容区域超出常规认知的区域。

### 概念

**View**的尺寸、位置、背景、margin、padding、绘制区域。

尺寸：对这个实在再熟悉不过，通常是调用```View.setMeasuredDimension```方法确认的结果

位置：相对于其父类所占的位置，通常是调用```View.layout```方法得到结果

**站位**：这个是我自己对上面两个属性的一个白话式的理解。一个```View```在界面上占有一定的区域，通常情况下，它所有的绘制都是在这个区域的，超出了这个区域的绘制是会被过滤掉，不会显示在界面上的。

背景：一个```View```在其所占区域的背景，永远是第一时间绘制，而之后的绘制，会覆盖其上的

margin: 与```View```的容器的距离，或与其兄弟```View```边界的距离，如```FrameLayout```容器中的```View```，则是与```FrameLayout```边界的距离，如```LinearLayout```, ```RelativeLayout```容器中的```View```则是与其位置相关联的```View```边界的距离，如果没有相关的```View```, 则是与其容器的边界的距离。总之，它表示与其他```View```的距离，在某种程度上会影响到其尺寸会位置，但是不会影响其绘制区域逻辑的改变（始终在绘制区域内）。

padding: 这是一个影响```View```自身内部可绘制区域的属性。如果是一个```ViewGroup```, 那么通常其中的```View```会根据padding来分配尺寸和位置。如果是```View```，会根据这个属性来分配绘制区域。我也通常使用这个属性来限制一个```View```的内容区域，如通常```TextView```会在两侧留白，```ImageView```显示图片时不会显示在整个区域内，而是会缩小一些，那么使用padding则是一个非常好的方法。

绘制区域：这个通常是对```View```而言的，对于```ViewGroup```没有特殊的要求，不会在其上进行绘制，通常使用背景即可（不过不要有固定思维，我们也可以在```ViewGroup```上进行绘制）。

### 打破习以为常的认知

认知：一个```View```所能绘制的区域是根据其位置、尺寸和padding决定的，能够绘制的区域通常是其所占区域和减去padding的结果，如果绘制的内容超出绘制区域，则不会显示在屏幕上，即使逻辑上是在超出区域部分进行了绘制。

padding部分是否一定不可被绘制：之前，我们使用padding来限制绘制，就是认为padding区域是不能被绘制的，但是如果```ViewGroup```设置属性```android:clipToPadding="false"```, 那么其中的子```View```是可以显示其容器的padding区域内的。

属性```android:clipToPadding```含义：```ViewGroup```的属性，绘制在padding区域的```View```是否会被剪切掉，默认为```true```. 模糊的概念：绘制在padding区域的```View```？为什么会有绘制在padding区域的```View```呢？一个```ViewGroup```, 它的padding属性对其尺寸和位置都没有影响，影响的是其绘制区域和其子```View```的位置、尺寸。正常情况下，其子```View```的绘制是不能显示在padding区域的，那是因为**绘制在padding区域的部分会被剪切掉**，这也是```android:clipToPadding```这个属性名字的来源。

例子：一个```FrameLayout```, 其paddingTop="16dp", 其有唯一的一个子```View```, 宽高固定80dp, 没有其他特殊的属性，那么一开始，这个子```View```距顶部16dp的位置，如果我们向上移动```View```20dp, 即top=-40dp, 那么实际上，只能显示它的一半内容，因为绘制在padding区域的部分被剪切掉了，**但是，如果```FrameLayout```设置属性```android:clipToPadding="false"```, 那么会保留绘制在padding区域的部分。

### clipToPadding与clipChildren的区别

这两者都是```ViewGroup```的属性，```clipToPadding```是控制绘制在padding区域部分的内容（位置在padding区域的子```View```）是否会被剪切掉。而```clipChildren```则是控制绘制超出```ViewGroup```的区域是否会被剪切掉。我们可以认为可以看成padding和margin的区别。

点击区域：```clipToPadding```绘制在padding区域的部分的子```View```可以被点击。```clipChildren```超出```ViewGroup```绘制的子```View```的部分区域，不能被点击。




