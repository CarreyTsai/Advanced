# 基础语法

##  定义包
包的声明应 处于源文件顶部
```kotlin
package my.demo

import java.util.***
```
目录与包的结构无需匹配：源代码可以在文件系统的任意位置。

源文件通常以包声明开头。
默认会导入很多包到kotlin文件中
- kotlin.*
- kotlin.annotation.*
- kotlin.collections.*
- kotlin.comparisons.* （自 1.1 起）
- kotlin.io.*
- kotlin.ranges.*
- kotlin.sequences.*
- kotlin.text.*


除了默认导入之外，可以导入单独的名称。如果名字冲突，可以使用 __as__ 关键字在本地重命名冲突项来消除歧义。

```kotlin
import foot.Bar
import bar.Bar as bBar
``` 
## 定义函数

- 函数声明
```kotlin
fun double(x;Int):Int{
    return 2*x
}
```
- 调用方法
```kotlin
val result = double2()
```
- 中缀表示法

函数还可以用中缀表示法调用，当
- 他们是成员函数或则扩展函数
- 他们只有一个参数
- 她们用infix 关键字标注
```kotlin
//给Int定义扩展
infix fun Int.shl(x:Int){
    ......
}
// 用中缀表示法调用扩展函数
 1 shl 2
 //等同于
 1.shl(2)
```
- 参数

函数参数使用 Pascal 表示法定义，即 name: type。参数用逗号隔开。每个参数必须有显式类型。

```kotlin
    fun powerOf(number:Int,exponent:Int){
        ...
    }
    //函数参数可以有默认值，当省略相应的参数时使用默认值。与其他语言相比，这可以减少重载数量
    fun read(b: Array<Byte>, off: Int = 0, len: Int = b.size()) {
    ……
    }

```
默认值通过类型后面的 = 及给出的值来定义。

覆盖方法总是使用与基类型方法相同的默认参数值。 当覆盖一个带有默认参数值的方法时，必须从签名中省略默认参数值：

- 命名参数

可以在调用函数时使用命名的函数参数。当一个函数有大量的参数或默认参数时这会非常方便。

给定以下函数
```kotlin
fun reformat(str: String,
             normalizeCase: Boolean = true,
             upperCaseFirstLetter: Boolean = true,
             divideByCamelHumps: Boolean = false,
             wordSeparator: Char = ' ') {
        ……
        }
```
我们可以使用默认参数来调用它
```kotlin
reformat(str)
```
然而，当使用非默认参数调用它时，该调用看起来就像
```kotlin
reformat(str, true, true, false, '_')
```
使用命名参数我们可以使代码更具有可读性

请注意，在调用 Java 函数时不能使用命名参数语法，因为 Java 字节码并不总是保留函数参数的名称。

- 返回 Unit 的函数

如果一个函数不返回任何有用的值，它的返回类型是 Unit。Unit 是一种只有一个值——Unit 的类型。这个值不需要显式返回
```kotlin
fun printHello(name: String?): Unit {
    if (name != null)
        println("Hello ${name}")
    else
        println("Hi there!")
    // `return Unit` 或者 `return` 是可选的
}
//Unit 返回类型声明也是可选的。上面的代码等同于

fun printHello(name: String?) {
    ……
}
```
- 单表达式函数

当函数返回单个表达式时，可以省略花括号并且在 = 符号之后指定代码体即可
```kotlin
fun double(x: Int): Int = x * 2
//当返回值类型可由编译器推断时，显式声明返回类型是可选的

fun double(x: Int) = x * 2
```
- 显式返回类型

具有块代码体的函数必须始终显式指定返回类型，除非他们旨在返回 Unit，在这种情况下它是可选的。 Kotlin 不推断具有块代码体的函数的返回类型，因为这样的函数在代码体中可能有复杂的控制流，并且返回类型对于读者（有时甚至对于编译器）是不明显的。

- 可变数量的参数（Varargs）

函数的参数（通常是最后一个）可以用 vararg 修饰符标记：
```kotlin
fun <T> asList(vararg ts: T): List<T> {
    val result = ArrayList<T>()
    for (t in ts) // ts is an Array
        result.add(t)
    return result
}
//允许将可变数量的参数传递给函数：

val list = asList(1, 2, 3)
```
在函数内部，类型 T 的 vararg 参数的可见方式是作为 T 数组，即上例中的 ts 变量具有类型 Array <out T>。

