### 1.四大组件

#### Activity

> 一个应用程序的显示组件，绘制用户界面的窗口，在屏幕上提供一个区域，可填满整个屏幕，也可浮动或小窗，允许用户进行交互操作

1. 四种启动模式与任务栈(Task)

   > 在AndroidManifest.xml -> <activity>中设置启动模式 - android:launchMode="singleInstance/singleTop/singleTask"
   >
   > standard、singleTop、singleTask针对的任务栈都是当前对应的app进程的，而singleInstance指向整个系统。

   1. 任务栈Task    用于存放Activity组件，LIFO，一个任务栈包含了Activity的集合，系统可通过Task有序管理Activities，只有栈顶Activity可以跟用户进行交互。

   2. Standard    默认配置的标准启动模式，每次启动一个Activity都会创建一个实例，无论其是否已存在于栈中。

   3. SingleTop    栈顶复用模式，假设ActivityA已位于栈顶，此时调用onNewIntent再创建ActivityA时将复用该ActivityA不再创建新实例

      > 应用场景：当前要跳转的页面已经在栈顶时，如消息通知跳转

   4. SingleTask    栈内复用模式，假设ActivityA已位于栈内，此时再创建ActivityA时将复用该ActivityA且将其上方的Activity全部出栈

      > 应用场景：有一个专用主页面作为基础的app，如Activity+ViewPager+多Fragment的情况。
      >
      > ​	> 如果其他APP进程开启了Activity1，此时会创建新的任务栈
      >
      > ​	> 如果以该模式启动的Activity1已经活动在后台的某任务栈中，启动后后台任务栈会一起切换到前台

   5. SingleInstance    全局唯一模式，采用一个单独的Task栈管理该Activity，具有全局唯一性，主要用于解决多应用程序共享Activity的问题。

      > 应用场景：系统内部应用如电话、短信等，通过Intent传播时会固定的调用这些应用。

2. 创建Activity的方式

   1. 自定义Activity类名，继承AppCompatActivity
   2. 重写onCreate()方法，调用setContentView()设置要显示的视图
   3. 在AndroidManifest.xml中注册并配置该Activity
   4. 启动Activity，startActivity(intent)
   5. 关闭Activity，调用finish()或手机返回键、home键

3. 生命周期管理

   > 运行状态、暂停状态、停止状态、销毁状态，根据回调方法划分为三个生存期：
   >
   > **完整生存期** Activity在onCreate()和onDestroy()之间的生存期。
   >
   > **可见生存期** Activity在onStart()和onStop()之间的生存期。
   >
   > **前台生存期** Activity在onResume()和onPause()之间的生存期。

   1. onCreate()    首次创建Activity时调用
   2. onStart()    在Activity即将对用户可见之前调用
   3. onResume()    在Activity即将开始与用户交互之前调用
   4. onPause()    当系统即将开始另一个Activity时调用
   5. onStop()    当Activity对用户不再可见时调用
   6. onDestroy()    在Activity被销毁前调用
   7. onRestart()    当Activity已停止并即将再次启动前调用

   ![](D:\Users\80350694\Desktop\文档输出\res\20210830152420851.png)

4. 生命周期方法回调实例

   1. 按下返回键

      - 启动应用 onCreate - onStart - onResume
      - 按下返回键 onPause - onStop - onDestroy
      - 从近期任务再次启动 onCreate - onStart - onResume

   2. 按下Home建

      - 启动应用 同上
      - 按下Home键 onPause - onStop
      - 从近期任务再次启动 onRestart - onStart - onResume

   3. 按下Home键后调整字体大小

      - 启动应用 同上
      - 按下Home键 同上
      - 调整字体大小 无回调
      - 从近期任务再次启动 onDestroy - onCreate - onStart - onResume

      >Configuration改变：屏幕旋转、字体大小改变、语言变化、键盘显隐、切换主题/字体 -> 资源配置改变 -> 重启应用onDestroy - onCreate
      >
      >应对措施：
      >
      >- 保存和恢复	onSaveInstanceState()/onRestoreInstanceState()
      >- configChanges    AndroidManifest中声明Configuration改变时的处理，再重写onConfigurationChanged()方法自定义处理

