## Kotlin语法

Kotlin 是JetBrains（就是开发很多IDE都公司，android官方开发平台android studio 就是基于Intellij，作为该平台的一个插件。）开发的基于JVM的语言。

Kotlin 是使用Java开发者思维被创建的，相比于java7的优势是：

* 它更加易表现：这是它最重要的优点之一。你可以编写少得多的代码。

* 它更加安全：Kotlin是空安全的，也就是说在我们编译时期就处理了各种null的情况，避免了执行时异常。如果一个对象可以是null，则我们需要明确地指定它，然后在使用它之前检查它是否是null。你可以节约很多调试空指针异常的时间，解决掉null引发的bug。

* 它是函数式的：Kotlin是基于面向对象的语言。但是就如其他很多现代的语言那样，它使用了很多函数式编程的概念，比如，使用lambda表达式来更方便地解决问题。其中一个很棒的特性就是Collections的处理方式。

* 它可以扩展函数：这意味着我们可以扩展类的更多的特性，甚至我们没有权限去访问这个类中的代码。

* 它是高度互操作性的：你可以继续使用所有的你用Java写的代码和库，因为两个语言之间的互操作性是完美的。甚至可以在一个项目中使用Kotlin和Java两种语言混合编程。

### Anko 
Anko 是JetBrains 开发的一个强大的库。 它主要的目的是用来替代以前的xml 方式来使用代码生成ui布局。可以尝试下。但是长时间使用XML布局会感觉XML容易多。但是习惯了就好了。

然而，这不是我们能在这个库中得到的唯一的功能。Anko 包含了很多的非常有帮助的函数和属性，来避免让我们写很多模版代码，我们将会通过例子来认识这个库。Anko的实现方式对学习大部分Kotlin语言都非常有帮助。

### 扩展函数

扩展函数是指在一个类上增加一种新的行为，甚至我们没有这个类代码的访问权限。这是一个在缺少有用函数的类上扩展的方法。在Java中，通常会实现很多带有static方法的工具类。Kotlin中扩展函数的一个优势是我们不需要在调用方法的时候把整个对象当作参数传入。扩展函数表现得就像是属于这个类的一样，而且我们可以使用`this`关键字和调用所有public方法。

例子：创建一个函数Toast 这个函数不需要传入任何Context，它 可以被任何Context或者它的子类调用，比如Activity和Service：（好神奇）

```kotlin
fun Context.toast(message: CharSequence, duration: Int = Toast.LENGTH_SHORT) {
    Toast.makeText(this, message, duration).show()
}
```
这个方法可以直接在activity 内部直接调用：
```kotlin
toast("Hello world!")
toast("Hello world!", Toast.LENGTH_LONG)
```
当然，Anko已经包括了自己的toast扩展函数，跟上面这个很相似。Anko提供了一些针对`CharSequence`和`resource`的函数，还有两个不同的toast和longToast方法：

```kotlin
toast("Hello world!")
longToast(R.id.hello_world)
```
扩展函数也可以是一个属性。所以我们可以通过相似的方法来扩展属性。下面的例子展示了使用他自己的getter/setter生成一个属性的方式。Kotlin由于互操作性的特性已经提供了这个属性，但理解扩展属性背后的思想是一个很不错的练习：

```kotlin
public var TextView.text: CharSequence
    get() = getText()
    set(v) = setText(v)
```
扩展函数并不是真正地修改了原来的类，它是以静态导入的方式来实现的。扩展函数可以被声明在任何文件中，因此有个通用的实践是把一系列有关的函数放在一个新建的文件里。

###  请求数据

kotlin扩展函数 让请求变得更简单，首先创建一个新的Request类：

```kotlin
public class Request(val url: String) {
    public fun run() {
        val forecastJsonStr = URL(url).readText()
        Log.d(javaClass.simpleName, forecastJsonStr)
    }
}
```
我们的请求很简单的接受一个url，然后读取结果，并打印日志。
实现非常简单。因为我们使用的'readText'，就是Kotlin 标准库中的扩展函数。

和java 代码比较， 我们仅使用标准库 就节省了大量的代码。 比如`HttpURLConnection`、`BufferedReader`和需要达到相同效果所必要的迭代结果，管理连接状态、reader等部分的代码。很明显，这些就是场景背后函数所作的事情，但是我们却不用关心。

为了可以执行请求，App必须要有Internet权限。所以需要在`AndroidManifest.xml`中添加：：

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

由于http请求不允许在main线程中执行。所以android中会使用`AsyncTask`,但是比较难用。
Anko 提供了非常简单的DSL 来处理异步任务。它满足大部分的需求，提供了个几本的`async`函数，用于在其他线程执行代码，也可以通过调用`uiThread`的方式回到主线程。

```kotlin
async() {
    Request(url).run()
    uiThread { longToast("Request performed") }
}
```
`UIThread`有一个很不错的一点就是可以依赖于调用者。如果它是被一个`Activity`调用的，那么如果`activity.isFinishing()`返回`true`，则`uiThread`不会执行，这样就不会在Activity销毁的时候遇到崩溃的情况了。

假如你想使用`Future`来工作，`async`返回一个Java `Future`。而且如果你需要一个返回结果的`Future`，你可以使用`asyncResult`。

真的很简单，对吧？而且比`AsyncTasks`更加具有可读性。现在，我仅仅给请求发送了一个url，来测试我们是否可以正确接收内容，这样我们才能在Activity中把它画出来。我很快会讲到怎么去进行json解析和转换成app中的数据类，但是在我们继续之前，学习什么是数据类也是很重要的。

检查代码并审查url请求和包结构的代码。你可以运行app并且确保你可以在打印的json日志和请求完毕之后的toast。

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

### 转换json 到数据类

当我们使用Gson来解析json到我们的类中，这些属性的名字必须要与json中的名字一样，或者可以指定一个`serialised name`（序列化名称）。一个好的实践是，大部分的软件结构中会根据我们app中布局来解耦成不同的模型。所以我喜欢使用声明简化这些类，因为我会在app其它部分使用它之前解析这些类。属性名称与json结果中的名字是完全一样的。

现在，为了返回被解析后的结果，我们的`Request`类需要进行一些修改。它将仍然只接收一个城市的`zipcode`作为参数而不是一个完整的url，因此这样变得更加具有可读性。现在，我会把这个静态的url放在一个`companion object`（伴随对象）中。如果我们之后还要对该API增加更多请求，我们需要提取它。

```kotlin

data class ForecastResult(val city: City, val list: List<Forecast>)

data class City(val id: Long, val name: String, val coord: Coordinates,
                val country: String, val population: Int)

data class Coordinates(val lon: Float, val lat: Float)

data class Forecast(val dt: Long, val temp: Temperature, val pressure: Float,
                    val humidity: Int, val weather: List<Weather>,
                    val speed: Float, val deg: Int, val clouds: Int,
                    val rain: Float)

data class Temperature(val day: Float, val min: Float, val max: Float,
                       val night: Float, val eve: Float, val morn: Float)

data class Weather(val id: Long, val main: String, val description: String,
                   val icon: String)

```
当我们使用Gson来解析json到我们的类中，这些属性的名字必须要与json中的名字一样，或者可以指定一个`serialised name`（序列化名称）。一个好的实践是，大部分的软件结构中会根据我们app中布局来解耦成不同的模型。所以我喜欢使用声明简化这些类，因为我会在app其它部分使用它之前解析这些类。属性名称与json结果中的名字是完全一样的。

现在，为了返回被解析后的结果，我们的`Request`类需要进行一些修改。它将仍然只接收一个城市的`zipcode`作为参数而不是一个完整的url，因此这样变得更加具有可读性。现在，我会把这个静态的url放在一个`companion object`（伴随对象）中。如果我们之后还要对该API增加更多请求，我们需要提取它。

> __Companion objects__
> Kotlin允许我们去定义一些行为与静态对象一样的对象。尽管这些对象可以用众所周知的模式来实现，比如容易实现的单例模式。

> 我们需要一个类里面有一些静态的属性、常量或者函数，我们可以使用`companion objecvt`。这个对象被这个类的所有对象所共享，就像Java中的静态属性或者方法。

以下是最后的代码：

```kotlin
public class ForecastRequest(val zipCode: String) {
    companion object {
        private val APP_ID = "15646a06818f61f7b8d7823ca833e1ce"
        private val URL = "http://api.openweathermap.org/data/2.5/" +
                "forecast/daily?mode=json&units=metric&cnt=7"
        private val COMPLETE_URL = "$URL&APPID=$APP_ID&q="
	}
	
    fun execute(): ForecastResult {
        val forecastJsonStr = URL(COMPLETE_URL + zipCode).readText()
        return Gson().fromJson(forecastJsonStr, ForecastResult::class.java)
	}
}

```

### 构建domain层

我们现在创建一个新的包作为`domain`层。这一层中会包含一些`Commands`的实现来为app执行任务。

首先，必须要定义一个`Command`：

```kotlin
public interface Command<T> {
	fun execute(): T
}
```

这个command会执行一个操作并且返回某种类型的对象，这个类型可以通过范型被指定。你需要知道一个有趣的概念，__一切kotlin函数都会返回一个值__。如果没有指定，它就默认返回一个`Unit`类。所以如果我们想让Command不返回数据，我们可以指定它的类型为Unit。

Kotlin中的接口比Java（Java 8以前）中的强大多了，因为它们可以包含代码。但是我们现在不需要更多的代码，以后的章节中会仔细讲这个话题。

第一个command需要去请求天气预报结构然后转换结果为domain类。下面这些类会在domain类中被定义：

```kotlin
data class ForecastList(val city: String, val country: String,
                        val dailyForecast:List<Forecast>)

data class Forecast(val date: String, val description: String, val high: Int,
                    val low: Int)
```

当更多的功能被增加，这些类可能会需要在以后被审查。但是现在这些类对我们来说已经足够了。

这些类必须从数据映射到我们的domain类，所以我接下来需要创建一个`DataMapper`：

```kotlin
public class ForecastDataMapper {
    fun convertFromDataModel(forecast: ForecastResult): ForecastList {
        return ForecastList(forecast.city.name, forecast.city.country,
                convertForecastListToDomain(forecast.list))
    private fun convertForecastListToDomain(list: List<Forecast>):
            List<ModelForecast> {
        return list.map { convertForecastItemToDomain(it) }
    }
    private fun convertForecastItemToDomain(forecast: Forecast): ModelForecast {
        return ModelForecast(convertDate(forecast.dt),
                forecast.weather[0].description, forecast.temp.max.toInt(),
                forecast.temp.min.toInt())
}
    private fun convertDate(date: Long): String {
        val df = DateFormat.getDateInstance(DateFormat.MEDIUM, Locale.getDefault())
		return df.format(date * 1000)
	}
}
```

当我们使用了两个相同名字的类，我们可以给其中一个指定一个别名，这样我们就不需要写完整的包名了：

```kotlin
import com.antonioleiva.weatherapp.domain.model.Forecast as ModelForecast
```

这些代码中另一个有趣的是我们从一个forecast list中转换为domain model的方法：

```kotlin
return list.map { convertForecastItemToDomain(it) }
```

这一条语句，我们就可以循环这个集合并且返回一个转换后的新的List。Kotlin在List中提供了很多不错的函数操作符，它们可以在这个List的每个item中应用这个操作并且任何方式转换它们。对比Java 7，这是Kotlin其中一个强大的功能。我们很快就会查看所有不同的操作符。知道它们的存在是很重要的，因为它们要方便得多，并可以节省很多时间和模版。

现在，编写命令前的准备就绪：

```kotlin
class RequestForecastCommand(val zipCode: String) :
				Command<ForecastList> {
	override fun execute(): ForecastList {
	    val forecastRequest = ForecastRequest(zipCode)
	    return ForecastDataMapper().convertFromDataModel(
	            forecastRequest.execute())
	}
}
```

### 在UI中绘制数据

`MainActivity`中的代码有些小的改动，因为现在有真实的数据需要填充到adapter中。异步调用需要被重写成：

```kotlin
async() {
	val result = RequestForecastCommand("94043").execute()
	uiThread{
		forecastList.adapter = ForecastListAdapter(result)
	}
}
```

Adapter也需要被修改：

```kotlin
class ForecastListAdapter(val weekForecast: ForecastList) :
        RecyclerView.Adapter<ForecastListAdapter.ViewHolder>() {
        
    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int):
            ViewHolder? {
        return ViewHolder(TextView(parent.getContext()))
    }
        
    override fun onBindViewHolder(holder: ViewHolder,
            position: Int) {
        with(weekForecast.dailyForecast[position]) {
            holder.textView.text = "$date - $description - $high/$low"
	} 
}
    override fun getItemCount(): Int = weekForecast.dailyForecast.size
    
    class ViewHolder(val textView: TextView) : RecyclerView.ViewHolder(textView)
}
```