只有一个参数可以标注为 vararg。如果 vararg 参数不是列表中的最后一个参数， 可以使用命名参数语法传递其后的参数的值，或者，如果参数具有函数类型，则通过在括号外部传一个 lambda。

当我们调用 vararg-函数时，我们可以一个接一个地传参，例如 asList(1, 2, 3)，或者，如果我们已经有一个数组并希望将其内容传给该函数，我们使用伸展（spread）操作符（在数组前面加 *）：
```kotlin
val a = arrayOf(1, 2, 3)
val list = asList(-1, 0, *a, 4)
```
- 函数作用域

在 Kotlin 中函数可以在文件顶层声明，这意味着你不需要像一些语言如 Java、C# 或 Scala 那样创建一个类来保存一个函数。此外除了顶层函数，Kotlin 中函数也可以声明在局部作用域、作为成员函数以及扩展函数。

- 局部函数

Kotlin 支持局部函数，即一个函数在另一个函数内部
```kotlin
fun dfs(graph: Graph) {
    fun dfs(current: Vertex, visited: Set<Vertex>) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v, visited)
    }

    dfs(graph.vertices[0], HashSet())
}
```

局部函数可以访问外部函数（即闭包）的局部变量，所以在上例中，visited 可以是局部变量。
```kotlin
fun dfs(graph: Graph) {
    val visited = HashSet<Vertex>()
    fun dfs(current: Vertex) {
        if (!visited.add(current)) return
        for (v in current.neighbors)
            dfs(v)
    }

    dfs(graph.vertices[0])
}
```
- 成员函数

成员函数是在类或对象内部定义的函数
```kotlin
class Sample() {
    fun foo() { print("Foo") }
}

//成员函数以点表示法调用

Sample().foo() // 创建类 Sample 实例并调用 foo
```

- 泛型函数

函数可以有泛型参数，通过在函数名前使用尖括号指定。
```kotlin
fun <T> singletonList(item: T): List<T> {
    // ……
}
```

- 内联函数

使用高阶函数会带来一些运行时的效率损失：每一个函数都是一个对象，并且会捕获一个闭包。 即那些在函数体内会访问到的变量。 内存分配（对于函数对象和类）和虚拟调用会引入运行时间开销。

但是在许多情况下通过内联化 lambda 表达式可以消除这类的开销。 下述函数是这种情况的很好的例子。即 lock() 函数可以很容易地在调用处内联。 考虑下面的情况：
```kotlin
lock(l) { foo() }
```
编译器没有为参数创建一个函数对象并生成一个调用。取而代之，编译器可以生成以下代码：
```kotlin
l.lock()
try {
    foo()
}
finally {
    l.unlock()
}

```
为了让编译器这么做，我们需要使用 inline 修饰符标记 lock() 函数：

```kotlin
inline fun lock<T>(lock: Lock, body: () -> T): T {
    // ……
}
```
inline 修饰符影响函数本身和传给它的 lambda 表达式：所有这些都将内联到调用处。

内联可能导致生成的代码增加，但是如果我们使用得当（不内联大函数），它将在性能上有所提升，尤其是在循环中的“超多态（megamorphic）”调用处。

- 禁用内联

如果你只想被（作为参数）传给一个内联函数的 lamda 表达式中只有一些被内联，你可以用 noinline 修饰符标记一些函数参数：
```kotlin
inline fun foo(inlined: () -> Unit, noinline notInlined: () -> Unit) {
    // ……
}
```
可以内联的 lambda 表达式只能在内联函数内部调用或者作为可内联的参数传递， 但是 noinline 的可以以任何我们喜欢的方式操作：存储在字段中、传送它等等。

需要注意的是，如果一个内联函数没有可内联的函数参数并且没有具体化的类型参数，编译器会产生一个警告，因为内联这样的函数很可能并无益处（如果你确认需要内联，则可以关掉该警告）。

- 非局部返回

