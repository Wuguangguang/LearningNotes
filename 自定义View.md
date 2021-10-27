### 自定义View

#### 1.储备知识

##### 1.1 ViewRoot

- 定义连接器，对应`ViewRootImpl`类

- 链接`WindowManager`和`DecorView`；完成View的绘制流程
- 具体描述：与WMS交互通讯，调整窗口大小及布局；向DecorView派发输入事件；完成三大绘制流程：`measure、layout、draw`

##### 1.2 DecorView

- 顶层View，即Android视图树的根节点，同时也是`FrameLayout`的子类
- 显示&加载布局，View层事件都先经过DecorView，再传递到View

- DecorView内含一个竖直方向的LinearLayout，分为titlebar|content两部分

  > Activity中通过`setContentView()`所设置的布局文件被加载到Content中，即DecorView的唯一子View(id = content)的FrameLayout中

  ``` java
  // 在代码中可通过content得到对应加载的布局
  // 1. 得到content
  ViewGroup content = (ViewGroup)findViewById(android.R.id.content);
  // 2. 得到设置的View
  ViewGroup rootView = (ViewGroup) content.getChildAt(0);
  ```

##### 1.3 Window

- 承载器，承载视图View的显示
- 具体描述：Window类=抽象类，PhoneWindow类=实现类；PhoneWindow类中有内部类DecorView=View的根布局；通过创建DecorView加载Activity中设置的布局
- 特别注意：Window类通过WindowManager将DecorView加载其中；将DecorView交给ViewRoot，及性能视图绘制、其他交互

##### 1.4 Activity

- 控制器，控制生命周期&处理事件
- 统筹视图的添加显示；通过其他回调与Window|View交互
- 不负责视图控制；由Window真正控制视图；1个Activity包含1个Window

##### 1.4 Window|Activity|DecorView|ViewRoot

![](D:\Users\80350694\Desktop\文档输出\res\VIew四部分的关系.webp)

##### 1.5 自定义View基础

1. 视图定义

   日常说的View，具体表现在显示在屏幕上的各种视图控件，如TextView、LinearLayout等

2. 视图分类

   - 单一视图：不包含子View的单一View，如TextView
   - 视图组：由多个View组成的ViewGroup，如LinearLayout

3. 视图类简介

   - View类是Android各种组件的基类
   - View的4个构造函数：

   ``` java
   // 构造函数1
   // 调用场景：View是在Java代码里面new的
   public View(Context context) {
       super(context);
   }
   // 构造函数2
   // 调用场景：View是在.xml里声明的
   // 自定义属性是从AttributeSet参数传进来的
   public View(Context context, AttributeSet attrs) {
       super(context, attrs);
   }
   // 构造函数3
   // 应用场景：View有style属性时
   // 一般是在第二个构造函数里主动调用；不会自动调用
   public View(Context context, AttributeSet attrs, int defStyleAttr) {
       super(context, attrs, defStyleAttr);
   }
   
   // 构造函数4
   // 应用场景：View有style属性时、API21之后才使用
   // 一般是在第二个构造函数里主动调用；不会自动调用
   public View(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes) {
       super(context, attrs, defStyleAttr, defStyleRes);
   }
   ```

4. 视图结构

   - 对于包含子View的视图组，结构为树形
   - ViewGroup下可能有多个ViewGroup|View

   在View的绘制过程中，**永远从View树结构的根节点开始(即从树的顶端开始)，一层一层、一个个分支地自上而下遍历进行（即树形递归）**，最终计算整个View树中各个View，从而最终确定整个View树的相关属性。

5. Android坐标系

   - 屏幕左上角为坐标原点
   - 向右为x轴增大方向
   - 向下为y轴增大方向

6. View位置描述

   - 视图的位置是相对于父控件而言的，四个顶点的位置描述分别由四个与父控件相关的值决定：Top|Left|Right|Bottom

#### 2.绘制准备

##### 2.1 DecorView的创建

DevorView是Window内的顶层View，绘制准备从创建DecorView开始

