# Kotlin 基础语法

 ### 怎么定义一个类

如果你想定义一个类，只要使用 class关键字。

```kotlin
class MainActivity{
  
}
```

它有一个默认唯一的构造器。你只需要在类名后面写上它的参数。如果这个类没有任何内容可以省略大括号

```kotlin
class TestClass(name:String,value:String)
```

那么构造函数的函数体在哪呢？你可以写在`init`块中：

```kotlin
class TestClass(name:String,value:String){
  init{
	......
	}
}
```
### 类继承

默认任何类都是继承自Any（与java 中的Object类似）但是我们可以继承其它类。所有的类默认都是不可继承的（final），所以我们只能继承那些明确声明`open`或者`abstract`的类：

```kotlin
open class Animal(name:String)
class Cat(name:String):Animal(name)
```

当我们只有单个构造器时，我们需要在从父类继承下来的构造器中指定需要的参数。这是用来替换Java中的`super`调用的。


### 函数

（我们Java中的方法）可以使用`fun`关键字就可以定义:

```  kotlin
fun onCreate(savedInstanceState: Bundle?) {

}
```

如果你没有指定它的返回值，它就会返回`Unit`，与Java中的`void`类似，但是`Unit`是一个真正的对象。你当然也可以指定任何其它的返回类型：

```kotlin
fun add(x: Int, y: Int) : Int {

    return x + y

}
```



> 小提示：分号不是必须的

>  就想你在上面的例子中看到的那样，我在每句的最后没有使用分号。当然你也可以使用分号，分号不是必须的，而且不使用分号是一个不错的实践。当你这么做了，你会发现这节约了你很多时间。

然而如果返回的结果可以使用一个表达式计算出来，你可以不使用括号而是使用等号：

```kotlin
fun add(x: Int,y: Int) : Int = x + y
```

### 构造方法和函数参数

Kotlin中的参数与Java中有些不同。如你所见，我们先写参数的名字再写它的类型：

``` kotlin
fun add(x: Int, y: Int) : Int {

    return x + y

}
```
我们可以给参数指定一个默认值使得它们变得可选，这是非常有帮助的。这里有一个例子，在Activity中创建了一个函数用来toast一段信息：
```kotlin
fun toast(message: String, duration: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(this, message, duration).show()
}
```
第二个参数（duration）指定了一个默认值。这意味着你调用的时候可以传入第二个值或者不传，这样可以避免你需要的重载函数：
```kotlin
toast("Hello")
toast("Hello", Toast.LENGTH_LONG)
```

这个与下面的Java代码是一样的：

```java
void toast(String message){
   Toast.makeText(this, message, Toast.LENGTH_SHORT).show();
}

void toast(String message, int length){
    Toast.makeText(this, message, length).show();
}
```
下一个例子
```kotlin
fun toast1(message: String,
                  tag: String = javaClass.simpleName,
                length: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(this, "[$tag] $message", length).show()
}
```
你可以使用参数名字来调用，这表示你可以通过在值前写明参数名来传入你希望的参数：
```kotlin
toast1(message = "Hello", length = Toast.LENGTH_SHORT)
```
* 小提示：String模版
> 你可以在String中直接使用模版表达式。它可以帮助你很简单地在静态值和变量的基础上编写复杂的String。在上面的例子中，我使用了"[$className] $message"。

> 如你所见，任何时候你使用一个`$`符号就可以插入一个表达式。如果这个表达式有一点复杂，你就需要使用一对大括号括起来："Your name is ${user.name}"。



### 基本类型
像integer，float或者boolean等类型仍然存在，但是它们全部都会作为对象存在的。基本类型的名字和它们工作方式都是与Java非常相似的，但是有一些不同之处你可能需要考虑到：
- 数字类型中不会自动转型。举个例子，你不能给`Double`变量分配一个`Int`。必须要做一个明确的类型转换，可以使用众多的函数之一：
```kotlin
val i:Int=2
val d: Double = i.toDouble()
```
- 字符（Char）不能直接作为一个数字来处理。在需要时我们需要把他们转换为一个数字：

```kotlin
val c:Char='c'
val i: Int = c.toInt()
```

- 位运算也有一点不同。在Android中，我们经常在`flags`中使用“或”，所以我使用"and"和"or来举例：

```java
// Java
int bitwiseOr = FLAG1 | FLAG2;
int bitwiseAnd = FLAG1 & FLAG2;
```

```kotlin
// Kotlin
val bitwiseOr = FLAG1 or FLAG2
val bitwiseAnd = FLAG1 and FLAG2
```
- 字面上可以写明具体的类型。这个不是必须的，但是一个通用的Kotlin实践时省略变量的类型，所以我们可以让编译器自己去推断出具体的类型。

```kotlin
val i = 12 // An Int
val iHex = 0x0f // 一个十六进制的Int类型
val l = 3L // A Long
val d = 3.5 // A Double
val f = 3.5F // A Float
```
- 一个String可以像数组那样访问，并且被迭代：
```kotlin
val s = "Example"
val c = s[2] // 这是一个字符'a'
// 迭代String
val s = "Example"
for(c in s){
    print(c)
}
```

### 变量

变量可以很简单地定义成可变(`var`)和不可变（`val`）的变量。这个与Java中使用的`final`很相似。但是__不可变__在Kotlin（和其它很多现代语言）中是一个很重要的概念。

一个不可变对象意味着它在实例化之后就不能再去改变它的状态了。如果你需要一个这个对象修改之后的版本，那就会再创建一个新的对象。这个让编程更加具有健壮性和预估性。在Java中，大部分的对象是可变的，那就意味着任何可以访问它这个对象的代码都可以去修改它，从而影响整个程序的其它地方。

不可变对象也可以说是线程安全的，因为它们无法去改变，也不需要去定义访问控制，因为所有线程访问到的对象都是同一个。

所以在Kotlin中，如果我们想使用不可变性，我们编码时思考的方式需要有一些改变。__一个重要的概念是：尽可能地使用`val`__。除了个别情况（特别是在Android中，有很多类我们是不会去直接调用构造函数的），大多数时候是可以的。

之前提到的另一件事是我们通常不需要去指明类的类型，它会自动从后面分配的语句中推断出来，这样可以让代码更加清晰和快速修改。我们在前面已经有了一些例子：

```kotlin
val s = "Example" // A String
val i = 23 // An Int
val actionBar = supportActionBar // An ActionBar in an Activity context
```

如果我们需要使用更多的范型类型，则需要指定：

```kotlin
val a: Any = 23
val c: Context = activity
```

### 属性
 属性与java中的字段是相同的，但是更强大。属性做的事情是在字段上加getter 和setter。我们通过一个例子来比较他们的不同之处。这是java中字段安全访问和修改所需要的代码：

 ```java
public class Person {
    private String name;
    public String getName() {
        return name;
    }
    public void setName(String name) { 
        this.name = name;
    }
}
...
Person person = new Person();
person.setName("name");
String name = person.getName();
```

在Kotlin中，只需要一个属性就可以了：

```kotlin
public class Person {
    var name: String = ""
}

...

val person = Person()
person.name = "name"
val name = person.name
```