在 Kotlin 中，我们可以只使用一个正常的、非限定的 return 来退出一个命名或匿名函数。 这意味着要退出一个 lambda 表达式，我们必须使用一个标签，并且在 lambda 表达式内部禁止使用裸 return，因为 lambda 表达式不能使包含它的函数返回：
```kotlin
fun foo() {
    ordinaryFunction {
        return // 错误：不能使 `foo` 在此处返回
    }
}
//但是如果 lambda 表达式传给的函数是内联的，该 return 也可以内联，所以它是允许的：

fun foo() {
    inlineFunction {
        return // OK：该 lambda 表达式是内联的
    }
}
```
这种返回（位于 lambda 表达式中，但退出包含它的函数）称为非局部返回。 我们习惯了在循环中用这种结构，其内联函数通常包含：
```kotlin
fun hasZeros(ints: List<Int>): Boolean {
    ints.forEach {
        if (it == 0) return true // 从 hasZeros 返回
    }
    return false
}
```
请注意，一些内联函数可能调用传给它们的不是直接来自函数体、而是来自另一个执行上下文的 lambda 表达式参数，例如来自局部对象或嵌套函数。在这种情况下，该 lambda 表达式中也不允许非局部控制流。为了标识这种情况，该 lambda 表达式参数需要用 crossinline 修饰符标记:
```kotlin
inline fun f(crossinline body: () -> Unit) {
    val f = object: Runnable {
        override fun run() = body()
    }
    // ……
}
```
break 和 continue 在内联的 lambda 表达式中还不可用，但我们也计划支持它们

- 具体化的类型参数

有时候我们需要访问一个作为参数传给我们的一个类型：
```kotlin
fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
    var p = parent
    while (p != null && !clazz.isInstance(p)) {
        p = p.parent
    }
    @Suppress("UNCHECKED_CAST")
    return p as T?
}
```
在这里我们向上遍历一棵树并且检查每个节点是不是特定的类型。 这都没有问题，但是调用处不是很优雅：
```kotlin
treeNode.findParentOfType(MyTreeNode::class.java)
```
我们真正想要的只是传一个类型给该函数，即像这样调用它：
```kotlin
treeNode.findParentOfType<MyTreeNode>()
```
为能够这么做，内联函数支持具体化的类型参数，于是我们可以这样写：
```kotlin
inline fun <reified T> TreeNode.findParentOfType(): T? {
    var p = parent
    while (p != null && p !is T) {
        p = p.parent
    }
    return p as T?
}
```
我们使用 reified 修饰符来限定类型参数，现在可以在函数内部访问它了， 几乎就像是一个普通的类一样。由于函数是内联的，不需要反射，正常的操作符如 !is 和 as 现在都能用了。此外，我们还可以按照上面提到的方式调用它：myTree.findParentOfType<MyTreeNodeType>()。

虽然在许多情况下可能不需要反射，但我们仍然可以对一个具体化的类型参数使用它：
```kotlin
inline fun <reified T> membersOf() = T::class.members

fun main(s: Array<String>) {
    println(membersOf<StringBuilder>().joinToString("\n"))
}
```
普通的函数（未标记为内联函数的）不能有具体化参数。 不具有运行时表示的类型（例如非具体化的类型参数或者类似于Nothing的虚构类型） 不能用作具体化的类型参数的实参。
- 内联属性（自 1.1 起）

inline 修饰符可用于没有幕后字段的属性的访问器。 你可以标注独立的属性访问器：
```kotlin
val foo: Foo
    inline get() = Foo()

var bar: Bar
    get() = ……
    inline set(v) { …… }
//你也可以标注整个属性，将它的两个访问器都标记为内联：

inline var bar: Bar
    get() = ……
    set(v) { …… }
//在调用处，内联访问器如同内联函数一样内联。
```
- 扩展函数

扩展函数在其自有章节讲述

- 高阶函数和 Lambda 表达式

高阶函数和 Lambda 表达式在其自有章节讲述

- 尾递归函数

Kotlin 支持一种称为尾递归的函数式编程风格。 这允许一些通常用循环写的算法改用递归函数来写，而无堆栈溢出的风险。 当一个函数用 tailrec 修饰符标记并满足所需的形式时，编译器会优化该递归，留下一个快速而高效的基于循环的版本。
```kotlin
tailrec fun findFixPoint(x: Double = 1.0): Double
        = if (x == Math.cos(x)) x else findFixPoint(Math.cos(x))
```
这段代码计算余弦的不动点（fixpoint of cosine），这是一个数学常数。 它只是重复地从 1.0 开始调用 Math.cos，直到结果不再改变，产生0.7390851332151607的结果。最终代码相当于这种更传统风格的代码：
```kotlin
private fun findFixPoint(): Double {
    var x = 1.0
    while (true) {
        val y = Math.cos(x)
        if (x == y) return y
        x = y
    }
}
```
要符合 tailrec 修饰符的条件的话，函数必须将其自身调用作为它执行的最后一个操作。在递归调用后有更多代码时，不能使用尾递归，并且不能用在 try/catch/finally 块中。目前尾部递归只在 JVM 后端中支持。

