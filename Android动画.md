### 动画分类

Android动画：属性动画 & 视图动画( 补间动画 & 逐帧动画)

#### 视图动画ViewAnimation

​	作用对象：视图View

##### 1.补间动画TweenAnimation

​	确定开始|结束的视图样式，中间动画变化过程由系统补全

1. 视图样式：平移Translate、缩放Scale、旋转Rotate、透明度Alpha
2. 特点：使用简单方便，已封装好基础动画效果 | 仅控制整体实现效果，无法控制属性
3. 应用场景：视图中标准、基础的动画效果 | Activity、Fragment的切换效果 | 视图组中子元素的出场效果

##### 2.逐帧动画FrameAnimation

​	将动画拆分为帧的形式，且定义每一帧=每一张图片，按顺序播放一组预先定义好的图片

1. 特点：使用简单方便 | 容易引发OOM[因使用大量尺寸较大的图片资源]
2. 应用场景：较为复杂的个性化动画效果

##### 3.视图动画的局限性

1. 作用对象局限在View上，有些情况下动画效果针对视图的某个属性，而非View

2. 只改变View的视觉效果，不能改变View的属性
3. 动画效果单一，只能实现简单动画需求，复杂动画效果难以实现

#### 属性动画PropertyAnimation

Android3.0后提供的全新动画模式，定义一个随时间更改任何对象属性的动画，无论其是否绘制到屏幕上

1. 原理：在一定时间间隔内，通过不断对值进行改变 & 不断将该值赋给对象的属性，实现对象在某属性上的动画效果
2. 作用对象：任意Java对象，不再局限于视图View对象
3. 特点：作用对象进行了扩展，动画效果丰富
4. 应用场景：与属性相关、更加复杂的动画效果

动画属性：

- 位移 translationX|translationY|translationZ
- 透明度 alpha，全透明到不透明：0f -> 1f
- 旋转 rotation，旋转一圈：0f -> 360f
- 缩放 水平缩放scaleX，垂直缩放scaleY

##### 1.ValueAnimator

先改变值，后手动赋值给对象的属性。`ValueAnimator`作为`ObjectAnimator`的父类，主要动态计算目标对象属性的值，然后设置给对象属性，达到动画效果。

``` java
//ValueAnimator实现
tvText.setOnClickListener {
    val valueAnimator = ValueAnimator.ofFloat(0f, 180f)
    valueAnimator.addUpdateListener {
           tvText.rotationY = it.animatedValue as Float //手动赋值
    }
    valueAnimator.start()
}
//ObjectAnimator实现
 ObjectAnimator.ofFloat(tvText, "rotationY", 0f, 180f).apply { start() }
//使用ValueAnimator实现动画，需要手动赋值给目标对象`tvText`的`rotationY`，而ObjectAnimator则是自动赋值，不需要手动赋值就可以达到效果。
```

##### 2.ObjectAnimator

先改变值，后自动赋值给对象的属性，父类为ValueAnimator。`ObjectAnimator`则在`ValueAnimator`的基础上极大地**简化**对目标对象的属性值的计算和添加效果，融合了 ValueAnimator 的计时引擎和值计算以及为目标对象的命名属性添加动画效果这一功能。

``` java
llAddAccount.setOnClickListener {
    val objectAnimation = ObjectAnimator.ofFloat(llAddAccount, "translationX", 0f, -70f)
    objectAnimation.start()
}
/*
ObjectAnimator.ofFloat(), param1:要实现动画效果的view, param2:属性名, param3:可变长参数，开始位置\中间位置\结束位置
注意事项：如果可变长参数只有一个值，值将作为动画结束值，此时属性必须拥有初始化值和getXXX方法。
translationX\translationY涉及到的位移都是相对自身位置而言。例如点A(x,y)->点B(x1,y1),那么ofFloat方法的可变长参数，第一个值应该0f,第二个值应该x1-x。
*/
```

##### 3.与视图动画的区别

是否改变动画本身的属性：使用视图动画时，无论动画结果在哪，该View的位置不变 & 响应区域都是在原地，不根据结果移动；属性动画通过改变属性从而使动画移动，位置&响应区域变化

#### 插值器|估值器

先由插值器根据时间流逝的百分比计算出目标对象的属性改变的**百分比**，再由估值器根据插值器计算出来的属性改变的百分比计算出目标对象属性对应类型的**值**。

**插值器决定变化趋势，估值器决定具体变化数值**

##### 1.插值器Interpolator

辅助动画实现的接口，设置属性值从初始值过渡到结束值的变化规律，确定了动画效果变化的模式。

系统默认的插值器是`AccelerateDecelerateInterpolator`，即先加速后减速。SDK内置的时间插值器：

- LinearInterpolator 匀速
- AccelerateInterpolator 持续加速
- DecelerateInterpolator 持续减速
- OvershootInterpolator 快速完成，超出再回到结束样式
- AccelerateDecelerateInterpolator 先加速后减速
- AnticipateInterpolator 先退后再加速
- AnticipateOvershootInterpolator 先退后再加速前进，超出终点后再回终点
- BounceInterpolator 最后阶段弹球效果
- CycleInterpolator 周期运动