5. 如何提升进入页面的速度
   1. 应用启动时间：从点击图标开始，创建一个进程，直到看见界面
   2. 应用启动关键流程：创建进程 - 创建、初始化Application类 - 创建Activity类 - **onCreate** - 配置主题信息 - **onStart** - **onResume** - **measure/layout/draw** - 显示
   3. 优化措施：
      1. 耗时任务异步处理
      2. 布局优化    减少布局层次、Merge标签、Viewstub、自定义控件
      3. 不可见视图延时加载    资源分开初始化、addView、页面分开加载
   
6. 内存泄漏问题

   `static、Handler、ServiceConnection`等在Activity中引发的内存泄漏问题，常基于生命周期解决，即`onDestroy`中进行置空、解绑等。

#### Service

> 在后台运行的组件，用于执行长时间运行的操作或为远程进程执行作业，可不提供界面。Activity等其他组件可启动服务并使服务运行或绑定到该组件以便进行交互。
>
> Service运行在主线程，在ANR机制中，Service响应时长不能超过20s，并不能进行耗时操作，可如果使用Thread异步处理，则可进行耗时操作。

1. 生命周期回调

   1. onCreate 创建Service时调用，完成一次性设置
   2. onStartCommand 通过startService启动服务时调用，一旦执行，服务便可后台无限期运行，用于计数，服务被调用的次数
   3. onBind 通过bindService与服务绑定时调用，通过返回IBinder提供一个接口供客户端与服务进行通信
   4. onUnbind 客户端与服务端解绑后调用
   5. onDestroy 在Service被销毁前调用，清理所有资源如注册的监听器、接收器等

   ![](D:\Users\80350694\Desktop\文档输出\res\20210830153913788.png)

2. 生命周期管理
   
   1. startService()    独立于Activity运行，保持一个单例，自行结束或被叫停
   2. bindService()    绑定于Activity运行，Activity结束时会被叫停
   3. 被启动又被绑定的Service，将一直在后台运行，保持单例，必须显式stop停止，unbind不会停止Service。
   4. 调用`bindService`启动Service时应保证在某处会调用`unbindService`解绑；调用`startService`一定要保证`stopService`出现。
   
3. 绑定Service
   1. 扩展Binder类    使用场景：应用程序私用，并于客户端运行同一进程
      1. 服务端提供客户端可调用的方法
      2. 扩展类返回服务实例
   2. Messenger    使用场景：不同进程间通信；服务每次处理一个请求
      1. 客户端使用IBinder将Message实例化
      2. Message对象向服务发送命令
   3. AIDL文件    使用场景：不同进程间通信；服务每次处理多个请求
      1. android接口定义语言
      2. 将对象解析为系统可识别的形态

4. 使用方法

   Service组件需要在AndroidManifest中注册

   ``` java
   <service android:name=".LocalService"/>	// 1.在AndroidManifest中注册
   startService(Intent);	// 2.启动Service
   bindService(Intent, ServiceConnection, Int);
   unBindService(ServiceConnection);	// 3.解绑，绑定启动时需要成对出现
   stopService(Intent);	// 4.暂停，开始启动时需要成对出现
   ```