> __with函数__
>  with是一个非常有用的函数，它包含在Kotlin的标准库中。它接收一个对象和一个扩展函数作为它的参数，然后使这个对象扩展这个函数。这表示所有我们在括号中编写的代码都是作为对象（第一个参数）的一个扩展函数，我们可以就像作为this一样使用所有它的public方法和属性。当我们针对同一个对象做很多操作的时候这个非常有利于简化代码。

### 扩展函数中的操作符

我们不需要去扩展我们自己的类，但是我需要去使用扩展函数扩展我们已经存在的类来让第三方的库能提供更多的操作。几个例子，我们可以去像访问List的方式去访问`ViewGroup`的view：

```kotlin
operator fun ViewGroup.get(position: Int): View = getChildAt(position)
```

现在真的可以非常简单地从一个`ViewGroup`中通过position得到一个view：

```kotlin
val container: ViewGroup = find(R.id.container)
val view = container[2]
```
### 简化setOnClickListener()

我们用Android中非常典型的例子去解释它是怎么工作的：`View.setOnClickListener()`方法。如果我们想用Java的方式去增加点击事件的回调，我首先要编写一个`OnClickListener`接口：

```java
public interface OnClickListener {
    void onClick(View v);
}
```

然后我们要编写一个匿名内部类去实现这个接口：

```java
view.setOnClickListener(new OnClickListener(){
	@Override
	public void onClick(View v) {
		Toast.makeText(v.getContext(), "Click", Toast.LENGTH_SHORT).show();
	}
})
```

我们将把上面的代码转换成Kotlin（使用了Anko的toast函数）：

```kotlin
view.setOnClickListener(object : OnClickListener {
	override fun onClick(v: View) {
		toast("Click")
	}
}
```

很幸运的是，Kotlin允许Java库的一些优化，Interface中包含单个函数可以被替代为一个函数。如果我们这么去定义了，它会正常执行：

```kotlin
fun setOnClickListener(listener: (View) -> Unit)
```

一个lambda表达式通过参数的形式被定义在箭头的左边（被圆括号包围），然后在箭头的右边返回结果值。在这个例子中，我们接收一个View，然后返回一个Unit（没有东西）。所以根据这种思想，我们可以把前面的代码简化成这样：

```kotlin
view.setOnClickListener({ view -> toast("Click")})
```

这是非常棒的简化！当我们定义了一个方法，我们必须使用大括号包围，然后在箭头的左边指定参数，在箭头的右边返回函数执行的结果。如果左边的参数没有使用到，我们甚至可以省略左边的参数：

```kotlin
view.setOnClickListener({ toast("Click") })
```

如果这个函数的最后一个参数是一个函数，我们可以把这个函数移动到圆括号外面：

```kotlin
view.setOnClickListener() { toast("Click") }
```

并且，最后，如果这个函数只有一个参数，我们可以省略这个圆括号：

```kotlin
view.setOnClickListener { toast("Click") }
```

比原始的Java代码简短了5倍多，并且更加容易理解它所做的事情。非常让人影响深刻。

### ForecastListAdapter的click listener

在前面一章，我这么艰苦地写了click listener的目的就是更好的在这一章中进行开发。然而现在是时候把你学到的东西用到实践中去了。我们从ForecastListAdapter中删除了listener接口，然后使用lambda代替：

```kotlin
public class ForecastListAdapter(val weekForecast: ForecastList,
								 val itemClick: (Forecast) -> Unit)
```

这个itemClick函数接收一个`forecast`参数然后不返回任何东西。`ViewHolder`中也可以这么修改：

```kotlin
class ViewHolder(view: View, val itemClick: (Forecast) -> Unit)
```

其它的代码保持不变。仅仅改变`MainActivity`：

```kotlin
val adapter = ForecastListAdapter(result) { forecast -> toast(forecast.date) }
```

我们可以简化最后一句。如果这个函数只接收一个参数，那我们可以使用`it`引用，而不用去指定左边的参数。所以我们可以这么做：

```kotlin
val adapter = ForecastListAdapter(result) { toast(it.date) }
```
### 扩展语言

多亏这些改变，我们可以去创建自己的`builder`和代码块。我们已经在使用一些有趣的函数，比如`with`。如下简单的实现：

```kotlin
inline fun <T> with(t: T, body: T.() -> Unit) { t.body() }
```

这个函数接收一个`T`类型的对象和一个被作为扩展函数的函数。它的实现仅仅是让这个对象去执行这个函数。因为第二个参数是一个函数，所以我们可以把它放在圆括号外面，所以我们可以创建一个代码块，在这这个代码块中我们可以使用`this`和直接访问所有的public的方法和属性：

```kotlin
with(forecast) {
	Picasso.with(itemView.ctx).load(iconUrl).into(iconView)
	dateView.text = date
	descriptionView.text = description
	maxTemperatureView.text = "$high"
	minTemperatureView.text = "$low"
	itemView.setOnClickListener { itemClick(this) }
}
```

> 内联函数
> 内联函数与普通的函数有点不同。一个内联函数会在编译的时候被替换掉，而不是真正的方法调用。这在一些情况下可以减少内存分配和运行时开销。举个例子，如果我们有一个函数，只接收一个函数作为它的参数。如果是一个普通的函数，内部会创建一个含有那个函数的对象。另一方面，内联函数会把我们调用这个函数的地方替换掉，所以它不需要为此生成一个内部的对象。

另一个例子：我们可以创建代码块只提供`Lollipop`或者更高版本来执行：

```kotlin
inline fun supportsLollipop(code: () -> Unit) {
	if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
		code()
	}
}
```

它只是检查版本，然后如果满足条件则去执行。现在我们可以这么做：

```kotlin
supportsLollipop {
	window.setStatusBarColor(Color.BLACK)
}
```

举个例子，Anko也是基于这个思想来实现`Android Layout`的`DSL`化。你也可以查看`Kotlin reference`中[`使用DSL来编写HTML`]的一个例子。

[`使用DSL来编写HTML`]: http://kotlinlang.org/docs/reference/type-safe-builders.html



### Kotlin Android Extensions

另一个Kotlin团队研发的可以让开发更简单的插件是`Kotlin Android Extensions`。当前仅仅包括了view的绑定。这个插件自动创建了很多的属性来让我们直接访问XML中的view。这种方式不需要你在开始使用之前明确地从布局中去找到这些views。

这些属性的名字就是来自对应view的id，所以我们取id的时候要十分小心，因为它们将会是我们类中非常重要的一部分。这些属性的类型也是来自XML中的，所以我们不需要去进行额外的类型转换。

`Kotlin Android Extensions`的一个优点是它不需要在我们的代码中依赖其它额外的库。它仅仅由插件组层，需要时用于生成工作所需的代码，只需要依赖于Kotlin的标准库。

那它背后是怎么工作的？该插件会代替任何属性调用函数，比如获取到view并具有缓存功能，以免每次属性被调用都会去重新获取这个view。需要注意的是这个缓存装置只会在`Activity`或者`Fragment`中才有效。如果它是在一个扩展函数中增加的，这个缓存就会被跳过，因为它可以被用在`Activity`中但是插件不能被修改，所以不需要再去增加一个缓存功能。

### 怎么去使用Kotlin Android Extensions

如果你还记得，现在项目已经准备好去使用Kotlin Android Extensions。当我们创建这个项目，我们就已经在`build.gradle`中增加了这个依赖：

```groovy
buldscript{
	repositories {
		jcenter()
	}
	dependencies {
		classpath "org.jetbrains.kotlin:kotlin-android-extensions:$kotlin_version"
	}
}
```

唯一一件需要这个插件做的事情是在类中增加一个特定的"手工"`import`来使用这个功能。我们有两个方法来使用它：

*  `Activities`或者`Fragments`的`Android Extensions`

这是最典型的使用方式。它们可以作为`activity`或`fragment`的属性是可以被访问的。属性的名字就是XML中对应view的id。

我们需要使用的`import`语句以`kotlin.android.synthetic`开头，然后加上我们要绑定到Activity的布局XML的名字：

```kotlin
import kotlinx.android.synthetic.activity_main.*
```

此后，我们就可以在`setContentView`被调用后访问这些view。新的Android Studio版本中可以通过使用`include`标签在Activity默认布局中增加内嵌的布局。很重要的一点是，针对这些布局，我们也需要增加手工的import：

```kotlin
import kotlinx.android.synthetic.activity_main.*
import kotlinx.android.synthetic.content_main.*
```

* `Views`的`Android Extensions`

前面说的使用还是有局限性的，因为可能有很多代码需要访问XML中的view。比如，一个自定义view或者一个adapter。举个例子，绑定一个xml中的view到另一个view。唯一不同的就是需要`import`：

```kotlin
import kotlinx.android.synthetic.view_item.view.*
```

如果我们需要一个adapter，比如，我们现在要从inflater的View中访问属性：

```kotlin
view.textView.text = "Hello"
```


现在是时候使用`Kotlin Android Extensions`来修改我们的代码了。修改相当简单。

我们从`MainActivity`开始。我们当前只是使用了`forecastList`的RecyclerView。但是我们可以简化一点代码。首先，为`activity_main`XML增加手工import：

```kotlin
import kotlinx.android.synthetic.activity_main.*
```

之前说过，我们使用id来访问views。所以我要修改`RecyclerView`的id，不使用下划线，让它更加适合Kotlin变量的名字。XML最后如下：

```xml
<FrameLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <android.support.v7.widget.RecyclerView
        android:id="@+id/forecastList"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>
</FrameLayout>
```

然后现在，我们可以不需要`find`这一行了：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    forecastList.layoutManager = LinearLayoutManager(this)
    ...
}
```

这已经是最小的简化了，因为这个布局非常简单。但是`ForecastListAdapter`也可以从这个插件中受益。这里你可以使用一个装置来绑定这些属性到view中，它可以帮助我们移除所有`ViewHolder`的`find`代码。

首先，为`item_forecast`增加手工导入：

```kotlin
import kotlinx.android.synthetic.item_forecast.view.*
```

然后现在我们可以在`ViewHolder`中使用包含在`itemView`中的属性。实际上你可以在任何view中使用这些属性，但是很显然如果view不包含要获取的子view就会奔溃。

现在我们可以直接访问view的属性了：

```kotlin
class ViewHolder(view: View, val itemClick: (Forecast) -> Unit) :
        RecyclerView.ViewHolder(view) {
    fun bindForecast(forecast: Forecast) {
        with(forecast){
	        Picasso.with(itemView.ctx).load(iconUrl).into(itemView.icon)
			itemView.date.text = date
			itemView.description.text = description
			itemView.maxTemperature.text = "${high.toString()}"
			itemView.minTemperature.text = "${low.toString()}"
			itemView.onClick { itemClick(forecast) }
		} 
	}
}
```
### Applicaton单例化

按照我们在Java中一样创建一个单例最简单的方式：

```kotlin
class App : Application() {
    companion object {
        private var instance: Application? = null
	    fun instance() = instance!!
    }
    override fun onCreate() {
        super.onCreate()
        instance = this
	}
}
```

为了这个Application实例被使用，要记得在`AndroidManifest.xml`中增加这个`App`：

```xml
<application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:theme="@style/AppTheme"
    android:name=".ui.App">
    ...
</application>
```

Android有一个问题，就是我们不能去控制很多类的构造函数。比如，我们不能初始化一个非null属性，因为它的值需要在构造函数中去定义。所以我们需要一个可null的变量，和一个返回非null值的函数。我们知道我们一直都有一个`App`实例，但是在它调用`onCreate`之前我们不能去操作任何事情，所以我们为了安全性，我们假设`instance()`函数将会总是返回一个非null的`app`实例。

但是这个方案看起来有点不自然。我们需要定义个一个属性（已经有了getter和setter），然后通过一个函数来返回那个属性。我们有其他方法去达到相似的效果么？是的，我们可以通过委托这个属性的值给另外一个类。这个就是我们知道的`委托属性`。

### 委托属性

我们可能需要一个属性具有一些相同的行为，使用`lazy`或者`observable`可以被很有趣地实现重用。而不是一次又一次地去声明那些相同的代码，Kotlin提供了一个委托属性到一个类的方法。这就是我们知道的`委托属性`。

当我们使用属性的`get`或者`set`的时候，属性委托的`getValue`和`setValue`就会被调用。

属性委托的结构如下：

```kotlin
class Delegate<T> : ReadWriteProperty<Any?, T> {
	fun getValue(thisRef: Any?, property: KProperty<*>): T {
		return ...
	}
	
	fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
		...
	}
}
```

这个T是委托属性的类型。`getValue`函数接收一个类的引用和一个属性的元数据。`setValue`函数又接收了一个被设置的值。如果这个属性是不可修改（val），就会只有一个`getValue`函数。

下面展示属性委托是怎么设置的：

```kotlin
class Example {
	var p: String by Delegate()
}
```

它使用了`by`这个关键字来指定一个委托对象。

###  标准委托

在Kotlin的标准库中有一系列的标准委托。它们包括了大部分有用的委托，但是我们也可以创建我们自己的委托。

* Lazy

它包含一个lambda，当第一次执行`getValue`的时候这个lambda会被调用，所以这个属性可以被延迟初始化。之后的调用都只会返回同一个值。这是非常有趣的特性， 当我们在它们第一次真正调用之前不是必须需要它们的时候。我们可以节省内存，在这些属性真正需要前不进行初始化。

```kotlin
class App : Application() {
    val database: SQLiteOpenHelper by lazy {
		MyDatabaseHelper(applicationContext)
	}

	override fun onCreate() {
	    super.onCreate()
	    val db = database.writableDatabase
    }
}
```

在这个例子中，database并没有被真正初始化，直到第一次调用`onCreate`时。在那之后，我们才确保applicationContext存在，并且已经准备好可以被使用了。`lazy`操作符是线程安全的。

如果你不担心多线程问题或者想提高更多的性能，你也可以使用`lazy(LazyThreadSafeMode.NONE){ ... }`。


* Observable

这个委托会帮我们监测我们希望观察的属性的变化。当被观察属性的`set`方法被调用的时候，它就会自动执行我们指定的lambda表达式。所以一旦该属性被赋了新的值，我们就会接收到被委托的属性、旧值和新值。

```kotlin
class ViewModel(val db: MyDatabase) {
	var myProperty by Delegates.observable("") {
	    d, old, new ->
	    db.saveChanges(this, new)
	}
}
```

这个例子展示了，一些我们需要关心的ViewMode，每次值被修改了，就会保存它们到数据库。


* Vetoable

这是一个特殊的`observable`，它让你决定是否这个值需要被保存。它可以被用于在真正保存之前进行一些条件判断。

```kotlin
var positiveNumber = Delegates.vetoable(0) {
    d, old, new ->
	new >= 0
}
```

上面这个委托只允许在新的值是正数的时候执行保存。在lambda中，最后一行表示返回值。你不需要使用return关键字（实质上不能被编译）。

* Not Null

有时候我们需要在某些地方初始化这个属性，但是我们不能在构造函数中确定，或者我们不能在构造函数中做任何事情。第二种情况在Android中很常见：在Activity、fragment、service、receivers……无论如何，一个非抽象的属性在构造函数执行完之前需要被赋值。为了给这些属性赋值，我们无法让它一直等待到我们希望给它赋值的时候。我们至少有两种选择方案。

第一种就是使用可null类型并且赋值为null，直到我们有了真正想赋的值。但是我们就需要在每个地方不管是否是null都要去检查。如果我们确定这个属性在任何我们使用的时候都不会是null，这可能会使得我们要编写一些必要的代码了。

第二种选择是使用`notNull`委托。它会含有一个可null的变量并会在我们设置这个属性的时候分配一个真实的值。如果这个值在被获取之前没有被分配，它就会抛出一个异常。

这个在单例App这个例子中很有用：

```kotlin
class App : Application() {
    companion object {
        var instance: App by Delegates.notNull()
	}
	
	override fun onCreate() {
        super.onCreate()
        instance = this
	}
}
```

* 从Map中映射值

另外一种属性委托方式就是，属性的值会从一个map中获取value，属性的名字对应这个map中的key。这个委托可以让我们做一些很强大的事情，因为我们可以很简单地从一个动态地map中创建一个对象实例。如果我们import `kotlin.properties.getValue`，我们可以从构造函数映射到`val`属性来得到一个不可修改的map。如果我们想去修改map和属性，我们也可以import `kotlin.properties.setValue`。类需要一个`MutableMap`作为构造函数的参数。

想象我们从一个Json中加载了一个配置类，然后分配它们的key和value到一个map中。我们可以仅仅通过传入一个map的构造函数来创建一个实例：

```kotlin
import kotlin.properties.getValue

class Configuration(map: Map<String, Any?>) {
    val width: Int by map
    val height: Int by map
    val dp: Int by map
    val deviceName: String by map
}
```

作为一个参考，这里我展示下对于这个类怎么去创建一个必须要的map：

```kotlin
conf = Configuration(mapOf(
    "width" to 1080,
    "height" to 720,
    "dp" to 240,
    "deviceName" to "mydevice"
))
```

### 怎么去创建一个自定义的委托

先来说说我们要实现什么，举个例子，我们创建一个`notNull`的委托，它只能被赋值一次，如果第二次赋值，它就会抛异常。

Kotlin库提供了几个接口，我们自己的委托必须要实现：`ReadOnlyProperty`和`ReadWriteProperty`。具体取决于我们被委托的对象是`val`还是`var`。

我们要做的第一件事就是创建一个类然后继承`ReadWriteProperty`：

```kotlin
private class NotNullSingleValueVar<T>() : ReadWriteProperty<Any?, T> {

		override fun getValue(thisRef: Any?, property: KProperty<*>): T {
	        throw UnsupportedOperationException()
        }
           
		override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
		}
}
```

这个委托可以作用在任何非null的类型。它接收任何类型的引用，然后像getter和setter那样使用T。现在我们需要去实现这些函数。

- Getter函数 如果已经被初始化，则会返回一个值，否则会抛异常。
- Setter函数 如果仍然是null，则赋值，否则会抛异常。

```kotlin
private class NotNullSingleValueVar<T>() : ReadWriteProperty<Any?, T> {
    private var value: T? = null
    override fun getValue(thisRef: Any?, property: KProperty<*>): T {
        return value ?: throw IllegalStateException("${desc.name} " +
                "not initialized")
	}
	
	override fun setValue(thisRef: Any?, property: KProperty<*>, value: T) {
		this.value = if (this.value == null) value
		else throw IllegalStateException("${desc.name} already initialized")
	}
}
```

现在你可以创建一个对象，然后添加函数使用你的委托：

```kotlin
object DelegatesExt {
    fun notNullSingleValue<T>():
            ReadWriteProperty<Any?, T> = NotNullSingleValueVar()
}
```
### 重新实现Application单例化

在这个情景下，委托就可以帮助我们了。我们直到我们的单例不会是null，但是我们不能使用构造函数去初始化属性。所以我们可以使用`notNull`委托：

```kotlin
class App : Application() {
    companion object {
        var instance: App by Delegates.notNull()
    }

	override fun onCreate() {
	        super.onCreate()
	        instance = this
	}
}
```

这种情况下有个问题，我们可以在app的任何地方去修改这个值，因为如果我们使用`Delegates.notNull()`，属性必须是var的。但是我们可以使用刚刚创建的委托，这样可以多一点保护。我们只能修改这个值一次：

```kotlin
companion object {
	var instance: App by DelegatesExt.notNullSingleValue()
}
```

尽管，在这个例子中，使用单例可能是最简单的方法，但是我想用代码的形式展示给你怎么去创建一个自定义的委托。

### 创建一个SQLiteOpenHelper

如你所知，Android使用SQLite作为它的数据库管理系统。SQLite是一个嵌入app的一个数据库，它的确是非常轻量的。这就是为什么这是手机app的不错的选择。

尽管如此，它的操作数据库的API在Android中是非常原生的。你将会需要编写很多SQL语句和你的对象与`ContentValues`或者`Cursors`之间的解析过程。很感激的，联合使用Kotlin和Anko，我们可以大量简化这些。

当然，有很多Android中可以使用的关于数据库的库，多亏Kotlin的互操作性，所有这些库都可以正常使用。但是针对一个简单的数据库来说可以不使用任何它们，之后的一分钟之内你就可以看到。
### ManagedSqliteOpenHelper

Anko提供了很多强大的SqliteOpenHelper来可以大量简化代码。当我们使用一个一般的`SqliteOpenHelper`，我们需要去调用`getReadableDatabase()`或者`getWritableDatabase()`，然后我们可以执行我们的搜索并拿到结果。在这之后，我们不能忘记调用`close()`。使用`ManagedSqliteOpenHelper`我们只需要：

```kotlin
forecastDbHelper.use {
	...
}
```

在lambda里面，我们可以直接使用`SqliteDatabase`中的函数。它是怎么工作的？阅读Anko函数的实现方式真是一件有趣的事情，你可以从这里学到Kotlin的很多知识：

```kotlin
public fun <T> use(f: SQLiteDatabase.() -> T): T {
    try {
	    return openDatabase().f()
	} finally {
	    closeDatabase()
    }
}
```

首先，`use`接收一个`SQLiteDatabase`的扩展函数。这表示，我们可以使用`this`在大括号中，并且处于`SQLiteDatabase`对象中。这个函数扩展可以返回一个值，所以我们可以像这么做：

```kotin
val result = forecastDbHelper.use {
    val queriedObject = ...
    queriedObject
}
```

要记住，在一个函数中，最后一行表示返回值。因为T没有任何的限制，所以我们可以返回任何对象。甚至如果我们不想返回任何值就使用`Unit`。

由于使用了`try-finall`，`use`方法会确保不管在数据库操作执行成功还是失败都会去关闭数据库。

而且，在`sqliteDatabase`中还有很多有用的扩展函数，我们会在之后使用到他们。但是现在让我们先定义我们的表和实现`SqliteOopenHelper`。

### 定义表

创建几个`objects`可以让我们避免表名列名拼写错误、重复等。我们需要两个表：一个用来保存城市的信息，另一个用来保存某天的天气预报。第二张表会有一个关联到第一张表的字段。

`CityForecastTable`提供了表的名字还有需要列：一个id（这个城市的zipCode），城市的名称和所在国家。

```kotlin
object CityForecastTable {
    val NAME = "CityForecast"
    val ID = "_id"
    val CITY = "city"
    val COUNTRY = "country"
}
```

`DayForecast`有更多的信息，就如你下面看到的有很多的列。最后一列`cityId`，用来保持属于的城市id。

```kotlin
object DayForecastTable {
	val NAME = "DayForecast"
	val ID = "_id"
	val DATE = "date"
	val DESCRIPTION = "description"
	val HIGH = "high"
	val LOW = "low"
	val ICON_URL = "iconUrl"
	val CITY_ID = "cityId"
}
```

### 实现SqliteOpenHelper

我们`SqliteOpenHelper`的基本组成是数据库的创建和更新，并提供了一个`SqliteDatebase`，使得我们可以用它来工作。查询可以被抽取出来放在其它的类中：

```kotlin
class ForecastDbHelper() : ManagedSQLiteOpenHelper(App.instance,
        ForecastDbHelper.DB_NAME, null, ForecastDbHelper.DB_VERSION) {
	...
}
```

我们在前面的章节中使用过我们创建的`App.instance`，这次我们同样的包括数据库名称和版本。这些值我们都会与`SqliteOpenHelper`一起定义在`companion object`中：

```kotlin
companion object {
    val DB_NAME = "forecast.db"
    val DB_VERSION = 1
    val instance: ForecastDbHelper by lazy { ForecastDbHelper() }
}
```

`instance`这个属性使用了`lazy`委托，它表示直到它真的被调用才会被创建。用这种方法，如果数据库从来没有被使用，我们没有必要去创建这个对象。一般`lazy`委托的代码块可以阻止在多个不同的线程中创建多个对象。这个只会发生在两个线程在同事时间访问这个`instance`对象，它很难发生但是发生具体还有看app的实现。无人如何，`lazy`委托是线程安全的。

为了去创建这些定义的表，我们需要去提供一个`onCreate`函数的实现。当没有库使用的时候，创建表会通过我们编写原生的包含我们定义好的列和类型的`CREATE TABLE`语句来实现。然而Anko提供了一个简单的扩展函数，它接收一个表的名字和一系列由列名和类型构建的`Pair`对象：

```kotlin
db.createTable(CityForecastTable.NAME, true,
        Pair(CityForecastTable.ID, INTEGER + PRIMARY_KEY),
        Pair(CityForecastTable.CITY, TEXT),
        Pair(CityForecastTable.COUNTRY, TEXT))