``` java
/**
  * 具体使用：Activity的setContentView()
  */
  @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }
/**
  * 源码分析：Activity的setContentView()
  */
   public void setContentView(int layoutResID) {
        // getWindow() 作用：获得Activity 的成员变量mWindow ->>分析1
        // Window类实例的setContentView（） ->>分析2
        getWindow().setContentView(layoutResID);
        initWindowDecorActionBar();
   }

/**
  * 分析1：成员变量mWindow
  */
  // 1. 创建一个Window对象（即 PhoneWindow实例）
  // Window类 = 抽象类，其唯一实现类 = PhoneWindow
  mWindow = new PhoneWindow(this, window);
  // 2. 设置回调，向Activity分发点击或状态改变等事件
  mWindow.setWindowControllerCallback(this);
  mWindow.setCallback(this);
  // 3. 为Window实例对象设置WindowManager对象
    mWindow.setWindowManager(
        (WindowManager)context.getSystemService(Context.WINDOW_SERVICE),
        mToken, mComponent.flattenToString(),
        (info.flags & ActivityInfo.FLAG_HARDWARE_ACCELERATED) != 0);

/**
  * 分析2：Window类实例的setContentView（）
  */
  public void setContentView(int layoutResID) {
        // 1. 若mContentParent为空，创建一个DecroView
        // mContentParent即为内容栏（content）对应的DecorView = FrameLayout子类
        if (mContentParent == null) {
            installDecor(); // ->>分析3
        } else {
            // 若不为空，则删除其中的View
            mContentParent.removeAllViews();
        }

        // 2. 为mContentParent添加子View
        // 即Activity中设置的布局文件
        mLayoutInflater.inflate(layoutResID, mContentParent);
        final Callback cb = getCallback();
        if (cb != null && !isDestroyed()) {
            cb.onContentChanged();//回调通知，内容改变
        }
    }

/**
  * 分析3：installDecor()
  * 作用：创建一个DecroView
  */
  private void installDecor() {
    if (mDecor == null) {
        // 1. 生成DecorView ->>分析4
        mDecor = generateDecor(); 
        mDecor.setDescendantFocusability(ViewGroup.FOCUS_AFTER_DESCENDANTS);
        mDecor.setIsRootNamespace(true);
        if (!mInvalidatePanelMenuPosted && mInvalidatePanelMenuFeatures != 0) {
            mDecor.postOnAnimation(mInvalidatePanelMenuRunnable);
        }
    }
    // 2. 为DecorView设置布局格式 & 返回mContentParent ->>分析5
    if (mContentParent == null) {
        mContentParent = generateLayout(mDecor); 
        ...
        } 
    }
}

/**
  * 分析4：generateDecor()
  * 作用：生成DecorView
  */
  protected DecorView generateDecor() {
        return new DecorView(getContext(), -1);
    }
    // 回到分析原处

/**
  * 分析5：generateLayout(mDecor)
  * 作用：为DecorView设置布局格式
  */
  protected ViewGroup generateLayout(DecorView decor) {
        // 1. 从主题文件中获取样式信息
        TypedArray a = getWindowStyle();
        // 2. 根据主题样式，加载窗口布局
        int layoutResource;
        int features = getLocalFeatures();
        // 3. 加载layoutResource
        View in = mLayoutInflater.inflate(layoutResource, null);
        // 4. 往DecorView中添加子View
        // 即文章开头介绍DecorView时提到的布局格式，那只是一个例子，根据主题样式不同，加载不同的布局。
        decor.addView(in, new ViewGroup.LayoutParams(MATCH_PARENT, MATCH_PARENT)); 
        mContentRoot = (ViewGroup) in;
        // 5. 这里获取的是mContentParent = 即为内容栏（content）对应的DecorView = FrameLayout子类
        ViewGroup contentParent = (ViewGroup)findViewById(ID_ANDROID_CONTENT); 
        return contentParent;
    }
```

1. 创建实现类PhoneWindow的实例对象
2. 为PhoneWindow类对象设置WindowManager对象
3. 为PhoneWindow类对象创建1个DecorView类对象(根据所选主题样式)
4. 为DecorView类对象中content增加Activity设置的布局文件

此时，DecorView(即顶层View)已创建和添加Activity中设置的布局文件，但目前仍不可见。

##### 2.2 DecorView的显示

1. 将DecorView对象添加到WindowManager中
2. 创建ViewRootImpl对象;
3. WindowManager将DecorView对象交给ViewRootImpl对象
4. ViewRootImpl对象通过Handler向主线程发送了一条触发遍历操作的消息：performTraversals()；该方法用于执行View的绘制流程（measure、layout、draw）。

ViewRootImpl对象中接收的各种变化（如来自WmS的窗口属性变化、来自控件树的尺寸变化、重绘请求等）都引发performTraversals()的调用及完成相关处理，并最终显示到可见的Activity中。

#### 3.绘制流程概述

View的绘制流程开始于：`ViewRootImpl`对象的`performTraversals()`方法。**`View`的绘制流程从顶级`View（DecorView）`的`ViewGroup`开始，一层一层从`ViewGroup`至子`View`遍历测绘**

``` java
/**
  * 源码分析：ViewRootImpl.performTraversals()
  */
  private void performTraversals() {
  		// 1. 执行measure流程
        // 内部会调用performMeasure()
        measureHierarchy(host, lp, res,desiredWindowWidth, desiredWindowHeight);
        // 2. 执行layout流程
        performLayout(lp, mWidth, mHeight);
        // 3. 执行draw流程
        performDraw();
    }
```

#### 4.详细介绍

##### 4.1 Measure

- 测量View的宽|高

  > 1. 某些情况下，需要多次测量才能确定View的最终宽高
  > 2. 该情况下，测量后得到的宽高可能不准确
  > 3. 建议在layout过程中`onLayout`获取最终宽高

- 具体流程：单一View只测量自身一个View；ViewGroup遍历调用子元素的measure()，各子元素再递归该流程，合并所有子View的尺寸得到ViewGroup的测量值

##### 4.2 Layout

- 计算视图的位置（计算四顶点位置）
- 具体流程：单一VIew只计算本身位置；ViewGroup计算本身位置后遍历子View计算位置，最终确定所有子View在父容器的位置

##### 4.3 Draw

- 绘制View视图
- 具体流程：单一View只绘制自身；ViewGroup绘制自身、遍历子View绘制、绘制装饰

#### 5.自定义View的步骤

##### 5.1 实现Measure|Layout|Draw流程

1. 单一View：重写onDraw，绘制View本身的内容
2. ViewGroup：
   1. onMeasure：1.遍历所有子View & 测量：measureChildren 2.合并所有子View尺寸，计算最终ViewGroup尺寸 3.存储测量后子View宽高：setMeasureDimension
   2. onLayout：1.遍历子View 2.计算当前子View的四个位置值 3.根据上述四个位置的计算值，设置子View的4个顶点
   3. onDraw：绘制父View本身的内容

##### 5.2 自定义属性

1. 在values目录下创建自定义属性的xml文件
2. 在自定义view的构造方法中加载自定义xml文件 & 解析属性值
3. 在布局文件中使用自定义属性