5. 与Activity通信

   基于`IBinder`实现，通常使用其实现类`Binder`并通过强制转换完成操作。

   ``` java
   bindService(Intent, ServiceConnection, Int);
   
   public class LocalService extends Service {
       private final IBinder binder = new ServiceBinder();
       @Nullable
       @Override
       public IBinder onBind(Intent intent) {
           return binder;
       }
       public class ServiceBinder extends Binder {
           LocalService getLocalService() {
               return LocalService.this;
           }
       }
   }
   
   ServiceConnection connection = new ServiceConnection() {	// // bindService()方法中的参数之一，用于对Service进行操作
       // Activity和Service绑定时调用
       @Override
       public void onServiceConnected(ComponentName name, IBinder binder) {
           // 基于Binder拿到我们要的Service
           service = ((LocalService.ServiceBinder)binder).getLocalService();
           // 干你需要干的事情
       }
       // Activity和Service解绑时调用
       @Override
       public void onServiceDisconnected(ComponentName name) {
           service = null;
       }
   };
   /* Int:	- BIND_AUTO_CREATE    收到绑定需求，若Service尚未创建则立即创建
    - BIND_DEBUG_UNBIND    用于测试使用
    - BIND_NOT_FOREGROUND     不允许此绑定将目标服务进程提升到前台调度优先级
    一般使用BIND_AUTO_CREATE
   */
   ```

#### BroadcastReceiver

> 广播，应用程序间的全局喇叭，通信的手段，如发生电量低、插入耳机等事件系统发出广播，能响应该广播的程序将做相应动作。
>
> 广播同样可引发ANR，耗时操作不允许超过10s，且广播内一般不会使用Thread完成耗时操作
>
> 若需要网络、电池等服务就需要全局广播；若只需要应用内通信则只需要应用内广播
>
> 应用内广播LocalBroadcast使用Handler的消息传输机制；全局广播Broadcast使用Binder机制

1. 注册广播

   1. 动态注册    非常驻型，依赖注册组件的生命周期，注册/注销需要成对出现，灵活，但必须在程序启动后才能接收广播

      ``` kotlin
      inner class TimeChangeReceiver() : BroadcastReceiver(){	// 继承BroadcastReceiver并重写onReceiver()
          override fun onReceive(context: Context?, intent: Intent?){
              Toast.makeText(context,"Time has changed.",Toast.LENGTH_SHORT).show()
          }
      }
      lateinit var timeChangeReceiver : TimeChangeReceiver	//Receiver对象
      override fun onCreate(savedInstanceState: Bundle?) {
          super.onCreate(savedInstanceState)
          setContentView(R.layout.activity_main)
          val intentFileter = IntentFilter()	// 需要监听的广播对应的action
          intentFileter.addAction("android.intent.action.TIME_TICK")
          timeChangeReceiver = TimeChangeReceiver()	
          registerReceiver(timeChangeReceiver,intentFileter)	//注册该广播接收器
      }
      override fun onDestroy() {
          super.onDestroy()
          unregisterReceiver(timeChangeReceiver)	//取消注册该广播接收器
      }
      ```

   2. 静态注册    在AndroidManifest中注册，常驻型，程序关闭亦可激活广播，由于存在安全性、流畅性问题，8.0系统后，所有隐式广播都不允许使用静态注册方式接收(隐式广播：未具体指定发送给哪个应用程序的广播)

      ``` xml
      <receiver android:name=".MyBroadcastReceiver"
                android:enabled="true"
                android:exported="true"
                <intent-filter>
      			<action android:name="BROADCASTRECEIVER"/>
      		  </intent-filter>
      </receiver>
      ```

2. 发送广播

   1. 无序广播(标准广播)`sendBroadcast`   完全异步执行，广播发出后，所有注册了该广播的Receiver都会同时受到该广播信息，没有先后顺序，无法被截断，效率较高。

      ``` java
      Intent intent = new Intent("com.example.broadcastreceiver.test");
      intent.setComponent(new ComponentName(getPackageName(), MyBroadcastReceiver.class.getName()));
      sendBroadcast(intent, null);
      ```

   2. 有序广播`sendOrderedBroadcast`    同步执行，同一时刻只会有一个Receiver接收广播，串行接收，高优先级Receiver可以截断、二次处理广播，使低优先级Receiver无法收到该广播、收到处理过的信息。

   3. 区别    静态广播：广播一直存在，消耗资源较大，耗电量大    动态广播：广播的生命周期较灵活，资源消耗少，响应速度快

3. 生命周期

   若使用静态广播进行注册，则每接受一次信息就会被销毁，下一次需要重建；动态广播则与其注册处的Avtivity具体操作有关。