1. 自定义插值器

   - 本质：根据动画的进度（0%-100%）计算出当前属性值改变的百分比

   - 具体使用：实现 `Interpolator` / `TimeInterpolator`接口 & 复写`getInterpolation()`

     >1. 补间动画 实现 `Interpolator`接口；属性动画实现`TimeInterpolator`接口
     >2. `TimeInterpolator`接口是属性动画中新增的，用于兼容`Interpolator`接口，这使得所有过去的`Interpolator`实现类都可以直接在属性动画使用

   - 自定义插值器的关键在于：**对input值 根据动画的进度（0%-100%）通过逻辑计算 计算出当前属性值改变的百分比**

   ``` java
   public interface Interpolator {  
        float getInterpolation(float input) {  
           // input值值变化范围是0-1，且随着动画进度（0% - 100% ）均匀变化
           // 即动画开始时，input值 = 0；动画结束时input = 1
           // 而中间的值则是随着动画的进度（0% - 100%）在0到1之间均匀增加
         return xxx；
         // 返回的值就是用于估值器继续计算的fraction值，下面会详细说明
       }  
       
   public interface TimeInterpolator {  
       float getInterpolation(float input);  
   }  
   ```

2. `PathInterpolator`贝塞尔曲线：**将任意一条曲线通过精确的数学公式表达出来**。

> ``` java
>//创建一个任意Path的插值器
> PathInterpolator(Path path)
> //创建一个二阶贝塞尔曲线的插值器
> PathInterpolator(float controlX, float controlY)
> //创建一个三阶贝塞尔曲线的插值器
> PathInterpolator(float controlX1, float controlY1, float controlX2, float controlY2)
> //实现ImageView的透明度的变化，采用ObjectAnimator来实现，设置PathInterpolator可以控制透明度变化的速率，达到更加“丝滑”的视觉效果。
> ObjectAnimator animator = ObjectAnimator.ofFloat(mImageView, "alpha", 1, 0);
> PathInterpolator pathInterpolator = new PathInterpolator(.24f, .9f, .24f, 1f);
> animator.setInterpolator(pathInterpolator);
> animator.setDuration(500);
> animator.start();
> ```

##### 2.估值器Evaluator

辅助插值器的接口，设置属性值从初始值过渡到结束值变化的具体数值，协助插值器实现非线性运动的动画效果

SDK内置估值器：

- IntEvaluator 以整型的形式从初始值 - 结束值 进行过渡
- FloatEvaluator 以浮点型的形式从初始值 - 结束值 进行过渡
- ArgbEvaluator 以Argb类型的形式从初始值 - 结束值 进行过渡

1. 自定义估值器
   - 本质：根据 插值器计算出当前属性值改变的百分比 & 初始值 & 结束值 来计算 当前属性具体的数值
   - 具体使用：自定义估值器需要实现 TypeEvaluator接口 & 复写`evaluate()`

``` java
// 自定义对象
data class Point(var x: Float, var y: Float)
// 自定义估值器
class PointEvaluator : TypeEvaluator<Point> {
    /**
     * 根据插值器计算出当前对象的属性的百分比fraction,估算去属性当前具体的值
     * @param fraction 属性改变的百分比，插值器getInterpolation（）的返回值
     * @param startValue 动画的初始值
     * @param endValue 动画的结束值
     */
    override fun evaluate(fraction: Float, startValue: Point?, endValue: Point?): Point {
        if (startValue == null || endValue == null) {
            return Point(0f, 0f)
        }

        return Point(	
            fraction * (endValue.x - startValue.x),
            fraction * (endValue.y - startValue.y)
        )
    }
}
// 使用
val animator= ValueAnimator.ofObject(	// 从左上角斜向移动到右下角
    PointEvaluator(), 
    Point(0f, 0f),//动画开始属性值
    Point(
        ScreenUtils.getScreenWidth(this).toFloat(),
        ScreenUtils.getScreenHeight(this).toFloat()
    )//动画结束值
)
animator.addUpdateListener {//手动更新TextView的x和y 属性
val point = it.animatedValue as Point
	tvText.x = point.x
    tvText.y = point.y 
    logError("point:${point}")
}
animator.duration = 5000
btnStart.setOnClickListener {
    animator.start()
}
```

#### 动画使用问题

1. OOM：过多&过大的图片资源[逐帧动画]
2. MemoryLeak：动画被设为无限循环播放repeatCount="infinite"，且Activity退出时未停止，持有Activity引用[属性动画]
3. 兼容性：属性动画在API11后引入
4. 屏幕适配：避免使用px，否则导致动画效果在不同设备上不相同
5. 与动画元素交互：视图动画不改变属性，属性动画改变属性，注意交互位置区域

#### 区别

1. Animation | Animator
   - Animation 是针对视图外观的动画实现，动画被应用时外观改变但视图的触发点不会发生变化，还是在原来定义的位置。 (也就是作用于视图动画)
   - Animator  是针对视图属性的动画实现，动画被应用时对象属性产生变化，最终导致视图外观变化。（也就是作用于属性动画）
2. AnimatorSet | PropertyValuesHolder
   - AnimatorSet可以将作用于多个view多个属性的动画集合起来，而PropertyValuesHolder针对同一个对象多个属性。
   - AnimatorSet多了playTogether（同时执行）、playSequentially（顺序执行）、play().with()、before、after这些方法协同工作。