如果没有任何指定，属性会默认使用getter和setter。当然它也可以修改为你自定义的代码，并且不修改存在的代码：
```kotlin
public classs Person {
    var name: String = ""
        get() = field.toUpperCase()
        set(value){
            field = "Name: $value"
        }
}
```

如果需要在getter和setter中访问这个属性自身的值，它需要创建一个`backing field`。可以使用`field`这个预留字段来访问，它会被编译器找到正在使用的并自动创建。需要注意的是，如果我们直接调用了属性，那我们会使用setter和getter而不是直接访问这个属性。`backing field`只能在属性访问器内访问。

就如在前面章节提到的，当操作Java代码的时候，Kotlin将允许使用属性的语法去访问在Java文件中定义的getter/setter方法。编译器会直接链接到它原始的getter/setter方法。所以当我们直接访问属性的时候不会有性能开销。

### 数据类
数据类是一种非常强大的类，它可以让你避免创建Java中的用于保存状态但又操作非常简单的POJO的模版代码。它们通常只提供了用于访问它们属性的简单的getter和setter。定义一个新的数据类非常简单：

```kotlin
data class Teacher(val name:String,val age:Int,val birthday:Date)
```
通过数据类，我们可以方便地得到很多有趣的函数，一部分是来自属性，我们之前已经讲过（从编写getter和setter函数）：

- equals(): 它可以比较两个对象的属性来确保他们是相同的。
- hashCode(): 我们可以得到一个hash值，也是从属性中计算出来的。
- copy(): 你可以拷贝一个对象，可以根据你的需要去修改里面的属性。
- 一系列可以映射对象到变量中的函数。

如果我们使用不可修改的对象，假如我们需要修改这个对象状态，必须要创建一个新的一个或者多个属性被修改的实例。这个任务是非常重复且不简洁的。

举个例子，如果我们需要修改`Teacher`中的`birthday`（温度），我们可以这么做：

```kotlin
 val teacher = Teacher("张三",30, Date())
 teacher.copy(birthday = Date(2000,1,1))
```

如上，我们拷贝了第一个Teacher对象然后只修改了`birthday`的属性而没有修改这个对象的其它状态。

> 当你使用Java类时小心“不可修改性”
> 如果你决定使用不可修改来工作，你需要意识到Java不是根据这种思想来设计的，在某些情况下，我们仍然可以修改这些状态。在上一个例子中，你还是可以访问`Date`对象，然后改变它的值。有个简单（不安全）的方法是记住所有需要修改状态的对象作为一个规则，然后必要的时候去拷贝一份。
>另外一个方法是封装这些类。你可以创建一个`ImmutableDate`类，它封装了`Date`并且不允许去修改它们的状态。决定哪种方式取决于你。


映射对象的每一个属性到一个变量中，这个过程就是我们知道的多声明。这就是为什么会有`componentX`函数被自动创建。使用上面的`Teacher`类举个例子：

```kotlin
val teacher = Teacher("张三",30, Date())
val (name, age, birthday) = teacher
```

上面这个多声明会被编译成下面的代码：

```kotlin
val name = f1.component1()
val age = f1.component2()
val birthday = f1.component3()
```

这个特性背后的逻辑是非常强大的，它可以在很多情况下帮助我们简化代码。举个例子，`Map`类含有一些扩展函数的实现，允许它在迭代时使用key和value：

```kotlin
for ((key, value) in map) {
	Log.d("map", "key:$key, value:$value")
}
```
### 操作符表

这里你可以看见一系列包括`操作符`和`对应方法`的表。对应方法必须在指定的类中通过各种可能性被实现。

__一元操作符__

| 操作符 | 函数|
|--|--|
| +a | a.unaryPlus() |
| -a | a.unaryMinus() |
| !a | a.not() |
| a++ | a.inc() |
| a-- | a.dec() |
<br/>

__二元操作符__

| 操作符 | 函数|
|----|----
| a + b | a.plus(b) |
| a - b | a.minus(b) |
| a * b | a.times(b) |
| a / b | a.div(b) |
| a % b | a.mod(b) |
| a..b | a.rangeTo(b) |
| a in b | a.contains(b) |
| a !in b | !a.contains(b) |
| a += b | a.plusAssign(b) |
| a -= b | a.minusAssign(b) |
| a *= b | a.timesAssign(b) |
| a /= b | a.divAssign(b) |
| a %= b | a.modAssign(b) |
<br/>
__数组操作符__

| 操作符 | 函数|
|----|--
| a[i] | a.get(i) |
| a[i, j] | a.get(i, j) |
| a[i\_1, ..., i\_n] | a.get(i\_1, ..., i\_n) |
| a[i] = b | a.set(i, b) |
| a[i, j] = b | a.set(i, j, b) |
| a[i\_1, ..., i\_n] = b | a.set(i\_1, ..., i\_n, b) |
<br/>
__等于操作符__

| 操作符 | 函数|
|----|--
| a == b | a?.equals(b) ?: b === null |
| a != b | !(a?.equals(b) ?: b === null) |

相等操作符有一点不同，为了达到正确合适的相等检查做了更复杂的转换，因为要得到一个确切的函数结构比较，不仅仅是指定的名称。方法必须要如下准确地被实现：

```kotlin
operator fun equals(other: Any?): Boolean
```
操作符`===`和`!==`用来做身份检查（它们分别是Java中的`==`和`!=`），并且它们不能被重载。

<br/>

__函数调用__

| 方法 | 调用 |
|----|--
| a(i) | a.invoke(i) |
| a(i, j) | a.invoke(i, j) |
| a(i\_1, ..., i\_n)| a.invoke(i\_1, ..., i\_n) |


你可以想象，Kotlin List是实现了数组操作符的，所以我们可以像Java中的数组一样访问List的每一项。除此之外：在可修改的List中，每一项也可以用一个简单的方式被直接设置：

```kotlin
val x = myList[2]
myList[2] = 4
```

如果你还记得，我们有一个叫ForecastList的数据类，它是由很多其他额外的信息组成的。有趣的是可以直接访问它的每一项而不是请求内部的list得到某一项。做一个完全不相关的事情，我要去实现一个`size()`方法，它能稍微能简化一点当前的Adapter：

```kotlin
data class ForecastList(val city: String, val country: String,
                        val dailyForecast: List<Forecast>) {
    operator fun get(position: Int): Forecast = dailyForecast[position]
    fun size(): Int = dailyForecast.size
}
```

它会使我们的`onBindViewHolder`更简单一点：

```kotlin
override fun onBindViewHolder(holder: ViewHolder,
        position: Int) {
	with(weekForecast[position]) {
	    holder.textView.text = "$date - $description - $high/$low"
	}
}
```

当然还有`getItemCount()`方法：

```kotlin
override fun getItemCount(): Int = weekForecast.size()
```

### 修饰符

* private

`private`修饰符是我们使用的最限制的修饰符。它表示它只能被自己所在的文件可见。所以如果我们给一个类声明为`private`，我们就不能在定义这个类之外的文件中使用它。

另一方面，如果我们在一个类里面使用了`private `修饰符，那访问权限就被限制在这个类里面了。甚至是继承这个类的子类也不能使用它。