```

- 第一个参数是表的名称
- 第二个参数，当是true的时候，创建之前会检查这个表是否存在。
- 第三个参数是一个`Pair`类型的`vararg`参数。`vararg`也存在在Java中，这是一种在一个函数中传入联系很多相同类型的参数。这个函数也接收一个对象数组。

Anko中有一种叫做`SqlType`的特殊类型，它可以与`SqlTypeModifiers`混合，比如`PRIMARY_KEY`。`+`操作符像之前那样被重写了。这个`plus`函数会把两者通过合适的方式结合起来，然后返回一个新的`SqlType`：

```kotlin
fun SqlType.plus(m: SqlTypeModifier) : SqlType {
    return SqlTypeImpl(name, if (modifier == null) m.toString()
            else "$modifier $m")
}
```

如你所见，她会把多个修饰符组合起来。

但是回到我们的代码，我们可以修改得更好。Kotlin标准库中包含了一个叫`to`的函数，又一次，让我们来展示Kotlin的强大之处。它作为第一参数的扩展函数，接收另外一个对象作为参数，把两者组装并返回一个`Pair`。

```kotlin
public fun <A, B> A.to(that: B): Pair<A, B> = Pair(this, that)
```

因为带有一个函数参数的函数可以被用于inline，所以结果非常清晰：

```kotlin
val pair = object1 to object2
```

然后，把他们应用到表的创建中：

```kotlin
db.createTable(CityForecastTable.NAME, true,
        CityForecastTable.ID to INTEGER + PRIMARY_KEY,
        CityForecastTable.CITY to TEXT,
        CityForecastTable.COUNTRY to TEXT)
```

这就是整个函数看起来的样子：

```kotlin
override fun onCreate(db: SQLiteDatabase) {
    db.createTable(CityForecastTable.NAME, true,
            CityForecastTable.ID to INTEGER + PRIMARY_KEY,
            CityForecastTable.CITY to TEXT,
            CityForecastTable.COUNTRY to TEXT)
            
    db.createTable(DayForecastTable.NAME, true,
            DayForecastTable.ID to INTEGER + PRIMARY_KEY + AUTOINCREMENT,
            DayForecastTable.DATE to INTEGER,
            DayForecastTable.DESCRIPTION to TEXT,
            DayForecastTable.HIGH to INTEGER,
            DayForecastTable.LOW to INTEGER,
            DayForecastTable.ICON_URL to TEXT,
            DayForecastTable.CITY_ID to INTEGER)
}
```

我们有一个相似的函数用于删除表。`onUpgrade`将只是删除表，然后重建它们。我们只是把我们数据库作为一个缓存，所以这是一个简单安全的方法保证我们的表会如我们所期望的那样被重建。如果我有很重要的数据需要保留，我们就需要优化`onUpgrade`的代码，让它根据数据库版本来做相应的数据转移。

```kotlin
override fun onUpgrade(db: SQLiteDatabase, oldVersion: Int, newVersion: Int) {
    db.dropTable(CityForecastTable.NAME, true)
    db.dropTable(DayForecastTable.NAME, true)
    onCreate(db)
}
```

### 依赖注入

我试图不去增加很复杂的结构代码，保持简洁可测试性的代码和好的实践，我想我应该用Kotlin从其它方面去简化代码。如果你想了解一些控制反转或者依赖注入的话题，你可以查看我关于[Android中使用Dagger注入]的一系列文章。第一篇文章有关于他们这个团队的简单描写。

一种简单的方式，如果我们想拥有一些独立于其他类的类，这样更容易测试，并编写代码，易于扩展和维护，这时我们需要使用依赖注入。我们不去在类内部去实例化，我们在其它地方提供它们（通常通过构造函数）或者实例化它们。用这种方式，我们可以用其它的对象来替代他们。比如可以实现同样的接口或者在tests中使用mocks。

但是现在，这些依赖必须要在某些地方被提供，所以依赖注入由提供合作者的类组成。这些通常使用依赖注入器来完成。[Dagger]可能是Android上最流行的依赖注入器。当然当我们提供依赖有一定复杂性的时候是个很好的替代品。

但是最小的替代是可以在这个构造函数中使用默认值。我们可以给构造函数的参数通过分配默认值的方式提供一个依赖，然后在不同的情况下提供不同的实例。比如，在我们的`ForecastDbHelper`我们可以用更智能的方式提供一个context：

```kotlin
class ForecastDbHelper(ctx: Context = App.instance) :
        ManagedSQLiteOpenHelper(ctx, ForecastDbHelper.DB_NAME, null,
        ForecastDbHelper.DB_VERSION) {
        ...
}
```

现在我们有两种方式来创建这个类：

```kotlin
val dbHelper1 = ForecastDbHelper() // 它会使用 App.instance
val dbHelper2 = ForecastDbHelper(mockedContext) // 比如，提供给测试tests
```

我会到处使用这个特性，所以我在解释清楚之后再继续往下。我们已经有了表，所以是时候开始对它们增加和请求了。但是在这之前，我想先讲讲结合和函数操作符。别忘了查看代码库找到最新的代码。

[Android中使用Dagger注入]: http://http://antonioleiva.com/dependency-injection-android-dagger-part-1/
[Dagger]: http://square.github.io/dagger/

### 创建数据库model类

但是首先，我们要去为数据库创建model类。你还记得我们之前所见的map委托的方式？我们要把这些属性直接映射到数据库中，反过来也一样。

我们先来看下`CityForecast`类：

```kotlin
class CityForecast(val map: MutableMap<String, Any?>,
                           val dailyForecast: List<DayForecast>) {
    var _id: Long by map
    var city: String by map
    var country: String by map
    
    constructor(id: Long, city: String, country: String,
                dailyForecast: List<DayForecast>)
    : this(HashMap(), dailyForecast) {
        this._id = id
        this.city = city
        this.country = country
    }
}
```

默认的构造函数会得到一个含有属性和对应的值的map，和一个dailyForecast。多亏了委托，这些值会根据key的名字会映射到相应的属性中去。如果我们希望映射的过程运行完美，那么属性的名字必须要和数据库中对应的名字一模一样。我们后面会讲原因。

但是，第二个构造函数也是必要的。这是因为我们需要从domain映射到数据库类中，所以不能使用map，从属性中设置值也是方便的。我们传入一个空的map，但是又一次，多亏了委托，当我们设置值到属性的时候，它会自动增加所有的值到map中。用这种方式，我们就准备好map来保存到数据库中了。使用了这些有用的代码，我将会看见它运行起来就像魔法一样神奇。

现在我们需要第二个类，DayForecast，它会是第二个表。它包括表中的每一列作为它的属性，它也有第二个构造函数。唯一不同之处就是不需要设置id，因为它将通过SQLite自增长。

```kotlin
class DayForecast(var map: MutableMap<String, Any?>) {
	var _id: Long by map
	var date: Long by map
	var description: String by map
	var high: Int by map
	var low: Int by map
	var iconUrl: String by map
	var cityId: Long by map
	
	constructor(date: Long, description: String, high: Int, low: Int,
				iconUrl: String, cityId: Long)
	: this(HashMap()) {
		this.date = date
		this.description = description
		this.high = high
		this.low = low
		this.iconUrl = iconUrl
		this.cityId = cityId
	}
}
```

这些类将会帮助我们SQLite表与对象之间的互相映射。

### 写入和查询数据库

`SqliteOpenHelper`只是一个工具，是SQL世界和OOP之间的一个通道。我们要新建几个类来请求已经保存在数据库中的数据，和保存新的数据。被定义的类会使用`ForecastDbHelper`和`DataMapper`来转换数据库中的数据到`domain models`。我仍旧使用默认值的方式来实现简单的依赖注入：

```kotlin
class ForecastDb(
    val forecastDbHelper: ForecastDbHelper = ForecastDbHelper.instance,
    val dataMapper: DbDataMapper = DbDataMapper()) {
    ...
}
```

所有的函数使用前面章节讲到过的`use()`函数。lambda返回的值也会被作为这个函数的返回值。所以让我们定义一个使用`zip code`和`date`来查询一个`forecast`的函数：

```kotlin
fun requestForecastByZipCode(zipCode: Long, date: Long) = forecastDbHelper.use {
	...
}
```

这么没有什么解释的：我们使用`use`函数返回的结果作为这个函数返回的结果。

#### 查询一个forecast

第一个要做的查询就是每日的天气预报，因为我们需要这个列表来创建一个`city`对象。Anko提供了一个简单的请求构建器，所以我们来利用下这个有利条件：

```kotlin
val dailyRequest = "${DayForecastTable.CITY_ID} = ? " +
    "AND ${DayForecastTable.DATE} >= ?"
    
val dailyForecast = select(DayForecastTable.NAME)
        .whereSimple(dailyRequest, zipCode.toString(), date.toString())
        .parseList { DayForecast(HashMap(it)) }
```

第一行，`dailyRequest`是查询语句中`where`的一部分。它是`whereSimple`函数需要的第一个参数，这与我们用一般的helper做的方式很相似。这里有另外一个简化的`where`函数，它需要一些tags和values来进行匹配。我不太喜欢这个方式，因为我觉得这个增加了代码的模版化，虽然这个对我们把values解析成String很有利。最后它看起来会是这样：

```kotlin
val dailyRequest = "${DayForecastTable.CITY_ID} = {id}" + "AND ${DayForecastTable.DATE} >= {date}"

val dailyForecast = select(DayForecastTable.NAME)
        .where(dailyRequest, "id" to zipCode, "date" to date)
        .parseList { DayForecast(HashMap(it)) }
```

你可以选择你喜欢的一种方式。`select`函数是很简单的，它仅仅是需要一个被查询表的名字。`parse`函数的时候会有一些魔法在里面。在这个例子中我们假设请求结果是一个list，使用了`parseList`函数。它使用了`RowParser`或`RapRowParser`函数去把cursor转换成一个对象的集合。这两个不同之处就是`RowParser`是依赖列的顺序的，而`MapRowParser`是从map中拿到作为column的key名的。

在它们之间有两个重载的冲突，所以我们不能直接使用简化的方式准确地创建需要的对象。但是没有什么是不能通过扩展函数来解决的。我创建了一个接收一个lambda函数返回一个`MapRowParser`的函数。解析器会调用这个lambda来创建这个对象：

```kotlin
fun <T : Any> SelectQueryBuilder.parseList(
    parser: (Map<String, Any>) -> T): List<T> =
        parseList(object : MapRowParser<T> {
            override fun parseRow(columns: Map<String, Any>): T = parser(columns)
})
```

这个函数可以帮助我们简单地去`parseList`查询的结果：

```kotlin
parseList { DayForecast(HashMap(it)) }
```

解析器接收的`immutable map`被我们转化成了一个`mutable map`（我们需要在`database model`中是可以修改的）通过使用相应的`HashMap`构造函数。在`DayForecast`中的构造函数中会使用到这个`HashMap`。

所以，这个查询返回了一个`Cursor`，要理解这个场景的背后到底发生了什么。`parseList`中会迭代它，然后得到`Cursor`的每一行直到最后一个。对于每一行，它会创建一个包含这列的key和给对应的key赋值后的map。然后把这个map返回给这个解析器。

如果查询没有任何结果，`parseList`会返回一个空的list。

下一步查询城市也是一样的方法：

```kotlin
val city = select(CityForecastTable.NAME)
        .whereSimple("${CityForecastTable.ID} = ?", zipCode.toString())
        .parseOpt { CityForecast(HashMap(it), dailyForecast) }
```

不同之处是：我们使用的是`parseOpt`。这个函数返回一个可null的对象。结果可以使一个null或者单个的对象，这取决于请求是否能在数据库中查询到数据。这里有另外一个叫`parseSingle`的函数，本质上是一样的，但是它返回的事一个不可null的对象。所以如果没有在数据库中找到这一条数据，它会抛出一个异常。在我们的例子中，第一次查询一个城市的时候，肯定是不存在的，所以使用`parseOpt`会更安全。我又创建了一个好用的函数来阻止我们需要的对象的创建：

```kotlin
public fun <T : Any> SelectQueryBuilder.parseOpt(
    parser: (Map<String, Any>) -> T): T? =
        parseOpt(object : MapRowParser<T> {
            override fun parseRow(columns: Map<String, Any>): T = parser(columns)
        })
```

最后如果返回的city不是null，我们使用`dataMapper`把它转换成`domain object`再返回它。否则，我们直接返回null。你应该记得，lambda的最后一行表示返回值。所以这里将会返回一个`CityForecast?`类型的对象：

```kotlin
if (city != null) dataMapper.convertToDomain(city) else null
```

`DataMapper`函数很简单：

```kotlin
fun convertToDomain(forecast: CityForecast) = with(forecast) {
    val daily = dailyForecast.map { convertDayToDomain(it) }
    ForecastList(_id, city, country, daily)
}

