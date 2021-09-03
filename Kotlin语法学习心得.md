### 变量与函数

#### 变量

1. 对变量只允许两种关键字：var、val，通过类型推导机制来判断变量的实际数据类型，支持null类型声明，使用`?`声明允许为null的情况。

   1. `val`    value，声明一个不可变的变量，该变量初始赋值后不可再改变，类似于final，线程安全，不需要访问控制。
   2. `var `   variable，声明一个可变的变量。

   > 使用val为了解决final不合理使用的问题，永远优先使用val声明变量，无法满足需求时再改为var。
   >
   > 好的编程习惯：除非一个变量明确允许被修改，否则都应该加上final限制。

   ``` kotlin
   val str = "Hello World."
   var test = 123
   var a: Int?	// 若对变量进行延迟赋值，则仍然需要显式的声明变量类型
   ```

2. 常量

   1. kotlin常量必须声明在对象或顶层中
   2. 新增修饰常量的`const`关键字
   3. 只有基本类型和String类型可以声明为常量

   ``` kotlin
   class Sample {
       companion object {
           const val CONST_NUMBER = 1	// 伴生对象
       }
   }
   const val CONST_SECOND_NUMBER = 2	// 顶层
   ```

3. 自动装箱不一定保持引用相等，但保证值相等，如Int类型在-128~127范围内引用相等。

   > 对数字**没有隐式拓宽转换**，必须进行明确的类型转换，如`toLong()`。

4. 区间变量Range

   ``` kotlin
   var range = 0..10 //表示一个[0,10]的区间
   var range = 0 until 10	//表示一个[0,10)的区间
   var range = 10 downTo 1	//表示一个[10,1]的降序区间
   var range = 0 until 10 step 2 //表示{0，2，4，6，8},step关键字跳过元素,每次循环递增2
   ```

5. `Any`是所有非空对象的父类，如Java.Object；Any?`是所有可空对象的父类。`

6. `Unit`类型类似于void，可用于函数无返回值的情况。Unit是一个完备类型，可作为类型参数，void不可以。

   ``` kotlin
   interface Test<T> {
   	fun test(): T
   }
   class NoResultClass: Test<Unit> {
       override fun test(): Unit { }
   }
   ```

7. 延迟初始化

   ``` kotlin
   private lateinit var adapter: MsgAdapter
   //lateinit关键字，不需要在后续使用中进行?判空处理，但有可能出现UninitialPropertyAccessException异常。使用时一定确保调用前已完成初始化。
   if( !::adapter.isInitialized) { }	//::adapter.isInitialized判断是否初始化的固定写法
   ```

#### 变量访问与判断

1. `==`比较数值相等性，`===`比较引用是否相等

   ``` kotlin
   val intV_1: Int = 200
   val intV_2: Int? = intV_1
   val intV_3: Int? = intV_1
   println(intV_2 == intV_3)	//true
   println(intV_2 === intV_3)	//false
   ```

2. `字符串内嵌表达式`：支持在字符串值中引用局部变量，只需要变量名前加上字符`$`，还可用{}进行表达式运算；可使用`索引运算符[]`访问包含的单个字符

   ``` kotlin
   val intV = 100
   println("intV value is $intV")
   println("(intV + 100) value is ${intV + 100}")
   println("${$}100.99")	//$100.99 表示$字符使用${$}
   val str = "leavesc"	
   println(str[1])	// 使用索引运算符[]
   ```

3. `Sequence`    又被称为惰性集合操作，`Sequence`可以在数据量较大或未知时，进行流式处理

   ``` kotlin
   /*
   惰性：	- 只有result被使用到时才会执行，之前的代码并不会立即执行
   	  - 当出现满足条件的首元素后，不会执行之后的元素遍历，即4不会被遍历
   println处理流程： 取出元素1 - map为2 - 判断2是否被3整除 ... 取出元素2 - map为4 - 判断4是否被3整除 ... 
   */
   val sequence = sequenceOf(1,2,3,4)
   val result: Sequence<Int> = sequence
   	.map { i -> 
       	println("Map $i")
           i * 2
       }
   	.filter { i -> 
       	println("Filter $i")
                i % 3 == 0
       }
   println(result.first())
   ```

   `Sequence`懒加载实现的优点：

   - 一旦满足遍历退出条件，就省略后续不必要的遍历过程
   - 像 `List` 这种实现 `Iterable` 接口的集合类，**每调用一次函数就会生成一个新的 `Iterable`**，下一个函数再基于新的 `Iterable` 执行，每次函数调用产生的临时 `Iterable` 会导致额外的内存消耗，而 `Sequence` 在整个流程中只有一个。