所以一等公民，类、对象、接口……（也就是包成员）如果被定义为`private`，那么它们只会对被定义所在的文件可见。如果被定义在了类或者接口中，那它们只对这个类或者接口可见。

* protected

这个修饰符只能被用在类或者接口中的成员上。一个包成员不能被定义为`protected`。定义在一个成员中，就与Java中的方式一样了：它可以被成员自己和继承它的成员可见（比如，类和它的子类）。

* internal

如果是一个定义为`internal`的包成员的话，对所在的整个`module`可见。如果它是一个其它领域的成员，它就需要依赖那个领域的可见性了。比如，如果我们写了一个`private`类，那么它的`internal`修饰的函数的可见性就会限制与它所在的这个类的可见性。

我们可以访问同一个`module`中的`internal`修饰的类，但是不能访问其它`module`的。

> 什么是`module`
>  根据Jetbrains的定义，一个`module`应该是一个单独的功能性的单位，它应该是可以被单独编译、运行、测试、debug的。根据我们项目不同的模块，可以在Android Studio中创建不同的`module`。在Eclipse中，这些`module`可以认为是在一个`workspace`中的不同的`project`。

* public

你应该可以才想到，这是最没有限制的修饰符。__这是默认的修饰符__，成员在任何地方被修饰为`public`，很明显它只限制于它的领域。一个定义为`public`的成员被包含在一个`private`修饰的类中，这个成员在这个类以外也是不可见的。


### 构造器

所有构造函数默认都是`public`的，它们类是可见的，可以被其它地方使用。我们也可以使用这个语法来把构造函数修改为`private`：

```kotlin
class C private constructor(a: Int) { ... }
```

我们已经准备好使用`public`来进行重构了，但是我们还有很多其它细节需要修改。比如，在`RequestForecastCommand`中，我们在构造函数中我们创建的属性`zipCode`可以定义为`private`：

```kotlin
class RequestForecastCommand(private val zipCode: String)
```

所作的事情就是我们创建了一个不可修改的属性zipCode，它的值我们只能去得到，不能去修改它。所以这个不大的改动让代码看起来更加清晰。如果我们在编写类的时候，你觉得某些属性因为是什么原因不能对别人可见，那就把它定义为`private`。

而且，在Kotlin中，我们不需要去指定一个函数的返回值类型，它可以让编译器推断出来。举个省略返回值类型的例子：

```kotlin
data class ForecastList(...) {
	fun get(position: Int) = dailyForecast[position]
	fun size() = dailyForecast.size()
}
```

我们可以省略返回值类型的典型情景是当我们要给一个函数或者一个属性赋值的时候。而不需要去写代码块去实现。


### 集合和函数操作符

在我们这个项目我们已经使用过集合了，但是现在是时候展示它们结合函数操作符之后有多强大了。关于函数式编程很不错的一点是我们不用去解释我们怎么去做，而是直接说我想做什么。比如，如果我想去过滤一个list，不用去创建一个list，遍历这个list的每一项，然后如果满足一定的条件则放到一个新的集合中，而是直接使用filer函数并指明我想用的过滤器。用这种方式，我们可以节省大量的代码。

虽然我们可以直接用Java中的集合，但是Kotlin也提供了一些你希望用的本地的接口：

- __Iterable__：父类。所有我们可以遍历一系列的都是实现这个接口。
- __MutableIterable__：一个支持遍历的同时可以执行删除的Iterables。
- __Collection__：这个类相是一个范性集合。我们通过函数访问可以返回集合的size、是否为空、是否包含一个或者一些item。这个集合的所有方法提供查询，因为connections是不可修改的。
- __MutableCollection__：一个支持增加和删除item的Collection。它提供了额外的函数，比如`add` 、`remove`、`clear`等等。
- __List__：可能是最流行的集合类型。它是一个范性有序的集合。因为它的有序，我们可以使用`get`函数通过position来访问。
- __MutableList__：一个支持增加和删除item的List。
- __Set__：一个无序并不支持重复item的集合。
- __MutableSet__：一个支持增加和删除item的Set。
- __Map__：一个key-value对的collection。key在map中是唯一的，也就是说不能有两对key是一样的键值对存在于一个map中。
- __MutableMap__：一个支持增加和删除item的map。

有很多不同集合可用的函数操作符。我想通过一个例子来展示给你看。知道有哪些可选的操作符是很有用的，因为这样会更容易分辨它们使用的时机。

### 总数操作符

#### any

如果至少有一个元素符合给出的判断条件，则返回true。

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
assertTrue(list.any { it % 2 == 0 })
assertFalse(list.any { it > 10 })
```

#### all

如果全部的元素符合给出的判断条件，则返回true。

```kotlin
assertTrue(list.all { it < 10 })
assertFalse(list.all { it % 2 == 0 })
```

#### count

返回符合给出判断条件的元素总数。

```kotlin
assertEquals(3, list.count { it % 2 == 0 })
```

#### fold

在一个初始值的基础上从第一项到最后一项通过一个函数累计所有的元素。

```kotlin
assertEquals(25, list.fold(4) { total, next -> total + next })
```

#### foldRight

与`fold`一样，但是顺序是从最后一项到第一项。

```kotlin
assertEquals(25, list.foldRight(4) { total, next -> total + next })
```

#### forEach

遍历所有元素，并执行给定的操作。

```kotlin
list.forEach { println(it) }
```

#### forEachIndexed

与`forEach`，但是我们同时可以得到元素的index。

```kotlin
list.forEachIndexed { index, value
		-> println("position $index contains a $value") }