private fun convertDayToDomain(dayForecast: DayForecast) = with(dayForecast) {
	Forecast(date, description, high, low, iconUrl)
}
```

最后完整的函数如下：

```kotlin
fun requestForecastByZipCode(zipCode: Long, date: Long) = forecastDbHelper.use {

	val dailyRequest = "${DayForecastTable.CITY_ID} = ? AND " +
	    "${DayForecastTable.DATE} >= ?"
	val dailyForecast = select(DayForecastTable.NAME)
	        .whereSimple(dailyRequest, zipCode.toString(), date.toString())
	        .parseList { DayForecast(HashMap(it)) }
	        
	val city = select(CityForecastTable.NAME)
	        .whereSimple("${CityForecastTable.ID} = ?", zipCode.toString())
	        .parseOpt { CityForecast(HashMap(it), dailyForecast) }

    if (city != null) dataMapper.convertToDomain(city) else null
}
```

另外一个Anko中好玩的功能我们在这里展示，那就是你可以使用`classParser()`来替代我们用的`MapRowParser`，它是基于列名通过反射的方式去生成对象的。我喜欢另一种方法因为我不需要使用反射并且还有控制权进行转换操作，但是在有时候可能对你有用。

#### 保存一个forecast

`saveForecast`函数只是从数据库中清除数据，然后转换`domain` model为数据库model，然后插入每一天的`forecast`和`city forecast`。这个结构比之前的更简单：它通过`use`函数从`database helper`中返回数据。在这个例子中我们不需要返回值，所以它将返回`Unit`。

```kotlin
fun saveForecast(forecast: ForecastList) = forecastDbHelper.use {
    ...
}
```

首先，我们清空这两个表。Anko没有提供比较漂亮的方式来做这个，但这并不意味着我们不行。所以我们创建了一个`SQLiteDatabase`的扩展函数来让我们可以像SQL查询一样来执行它：

```kotlin
fun SQLiteDatabase.clear(tableName: String) {
	execSQL("delete from $tableName")
}
```

清空这两个表：

```kotlin
clear(CityForecastTable.NAME)
clear(DayForecastTable.NAME)
```

现在，是时候去转换执行`insert`后返回的数据了。在这一点上你可能直到我是`with`函数的粉丝：

```kotlin
with(dataMapper.convertFromDomain(forecast)) {
	...
}
```

从`domain model`转换的方式也是很直接的：

```kotlin
fun convertFromDomain(forecast: ForecastList) = with(forecast) {
    val daily = dailyForecast.map { convertDayFromDomain(id, it) }
    CityForecast(id, city, country, daily)
}

private fun convertDayFromDomain(cityId: Long, forecast: Forecast) =
    with(forecast) {
        DayForecast(date, description, high, low, iconUrl, cityId)
	}
```

在代码块，我们可以在不使用引用和变量的情况下使用`dailyForecast`和`map`，只是像我们在这个类内部一样就可以了。针对插入我们使用另外一个Anko函数，它需要一个表名和一个`vararg`修饰的`Pair<String, Any>`作为参数。这个函数会把`vararg`转换成Android SDK需要的`ContentValues`对象。所以我们的任务组成是把`map`转换成一个`vararg`数组。我们为`MutableMap`创建了一个扩展函数：

```kotlin
fun <K, V : Any> MutableMap<K, V?>.toVarargArray():
    Array<out Pair<K, V>> =  map({ Pair(it.key, it.value!!) }).toTypedArray()
```

它是支持可null的值的（这是`map delegate`的条件），把它转换为非null值（`select`函数需要）的`Array`所组成的`Pairs`。不用担心就算你不完全理解这个函数，我很快就会讲到可空性。

所以，这个新的函数我们可以这么使用：

```kotlin
insert(CityForecastTable.NAME, *map.toVarargArray())
```

它在`CityForecast`中插入了一个一行新的数据。在`toVarargArray`函数结果前面使用`*`表示这个array会被分解成为一个`vararg`参数。这个在Java中是自动处理的，但是我们需要在Kotlin中明确指明。

每天的天气预报也是一样了：

```kotlin
dailyForecast.forEach { insert(DayForecastTable.NAME, *it.map.toVarargArray()) }
```

所以，通过`map`的使用，我们可以用很简单的方式把类转换为数据表，反之亦然。因为我们已经新建了扩展函数，我们可以在别的项目中使用，这个才是真正可贵的地方。

这个函数的完整代码如下：

```kotlin
fun saveForecast(forecast: ForecastList) = forecastDbHelper.use {
	clear(CityForecastTable.NAME)
	clear(DayForecastTable.NAME)
	
	with(dataMapper.convertFromDomain(forecast)) {
	    insert(CityForecastTable.NAME, *map.toVarargArray())
	    dailyForecast forEach {
	        insert(DayForecastTable.NAME, *it.map.toVarargArray())
	    }
	}
}
```
### 可null类型怎么工作

大部分现代语言使用某些方法去解决了这个问题，Kotlin的方法跟别的相似的语言比是相当另类和不同的。但是黄金准则还是一样：如果变量是可以是null，编译器强制我们去用某种方式去处理。

指定一个变量是可null是通过__在类型的最后增加一个问号__。因为在Kotlin中一切都是对象（甚至是Java中原始数据类型），一切都是可null的。所以，当然我们可以有一个可null的integer：

```kotlin
val a: Int? = null
```

一个可nul类型，你在没有进行检查之前你是不能直接使用它。这个代码不能被编译：

```kotlin
val a: Int? = null
a.toString()
```

前一行代码标记为可null，然后编译器就会知道它，所以在你null检查之前你不能去使用它。还有一个特性是当我们检查了一个对象的可null性，之后这个对象就会自动转型成不可null类型，这就是Kotlin编译器的智能转换：

```kotlin
vala:Int?=null
...
if(a!=null){
	a.toString()
}
```

在`if`中，`a`从`Int?`变成了`Int`，所以我们可以不需要再检查可null性而直接使用它。`if`代码之外，当然我们又得检查处理。这仅仅在变量当前不能被改变的时候才有效，因为否则这个value可能被另外的线程修改，这时前面的检查会返回false。`val`属性或者本地（`val or var`）变量。

这听起来会让事情变得更多。难道我们不得不去编写大量代码去进行可null性的检查？当然不是，首先，因为大多数时候你不需要使用null类型。Null引用没有我们想象中的有用，当你想弄清楚一个变量是否可以为null时你就会发现这一点。但是Kotlin也有它自己的使处理更简洁的方案。举个例子，我们如下简化代码：

```kotlin
val a: Int? = null
...
a?.toString()
```

这里我们使用了安全访问操作符(`?`)。只有这个变量不是null的时候才会去执行前面的那行代码。否则，它不会做任何事情。并且我们甚至可以使用__Elvis operator__(`?:`)：

```kotlin
val a:Int? = null
val myString = a?.toString() ?: ""
```

因为在Kotlin中`throw`和`return`都是表达式，他们可以用在__Elvis operator__操作符的右边：

```kotlin
val myString = a?.toString() ?: return false

val myString = a?.toString() ?: throw IllegalStateException()
```

然后，我们可能会遇到这种情景，我们确定我们是在用一个非null变量，但是他的类型却是可null的。我们可以使用`!!`操作符来强制编译器执行可null类型时跳过限制检查：

```kotlin
val a: Int? = null
a!!.toString()
```

上面的代码将会被编译，但是很显然会奔溃。所以我们要确保只能在特定的情况下使用。通常我们可以自己选择作为解决方案。如果一份代码满篇都是`!!`，那就有股代码没有被正确处理的气味了。

### 可null性和Java库

好了，前面的章节解释了使用Kotlin代码完美地工作。但是与普通的Java库和Android SDK会发生什么呢？在Java中，所有对象可以被定义为null。所以我们不得不处理大量潜在的在现实中不可能是null的null变量。这意味着我们的代码最后可能会有几百个`!!`操作符，这绝对不是一个好的主意。

当我们去处理Android SDK时，你可能看见所有Java方法的参数被标记为单个的`!`。比如，Java中在一些获取对象的方法在Kotlin中显示返回`Any!`。这表示让开发者自己决定是否这个变量是否可null。

很幸运，新版本的Android开始使用`@Nullable`和`@NonNull`注解来辨别参数是否可以是null或者否个函数是否可以返回null。当我们怀疑时，我们可以进入源码去检查是否会接收到一个null对象。我的猜想是在以后，编译器能够读取这些注解，然后强制（或者至少是建议）一个更好的方法。

现在开始，当一个Jetbrains的`@Nullable`注解（这个与Android的注解不同）被注解在一个非null的变量时，就会获得一个警告。相对的没有发生在`@NotNull`注解上。

所以我们来举个例子，如果我们创建了一个Java的测试类：

```java
import org.jetbrains.annotations.Nullable;
public class NullTest {

	@Nullable
	public Object getObject(){
		return "";
	}
}
```

然后在Kotlin中使用：

```kotlin
val test = NullTest()
val myObject: Any = test.getObject()
```

我们会发现，在`getObject`函数上会显示一个警告。但是这只是从现在才开始的编译器检查，并且它还不认识Android的注解，所以我们可能不得不花更多的时间来等待一个更智能的方式。不管怎么样，使用源码注解的方式和一些Androd SDK的知识，我们也很难犯错误。

比如重写`Activity`的`onCraete`函数，我们可以决定是否让`savedInstanceState`可null：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
}

override fun onCreate(savedInstanceState: Bundle) {
}
```

这两种方法都会被编译，但是第二种是错误的，因为一个Activity很可能接收到一个null的bundle。只要小心一点点就足够了。当你有疑问时，你可以就用可null的对象然后处理掉用可能的null。记住，如果你使用了 `!!`，可能是因为你确信对象不可能为null，如果是这样，请定义为非null。

这个灵活性在Java库中真的很有必要，而且随着编译器的进化，我们将可能看到更好的交互（可能是基于注解的），但是现在来说这个机制已经足够灵活了。

### 创建业务逻辑来访问数据

在实现访问服务器和与本地数据库交互之后，是时候把事情整合起来了。逻辑步骤如下：

- 从数据库获取数据
- 检查是否存在对应星期的数据
- 如果有，返回UI并且渲染
- 如果没有，请求服务器获取数据
- 结果被保存在数据库中并且返回UI渲染

但是我们的`commands`不应该去处理所有这些逻辑。数据源应该是一个具体的实现，这样就可以被容易地修改，所以增加一些额外的代码，然后把`command`从数据访问中抽象出来听起来是个不错的方式。在我们的实现中，它会遍历整个list直到结果被找到。

所以我们先来给接口定义一些我们实现`provider`需要使用到的数据源：

```kotlin
interface ForecastDataSource {
    fun requestForecastByZipCode(zipCode: Long, date: Long): ForecastList?
}
```

`provider`需要一个接收`zip code`和一个`date`，然后它应该根据那一天返回一周的天气预报。

```kotlin
class ForecastProvider(val sources: List<ForecastDataSource> =
        ForecastProvider.SOURCES) {
        
	companion object {
	    val DAY_IN_MILLIS = 1000 * 60 * 60 * 24
	    val SOURCES = listOf(ForecastDb(), ForecastServer())
	}
    ...
}
```

`forecast provider`接收一个数据源列表，通过构造函数传入（比如用于测试），但是我设置了source的默认值为被定义在`companion object`中的`SOURCES`List。我将使用数据库的数据源和服务端数据源。顺序是很重要的，因为它会根据顺序去遍历这个sources，然后一旦获取到有效的返回值就会停止查询。逻辑顺序是先在本地查询（本地数据库中），然后再通过API查询。

所以主函数的代码如下：

```kotlin
fun requestByZipCode(zipCode: Long, days: Int): ForecastList
            = sources.firstResult { requestSource(it, days, zipCode) }
```

它会得到第一个不是null的结果然后返回。当我在第18章中讲到的大量的函数操作符中搜索后，我没有找到完全符合我想要的。所以当我去查看Kotlin的源码时，我直接拷贝了`first`函数然后修改它们来达到我想要的目的：

```kotlin
inline fun <T, R : Any> Iterable<T>.firstResult(predicate: (T) -> R?) : R {
	for (element in this){
		val result = predicate(element)
		if (result != null) return result
	}
	throw NoSuchElementException("No element matching predicate was found.")
}
```

该函数接收一个断言函数，它接收一个`T`类型的对象然后返回一个`R?`类型的值。这表示`predicate`可以返回null类型，但是我们的`firstResult`不能返回null。这就是为什么返回`R`的原因。