4. 应用内广播     LocalBroadcastManager

   发送的广播只会在App内传播，不泄露给其他App，保障数据安全性；无法接收到其他App的广播；相较于全局广播效率更高

   ``` java
   networkReceiver = new NetworkReceiver();	// 注册
   localBroadcastManager = LocalBroadcastManager.getInstance(this); // 以单例模式进行创建
   localBroadcastManager.registerReceiver(networkReceiver, new IntentFilter("需要去过滤的信息"));
   localBroadcastManager.sendBroadcast(Intent);	// 发送消息
   localBroadcastManager.unregisterReceiver(networkReceiver);	// 注销
   ```


#### ContentProvider

#### Intent

> Intent是各组件交互的一种重要方式，不仅可以指明当前组件想执行的动作，也可以在不同组件间传递数据。
>
> Activity、Service、BroadcastReceiver间的通信使用Intent

1. Intent类型

   1. 显式Intent    知道要启动的组件名，只有一个组件能处理该Intent 

      ``` kotlin
      Intent(Context packageContext, Class<?> cls)
      // param1:启动Activity的上下文 | param2:指定想要启动的Activity
      Context.startActivity(intent: Intent)
      ```

   2. 隐式Intent    声明要执行的常规操作，允许多个应用响应该Intent

      > 并不明确指出想要启动哪一个Activity，而制定一系列更抽象的action、category信息，交由系统分析该Intent，找到合适的Activity启动。
      >
      > 在配置文件中某activity内使用<intent-filter>标签添加想要该activity相应的action、category
      >
      > 可用于启动其他程序的Activity，如在本程序打开浏览器等。只需要指定希望响应该Intent的action、category等，在startActivity(intent)后，可以响应该Intent的所有应用程序都会给出答复。

   3. 启动Service必须使用显式Intent，隐式Intent启动Service存在安全隐患（无法确定哪些服务会响应Intent，且用户无法知悉哪些服务已启动），Android5.0开始使用隐式Intent调用bindService()，系统将引发异常、

2. 隐式Intent的匹配规则

   1. Action   Intent指定的Action必须与过滤器中某个Action匹配
   2. Category    Intent的每个Category必须与过滤器中某个Category匹配
   3. Data（包括URL和数据类型）URL和数据类型是否指定、是否匹配分为4种匹配类型

3. Intent的信息传递

   1. 基础类型    8种基本数据类型+String+CharSequence

   2. 传递复杂对象

      1. Serializable    序列化，将一个对象转换为可存储或可传输状态
         - 保存对象到本地文件、数据库、网络流；只需要实现Serializable接口；数据持久化保存；数据存储在磁盘或网络传输数据
      2. Parcelable    解包打包，将一个完整对象分解，每一部分都是Intent支持的基础类型
         - 不同组件及不同程序间高效传输数据；需要自定义如何打包解包；性能更优；内存间数据传输
      3. 优先选择Parcelable接口，有数据存储或网络传输需求再考虑Serializable接口

   3. 传递数据的风险

      1. 传递数据量过大，如list.size过大，可能导致ActivityB无法启动，抛出TransactionToolLargeException异常
         - 限制传递的数据量（多个传输对象共用1M的缓冲区）
         - 调整数据传输方式（static、单例、数据库）
      2. 接受端无法创建对象
         - 将类作为公共类放入系统中
         - 封装为jar包

   > Intent中使用putExtra存放的数据最终封装进入`Bundle`，即Activity间数据传递的媒介

   ``` java
   Intent intent = new Intent(this, SecActivity.class);	// 定义要跳转的目标Activity
   intent.putExtra("parameter key", "parameter value");	// 将数据放入Intent
   startActivity(intent);	//不带回调方法的启动
   startActivityForResult(intent, REQUEST_CODE);	//带回调方法的启动
   // 如果要从目标Activity中接收返回的数据，就必须重写onActivityResult方法，根据requestCode判断是否为本Activity发出的启动请求。
   @Override
   protected void onActivityResult(int requestCode, int resultCode, @Nullable Intent data) {
       if(requestCode == REQUEST_CODE) {
           // 进行相应的处理
       }
       super.onActivityResult(requestCode, resultCode, data);
   }
   // 从Intent中拿取参数
   getIntent().getStringExtra(key);
   getIntent().getBooleanExtra(key);
   // 通过setResult进行数据的回调
   setResult(resultCode);
   setResult(resultCode, intent);
   // 目标Activity重写onKeyDown，设置返回跳转Activity的数据
   @Override
   public boolean onKeyDown(int keyCode, KeyEvent event) {
       if (keyCode == KeyEvent.ACTION_DOWN && event.getAction() == KeyEvent.ACTION_DOWN) {	//按下返回键
           Intent intent = new Intent();
           intent.putExtra("parameter key", "parameter value");
           // setResult(resultCode);
           setResult(resultCode, intent);	// 携带数据的方式
       }
       return super.onKeyDown(keyCode, event);
   }
   ```