```

#### max

返回最大的一项，如果没有则返回null。

```kotlin
assertEquals(6, list.max())
```

#### maxBy

根据给定的函数返回最大的一项，如果没有则返回null。

```kotlin
// The element whose negative is greater
assertEquals(1, list.maxBy { -it })
```

#### min

返回最小的一项，如果没有则返回null。

```kotlin
assertEquals(1, list.min())
```

#### minBy

根据给定的函数返回最小的一项，如果没有则返回null。

```kotlin
// The element whose negative is smaller
assertEquals(6, list.minBy { -it })
```

#### none

如果没有任何元素与给定的函数匹配，则返回true。

```kotlin
// No elements are divisible by 7
assertTrue(list.none { it % 7 == 0 })
```

#### reduce

与`fold`一样，但是没有一个初始值。通过一个函数从第一项到最后一项进行累计。

```kotlin
assertEquals(21, list.reduce { total, next -> total + next })
```

#### reduceRight

与`reduce`一样，但是顺序是从最后一项到第一项。

```kotlin
assertEquals(21, list.reduceRight { total, next -> total + next })
```

#### sumBy

返回所有每一项通过函数转换之后的数据的总和。

```kotlin
assertEquals(3, list.sumBy { it % 2 })
```

### 过滤操作符

#### drop

返回包含去掉前n个元素的所有元素的列表。

```kotlin
assertEquals(listOf(5, 6), list.drop(4))
```

#### dropWhile

返回根据给定函数从第一项开始去掉指定元素的列表。

```kotin
assertEquals(listOf(3, 4, 5, 6), list.dropWhile { it < 3 })
```

#### dropLastWhile

返回根据给定函数从最后一项开始去掉指定元素的列表。

```kotlin
assertEquals(listOf(1, 2, 3, 4), list.dropLastWhile { it > 4 })
```

#### filter

过滤所有符合给定函数条件的元素。

```kotin
assertEquals(listOf(2, 4, 6), list .ilter { it % 2 == 0 })
```

#### filterNot

过滤所有不符合给定函数条件的元素。

```kotin
assertEquals(listOf(1, 3, 5), list.filterNot { it % 2 == 0 })
```

#### filterNotNull

过滤所有元素中不是null的元素。

```kotlin
assertEquals(listOf(1, 2, 3, 4), listWithNull.filterNotNull())
```

#### slice

过滤一个list中指定index的元素。

```kotlin
assertEquals(listOf(2, 4, 5), list.slice(listOf(1, 3, 4)))
```

#### take

返回从第一个开始的n个元素。

```kotlin
assertEquals(listOf(1, 2), list.take(2))
```

#### takeLast

返回从最后一个开始的n个元素

```kotlin
assertEquals(listOf(5, 6), list.takeLast(2))
```

#### takeWhile

返回从第一个开始符合给定函数条件的元素。

```kotlin
assertEquals(listOf(1, 2), list.takeWhile { it < 3 })
```

### 映射操作符

#### flatMap

遍历所有的元素，为每一个创建一个集合，最后把所有的集合放在一个集合中。

```kotlin
assertEquals(listOf(1, 2, 2, 3, 3, 4, 4, 5, 5, 6, 6, 7), 
list.flatMap { listOf(it, it + 1) })
```

#### groupBy

返回一个根据给定函数分组后的map。

```kotlin
assertEquals(mapOf("odd" to listOf(1, 3, 5), "even" to listOf(2, 4, 6)), list.groupBy { if (it % 2 == 0) "even" else "odd" })
```

#### map

返回一个每一个元素根据给定的函数转换所组成的List。

```kotlin
assertEquals(listOf(2, 4, 6, 8, 10, 12), list.map { it * 2 })
```

#### mapIndexed

返回一个每一个元素根据给定的包含元素index的函数转换所组成的List。

```kotlin
assertEquals(listOf (0, 2, 6, 12, 20, 30), list.mapIndexed { index, it -> index * it })
```

#### mapNotNull

返回一个每一个非null元素根据给定的函数转换所组成的List。

```kotlin
assertEquals(listOf(2, 4, 6, 8), listWithNull.mapNotNull { it * 2 })
```

### 元素操作符

#### contains

如果指定元素可以在集合中找到，则返回true。

```kotlin
assertTrue(list.contains(2))
```

#### elementAt

返回给定index对应的元素，如果index数组越界则会抛出`IndexOutOfBoundsException`。

```kotlin
assertEquals(2, list.elementAt(1))
```

#### elementAtOrElse

返回给定index对应的元素，如果index数组越界则会根据给定函数返回默认值。

```kotlin
assertEquals(20, list.elementAtOrElse(10, { 2 * it }))
```

#### elementAtOrNull

返回给定index对应的元素，如果index数组越界则会返回null。

```kotlin
assertNull(list.elementAtOrNull(10))
```

#### first

返回符合给定函数条件的第一个元素。

```kotlin
assertEquals(2, list.first { it % 2 == 0 })
```

#### firstOrNull

返回符合给定函数条件的第一个元素，如果没有符合则返回null。

```kotlin
assertNull(list.firstOrNull { it % 7 == 0 })
```

#### indexOf

返回指定元素的第一个index，如果不存在，则返回`-1`。

```kotlin
assertEquals(3, list.indexOf(4))
```

#### indexOfFirst

返回第一个符合给定函数条件的元素的index，如果没有符合则返回`-1`。

```kotlin
assertEquals(1, list.indexOfFirst { it % 2 == 0 })
```

#### indexOfLast

返回最后一个符合给定函数条件的元素的index，如果没有符合则返回`-1`。

```kotlin
assertEquals(5, list.indexOfLast { it % 2 == 0 })
```

#### last

返回符合给定函数条件的最后一个元素。

```kotlin
assertEquals(6, list.last { it % 2 == 0 })
```

#### lastIndexOf

返回指定元素的最后一个index，如果不存在，则返回`-1`。

#### lastOrNull

返回符合给定函数条件的最后一个元素，如果没有符合则返回null。

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
assertNull(list.lastOrNull { it % 7 == 0 })
```

#### single

返回符合给定函数的单个元素，如果没有符合或者超过一个，则抛出异常。

```kotlin
assertEquals(5, list.single { it % 5 == 0 })
```

#### singleOrNull

返回符合给定函数的单个元素，如果没有符合或者超过一个，则返回null。

```kotlin
assertNull(list.singleOrNull { it % 7 == 0 })
```

### 生产操作符

#### merge

把两个集合合并成一个新的，相同index的元素通过给定的函数进行合并成新的元素作为新的集合的一个元素，返回这个新的集合。新的集合的大小由最小的那个集合大小决定。

```kotlin
val list = listOf(1, 2, 3, 4, 5, 6)
val listRepeated = listOf(2, 2, 3, 4, 5, 5, 6)
assertEquals(listOf(3, 4, 6, 8, 10, 11), list.merge(listRepeated) { it1, it2 -> it1 + it2 })
```

#### partition

把一个给定的集合分割成两个，第一个集合是由原集合每一项元素匹配给定函数条件返回`true`的元素组成，第二个集合是由原集合每一项元素匹配给定函数条件返回`false`的元素组成。

```kotlin
assertEquals(
	Pair(listOf(2, 4, 6), listOf(1, 3, 5)), 
	list.partition { it % 2 == 0 }
)
```

#### plus

返回一个包含原集合和给定集合中所有元素的集合，因为函数的名字原因，我们可以使用`+`操作符。

```kotlin
assertEquals(
	listOf(1, 2, 3, 4, 5, 6, 7, 8), 
	list + listOf(7, 8)
)
```

#### zip

返回由`pair`组成的List，每个`pair`由两个集合中相同index的元素组成。这个返回的List的大小由最小的那个集合决定。

```kotin
assertEquals(
	listOf(Pair(1, 7), Pair(2, 8)), 
	list.zip(listOf(7, 8))
)
```

#### unzip

从包含pair的List中生成包含List的Pair。

```kotlin
assertEquals(
	Pair(listOf(5, 6), listOf(7, 8)), 
	listOf(Pair(5, 7), Pair(6, 8)).unzip()
)
```

### 顺序操作符

#### reverse

返回一个与指定list相反顺序的list。

```kotlin
val unsortedList = listOf(3, 2, 7, 5)
assertEquals(listOf(5, 7, 2, 3), unsortedList.reverse())
```

#### sort

返回一个自然排序后的list。

```kotlin
assertEquals(listOf(2, 3, 5, 7), unsortedList.sort())
```

#### sortBy

返回一个根据指定函数排序后的list。

```kotlin
assertEquals(listOf(3, 7, 2, 5), unsortedList.sortBy { it % 3 })
```

#### sortDescending

返回一个降序排序后的List。

```kotlin
assertEquals(listOf(7, 5, 3, 2), unsortedList.sortDescending())
```

#### sortDescendingBy

返回一个根据指定函数降序排序后的list。