它怎么工作呢？它将遍历集合中的每一个元素然后执行这个断言函数。当这个断言函数的结果返回不是null时，这个结果就会被返回。

如果我们可以允许sources返回null，那我们就可以使用`firstOrNull`函数来代替。不同之处就是最后一行的返回null和抛异常。但是我现在不在代码里面去处理这些细节了。

在我们的例子中`T = ForecastDataSource`，`R = ForecastList`。但是记住在`ForecastDataSource`中指定的函数返回一个`ForecastList?`，也就是`R?`，所以一切都是匹配得这么完美。`requestSource`让前面的函数看起来更有可读性：

```kotlin
fun requestSource(source: ForecastDataSource, days: Int, zipCode: Long):
        ForecastList? {
    val res = source.requestForecastByZipCode(zipCode, todayTimeSpan())
    return if (res != null && res.size() >= days) res else null
}
```

如果结果不是null并且数量也参数匹配，那这个查询被执行且只会返回一个数据。否则，数据源没有足够的数据来返回一个成功的结果。

函数`todayTimeSpan()`计算今天毫秒级的时间，并排除掉“时差”。其中一些数据源（我们例子中的数据库）可能会需要它。因为如果我们没有指定更多的信息，服务端默认就是今天，所以我们不需要设置它。

```kotlin
private fun todayTimeSpan() = System.currentTimeMillis() / DAY_IN_MILLIS * DAY_IN_MILLIS
```

这个类完整的代码如下：

```kotlin
class ForecastProvider(val sources: List<ForecastDataSource> =
        ForecastProvider.SOURCES) {

	companion object {
        val DAY_IN_MILLIS = 1000 * 60 * 60 * 24;
        val SOURCES = listOf(ForecastDb(), ForecastServer())
    }

	fun requestByZipCode(zipCode: Long, days: Int): ForecastList
            = sources.firstResult { requestSource(it, days, zipCode) }

	private fun requestSource(source: RepositorySource, days: Int,
            zipCode: Long): ForecastList? {
        val res = source.requestForecastByZipCode(zipCode, todayTimeSpan())
        return if (res != null && res.size() >= days) res else null
    }

	private fun todayTimeSpan() = System.currentTimeMillis() /
            DAY_IN_MILLIS * DAY_IN_MILLIS
}
```

我们已经定义了一个`ForecastDb`。现在我们需要去实现`ForcastDataSource`：

```kotlin
class ForecastDb(val forecastDbHelper: ForecastDbHelper =
        ForecastDbHelper.instance, val dataMapper: DbDataMapper = DbDataMapper())
        : ForecastDataSource {
        
    override fun requestForecastByZipCode(zipCode: Long, date: Long) =
            forecastDbHelper.use {
            ...
	}
	...
}
```

`ForecastServer`还没有还被实现，但是这是非常简单的。它在从服务端接收到数据之后就会使用`ForecastDb`去保存到数据库。用这种方式，我们就可以缓存这些数据到数据库中，提供给以后的查询。

```kotlin
class ForecastServer(val dataMapper: ServerDataMapper = ServerDataMapper(),
        val forecastDb: ForecastDb = ForecastDb()) : ForecastDataSource {
        
    override fun requestForecastByZipCode(zipCode: Long, date: Long):
            ForecastList? {
        val result = ForecastByZipCodeRequest(zipCode).execute()
        val converted = dataMapper.convertToDomain(zipCode, result)
        forecastDb.saveForecast(converted)
        return forecastDb.requestForecastByZipCode(zipCode, date)
	}
}
```

它也是使用了之前我们创建的`data mapper`，最然我们修改一些函数的名字来让它更加与我们之前用在`database model`的mapper更相似。你可以查看`provider`来查看细节。

被重写的方法用来请求服务器，转换结果到`domain objects`并保存它们到数据库。它最后查询数据库返回数据，这是因为我们需要使用到插入到数据库中的字增长id。

这就是`provider`被实现的最后的一步了。现在我们需要开始使用它。`ForecastCommand`不会再直接与服务端交互，也不会转换数据到`domain model`。

```kotlin
RequestForecastCommand(val zipCode: Long,
        val forecastProvider: ForecastProvider = ForecastProvider()) :
        Command<ForecastList> {
        
	companion object {
		val DAYS = 7
	}
	
	override fun execute(): ForecastList {
		return forecastProvider.requestByZipCode(zipCode, DAYS)
	}
}
```

其它修改的地方包括重命名和包的结构调整。在[Kotlin for Android Developers repository]查看相应的提交。

[Kotlin for Android Developers repository]: https://github.com/antoniolg/Kotlin-for-Android-Developers

# 准备请求

因为我们需要知道哪一个item我们要在详情界面中显示出来，所以逻辑告诉我们需要发送一个天气预报的`id`到详情界面。所以`domain model`需要一个新的`id`属性：

```kotlin
data class Forecast(val id: Long, val date: Long, val description: String,
    val high: Int, val low: Int, val iconUrl: String)
```

`ForecastProvider`也需要一个新的函数，它返回通过`id`请求后的结果。`DetailActivity`将需要通过接收到的`id`来执行请求获取天气预报数据。因为所有的请求都会迭代所有的数据源并且返回第一个非null的结果，我们可以抽取并定义一个新的函数：

```kotlin
private fun <T : Any> requestToSources(f: (ForecastDataSource) -> T?): T
        = sources.firstResult { f(it) }
```

这个函数使用一个非null类型作为范型。它会接收一个函数，并返回一个可null的对象。其中这个接收的函数接收一个`ForecastDataSource`，并返回一个可null范型的对象。我们可以重写上一个请求并如下写一个新的：

```kotlin
fun requestByZipCode(zipCode: Long, days: Int): ForecastList = requestToSources {
	val res = it.requestForecastByZipCode(zipCode, todayTimeSpan())
	if (res != null && res.size() >= days) res else null
}

fun requestForecast(id: Long): Forecast = requestToSources {
	it.requestDayForecast(id)
}
```

现在数据源需要去实现一个新的函数：

```kotlin
fun requestDayForecast(id: Long): Forecast?
```

`ForcastDb`将总是会拿到所需的在上一次请求被缓存的值，所以我们可以通过这种方式去获取它：

```kotlin
override fun requestDayForecast(id: Long): Forecast? = forecastDbHelper.use {
    val forecast = select(DayForecastTable.NAME).byId(id).
            parseOpt { DayForecast(HashMap(it)) }
    if (forecast != null) dataMapper.convertDayToDomain(forecast) else null
}
```

`select`从查询与之前的非常相似。我创建了另一个名为`byId`的工具函数，因为通过`id`来请求是很通用的，像这样使用一个函数可以简化处理过程也更具可读性。函数的实现也是相当简单：

```kotlin
fun SelectQueryBuilder.byId(id: Long): SelectQueryBuilder
        = whereSimple("_id = ?", id.toString())
```

它只是使用了`whereSimple`函数实现使用`_id`字段来查询数据。这个函数相当普通，但是如你所见，你可以根据你数据库结构的需要来创建需要的扩展函数，它可以大量地简化你代码的可读性。`DataMapper`有一些不值得一提的轻微改变。你可以通过代码库来查看它们。

另一方面，`ForecastServer`将不会再使用，因为信息总是会被缓存在数据库中。我们可以在一些奇怪的场景下实现一些代码保护，但是我们在这个例子中没有做任何处理，所以如果发生它也会只是抛出一个异常：

```kotlin
override fun requestDayForecast(id: Long): Forecast?
        = throw UnsupportedOperationException()
```

> `try`和`throw`是表达式
> > 在Kotlin中，几乎一切都是表达式，也就是说一切都会返回一个值。这在函数式编程中是非常重要的，当你使用`try-catch`处理边界的问题或者当抛出异常的时候。比如，在上一个例子中，我们可以给结果分配一个exception就算他们不是相同的类型，而不是必须要去创建一个完整的代码块。当我们需要在一个`when`分支中抛出一个exception的时候也是非常有用：
>>```kotlin
	val x = when(y){ 
				in 0..10 -> 1
                in 11..20 -> 2
                else -> throw Exception("Invalid")
	}
>>```
>> `try-catch`中也是一样，我们可以根据try的结果分配一个值：
>> ```kotlin
	val x = try{ doSomething() }catch{ null }
>> ```

最后一件我们需要做的事就是在新的activity中创建一个command来执行请求：

```kotlin
class RequestDayForecastCommand(
        val id: Long,
        val forecastProvider: ForecastProvider = ForecastProvider()) :
        Command<Forecast> {
    override fun execute() = forecastProvider.requestForecast(id)
}
```

请求返回一个将用于activity绘制UI的`Forecast`结果。

# 提供一个新的activity

现在我们准备去创建一个`DetailActivity`。我们详情activity将会接收一组从主activity传过来的参数：`forecast id`和`城市名称`。第一个参数将会用来从数据库中请求数据，城市名称用于显示在toolbar上。所以我们首先需要定义一组参数的名字：

```kotlin
public class DetailActivity : AppCompatActivity() {
	companion object {
	    val ID = "DetailActivity:id"
	    val CITY_NAME = "DetailActivity:cityName"
    }
    ...
}
```

在`onCreate`函数中，第一步是去设置content view。UI是非常简单的，但是对于这个app来说是足够了：

```kotlin
<LinearLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	xmlns:tools="http://schemas.android.com/tools"
	android:layout_width="match_parent"
	android:layout_height="match_parent"
	android:orientation="vertical"
	android:paddingBottom="@dimen/activity_vertical_margin"
	android:paddingLeft="@dimen/activity_horizontal_margin"
	android:paddingRight="@dimen/activity_horizontal_margin"
	android:paddingTop="@dimen/activity_vertical_margin">
	
	<LinearLayout
		android:layout_width="match_parent"
		android:layout_height="wrap_content"
		android:orientation="horizontal"
		android:gravity="center_vertical"
		tools:ignore="UseCompoundDrawables">
	
		<ImageView
			android:id="@+id/icon"
			android:layout_width="64dp"
			android:layout_height="64dp"
			tools:src="@mipmap/ic_launcher"
			tools:ignore="ContentDescription"/>
		
		<TextView
			android:id="@+id/weatherDescription"
			android:layout_width="wrap_content"
			android:layout_height="wrap_content"
			android:layout_margin="@dimen/spacing_xlarge"
			android:textAppearance="@style/TextAppearance.AppCompat.Display1"
		    tools:text="Few clouds"/>
    </LinearLayout>
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <TextView
            android:id="@+id/maxTemperature"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:layout_margin="@dimen/spacing_xlarge"
            android:gravity="center_horizontal"
            android:textAppearance="@style/TextAppearance.AppCompat.Display3"
            tools:text="30"/>
        <TextView
            android:id="@+id/minTemperature"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:layout_margin="@dimen/spacing_xlarge"
            android:gravity="center_horizontal"
            android:textAppearance="@style/TextAppearance.AppCompat.Display3"
            tools:text="10"/>
    </LinearLayout>
</LinearLayout>
```

然后在`onCreate`代码中去设置它。使用城市的名字设置成toolbar的title。`intent`和`title`通过下面的方法被自动影射到属性：

```kotlin
setContentView(R.layout.activity_detail)
title = intent.getStringExtra(CITY_NAME)
```

`onCreate`实现的另一部分是调用command。这与我们之前做的非常相似：

```kotlin
async {
       val result = RequestDayForecastCommand(intent.getLongExtra(ID, -1)).execute()
       uiThread { bindForecast(result) }
}
```

当结果从数据库中获取之后，`bindForecast`函数在UI线程中被调用。我们在这个activity中又一次使用了Kotlin Android Extensions插件来实现不使用`findViewById`来从XML中获取到属性：

```kotlin
import kotlinx.android.synthetic.activity_detail.*
...

private fun bindForecast(forecast: Forecast) = with(forecast) {
       Picasso.with(ctx).load(iconUrl).into(icon)
       supportActionBar.subtitle = date.toDateString(DateFormat.FULL)
       weatherDescription.text = description
       bindWeather(high to maxTemperature, low to minTemperature)
}
```

这里有一些有趣的地方。比如，我创建了另一个扩展函数来转换一个`Long`对象到一个用于显示的日期字符串。记住我们在adapter中也使用了，所以明确定义它为一个函数是个不错的实践：

```kotlin
fun Long.toDateString(dateFormat: Int = DateFormat.MEDIUM): String {
    val df = DateFormat.getDateInstance(dateFormat, Locale.getDefault())
    return df.format(this)
}
```

我会得到一个`date format`（或者使用默认的DateFormat.MEDIUM）并转换`Long`为一个用户可以理解的`String`。