#### Fragment

> 一种嵌入在Activity中的UI片段，主要用于兼顾平板与手机不同屏幕尺寸的自适应性，也可用于减轻Activity，提高加载效率。

#### 消息机制

1. 相关名词

   1. UI线程：即主线程，系统创建UI线程时会初始化一个`Looper`对象，同时创建与之关联的`MessageQueue`
   2. Handler：发送与处理信息，当前线程必须有`Looper`对象才可正常工作
   3. Message：Handler接受与处理的消息对象
   4. MessageQueue：消息队列，FIFO管理Message，初始化`Looper`对象时会创建。
   5. Looper：每个线程唯一，管理`MessageQueue`，不断从中取出Message分发给对应Handler处理

   > 当子线程想修改Activity的UI组件时，新建一个Handler对象，通过该对象向主线程发送信息；信息Message先进入主线程MessageQueue进行等待，由Looper按序取出Message进行处理，根据Message的`what`属性分发给相应Handler。

#### 事件分发机制

1. 事件序列解析    `点击/滑动`

   > MotionEvent/事件类型 - 具体操作
   >
   > ACTION_DOWN - 点下View
   >
   > ACTION_UP - 抬起View
   >
   > ACTION_MOVE - 滑动View
   >
   > ACTION_CANCEL - 非人为因素取消
   >
   > 事件序列的一般组成
   >
   > ​	点击： Down -> Up
   >
   > ​	滑动： Down -> Move -> Move -> Up

2. 事件分发

   1. 使用函数
       1. dispatchTouchEvent    用于事件分发
       2. onTouchEvent    消费事件
       3. onInterceptTouchEvent    判断是否拦截事件，仅存在于ViewGroup
    2. 分发对象    Activity、ViewGroup、View

### 2.UI组件

#### ViewGroup

1. LinearLayout

   线性布局，控件在线性方向上依次排列，orientation指定线性方向为vertical/horizontal，layout_gravity指定布局中对齐方式，layout_weight使用比例指定控件大小

2. RelativeLayout

   相对布局，通过相对定位方式指定位置。

3. FrameLayout

   帧布局，所有控件都默认摆放在布局的左上角。

4. ConstraintLayout

#### View

visibility: visible表示可见；invisible表示不可见，但任然占据位置与大小；gone表示不可见且不占据任何空间

1. TextView

   1. gravity指定文字的对齐方式，可以用"|"指定多个值。
   2. textColor/textSize等等...

2. Button

   1. textAllCaps="false" 指定系统保留原始文字，而不是全部转化为大写[Button组件]
   2. 可以使用单抽象方法接口+Lambda表达式注册点击监听事件，也可以通过实现接口方式注册。

3. EditText

   1. maxLines指定EditText的最大行数，保证拉伸长度限制。

4. ImageView

   1. 调用setImageResource(Id)动态更改图片。

5. ProgressBar

   1. 用于显示一个进度条，表示正在加载一些数据。
   2. style="?android:attr/progressBarStyleHorizontal"指定为水平进度条，android:max指定最大值
   3. 可根据progressBar.progress属性动态更改进度条进度。