```kotlin
assertEquals(listOf(2, 5, 7, 3), unsortedList.sortDescendingBy { it % 3 })
```

# If表达式

在Kotlin中一切都是表达式，也就是说一切都返回一个值。如果`if`条件不含有一个exception，那我们可以像我们平时那样使用它：

```kotlin
if(x>0){
	toast("x is greater than 0")
}else if(x==0){ 
	toast("x equals 0")
}else{
	toast("x is smaller than 0")
}
```

我们也可以把结果赋值给一个变量。我们在我们的代码中使用了很多次：

```kotlin
val res = if (x != null && x.size() >= days) x else null
```

这也说明我也不需要像Java那种有一个三元操作符，因为我们可以使用它来简单实现：

```kotlin
val z = if (condition) x else y
```

所以`if`表达式总是返回一个value。如果一个分支返回了Unit，那整个表达式也将返回Unit，它是可以被忽略的，这种情况下它的用法也就跟一般Java中的`if`条件一样了。

# When表达式

`when`表达式与Java中的`switch/case`类似，但是要强大得多。这个表达式会去试图匹配所有可能的分支直到找到满意的一项。然后它会运行右边的表达式。与Java的`switch/case`不同之处是参数可以是任何类型，并且分支也可以是一个条件。

对于默认的选项，我们可以增加一个`else`分支，它会在前面没有任何条件匹配时再执行。条件匹配成功后执行的代码也可以是代码块：

```kotlin
when (x){
	1 -> print("x == 1") 
	2 -> print("x == 2") 
	else -> {
		print("I'm a block")
		print("x is neither 1 nor 2")
    }
}
```

因为它是一个表达式，它也可以返回一个值。我们需要考虑什么时候作为一个表达式使用，它必须要覆盖所有分支的可能性或者实现`else`分支。否则它不会被编译成功：

```kotlin
val result = when (x) {
    0, 1 -> "binary"
	else -> "error"
}
```

如你所见，条件可以是一系列被逗号分割的值。但是它可以更多的匹配方式。比如，我们可以检测参数类型并进行判断：

```kotlin
when(view) {
    is TextView -> view.setText("I'm a TextView")
    is EditText -> toast("EditText value: ${view.getText()}")
    is ViewGroup -> toast("Number of children: ${view.getChildCount()} ")
    else -> view.visibility = View.GONE
}
```

再条件右边的代码中，参数会被自动转型，所以你不需要去明确地做类型转换。

它还让检测参数否在一个数组范围甚至是集合范围成为可能（我会在这章节的后面讲这个）：

```kotlin
val cost = when(x) {
	in 1..10 -> "cheap"
	in 10..100 -> "regular"
	in 100..1000 -> "expensive"
	in specialValues -> "special value!"
	else -> "not rated"
}
```

或者你甚至可以从对参数做需要的几乎疯狂的检查摆脱出来。它可以使用简单的`if/else`链替代：

```kotlin
valres=when{
	x in 1..10 -> "cheap"
	s.contains("hello") -> "it's a welcome!"
	v is ViewGroup -> "child count: ${v.getChildCount()}"
	else -> ""
}
```

### For循环

虽然你在使用了collections的函数操作符之后不会再过多地使用for循环，但是for循环再一些情况下仍然是很有用的。提供一个迭代器它可以作用在任何东西上面：

```kotlin
for (item in collection) {
	print(item)
}
```

如果你需要更多使用index的典型的迭代，我们也可以使用`ranges`（反正它通常是更加智能的解决方案）：

```kotlin
for (index in 0..viewGroup.getChildCount() - 1) {
    val view = viewGroup.getChildAt(index)
    view.visibility = View.VISIBLE
}
```

在我们迭代一个array或者list，一系列的index可以用来获取到指定的对象，所以上面的方式不是必要的：

```kotlin
for (i in array.indices)
	print(array[i])
```

# While和do/while循环

你也可以使用`while`循环，尽管它们两个都不是特别常用的。它们通常可以更简单、视觉上更容易理解的方式去解决一个问题，两个例子：

```kotlin
while(x > 0){ 
	x--
}

do{
	val y = retrieveData()
} while (y != null) // y在这里是可见的!
```

# Ranges

很难解释`control flow`，如果不去讲讲`ranges`的话。但是它们的范围要宽得多。`Range`表达式使用一个`..`操作符，它是被定义实现了一个`RangTo`方法。

`Ranges`帮助我们使用很多富有创造性的方式去简化我们的代码。比如我们可以把它：

```kotlin
if(i >= 0 && i <= 10) 
	println(i)
```

转化成：

```kotlin
if (i in 0..10)
    println(i)
```

`Range`被定义为可以被比较的任意类型，但是对于数字类型，比较器会通过转换它为简单的类似Java代码来避免额外开销的方式来优化它。数字类型的`ranges`也可以被迭代，编译器会转换它们为与Java中使用index的for循环的相同字节码的方式来进行优化：

```kotlin
for (i in 0..10)
    println(i)
```

`Ranges`默认会自增长，所以如果像以下的代码：

```kotlin
for (i in 10..0)
    println(i)
```

它就不会做任何事情。但是你可以使用`downTo`函数：

```kotlin
for(i in 10 downTo 0)
	println(i)
```

我们可以在`range`中使用`step`来定义一个从1到一个值的不同的空隙：

```kotlin
for (i in 1..4 step 2) println(i)

for (i in 4 downTo 1 step 2) println(i)
```

如果你想去创建一个open range（不包含最后一项，译者注：类似数学中的开区间），你可以使用`until`函数：

```kotlin
for (i in 0 until 4) println(i)
```

这一行会打印从0到3，但是会跳过最后一个值。这也就是说`0 until 4 == 0..3`。在一个list中迭代时，使用`(i in 0 until list.size)`比`(i in 0..list.size - 1)`更加容易理解。

就如之前所提到的，使用`ranges`确实有富有创造性的方式。比如，一个简单的方式去从一个`ViewGroup`中得到一个Views列表可以这么做：

```kotlin
val views = (0..viewGroup.childCount - 1).map { viewGroup.getChildAt(it) }
```

混合使用`ranges`和`函数操作符`可以避免我们使用明确地循环去迭代一个集合，还有明确地去创建一个我们用来添加views的list。所有的事情都在一行代码中做好了。

如果你想知道更多`ranges`的实现方式和更多的范例和有用的信息，你可以进入[Kotlin reference]

[Kotlin reference]:  https://kotlinlang.org/docs/reference/ranges.html


# 接口

Kotlin中的接口比Java 7中要强大得多。如果你使用Java 8，它们非常相似。在Kotlin中，我们可以像Java中那样使用接口。想象我们有一些动物，它们的其中一些可以飞行。这个是我们针对飞行动物的接口：

```kotlin
interface FlyingAnimal {
	fun fly()
}
```

鸟和蝙蝠都可以通过扇动翅膀的方式飞行。所以我们为它们创建两个类：

```kotlin
class Bird : FlyingAnimal {
    val wings: Wings = Wings()
    override fun fly() = wings.move()
}

class Bat : FlyingAnimal {
    val wings: Wings = Wings()
    override fun fly() = wings.move()
}
```