另一个有趣的地方是`bindWeather`函数。它会接收一个`vararg`的由`Int`和`TextView`组成的`pairs`，并且根据温度给`TextView`设置不同的`text`和`text color`。

```kotlin
private fun bindWeather(vararg views: Pair<Int, TextView>) = views.forEach {
    it.second.text = "${it.first.toString()}￿￿"
    it.second.textColor = color(when (it.first) {
        in -50..0 -> android.R.color.holo_red_dark
        in 0..15 -> android.R.color.holo_orange_dark
        else -> android.R.color.holo_green_dark
	})
}
```

每一个pair，它会设置一个`text`来显示温度和一个根据温度匹配的不同的颜色：低温度用红色，中温度用橙色，其它用绿色。温度值是比较随机的，但是这个是使用`when`表达式让代码变得简短精炼的很好的代表。

`color`是我想念的Anko中的一个扩展函数，它可以很简洁的方式从resources中获取一个color，类似于我们在其它地方使用到的`dimen`。我们写下这一行的时候，当前`support library`依赖`ContextCompat`来从不同的Android版本中获取一个color：

```kotlin
public fun Context.color(res: Int): Int = ContextCompat.getColor(this, res)
```

`AndroidManifest`也需要知道新activity的存在：

```kotlin
<activity
    android:name=".ui.activities.DetailActivity"
    android:parentActivityName=".ui.activities.MainActivity" >
    <meta-data
        android:name="android.support.PARENT_ACTIVITY"
        android:value="com.antonioleiva.weatherapp.ui.activities.MainActivity" />
</activity>
```

# 启动一个activity：reified函数

最后一步是从`main activity`启动一个`detail activity`。我们可以如下重写adapter实例：

```kotlin
val adapter = ForecastListAdapter(result) {
	val intent = Intent(MainActivity@this, javaClass<DetailActivity>())
	intent.putExtra(DetailActivity.ID, it.id)
	intent.putExtra(DetailActivity.CITY_NAME, result.city)
	startActivity(intent)
}
```

但是这是非常冗长的。一如既往地，Anko提供了简单得多的方式通过`reified function`来启动一个activity：

```kotlin
val adapter = ForecastListAdapter(result) {
    startActivity<DetailActivity>(DetailActivity.ID to it.id,
            DetailActivity.CITY_NAME to result.city)
}
```

`reified function`背后到底有什么魔法呢？就像你可能知道的那样，当我们在Java中创建一个范型函数，我们没有办法得到范型类型的Class。一个流行的变通方法是作为参数传入一个Class。在Kotlin中，一个内联（`inline`）函数可以被具体化（`reified`），这意味着我们可以在函数中得到并使用范型类型的Class。Anko真正使用它的一个简单的例子接下来会讲到（在这个例子中我只使用了`String`）：

```kotlin
public inline fun <reified T: Activity> Context.startActivity(
        vararg params: Pair<String, String>) {
    val intent = Intent(this, T::class.javaClass)
    params forEach { intent.putExtra(it.first, it.second) }
    startActivity(intent)
}
```

真正的实现要更加复杂一点因为它使用了一个很长的令人讨厌的`when`表达式来增加由类型决定的额外信息，但是在概念上来说它没有增加其它更有用的知识。

`Reified`函数是有一个可以简化代码和提高理解性的语法糖。在这个例子中，它通过获取到了范型类型的`javaClass`来创建了一个intent，迭代所有参数并增加到intent，然后使用Intent来启动activity。`reified`限制于activity的子类。

剩下的一点细节在代码库中已经说明。我们现在有一个非常简单（但是完整）的主从视图（`master-detail`）的App，它使用Kotlin实现，没有使用一行Java代码。




# 在我们的App中实现一个例子

接口可以被用来从类中提取出相似行为的通用代码。比如，我们可以创建一个接口用于处理app的toolbar。`MainActivity`和`DetailActivity`在处理`toolbar`时会共享这些相似的代码。

但是受限，我们需要做出一些改变，使用被定义在布局中`toolbar`，而不是标准的`ActionBar`。第一件事是继承`NoActionBar`主题。这样`toolbar`不会自动被包含进来：

```kotlin
<style name="AppTheme" parent="Theme.AppCompat.Light.NoActionBar">
	<item name="colorPrimary">#ff212121</item>
	<item name="colorPrimaryDark">@android:color/black</item>
</style>
```

我们使用`light`主题。然后我们创建一个`toolbar`的布局，我们稍后会在其它的布局中使用到它：

```kotlin
<android.support.v7.widget.Toolbar
	xmlns:app="http://schemas.android.com/apk/res-auto"
	xmlns:android="http://schemas.android.com/apk/res/android"
	android:id="@+id/toolbar"
	android:layout_width="match_parent"
	android:layout_height="?attr/actionBarSize"
	android:background="?attr/colorPrimary"
	app:theme="@style/ThemeOverlay.AppCompat.Dark.ActionBar"
	app:popupTheme="@style/ThemeOverlay.AppCompat.Light"/>
```

`toolbar`指定了它自己的背景，一个针对自己的`dark`主题和一个针对生成的弹出框的`light`主题（`overflow menu`实例）。我们现在已经有了相同的主题：`light`主题和`dark`主题的`Action Bar`。

下一步我们将修改`MainActivity`的布局，增加一个toolbar：

```kotlin
<FrameLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	android:layout_width="match_parent"
	android:layout_height="match_parent">

	<android.support.v7.widget.RecyclerView
		android:id="@+id/forecastList"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		android:clipToPadding="false"
		android:paddingTop="?attr/actionBarSize"/>
	
	<include layout="@layout/toolbar"/>
</FrameLayout>
```

现在toolbar被增加到布局中，我们可以开始使用它。我们创建了一个接口，它可以让我们：

- 改变title
- 指定是否显示上一步的导航动作
- 滚动时的toolbar动画
- 给所有的activity设置相同的菜单，甚至行为

然后让我们定义`ToolbarManager`：

```kotlin
interface ToolbarManager {
    val toolbar: Toolbar
    ...
}
```

它将需要一个toolbar属性。接口是无状态的，所以属性可以被定义，但是不能赋值。子类会实现这个接口并重写这个属性。

另一方面，我们可以不使用重写来实现无状态的属性。也就是说属性不需要维护一个`backup field`。一个处理toolbar title属性的例子：

```kotlin
var toolbarTitle: String
    get() = toolbar.title.toString()
    set(value) {
        toolbar.title = value
    }
```

因为属性仅仅使用了toolbar，它不需要保存任何新的状态。

我们现在创建了一个新的函数用来初始化toolbar，inflate一个menu并且设置一个listener：

```kotlin
fun initToolbar(){
	toolbar.inflateMenu(R.menu.menu_main)
    toolbar.setOnMenuItemClickListener {
		when (it.itemId) {
		    R.id.action_settings -> App.instance.toast("Settings")
		    else -> App.instance.toast("Unknown option")
		}
		true
	}
}
```

我们可以增加一个函数用来开启toolbar上面导航icon，设置一个箭头的icon并设置一个当icon被按压时触发的事件：

```kotlin
fun enableHomeAsUp(up: () -> Unit) {
       toolbar.navigationIcon = createUpDrawable()
       toolbar.setNavigationOnClickListener { up() }
}

private fun createUpDrawable() = with (DrawerArrowDrawable(toolbar.ctx)){
    progress = 1f
	this
}
```

这个函数接收一个listener，使用[DrawerArrowDrawable]来创建一个最后状态（当箭头已经显示时）的drawable，然后把listener设置给toolbar。

最后，接口将会提供一个函数，它允许toolbar可以attached到一个scroll上面，并且根据scroll的方向来执行动画。当往下滚动时toolbar会消失 ，往上滚动toolbar会再次显示：

```kotlin
fun attachToScroll(recyclerView: RecyclerView) {
    recyclerView.addOnScrollListener(object : RecyclerView.OnScrollListener() {
		override fun onScrolled(recyclerView: RecyclerView?, dx: Int, dy: Int) {
			if (dy > 0) toolbar.slideExit() else toolbar.slideEnter()
	    }
    })
}
```

我们会创建两个用于view从屏幕中显示或者消失动画的扩展函数。我们会检查是否动画之前没有执行过。这种方式可以避免每次不同的滚动view都会执行动画：

```kotlin
fun View.slideExit() {
	if (translationY == 0f) animate().translationY(-height.toFlat())
}

fun View.slideEnter() {
	if (translationY < 0f) animate().translationY(0f)
}
```

在`toobar manager`实现之后，是时候在`MainActivity`中使用它了。我们首先指定toolbar属性。我们可以使用`lazy`委托实现，这样会在我们第一次使用它的时候才会inflate：

```kotlin
override val toolbar by lazy { find<Toolbar>(R.id.toolbar) }
```

`MainActivity`将会仅仅初始化toolbar并attach到`RecyclerView`的滚动并修改toolbar的title：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) { 
super.onCreate(savedInstanceState) 
setContentView(R.layout.activity_main) initToolbar()
    forecastList.layoutManager = LinearLayoutManager(this)
    attachToScroll(forecastList)
    async {
        val result = RequestForecastCommand(94043).execute()
        uiThread {
			val adapter = ForecastListAdapter(result) {
		    startActivity<DetailActivity>(DetailActivity.ID to it.id,
			            DetailActivity.CITY_NAME to result.city)
			}
			forecastList.adapter = adapter
			toolbarTitle = "${result.city} (${result.country})"
		} 
	}
}
```

`DetailActivity`也需要一些布局上的修改：

```kotlin
<LinearLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical">
    
    <include layout="@layout/toolbar"/>
    
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:gravity="center_vertical"
		android:paddingTop="@dimen/activity_vertical_margin"
        android:paddingLeft="@dimen/activity_horizontal_margin"
        android:paddingRight="@dimen/activity_horizontal_margin"
        tools:ignore="UseCompoundDrawables">
       ....
    </LinearLayout>
    
</LinearLayout>
```

使用相同的方式去指定toolbar属性。`DetailActivity`也会初始化toolbar，设置title并且开启导航返回icon：

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.activity_detail)
    
	initToolbar()
	toolbarTitle = intent.getStringExtra(CITY_NAME) 
	enableHomeAsUp { onBackPressed() }
	...
}
```

接口可以帮助我们从类中提取出公共的代码来共享相似的行为。可以作为让我们代码精炼合理简洁可复用的替代方案。思考哪方面接口可以帮助你写出更好的代码。

[DrawerArrowDrawable]: https://developer.android.com/reference/android/support/v7/graphics/drawable/DrawerArrowDrawable.html


# 创建一个设置activity

当toolbar上溢出菜单（`overflow menu`）的`settings`选项被点击时，需要打开一个新的Activity。所以首先要做的事情时需要一个新的`SettingActivity`：

```kotlin
class SettingsActivity : AppCompatActivity() {
	
	override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_settings)
        setSupportActionBar(toolbar)
        supportActionBar.setDisplayHomeAsUpEnabled(true)
    }

	override fun onOptionsItemSelected(item: MenuItem) = when (item.itemId) {
        android.R.id.home -> { onBackPressed(); true }
        else -> false
	}
}
```

当用户离开这个界面的时我们需要保存用户`preference`（偏好），所以我们需要像处理`Back`一样处理`Up`动作，重定向动作到`onBackPressed`。现在，让我们要创建一个XML布局。对于这个`preference`来说一个简单`EditText`就足够了：

```xml
<FrameLayout
	xmlns:android="http://schemas.android.com/apk/res/android"
	android:layout_width="match_parent"
	android:layout_height="match_parent">

	<include layout="@layout/toolbar"/>
	<LinearLayout
		android:orientation="vertical"
		android:layout_width="match_parent"
		android:layout_height="match_parent"
		android:layout_marginTop="?attr/actionBarSize"
		android:padding="@dimen/spacing_xlarge">
	
		<TextView
			android:layout_width="wrap_content"
			android:layout_height="wrap_content"
			android:text="@string/city_zipcode"/>
		
		<EditText
			android:id="@+id/cityCode"
			android:layout_width="match_parent"
			android:layout_height="wrap_content"
			android:hint="@string/city_zipcode"
			android:inputType="number"/>

	</LinearLayout>
</FrameLayout>
```

然后只需要在`AndroidManifest.xml`中声明这个activity：

```xml
<activity
android:name=".ui.activities.SettingsActivity"
android:label="@string/settings"/>
```


# 测试

# Unit testing

我不会对`unit testing`（单元测试）是什么的话题展开讨论。存在很多定义，但是都有一些细微的不同。一个普通的观点可能是`unit testing`验证一个单位（`unit`）的源代码的测试。一个单位（`unit`）包含什么就留给读者了。在我们的例子中，我仅仅去定义了一个`unit test`作为一个不需要设备运行的测试。IDE将会运行这些测试然后显示最后的结果分辩哪些测试成功哪些测试失败了。