4. 原始字符串： 三引号"""括起来，内部没有转义且可以包含换行及任何其它字符，也支持字符串模板${s.length}\$a

   ``` kotlin
   val str = """{"gender":“男”,"name":"zhangsan"}"""
   //等价于
   val str = "{\"gender\":\"男\",\"name\":\"zhangsan\"}"
   ```

- 每一种数据类都提供了若干相应类，如`IntArray\ByteArray\BooleanArray`，将被编译为普通java数组`int[]\byte[]\boolean[]`等

  ``` kotlin
  val intArray = IntArray(5)
  val doubleArray = DoubleArray(5) { Random().nextDouble()}
  val charArray = charArrayOf('H','e','l')
  val array2 = arrayOfNulls<String>[10]	//包含元素均为null，只用来创建包含元素类型可空的数组
  ```

#### 函数

1. 用单行表达式与等号定义的函数叫做**表达式函数体**，可省略返回值类型；对于一般有返回值的**代码块函数体**，必须显式写出返回类型和return语句。	

   ``` kotlin
   //常用函数用法,代码块函数体
   fun methodName(param1: Int, param2: Int) : Int {
       return 0
   }
   //语法糖,不必编写函数体，直接将唯一一行代码用=替代return，表达式函数体
   fun largeNumber(num1: Int, num2: Int): Int = max(num1,num2)
   ```

### 常用关键字

####  `if-else`

``` kotlin
//完全等同于java的语法
if(num1 > num2) {
    value = num1
} else {
    value = num2
}
//返回值简化,类似于三目运算value = num1>num2 ? num1 : num2
val value = if(num1 > num2) num1 else num
```

#### `when`

1. 控制流：分支条件可以是任意表达式	2.表达式：符合条件的分支的值就是整个表达式的值,类似if、try catch

``` kotlin
//类似于switch语句，但允许传入任意类型的参数，类型匹配、
when (value) {
    in 4..9 -> println("in 4..9") //区间判断
    3 -> println("value is 3")    //相等性判断
    2, 6 -> println("value is 2 or 6")    //多值相等性判断
    is Int -> println("is Int")   //类型判断
    else -> println("else")       //如果以上条件都不满足，则执行 else
}
```

#### 循环语句

1.where循环与java语法完全相同	2.for-i循环直接被舍弃，而将for-each变为了for-in循环

``` kotlin
for(i in 0..10) {
    println(i)
}
/
for(i in 0 until 10 step 2) {
    println(i) //0.2.4.6.8
}
// for循环通过索引遍历
for(index in items.indices) {
	println("${items[inedx]}")
}
```

#### 空安全

Kotlin将空指针异常检查提前到编译期，若代码存在异常风险则编译报错。

2. `?.`：安全调用运算符，把null检查和方法调用合并为一个操作

2. `?:`：Elvis运算符，用于替代`?.`直接返回默认值null的情况，若运算数1不为null，则结果为运算数1，否则为运算数2

5. `!!`：非空断言，确信该处对象不为空，跳过空指针检查，若该变量为null值则抛出NPE异常

``` kotlin
if(name != null) {
    println(name.toUpperCase())
}else {
    println(null)
}
// 等价于
println(name?.toUpperCase())
// ?:
if(name != null) {
    println(name)
}else {
    println("default")
}
// 等价于
println(name ?: "default")
```

#### `object`

1. 创建一个继承自某类型的匿名类对象，且该类内部可修改外部变量

   ``` kotlin
   fun showFragment() {
       var position = 0
       TransitionSet().addListener(object: TransitionListenerAdapter() {
           override fun onTransitionStart(transition: Transition?) {
               position++	//可访问、修改外部变量
               //java中 匿名内部类若需要访问、修改外部变量，需要使用final限制该类
           }
       })
   }
   ```

2. 匿名对象只有定义为局部变量、private成员变量时，才可体现其真实类型；匿名对象为public函数返回值或public属性时，只能将其视为Any，匿名对象中添加的属性、方法不可被访问

   ``` kotlin
   private fun foo() = object {
       val x:Int = 1
   }
   fun PublicFoo() = object {
       val x:Int = 1
   }
   fun test() {
       val adHoc = object {	//匿名局部对象变量
           var x:Int = 0
       }
       val x0 = adHoc.x
       val x1 = foo().x
       var x2 = publicFoo().x	//error	定义为public的匿名对象的属性、方法不可被访问
   }
   ```

3. 创建单例类对应于Java的饿汉式单例模式，始终线程安全

   ``` kotlin
   object Singleton { }
   ```

4. companion伴生对象，类比static的静态方法、成员，也可用顶层方法实现。

   - 一个类中只可定义一个`companion object`，可以直接使用类名引用伴生对象，伴生对象在类加载时初始化。对应于Java生成一个静态内部类
   - @JvmStatic注解将使方法编译为真正的静态方法，只能加在单例类或companion object的方法上。
   - 顶层方法指没有定义在任何类中的方法，编写在.kt文件中的方法都是顶层方法。

   > - 如果实现工具类功能，直接创建文件，写top_level顶层函数
   > - 如果需要继承其他类或实现接口，使用`object`或`companion object`

#### `data`

data 数据类，简化Java生成POJO的大量模板代码，自动生成hashCode、equals、componentx、copy、toString等方法。

``` kotlin
data class cellphone(val brand: String, val price: Double)
```

对于data数据类：

- 主构造函数需要至少一个参数，可以有默认值
- 主构造函数所有参数都需要标记为val、var，表示为该数据类的成员变量
- 数据类是final类，不能为抽象、开放、密封、内部的
- 数据类可以有超类

对于data数据类由kotlin编译器自动从主构造函数中声明的所有属性导出的方法：

- equals|hashcode    对象比较
- toString    输出对象字符串`User(name=john,age=42)`
- componentN    按声明顺序对应于所有属性，可用于解构声明
- copy   复制一个对象改变一些属性，其余部分不变

``` kotlin
val jack = User("Jack",1)
val olderJack = jack.copy(age = 2)	//copy一个对象，指定修改的部分
//componentN函数的解构声明
val (name,age) = jack
println("$name,$age years of age")	//Jack,1 years of age
val (name,_) = jack	//只取某参数，可以使用_占位符替代。
val (name) = jack	
val (_,age) = jack
```

#### 类型的判断和强转

`as`：强制类型转换，类比Java.instanceOf；`as?`：安全的强制类型转换，避免了类型错误抛出异常

``` kotlin
class NewActivity: MainActivity() {
    fun action() {}
}
// java
void main() {
    Activity activity = new NewActivity();	// 子类对象指向父类引用，java中称为多态
    if(activity instanceof NewActivity) {
		((NewActivity)activity).action();
    }
}
// kotlin
fun main() {
    var activity: Activity = NewActivity()
    activity.action()	// 无法直接调用
    // 使用is关键字进行类型判断
   if(activity is NewActivity) {
       activity.action()	// 正常执行
   }
   // 使用as关键字直接进行强转调用
   (activity as NewActivity).action()	// 正常运行，但有强转错误类型的异常隐患
   // 使用as?关键字进行安全强转
   (activity as? NewActivity).action()	// 强转成功就执行，否则不执行任何操作
}
```



### 面向对象

#### 实例化对象

``` kotlin
val p = Person()	//不需要new关键字	
```

#### `open`

需要为基类加上open关键字，该类才可被子类继承。默认任何非抽象类都不可被继承。

> 如果一个类不是专门为继承设计，则应加上final声明，禁止其被继承

1. `override`重写方法有遗传性，可被其子类重写，函数可见性继承自父类，需要加上`final`关闭遗传

2. `open`没有遗传性，子类仍然需要加上`open`来表示可继承

``` kotlin
open class Person {
    var name = ""
    var age = 0
    fun eat() {
        println(name + "is eating. He is " + age + " years old.")
    }
}
```

#### 继承、接口

继承基类、接口都使用`:`关键字，使用`,`隔开，基类需要加括号，接口不需要。

``` kotlin
class Student : Person(),Comparable {	// 子类主构造函数调用父类中的构造函数，直接通过括号指定
    ...	
}
// 接口函数可默认实现
interface Study {
    fun readBooks()
    fun doHomework() {
        println("...")
    }
}
```

#### 主构造函数

最常用的构造函数，没有函数体，直接定义在类名后

如果在主构造函数的参数声明时加上 `var` 或者 `val`，就等价于在类中创建了该名称的属性，并且初始值就是主构造器中该参数的值。

``` kotlin
class Student(val sno: String, val grade: Int) : Person() { }
//init方法可以编写主构造函数中的逻辑
init{
    println("sno is " + sno)
    println("grade is " + grade)
}
```

#### 次构造函数

所有的次构造函数都必须调用主构造函数,通过constructor关键字定义

``` kotlin
class Student(val sno: String, name: String): Person(name) {
    constructor(name:String) : this("", name) { }
    constructor() : this("", "") { }
}
```

#### 可见性修饰符

1. public    所有类可见，Kotlin默认为public
2. protected  对当前类、子类可见
3. internal    Kotlin同一模块中的类可见
4. private    对当前类、同包下类可见



### Lambda表达式

>  Lambda是一小段可以作为参数传递的代码
>
> {参数名1：参数类型，参数名2：参数类型 -> 函数体}，最后一行代码会自动作为Lambda表达式的返回值

#### 使用方法

``` kotlin
// 创建一个Thread的完整写法
Thread(object: Runnable {
    override fun run() {
        ...
    }
})
// 满足SAM(Single Abstract Method)，单一方法接口，可简化匿名类写法
Thread({
    ...
})
// 使用闭包，当函数的最后一个参数为Lambda表达式时，可以将Lambda写在括号外
Thread {
    ...
}
//View.OnClickListener接口的函数式API写法
button.setOnClickListener(new View.OnClickListener() {
	@Override
	public void onClick(View v) {}
})
button.setOnClickListener {}
```

### 其它

#### 函数参数默认值

- 默认参数必须按照函数声明参数顺序给定，只有末尾参数可以省略。

- 命名参数可省略任何有默认值的参数，也可按任何顺序传入参数

``` kotlin
fun printParams(num: Int, str: String = "hello") { }
//键值对传参
printParams(str = "world", num = 123)
printParams(num = 444, "hello")	//error，指定了一个参数的名称后，之后的所有参数都需要标明名称
```

- 可变参数可以把任意个数的参数打包数组中传给函数，使用`vararg`关键字声明。Kotlin需要将数组解包才能传给可变参数

``` kotlin
fun compute(vararg name: String) {	//可变参数
    name.forEach { println(it)}
}
compute()
compute("a")
compute("a","B","c")
val names = arrayOf("leavesc","LeaveSC","asda")
compute(* names)	//显式的解包数组
```

#### 标准函数

1. with|run|apply

   标准函数指Standard.kt文件中定义的函数

   ``` kotlin
   val list = listOf("Apple","Banana","Orange","Pear","Grape")
   //with,param1:任意类型对象，param2:Lambda表达式，param1为表达式上下文，表达式最后一行代码为返回值。
   val result = with(StringBuilder()) {
       append("Start eating.\n")
       for(fruit in list) {
           append(fruit).append("\n")
       }
       append("Ate all fruits.")
       toString()
   }
   //run,在某个对象的基础上调用，只接收一个Lambda表达式，且提供对象的上下文，最后一行代码为返回值。
   val result = StringBuilder().run {
       append("Start eating.\n")
       for(fruit in list) {
           append(fruit).append("\n")
       }
       append("Ate all fruits.")
       toString()
   }
   //apply,无法指定返回值，只能返回调用对象本身
   val result = StringBuilder().apply {
       append("Start eating.\n")
       for(fruit in list) {
           append(fruit).append("\n")
       }
       append("Ate all fruits.")
   }
   println(result.toString())
   ```

#### 扩展函数和运算符重载

1. 扩展函数

   可以定义在任何一个现有类中，不一定需要创建新文件，不过最好定义为顶层方法

   ``` kotlin
   fun ClassName.methodName(param1: Int, param2: Int): Int {
       return 0
   }
   ```

   - 扩展函数是**静态分发**的，由函数调用所在表达式的类型决定，而非由运行时求值结果决定

     ``` kotlin
     open class Shape
     class Rectangle: Shape()
     fun Shape.getName() = "Shape"
     fun Rectangle.getName() = "Rectangle"
     fun printClassName(s: Shape) {	//只取决于参数s的声明类型
         println(s.getName())
     }
     printClassName(Rectangle())	//输出Shape
     ```

   - 如果一个类有同名、同接收者类型、同参数的成员函数、扩展函数，成员函数优先

     ``` kotlin
     class Example {
         fun printFunctionType() { println("class method.")}
     }
     fun Example.printFunctionType() { println("extension method.")}
     Example().printFunctionType()	//Class method.
     ```

   - 扩展函数内可使用this，但不能访问私有、保护成员

2. 扩展属性

   扩展属性没有真的为类添加属性，只通过get、set显式定义，相当于定义了属性访问器方法，没有幕后字段field，因此没有初始化器

   ``` kotlin
   val View.isLayoutRtl: Boolean // (= false) error
   	get() = this.layoutDirection == View.LAYOUT_DIRECTION_RTL
   ```

3. 运算符重载

   operator关键字，在指定函数之前修饰即可实现运算符重载（plus\minus\times\div\rem)

   ``` kotlin
   operator fun plus(money: Money): Money {
       val sum = value + money.value
       return Money(sum)
   }
   val money1 = Money(5)
   val money2 = Money(10)
   val money3 = money1 + money2
   //还可根据不同参数类型对同一函数实现多个重载
   operator fun plus(newValue: Int): Money {
       val sum = value + newValue
       return Money(sum)
   }
   ```

### 泛型和委托

#### 泛型

1. 泛型允许在不指定具体类型的情况下进行编程，代码具有更好的扩展性。

``` kotlin
class MyClass<T> {
    fun method(param: T): T {
        return param
    }
}
val myClass = MyClass<Int>()
val result = myClass.method(123)
//泛型方法
class MyClass {
    fun <T> method(param: T): T {
        return param
    }
}
val myClass = MyClass()
val result = myClass.method<Int>(123)
//对泛型进行类型限制，泛型的上界默认为Any?
fun <T: Number> method(param: T): T {
    return param
}
```

#### 类委托和委托类型

委托模式：有两个对象参与处理请求，接受请求的对象将请求委托给另一对象来处理，通过关键字`by`指定实现者

1. 类委托：将一个类的具体实现委托给另一个类完成，如Set -> HashSet。大部分方法实现调用辅助对象方法，小部分自己重写，或加入独有方法

   ``` kotlin
   class MySet<T>(val helperSet: HashSet<T>): Set<T> {
       override val size: Int
       	get() = helperSet.size
       override fun contains(element: T) = helperSet.contains(element)
       ...
   }
   ```

   委托的关键字为`by`，可免去模板代码，只需要写特定功能。

   ``` kotlin
   class MySet<T>(val helperSet: HashSet<T>): Set<T> by helperSet {
   	fun helloworld() = println("Hello World.")
       override fun isEmpty() = false
   }
   ```

2. 委托属性：一个类的某属性值不是在类中直接定义，而委托给一个委托类，实现对这一类的属性的统一管理

3. lazy函数：把想要延迟执行的代码放到by lazy代码块中，当该属性首次被调用时，才执行代码块。

   ``` kotlin
   val p by lazy{ ... }
   //实际是通过by将p委托给lazy{ ... }这一高阶函数
   ```

4. lateinit var    延迟初始化，让编译器在检查时不因为属性变量未初始化而报错

   ``` kotlin
   lateinit var myParam: String
   ```

   只能修饰类属性，不能修饰局部变量，且只能用来修饰对象

### 协程

1. 基本用法

   ``` groovy
   // 在模块级build.gradle中添加协程依赖库
   implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.1.1"
   implementation "org.jetbrains.kotlinx:kotlinx-coroutines-android0:1.1.1"
   // 伪代码
   launch({
       val user = api.getUser()	// 网络请求(IO线程)
       nameTv.text = user.name	// 更新UI(主线程)
   })
   // 闭包简化写法
   launch {
       ...
   }
   ```

   ``` kotlin
   // 方法一，使用runBlocking顶层函数；线程阻塞的，常用于单元测试
   runBlocking {
       getImage(imageId)
   }
   //方法二，使用GlobalScope单例对象，可直接调用launch开启协程；线程不阻塞的，但生命周期与app一致，不可取消
   GlobalScope.launch {
       getImage(imageId)
   }
   //方法三，通过CoroutineContext创建一个CoroutineScope对象，在该对象中开启协程；推荐用法，生命周期与context一致，可管理控制
   val coroutineScope = CoroutineScope(context)
   coroutineScope.launch {
       getImage(imageId)
   }
   ```

2. 参数、方法

   1. 常见的Dispatches：`Dispatches.Main`、`Dispatches.IO`（IO密集型任务，如读写文件、数据库、网络请求）、`Dispatches.Default`（CPU密集型任务，如计算）

      ``` kotlin
      coroutineScope.launch(Dispatches.IO) {
          ...
      }
      ```

   2. `launch`：创建一个新协程，并在指定线程上运行

      `launch`与`async`：

      ​	相同点：都可启动一个协程，都返回Coroutine

      ​	不同点：`async`返回的Coroutine实现了Deferred接口，调用Deferred.await()即可得到结果

      ``` kotlin
      coroutineScope.launch(Dispatches.Main) {
          val avatar: Deferred = async{ api.getAvatar(user) }
          val logo: Deferred = async{ api.getCompanyLogo(user) }
          show(avatar.await(), logo.await())
      }
      ```

   3. `withContext`：切换到指定的线程，且在其闭包执行完毕后**自动切换回到原线程**；自动切换消除线程切换的嵌套，因此可放入单独函数内

      ``` kotlin
      coroutineScope.launch(Dispatches.Main) {
          val image = withContext(Dispatches.Io) {
              getImage(imageId)
          }
          avatarIv.setImageBitmap(image)
      }
      // withContext放入单独函数内
      suspend fun getImage(imageId: Int) = withContext(Dispatches,IO) {
          ...
      }	// 此时修改为val image = getImage(imageId)
      ```

   4. `delay` ：等待一段时间后再继续执行代码

   5. `suspend`：声明一个挂起函数，挂起函数必须在协程或另一挂起函数中调用

      1. 线程执行到suspend函数时，1.后台线程 被系统回收或继续执行别的后台任务    2.主线程 继续工作
      2. 协程执行到suspend函数时，在withContext指定的线程继续向下执行，执行完成后，自动切换回原线程
      3. 自定义suspend函数的时机：1.进行耗时操作：IO操作和CPU计算工作时 2.需要等待的操作：5秒后再继续操作的情况

      > 单独使用`suspend`关键字并不能声明一个完整的挂起函数，需要内部使用挂起函数API：`withContext、delay`指定要切换到的目标线程

   6. 挂起：**一个稍后会被自动切回来的线程调度操作**，该协程从当前线程挂起，协程与线程脱离

3. 完整例子

   ``` kotlin
   coroutineScope.launch(Dispatches.Main) {	// 主线程开启协程
       val user = api.getUser()	// IO线程执行网络请求
       nameTv.text = user.name	// 主线程更新UI
   }
   // java
   api.getUser(new Callback<User>() {
      @Override
       public void success(User user) {
           runOnUiThread(new Runnable() {	// 切换到主线程更新UI
               @Override 
               public void run() {
                   nameTv.setText(user.name);
               }
           })
       }
       @Override 
       public void failure(Exception e) {
           ...
       }
   });
   ```

4. 案例情况：多个网络请求需要等待同步结束后再执行UI操作

   ``` kotlin
   // 回调式写法
   api.getAvatar(user) { avatar ->
   	api.getCompanyLogo(user) { logo -> 
       	show(merge(avatar, logo))
       }
   }
   // 协程,协作式任务
   coroutineScope.launch(Dispatches.Main) {
       val avatar = async { api.getAvatar(user) }
       val logo = async { api.getCompanyLogo(user) }
       val merged = suspendingMerge(avatar, logo)
       show(merged)
   }
   ```

5. 总结
   1. 协程就是切线程；
   2. 挂起就是可以自动切回来的切线程；
   3. 挂起的非阻塞式指的是它能用看起来阻塞的代码写出非阻塞的操作；