当两个类继承自一个接口，非常典型的是它们两者共享相同的实现。但是Java 7中的接口只能定义行为，但是不能去实现它。

Kotlin接口在某一方面它可以实现函数。它们与类唯一的不同之处是它们是无状态（stateless）的，所以属性需要子类去重写。类需要去负责保存接口属性的状态。

我们可以让接口实现`fly`函数：

```kotlin
interface FlyingAnimal {
    val wings: Wings
    fun fly() = wings.move()
}
```

就像提到的那样，类需要去重写属性：

```kotlin
class Bird : FlyingAnimal {
	override val wings: Wings = Wings()
}

class Bat : FlyingAnimal {
	override val wings: Wings = Wings()
}
```

现在鸟和蝙蝠都可以飞行了：

```kotlin
val bird = Bird()
val bat = Bat()

bird.fly()
bat.fly()
```

# 委托

[委托模式]是一个很有用的模式，它可以用来从类中抽取出主要负责的部分。委托模式是Kotlin原生支持的，所以它避免了我们需要去调用委托对象。委托者只需要指定实现的接口的实例。

在我们前面的例子中，我们可以通过构造函数指定动物怎么飞行，而不是实现它。比如，一个使用翅膀飞行的动物可以用这种方式指定：

```kotlin
interface CanFly {
	fun fly()
}

class Bird(f: CanFly) : CanFly by f
```

我们可以使用接口来指示鸟可以飞行，但是鸟的飞行方式被定义在一个委托中，这个委托定义在构造函数中，所以我们可以针对不同的鸟使用不同的飞行方式。动物使用翅膀飞行的方式被定义在另一个类中：

```kotlin
class AnimalWithWings : CanFly {
    val wings: Wings = Wings()
    override fun fly() = wings.move()
}
```

动物扇动翅膀来飞行。所以我们可以创建一个鸟，它使用翅膀飞行：

```kotlin
val birdWithWings = Bird(AnimalWithWings())
birdWithWings.fly()
```

但是现在翅膀可以被别的不是鸟类的动物使用。如果我们假设蝙蝠使用翅膀，我们可以直接指定委托来实例化对象：

```kotlin
class Bat : CanFly by AnimalWithWings()
...
val bat = Bat()
bat.fly()
```

[委托模式]: https://en.wikipedia.org/wiki/Delegation_pattern



# 泛型

### 基础

举个例子，我们可以创建一个指定泛型类：

```kotlin
class TypedClass<T>(parameter: T) {
    val value: T = parameter
}
```

这个类现在可以使用任何的类型初始化，并且参数也会使用定义的类型，我们可以这么做：

```kotlin
val t1 = TypedClass<String>("Hello World!")
val t2 = TypedClass<Int>(25)
```

但是Kotlin很简单并且缩减了模版代码，所以如果编译器能够推断参数的类型，我们甚至也就不需要去指定它：

```kotlin
val t1 = TypedClass("Hello World!")
val t2 = TypedClass(25)
val t3 = TypedClass<String?>(null)
```

如第三个对象接收一个null引用，那仍然还是需要指定它的类型，因为它不能去推断出来。

我们可以像Java中那样在定义中指定的方式来增加类型限制。比如，如果我们想限制上一个类中为非null类型，我们只需要这么做：

```kotlin
class TypedClass<T : Any>(parameter: T) { 
	val value: T = parameter
}
```

如果你再去编译前面的代码，你将看到`t3`现在会抛出一个错误。可null类型不再被允许了。但是限制明显可以更加严厉。如果我们只希望`Context`的子类该怎么做？很简单：

```kotlin
class TypedClass<T : Context>(parameter: T) { 
	val value: T = parameter
}
```

现在所有继承`Context`的类都可以在我们这个类中使用。其它的类型是不被允许的。

当然，可以使用函数中。我们可以相当简单地构建泛型函数：

```kotlin
fun <T> typedFunction(item: T): List<T> {
	...
}
```

# 变体

这是真的是最难理解的部分之一。在Java中，当我们使用泛型的时候会出现问题。逻辑告诉我们`List<String>`应该可以转型为`List<Object>`，因为它有更弱的限制。但是我们来看下这个例子：

```kotlin
List<String> strList = new ArrayList<>();
List<Object> objList = strList;
objList.add(5);
String str = objList.get(0);
```

如果Java编译器允许我们这么做，我们可以增加一个`Integer`到`Object` List，但是它明显会在某一时刻奔溃。这就是为什么语言中增加了通配符。通配符可以在限制这个问题中可以增加灵活性。

如果我们增加了`? extends Object`，我们使用了协变（`covariance`），它表示我们可以处理任何使用了类型，比Object更严格的对象，但是我们只有使用`get`操作时是安全的。如果我们想去拷贝一个`Strings`集合到`Objects`集合中，我们应该是允许的，对吧？然后，如果我们这样：

```kotlin
List<String> strList = ...;
List<Object> objList = ...;
objList.addAll(strList);
```

这样是可以的，因为定义在`Collection`接口中的`addAll()`是这样的：

```java
List<String>
interface Collection<E> ... {
	void addAll(Collection<? extends E> items);
}
```

否则，没有通配符，我们不会允许在这个方法中使用`String` List。相反地，当然会失败。我们不能使用`addAll()`来增加一个`Objects` List到`Strings` List中。因为我们只是用那个方法从`collection`中获取元素，这是一个完美的协变（`covariance`）的例子。

另一方面，我们可以在对立面上发现逆变（`contravariance`）。按照集合的例子，如果我们想把传过来的参数增加到集合中去，我们可以增加更加限制的类型到泛型集合中。比如，我们可以增加`Strings`到`Object`List：

```java
void copyStrings(Collection<? super String> to, Collection<String> from) {
    to.addAll(from);
}
```

增加`Strings`到另一个集合中唯一的限制就是那个集合接收`Strings`或者父类。

但是通配符都有它的限制。通配符定义了使用场景变体（`use-site variance`），这意味着当我们使用它的时候需要声明它。这表示每次我们声明一个泛型变量时都会增加模版代码。

让我们看一个例子。使用我们之前相似的类：

```java
class TypedClass<T> {
    public T doSomething(){
	    ...
    }
}
```

这些代码不会被编译：

```java
TypedClass<String> t1 = new TypedClass<>();
TypedClass<Object> t2 = t1;
```

尽管它的确没有意义，因为我们仍然保持了类中的所有的方法并且没有任何损坏。我们需要指定的类型可以有一个更加灵活的定义。

```kotlin
TypedClass<String> t1 = new TypedClass<>();
TypedClass<? extends String> t2 = t1;
```

这会让代码更加难以理解，而且增加了一些额外的模版代码。

另一方面，Kotlin通过内部声明变体（`declaration-site variance`）可以使用更加容易的方式来处理。这表示当我们定义一个类或者接口的时候我们可以处理弱限制的场景，我们可以在其它地方直接使用它。

所以让我们看看它在Kotlin中是怎么工作的。相比冗长的通配符，Kotlin仅仅使用`out`来针对协变（`covariance`）和使用`in`来针对逆变（`contravariance`）。在这个例子中，当我们类产生的对象可以被保存到弱限制的变量中，我们使用协变。我们可以直接在类中定义声明｛