`Unit testing`通常使用`JUnit`库。所以让我们增加这个依赖到`build.gradle`。因为这个依赖只会在跑测试的时候才会用到，所以我们可以使用`testCompile`而不是`compile`。用这种方式，这个库会在正式编译时忽略掉，可以减少APK的大小：

```groovy
dependencies {
	...
	testCompile 'junit:junit:4.12'
}
```

现在同步gradle来获取该库并加入到你的项目中。为了开启`unit testing`，打开`Build Variants`tab（你可能可以在IDE的左边找到它），点击`Test Artifact`下拉，你应该选择`Unit Tests`。

另一件你需要做的事情是创建一个新的文件夹。在src下面，你可能已经有`androidTest`和`main`了。创建另一个名为`test`的文件夹，再在它下面创建一个`java`文件夹。所以现在你应该有一个名为`src/test/java`绿色的文件夹。这是IDE发现我们在使用`Unit Test`模式好的迹象，这个文件夹中将会包括一些测试文件。

我们来写一个非常简单的测试来看看一切是不是正常运行了。使用合适的包名（我的是`com.antonioleiva.weatherapp`，但是你需要使用你app中的主包名）创建一个新的名为`SimpleTest`的Kotlin类。当你创建完，编写如下简单的测试：

```kotlin
import org.junit.Test
import kotlin.test.assertTrue
class SimpleTest {
	@Test fun unitTestingWorks() {
	    assertTrue(true)
	}
}
```

使用`@Test`注解来辨别该函数为是一个测试。确认是`org.unit.Test`。然后增加一个简单的断言。它只是判断了true是否是true，它显然会成功。这个测试只是用开确认一切配置正确。

执行测试，只需要在你在`test`下创建的新的`java`文件夹上右击，然后选择`Run All Tests`。当编译完成后，它会运行测试并会看见结果简介的显示。你应该可以看见我们的测试通过了。

现在是时候创建一个真正的测试了。所有使用Android框架来处理的测试可能都需要一个`instrumentation test`或者使用更复杂的像[Robolectric]库。所以在这些例子中我会不使用框架的任何东西。举个例子，我将测试从`Long`转`String`的扩展函数。

创建一个新的名为`ExtensionTests`的文件，然后增加如下测试：

```kotlin
class ExtensionsTest {
	
	@Test fun testLongToDateString() {
        assertEquals("Oct 19, 2015", 1445275635000L.toDateString())
    }

	@Test fun testDateStringFullFormat() {
        assertEquals("Monday, October 19, 2015",
            1445275635000L.toDateString(DateFormat.FULL))
    }
}
```

这些测试检测`Long`实例是否可以转换成一个`String`。第一个测试默认行为（使用DateFormat.MEDIUM)），而第二个指定一个不同的格式。运行这些测试然后你会看到它们都通过了。我建议你修改它们然后看看它们失败是怎么样的。

如果你在Java中使用过测试，你将会发现这并没有什么太多的不同。我会演示一个简单的例子，我们可以对`ForecastProvider`进行一些测试。我们可以使用`Mockito`库来模拟其它的类然后独立测试provider：

```groovy
dependencies {
    ...
    testCompile "junit:junit:4.12"
    testCompile "org.mockito:mockito-core:1.10.19"
}
```

现在创建了一个`ForecastProviderTest`。我们要去测试`ForecastProvider`，使用`DataSource`来返回结果，看它结果是否为null。所以首先我们需要模拟一个`ForecastDataSource`：

```kotlin
val ds = mock(ForecastDataSource::class.java)
`when`(ds.requestDayForecast(0)).then {
    Forecast(0, 0, "desc", 20, 0, "url")
}
```

如你所见，我们需要在`when`上加反引号。因为`when`在Kotlin中是一个保留关键字，所以如果我们在一些Java代码中使用到它我们需要避免它。现在我们用这个数据源创建了一个provider，然后检测调用那个方法之后的结果是否为null：

```kotlin
val provider = ForecastProvider(listOf(ds))
assertNotNull(provider.requestForecast(0))
```

这是完整的测试函数：

```kotlin
@Test fun testDataSourceReturnsValue() {
    val ds = mock(ForecastDataSource::class.java)
    `when`(ds.requestDayForecast(0)).then {
           Forecast(0, 0, "desc", 20, 0, "url")
    }
    
	val provider = ForecastProvider(listOf(ds))
	assertNotNull(provider.requestForecast(0))
}
```

如果你运行它，你将会看见它会出错。多亏这个测试，我们在自己的代码中发现了某些错误。测试失败是因为`ForecastProvider`在使用之前正在它的`companion object`中初始化。我们可以通过构造函数的方式在`ForecastProvider`中增加一些数据源，这个静态的List就永远不会被使用，所以它应该是使用`lazy`加载：

```kotlin
companion object {
	val DAY_IN_MILLIS = 1000 * 60 * 60 * 24
    val SOURCES by lazy { listOf(ForecastDb(), ForecastServer()) }
}
```

如果你现在再次去运行，你会发现现在会通过所有的测试。
我们也可以测试一些比如当数据源返回null的时候，它会便利下一个数据源来得到结果：

```kotlin
@Test fun emptyDatabaseReturnsServerValue() {
    val db = mock(ForecastDataSource::class.java)
    val server = mock(ForecastDataSource::class.java)
    `when`(server.requestForecastByZipCode(
            any(Long::class.java), any(Long::class.java)))
            .then {
                ForecastList(0, "city", "country", listOf())
    val provider = ForecastProvider(listOf(db, server))
    assertNotNull(provider.requestByZipCode(0, 0))
}
```

如你所见，通过使用参数的默认值这种简单的依赖倒置足够让我们实现一些简单的`unit tests`。对于这个provider还有很多我们可以测试的东西，但是这个例子足够让我们学会使用`unit testing`工具了。

[Robolectric]: http://robolectric.org/

# Instrumentation tests

`Instrumentation tests`有一点不同。它们通常被使用在UI交互上，我们需要一个应用程序实例跑的同时执行测试。达到这个，我们就需要在设备上部署并运行。

这类的测试必须要放在`androidTest`文件夹中，我们必须要修改`Build Variants`区域的`Test Artifact`为`Android Instrumentation Tests`。实现instrumentation的官方库是[Espresso]，它通过`Actions`、`filter`以及检测结果的`ViewMatchers`和`Matchers`可以帮助我们更简单地使用。

配置比之前更加难一点。我们需要下载额外的库和`Gradle`的配置。好事是Kotlin的测试不需要添加额外的东西，所以如果你已经知道怎么去配置`Espresso`，它将对你来说是很简单的。

首先，在`defaultConfig`中指定`test runner`：

```groovy
defaultConfig {
    ...
    testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
}
```

当你处理完该runner，然后增加其它的依赖，这次是用`androidTestCompile`。这种方式，这些库只会再编译运行`instrumentation tests`的时候才被增加：

```groovy
dependencies {
    ...
    androidTestCompile "com.android.support:support-annotations:$support_version"
    androidTestCompile "com.android.support.test:runner:0.4.1"
    androidTestCompile "com.android.support.test:rules:0.4.1"
    androidTestCompile "com.android.support.test.espresso:espresso-core:2.2.1"
    androidTestCompile ("com.android.support.test.espresso:espresso-contrib:2.2.1"){
	    exclude group: 'com.android.support', module: 'appcompat'
		exclude group: 'com.android.support', module: 'support-v4'
		exclude module: 'recyclerview-v7'
}
```

我不想花大量的时间去讲这些，但是这里有为什么需要这些库的简短原因：

- `support-annotations`：其它库中需要使用到。
- `runner`：这是`test runner`，就是我们再`defaultConfig`中指定的那个。
- `rules`：包括一些测试inflate启动activity的规则。我们将会在我们的例子中使用这些规则。
- `espresso-core`：`Espresso`的基本实现，它让`instrument tests`更加容易。
- `espresso-contrib`：它增加了其它额外的功能，比如支持`RecyclerView`测试。我们不得不排除掉一些它的依赖，因为我们已经在这个项目中使用到了，否则测试会出错。

我们现在来创建一个简单的例子。测试将会点击forecast列表的第一行，然后它会判断是否能找到一个id为`R.id.weatherDescription`的view。这个view是在`DetailActivity`中的，这表示我们在测试在RecyclerView里面点击后是否可以成功地导航到详情页面。

```kotlin
class SimpleInstrumentationTest {

	@get:Rule
    val activityRule = ActivityTestRule(MainActivity::class.java)
    
    ...
}
```

首先我们需要指定它运行时使用`AndroidJUnit4`。然后，创建一个activity规则，它会实例化一个测试需要的activity。在Java中，你可以使用`@Rule`。但是如你所知，字段和属性是不一样的，所以如果你像那样去使用的话，执行会失败因为访问属性中的字段是不是public的。你需要加注解的是在getter上面。Kotlin允许指定`get`或者`set`在Rule的名字前面。在这个例子中，只要些`@get:Rule`。

之后，我们已经准备好创建第一个测试了：

```kotlin
@Test fun itemClick_navigatesToDetail() {
    onView(withId(R.id.forecastList)).perform(
            RecyclerViewActions
                .actionOnItemAtPosition<RecyclerView.ViewHolder>(0, click()))
    onView(withId(R.id.weatherDescription))
           .check(matches(isAssignableFrom(TextView::class.java)))
}
```

函数加上了`@Test`注解，这根我们使用`unit test`的方式一样。我们可以开始在测试体中使用`Espresso`。它首先在`RecyclerView`的第一个position中执行了一个点击。然后它检测是否可以找到一个指定id的view且这个view是一个`TextView`。

要运行这个测试，点击顶部的`Run configurations`下拉选择`Edit Configurations...`按下`+`图标，选择`Android Tests`，然后选择`app`模块。现在，在`target device`中选择你喜欢的`target`。点击`OK`然后运行。你应该可以看到App是怎样在你的设备中开始的，它会测试第一个position，打开详情页面然后再次关闭app。

现在我们要做一个更加复杂一点的事情。测试会从toolbar众打开一个溢出菜单，点击`settings`栏，改变城市的`code`，然后检测toolbar的标题是否改变成了对应的标题。

```kotlin
@Test fun modifyZipCode_changesToolbarTitle() {
	openActionBarOverflowOrOptionsMenu(activityRule.activity)
	onView(withText(R.string.settings)).perform(click())
	onView(withId(R.id.cityCode)).perform(replaceText("28830"))
	pressBack()
	onView(isAssignableFrom(Toolbar::class.java))
	        .check(matches(
	            withToolbarTitle(`is`("San Fernando de Henares (ES)"))))
}
```

这个测试实际做的事情：

- 它首先使用`openActionBarOverflowOrOptionsMenu`打开溢出菜单。
- 然后它根据`Settings`文本查找一个view，然后点击这个它。
- 之后，设置界面就会被打开，所以它会查找一个`EditText`并且替换成一个新的`code`。
- 它会点击返回按钮。它会把新的值保存在preferences中，然后关闭Activity。
- 因为`MainActivity`的`onResume`会调用，请求会再调用一次。这时它会获取到新城市的forecast。
- 最后一行将会检测Toolbar我们看到的title是否是新的城市的title。

这不是一个toolbar的title的默认匹配器，但是`Espresso`是很容易扩展的，所以我们可以创建一个新的matcher来实现检测：

```kotlin
private fun withToolbarTitle(textMatcher: Matcher<CharSequence>): Matcher<Any> =
        object : BoundedMatcher<Any, Toolbar>(Toolbar::class.java) {

	override fun matchesSafely(toolbar: Toolbar): Boolean {
		return textMatcher.matches(toolbar.title)
	}

	override fun describeTo(description: Description) {
		description.appendText("with toolbar title: ")
		textMatcher.describeTo(description)
	}                
}
```

`matchSafely`函数是我们检测的地方，而`describeTo`函数为matcher增加了一些新的信息。

这章特别有趣，因为我们看到了在Kotlin中怎么样去完美和谐地整合测试，它们可以没有任何问题地整合测试。查看代码然后你自己运行一下吧。


[Espresso]: https://google.github.io/android-testing-support-library/


[Bruce Eckel]: http://www.mindview.net/Etc/Discussions/CheckedExceptions
[Rod Waldhoff]: http://radio-weblogs.com/0122027/stories/2003/04/01/JavasCheckedExceptionsWereAMistake.html
[Anders Hejlsberg]: http://www.artima.com/intv/handcuffs.html