6. AlertDialog

   1. 在当前界面弹出一个对话框，置顶于所有元素之上，用于提示一些重要内容或警告信息。

   ``` kotlin
   AlertDialog.Builder(this).apply {
       setTitle("This is Dialog")	//标题
       setMessage("Something important")	//信息内容
       setCancelable(false)	//不可取消
       setPositiveButton("OK"){
           dialog, which ->
       }
       setNegativeButton("Cancel"){
           dialog, which ->
       }
       show()
   }
   ```

#### 常用组件

1. RecyclerView

   > ListView的扩展性较差、运行效率偏低，只能实现纵向滚动效果，无法横向滚动。
   >
   > RecyclerView相对于ListView的扩展性提升源于对布局排列的管理。ListView自身管理布局排列，而RecyclerView令LayoutManager代为管理。LayoutManager提供了一系列扩展接口比如GridLayoutManager网格布局、StaggeredGridLayoutManager瀑布流布局等，只需要实现相应接口就能定制不同布局。

   1. 定制Adapter

      ``` kotlin
      class FruitAdapter(val fruitList:List<Fruit>) : RecyclerView.Adapter<FruitAdapter.ViewHolder>() {
      	/*
          内部类ViewHolder继承自RecyclerView.ViewHolder，用于缓存子项控件
           */
          inner class ViewHolder(view : View) : RecyclerView.ViewHolder(view){
              val fruitImage : ImageView = view.findViewById(R.id.fruitImage)
              val fruitName : TextView = view.findViewById(R.id.fruitName)
          }
          /*
          用于创建ViewHolder实例，首先渲染相应布局文件，随后将View实例传入ViewHolder
           */
          override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
              val view = LayoutInflater.from(parent.context).inflate(R.layout.fruit_item,parent,false)
              return ViewHolder(view)
          }
          /*
          用于对RecyclerView的子项赋数据值，在每个子项被滚动到屏幕内时执行
           */
          override fun onBindViewHolder(holder: ViewHolder, position: Int) {
              val fruit = fruitList[position]
              holder.fruitName.text = fruit.name
              holder.fruitImage.setImageResource(fruit.imgId)
          }
      	override fun getItemCount(): Int {
              return fruitList.size
          }
      }
      ```

### 3.数据存储与持久化

#### 文件存储

> 最基本的数据存储方式，不对存储内容进行任何格式化处理，适合存储简单的文本数据或二进制数据。若用此方式存储较复杂结构化数据，则需要自定义一套格式规范。

Context类提供了一个openFileOutput(fileName,MODE)方法用于将数据存储到指定文件中，操作模式有MODE_PRIVATE和MODE_APPEND两种，分别为覆盖和追加方式。[MODE_WORLD_READABLE和MODE_WORLD_WRITEABLE两种方式安全漏洞较大已废弃。]

``` kotlin
// 将数据存储到指定文件种
String inputText = "xxxxxxxx"
try{
    val output = openFileOutput("data",Context.MODE_PRIVATE)	// openFileOutput返回FileOutputStream对象，使用JavaIO完成文件数据写入
    val writer = BufferedWriter(OutputStreamWriter(output))	// 使用字符->字节转换流 获得一个带缓冲的字符输出流对象writer	
    writer.use{	// 内置的扩展函数，调用者作为上下文，可直接使用it，会保证执行完毕后自动关闭外层流。
        it.write(inputText)
    }
}catch( e: IOException){
    e.printStackTrace()
}
```

#### SharedPreference

#### 数据库存储

### 4.网络库与异步请求

#### Retrofit

#### RxJava

### 5.Jetpack框架

#### ViewModel

#### ViewBinding

#### LiveData/LiveCycle

### 6.项目架构

#### MVC

#### MVP

#### MVVM

### 7.获取控件方式变化

#### findViewById

#### ButterKnife

#### Kotlin-Android-Extensions

#### ViewBinding