```kotlin
class TypedClass<out T>() {
    fun doSomething(): T {
	    ...
	}
}
```

这就是所有我们需要的。现在，在Java中不能编译的代码在Kotlin中可以完美运行：

```kotlin
val t1 = TypedClass<String>()
val t2: TypedClass<Any> = t1
```

如果你已经使用了这些概念，我确信你可以很简单地在Kotlin使用`in`和`out`。否则，你也只是需要一些联系和概念上的理解。

# 泛型例子

理论之后，我们转移到一些实际功能上面，这会让我们更加简单地掌握它。为了不重复发明轮子，我使用三个Kotlin标准库中的三个函数。这些函数让我们仅使用泛型的实现就可以做一些很棒的事情。它可以鼓舞你创建自己的函数。

#### let

`let`实在是一个简单的函数，它可以被任何对象调用。它接收一个函数（接收一个对象，返回函数结果）作为参数，作为参数的函数返回的结果作为整个函数的返回值。它在处理可null对象的时候是非常有用的，下面是它的定义：

```kotlin
inline fun <T, R> T.let(f: (T) -> R): R = f(this)
```

它使用了两个泛型类型：`T` 和 `R`。第一个是被调用者定义的，它的类型被函数接收到。第二个是函数的返回值类型。

我们怎么去使用它呢？你可能还记得当我们从数据源中获取数据时，结果可能是null。如果不是null，则把结果映射到`domain model`并返回结果，否则直接返回null：

```kotlin
if (forecast != null) dataMapper.convertDayToDomain(forecast) else null
```

这代码是非常丑陋的，我们不需要使用这种方式去处理可null对象。实际上如果我们使用`let`，都不需要`if`：

```kotlin
forecast?.let { dataMapper.convertDayToDomain(it) }
```

多亏`?.`操作符，`let`函数只会在`forecast`不是null的时候才会执行。否则它会返回null。也就是我们想达到的效果。

#### with

本书中我们大量讲了这个函数。`with`接收一个对象和一个函数，这个函数会作为这个对象的扩展函数执行。这表示我们根据推断可以在函数内使用`this`。

```kotlin
inline fun <T, R> with(receiver: T, f: T.() -> R): R = receiver.f()
```

泛型在这里也是以相同的方式运行：`T`代表接收类型，`R`代表结果。如你所见，函数通过`f: T.() -> R`声明被定义成了扩展函数。这就是为什么我们可以调用`receiver.f()`。

通过这个app，我们有几个例子：

```kotlin
fun convertFromDomain(forecast: ForecastList) = with(forecast) {
    val daily = dailyForecast map { convertDayFromDomain(id, it) }
    CityForecast(id, city, country, daily)
}
```

#### apply

它看起来于`with`很相似，但是是有点不同之处。`apply`可以避免创建builder的方式来使用，因为对象调用的函数可以根据自己的需要来初始化自己，然后`apply`函数会返回它同一个对象：

```kotlin
inline fun <T> T.apply(f: T.() -> Unit): T { f(); return this }
```

这里我们只需要一个泛型类型，因为调用这个函数的对象也就是这个函数返回的对象。一个不错的例子：

```kotlin
val textView = TextView(context).apply {
    text = "Hello"
    hint = "Hint"
    textColor = android.R.color.white
}
```

它创建了一个`TextView`，修改了一些属性，然后赋值给一个变量。一切都很简单，具有可读性和坚固的语法。让我们用在当前的代码中。在`ToolbarManager`中，我们使用这种方式来创建导航drawable：

```kotlin
private fun createUpDrawable() = with(DrawerArrowDrawable(toolbar.ctx)) {
    progress = 1f
	this
}
```

使用`with`和返回`this`是非常清晰的，但是使用`apply`可以更加简单：

```kotlin
private fun createUpDrawable() = DrawerArrowDrawable(toolbar.ctx).apply {
	progress = 1f
}
```

你可以在`Kotlin for Android Developer`代码库中查看这些小的优化。


# 访问Shared Preferences

你可能知道什么是Android [Shared Preferences]。可以通过Android框架简单存储的一系列key和value对。这些`preferences`与SDK的一部分融为一体，使得任务变得更加容易。而且从Android 6.0（Marshmallow），`shared preferences`可以自动被云存储，所以当一个用户在一个新的设备上面恢复App的时候，它们的`preferences`也会被恢复。

多亏使用了属性委托，我们可以使用非常简单的方式来处理`preferences`。我们可以创建一个委托，当`get`被调用时去查询，当`set`被调用时去执行保存操作。

因为我们想去保存`zip code`，它是一个long型，所以让我们创建一个Long属性的委托吧。在`DelegatesExtensions.kt`中，实现一个新的`LongPreference`类：

```kotlin
class LongPreference(val context: Context, val name: String, val default: Long)
    :  ReadWriteProperty<Any?, Long> {

	val prefs by lazy {
        context.getSharedPreferences("default", Context.MODE_PRIVATE)
    }

	override fun getValue(thisRef: Any?, property: KProperty<*>): Long {
        return prefs.getLong(name, default)
	}
	
	override fun setValue(thisRef: Any?, property: KProperty<*>, value: Long) {
        prefs.edit().putLong(name, value).apply()
    }
}
```

首先，我们使用`lazy`委托的方式创建一个preferences。这样的话，如果我们没有使用这个属性，这个委托就不会请求这个`SharedPreferences`对象。

当`get`被调用，它的实现是使用preferences实例去获取一个委托声明中指定名字的long属性，如果没有找到这个属性，则默认使用default。当一个值被`set`，拿到`preferences editor`并使用属性名保存。

我们可以在`DelegatesExt`中定义一个新的委托，这样我们访问时就简单很多：

```kotlin
object DelegatesExt {
    ....
    fun longPreference(context: Context, name: String, default: Long) =
        LongPreference(context, name, default)
}
```

在`SettingActivity`，现在可以定义一个属性去处理`zip code`偏好。我创建了两个常量用来作为名字和属性的默认值。这种方式可以在App其他地方使用：

```kotlin
companion object {
    val ZIP_CODE = "zipCode"
    val DEFAULT_ZIP = 94043L
}

var zipCode: Long by DelegatesExt.longPreference(this, ZIP_CODE, DEFAULT_ZIP)
```

现在preference工作起来就非常简单了，我们可以从属性中得到并赋值给`EditText`：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    ...
    cityCode.setText(zipCode.toString())
}
```

我们不能使用自动生成的属性`text`，因为`EditText`在`getText`中返回的是`Editable`，所以该属性默认为该值。如果我尝试去分配一个`String`，编译器会报错，使用`setText()`就足够了。

现在具备了所有要实现`onBackPressed`的东西。这里，一个属性的新值会被储存：

```kotlin
override fun onBackPressed() {
    super.onBackPressed()
    zipCode = cityCode.text.toString().toLong()
}
```

`MainActivity`需要一些小的改变。首先，它也需要一个`zip code`属性。

```kotlin
val zipCode: Long by DelegatesExt.longPreference(this, SettingsActivity.ZIP_CODE,
            SettingsActivity.DEFAULT_ZIP)
```

然后，我把`forecast load`的代码移动到了`onResume`，这样每次activity resumed，它都会刷新数据，以防`code zip`被修改。当然这里有更加复杂一点的方式去做，比如通过在请求forecast之前检查是否`zip code`真的改变了。但是我像保持这个例子的简单性，而且因为请求的数据已经保存在本地数据库中了，所以这个解决方案也不算太坏：

```kotlin
override fun onResume() {
    super.onResume()
    loadForecast()
}

private fun loadForecast() = async {
    val result = RequestForecastCommand(zipCode).execute()
    uiThread {
	    val adapter = ForecastListAdapter(result) {
		    startActivity<DetailActivity>(DetailActivity.ID to it.id,
		            DetailActivity.CITY_NAME to result.city)
		}
		forecastList.adapter = adapter
		toolbarTitle = "${result.city} (${result.country})"
	} 
}
```

`RequestForecastCommand`现在使用`zipCode`而不是之前的是一个固定值。

这里还有意见我们必须要做的事情：当溢出菜单的`settings`点击时启动这个`setting activity`。在`ToolbarManager`中的`initToolbar`函数需要有一些小的修改：

```kotlin
when (it.itemId) {
    R.id.action_settings -> toolbar.ctx.startActivity<SettingsActivity>()
    else -> App.instance.toast("Unknown option")
}
```

[Shared Preferences]: http://developer.android.com/training/basics/data-storage/shared-preferences.html

# 泛型preference委托

现在我们已经是泛型专家了，为什么不扩展`LongPreference`为支持所有`Shared Preferences`支持的类型呢？我们来创建一个`Preference`委托：

```kotlin
class Preference<T>(val context: Context, val name: String, val default: T)
    : ReadWriteProperty<Any?, T> {
	
	val prefs by lazy {
	    context.getSharedPreferences("default", Context.MODE_PRIVATE)
    }
	
	override fun getValue(thisRef: Any?, property: KProperty<*>): T {
		return findPreference(name, default)
	}

	override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
        putPreference(name, value)
    }
	...
}
```

这个preference与我们之前使用的非常相似。我们仅仅替换了`Long`为泛型类型`T`，然后调用了两个函数来做具体重要的工作。这些函数非常简单，尽管有些重复。它们会检查类型然后使用指定的方式来操作。比如，`findPrefernce`函数如下：

```kotlin
private fun <T> findPreference(name: String, default: T): T = with(prefs) {
        val res: Any = when (default) {
        is Long -> getLong(name, default)
        is String -> getString(name, default)
        is Int -> getInt(name, default)
        is Boolean -> getBoolean(name, default)
        is Float -> getFloat(name, default)
        else -> throw IllegalArgumentException(
            "This type can be saved into Preferences")
    }

	res as T
}
```

`putPreference`函数也是一样，但是在`when`最后通过`apply`，使用`preferences editor`保存结果：

```kotlin
private fun <U> putPreference(name: String, value: U) = with(prefs.edit()) {
    when (value) {
        is Long -> putLong(name, value)
        is String -> putString(name, value)
        is Int -> putInt(name, value)
        is Boolean -> putBoolean(name, value)
        is Float -> putFloat(name, value)
        else -> throw IllegalArgumentException("This type can be saved into Pref\
erences")
    }.apply()
}
```

现在修改`DelegateExt`：

```kotlin
object DelegatesExt {
	...
	fun preference<T : Any>(context: Context, name: String, default: T)
	    = Preference(context, name, default)
}
```

这章之后，用户可以访问设置界面并修改`zip code`。然后当我们返回主界面，forecast会自动重新刷新并显示新的信息。查看代码库中其余xi wei

# 内部类

在Java中，我们可以在类的里面再定义类。如果它是一个通常的类，它不能去访问外部类的成员（就如Java中的static）：

```kotlin
class Outer {
  private val bar: Int = 1
	class Nested {
		fun foo() = 2
	}
}

val demo = Outer.Nested().foo() // == 2
```

如果需要去访问外部类的成员，我们需要用`inner`声明这个类：

```kotlin
class Outer {
	private val bar: Int = 1
	inner class Inner{
		fun foo() = bar
	}
}

val demo = Outer().Inner().foo() // == 1
```

# 枚举

Kotlin也提供了枚举（`enums`）的实现：

```kotlin
enum class Day {
    SUNDAY, MONDAY, TUESDAY, WEDNESDAY,
    THURSDAY, FRIDAY, SATURDAY
}
```

枚举可以带有参数：

```kotlin
enum class Icon(val res: Int) {
	UP(R.drawable.ic_up),
	SEARCH(R.drawable.ic_search),
	CAST(R.drawable.ic_cast)
}

val searchIconRes = Icon.SEARCH.res
```

枚举可以通过`String`匹配名字来获取，我们也可以获取包含所有枚举的`Array`，所以我们可以遍历它。

```kotlin
val search: Icon = Icon.valueOf("SEARCH")
val iconList: Array<Icon> = Icon.values()
```

而且每一个枚举都有一些函数来获取它的名字、声明的位置：

```kotlin
val searchName: String = Icon.SEARCH.name()
val searchPosition: Int = Icon.SEARCH.ordinal()
```

枚举根据它的顺序实现了 `Comparable`接口，所以可以很方便地把它们进行排序。

# 密封（Sealed）类

密封类用来限制类的继承关系，这意味着密封类的子类数量是固定的。看起来就像是枚举那样，当你想在一个密封类的子类中寻找一个指定的类的时候，你可以事先知道所有的子类。不同之处在于枚举的实例是唯一的，而密封类可以有很多实例，它们可以有不同的状态。

我们可以实现，比如类似Scala中的`Option`类：这种类型可以防止null的使用，当对象包含一个值时返回`Some`类，当对象为空时则返回`None`：

```kotlin
sealed class Option<out T> {
    class Some<out T> : Option<T>()
    object None : Option<Nothing>()
}
```

有一件关于密封类很不错的事情是当我们使用`when`表达式时，我们可以匹配所有选项而不使用`else`分支：

```kotlin
val result = when (option) {
    is Option.Some<*> -> "Contains a value"
    is Option.None -> "Empty"
}
```
# 异常（Exceptions）

在Kotlin中，所有的`Exception`都是实现了`Throwable`，含有一个`message`且未经检查。这表示我们不会强迫我们在任何地方使用`try/catch`。这与Java中不太一样，比如在抛出`IOException`的方法，我们需要使用`try-catch`包围代码块。通过检查exception来处理显示并不是一个好的方法。像[Bruce Eckel]、[Rod Waldhoff]或[Anders Hejlsberg]等人可以给你关于这个更好的观点。

抛出异常的方式与Java很类似：

```kotlin
throw MyException("Exception message")
```

`try`表达式也是相同的：

```kotlin
try{
	// 一些代码
}
catch (e: SomeException) {
	// 处理
}
finally {
	// 可选的finally块
}
```

在Kotlin中，`throw`和`try`都是表达式，这意味着它们可以被赋值给一个变量。这个在处理一些边界问题的时候确实非常有用：

```kotlin
val s = when(x){
	is Int -> "Int instance"
	is String -> "String instance"
	else -> throw UnsupportedOperationException("Not valid type")
}
```

或者

```kotlin
val s = try { x as String } catch(e: ClassCastException) { null }
```