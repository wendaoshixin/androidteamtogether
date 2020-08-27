# Kotlin开发规范与注意事项

# 规范

### 1.目录结构

> 如果 Kotlin 文件包含单个类（以及可能相关的顶层声明），那么文件名应该与该类的名称相同，并追加 .kt 扩展名。如果文件包含多个类或只包含顶层声明， 那么选择一个描述该文件所包含内容的名称，并以此命名该文件。 使用首字母大写的驼峰风格（例如 ProcessDeclarations.kt）。
> 文件的名称应该描述文件中代码的作用。因此，应避免在文件名中使用诸如“Util”之类的无意义词语。

### 2.源文件组织

> 鼓励多个声明（类、顶级函数或者属性）放在同一个 Kotlin 源文件中， 只要这些声明在语义上彼此紧密关联并且文件保持合理大小 （不超过几百行）。
> 特别是在为类定义与类的所有客户都相关的扩展函数时， 请将它们放在与类自身定义相同的地方。而在定义仅对指定客户有意义的扩展函数时，请将它们放在紧挨该客户代码之后。不要只是为了保存 “Foo 的所有扩展函数”而创建文件。

### 3.类布局

> 通常，一个类的内容按以下顺序排列:
>
> * 属性声明与初始化块
> * 次构造函数
> * 方法声明
> * 伴生对象
>
> 不要按字母顺序或者可见性对方法声明排序，也不要将常规方法与扩展方法分开。而是要把相关的东西放在一起，这样从上到下阅读类的人就能够跟进所发生事情的逻辑。选择一个顺序（高级别优先，或者相反）并坚持下去。
> 将嵌套类放在紧挨使用这些类的代码之后。如果打算在外部使用嵌套类，而且类中并没有引用这些类，那么把它们放到末尾，在伴生对象之后。

### 4.接口实现布局

> 在实现一个接口时，实现成员的顺序应该与该接口的成员顺序相同（如果需要， 还要插入用于实现的额外的私有方法）

### 5.重载布局

> 在类中总是将重载放在一起。

## 6.命名规则

> 在 Kotlin 中，包名与类名的命名规则非常简单:
>
> * 包的名称总是小写且不使用下划线（org.example.project）。 通常不鼓励使用多个词的名称，但是如果确实需要使用多个词，可以将它们连接在一起或使用驼峰风格（org.example.myProject）。
> * 类与对象的名称以大写字母开头并使用驼峰风格：

```kotlin
open class DeclarationProcessor { /*……*/ }

object EmptyDeclarationProcessor : DeclarationProcessor() { /*……*/ }
```

### 7.函数名

**函数、属性与局部变量的名称以小写字母开头、使用驼峰风格而不使用下划线**:

```kotlin
fun processDeclarations() { /*……*/ }
var declarationCount = 1
```

**例外：用于创建类实例的工厂函数可以与抽象返回类型具有相同的名称：**

```kotlin
interface Foo { /*……*/ }

class FooImpl : Foo { /*……*/ }

fun Foo(): Foo { return FooImpl() }
```

### 8.测试方法的名称

**当且仅当在测试中，可以使用反引号括起来的带空格的方法名。 （请注意，Android 运行时目前不支持这样的方法名。）测试代码中也允许方法名使用下划线。**

```kotlin
class MyTestCase {
     @Test fun `ensure everything works`() { /*...*/ }
     
     @Test fun ensureEverythingWorks_onAndroid() { /*...*/ }
}
```

### 9.属性名

**常量名称（标有 const 的属性，或者保存不可变数据的没有自定义 get 函数的顶层/对象 val 属性）应该使用大写、下划线分隔的名称：**

```kotlin
const val MAX_COUNT = 8
val USER_NAME_FIELD = "UserName"
```

**保存带有行为的对象或者可变数据的顶层/对象属性的名称应该使用驼峰风格名称：**

```kotlin
val mutableCollection: MutableSet<String> = HashSet()
```

**保存单例对象引用的属性的名称可以使用与 `object` 声明相同的命名风格：**

```kotlin
val PersonComparator: Comparator<Person> = /*...*/
```

**对于枚举常量，可以使用大写、下划线分隔的名称 （`enum class Color { RED, GREEN }`）也可使用首字母大写的常规驼峰名称，具体取决于用途。**

**如果一个类有两个概念上相同的属性，一个是公共 API 的一部分，另一个是实现细节，那么使用下划线作为私有属性名称的前缀：**

```kotlin
    class C {
    private val _elementList = mutableListOf<Element>()
    val elementList: List<Element>
         get() = _elementList

}
```

**选择好名称**

> 类的名称通常是用来解释类是什么的名词或者名词短语：List、 PersonReader。
> 方法的名称通常是动词或动词短语，说明该方法做什么：close、 readPersons。 修改对象或者返回一个新对象的名称也应遵循建议。例如 sort 是对一个集合就地排序，而 sorted 是返回一个排序后的集合副本。
> 名称应该表明实体的目的是什么，所以最好避免在名称中使用无意义的单词 （Manager、 Wrapper 等）
>
> 当使用首字母缩写作为名称的一部分时，如果缩写由两个字母组成，就将其大写（IOStream）； 而如果缩写更长一些，就只大写其首字母（XmlFormatter、 HttpInputStream）。



#### 10.格式化

**使用 4 个空格缩进。不要使用 tab。**
**对于花括号，将左花括号放在结构起始处的行尾，而将右花括号放在与左括结构横向对齐的单独一行。**

* **横向空白**

> 在二元操作符左右留空格（a + b）。例外：不要在“range to”操作符（0..i）左右留空格。
> 不要在一元运算符左右留空格（a++）
> 在控制流关键字（if、 when、 for 以及 while）与相应的左括号之间留空格。
> 不要在主构造函数声明、方法声明或者方法调用的左括号之前留空格。

```   kotlin
class A(val x: Int)
fun foo(x: Int) { …… }
fun bar() {
    foo(1)
}
```

**绝不在. 或者 ?. 左右留空格：foo.bar().filter { it > 2 }.joinToString(), foo?.bar()**
**在 // 之后留一个空格：// 这是一条注释**
**不要在用于指定类型参数的尖括号前后留空格：class Map<K, V> { …… }**
**不要在 :: 前后留空格：Foo::class、 String::length**
**不要在用于标记可空类型的 ? 前留空格：String?**
**作为一般规则，避免任何类型的水平对齐。将标识符重命名为不同长度的名称不应该影响声明或者任何用法的格式。**

* **冒号**

> 在以下场景中的 : 之前留一个空格：
> •	当它用于分隔类型与超类型时；
> •	当委托给一个超类的构造函数或者同一类的另一个构造函数时；
> •	在 object 关键字之后。
> 而当分隔声明与其类型时，不要在 : 之前留空格。
> 在 : 之后总要留一个空格。

```kotlin
abstract class Foo<out T : Any> : IFoo {
  abstract fun foo(a: Int): T
}
class FooImpl : Foo() {
    constructor(x: String) : this(x) { /*……*/ }   
    val x = object : IFoo { /*……*/ } 
}
```

* **类头格式化**

**具有少数主构造函数参数的类可以写成一行:**

``` class Person(id: Int, name: String)```

**具有较长类头的类应该格式化，以使每个主构造函数参数都在带有缩进的独立的行中。 另外，右括号应该位于一个新行上。如果使用了继承，那么超类的构造函数调用或者所实现接口的列表应该与右括号位于同一行：**

```kotlin
 class Person(
   id: Int,
    name: String,
    surname: String
) : Human(id, name) { /*……*/ }
```

**对于多个接口，应该将超类构造函数调用放在首位，然后将每个接口应放在不同的行中：**

``` kotlin
   class Person(
    id: Int,
    name: String,
    surname: String
) : Human(id, name),
    KotlinMaker { /*……*/ }
```

**对于具有很长超类型列表的类，在冒号后面换行，并横向对齐所有超类型名：**

``` kotlin
class MyFavouriteVeryLongClassHolder :
   MyLongHolder<MyFavouriteVeryLongClass>(),
    SomeOtherInterface,
    AndAnotherOne {
    fun foo() { /*...*/ }
}
```

**构造函数参数使用常规缩进（4 个空格）。**

**理由：这确保了在主构造函数中声明的属性与 在类体中声明的属性具有相同的缩进。**



* **修饰符**

**如果一个声明有多个修饰符**，请始终按照以下顺序安放：

**public / protected / private / internal**
**expect / actual**
**final / open / abstract / sealed / const**
**external**
**override**
**lateinit**
**tailrec**
**vararg**
**suspend**
**inner**
**enum / annotation**
**companion**
**inline**
**infix**
**operator**
**data**

**将所有注解放在修饰符前**：

```  kotlin
@Named("Foo")
private val foo: Foo
```

**除非你在编写库，否则请省略多余的修饰符（例如 public）。**

* **注解格式化**

注解通常放在单独的行上，在它们所依附的声明之前，并使用相同的缩进：

```kotlin
@Target(AnnotationTarget.PROPERTY)
annotation class JsonExclude
//无参数的注解可以放在同一行：
@JsonExclude @JvmField
var x: String
//无参数的单个注解可以与相应的声明放在同一行：
@Test fun foo() { /*……*/ }
```

* **文件注解**

文件注解位于文件注释（如果有的话）之后、package 语句之前，并且用一个空白行与 package 分开（为了强调其针对文件而不是包）

``` kotlin
/** 授权许可、版权以及任何其他内容 */
@file:JvmName("FooBar")
package foo.bar
```

* **函数格式化**

如果函数签名不适合单行，请使用以下语法：

```kotlin
fun longMethodName(
    argument: ArgumentType = defaultValue,
    argument2: AnotherArgumentType
): ReturnType {
    // 函数体
}
```

函数参数使用常规缩进（4 个空格）。

理由：与构造函数参数一致

对于由单个表达式构成的函数体，优先使用表达式形式。

```kotlin
fun foo(): Int {     // 不良
    return 1 
}

fun foo() = 1        // 良好
```

* **表达式函数体格式化**

如果函数的表达式函数体与函数声明不适合放在同一行，那么将 = 留在第一行。 将表达式函数体缩进 4 个空格。

``` kotlin
 fun f(x: String) =
     x.length
```

* 属性格式化

```kotlin
val isEmpty: Boolean get() = size == 0
```

对于更复杂的属性，总是将 `get` 与 `set` 关键字放在不同的行上：

```kotlin
val foo: String
    get() { /*……*/ }
```

对于具有初始化器的属性，如果初始化器很长，那么在等号后增加一个换行并将初始化器缩进四个空格：

```kotlin
private val defaultCharset: Charset? =
    EncodingRegistry.getInstance().getDefaultCharsetForPropertiesFiles(file)
```

* **格式化控制流语句**

如果 if 或 when 语句的条件有多行，那么在语句体外边总是使用大括号。 将该条件的每个后续行相对于条件语句起始处缩进 4 个空格。 将该条件的右圆括号与左花括号放在单独一行：

```kotlin
if (!component.isSyncing &&
    !hasAnyKotlinRuntimeInScope(module)
) {
    return createKotlinNotConfiguredPanel(module)
}
```

理由：对齐整齐并且将条件与语句体分隔清楚
将 else、 catch、 finally 关键字以及 do/while 循环的 while 关键字与之前的花括号放在相同的行上：

```kotlin
if (condition) {
    // 主体
} else {
    // else 部分
}

try {
    // 主体
} finally {
    // 清理
}
```

在 when 语句中，如果一个分支不止一行，可以考虑用空行将其与相邻的分支块分开：

```kotlin
private fun parsePropertyValue(propName: String, token: Token) {
    when (token) {
        is Token.ValueToken ->
            callback.visitValue(propName, token.value)

        Token.LBRACE -> { // ……
        }
    }
}
```

将短分支放在与条件相同的行上，无需花括号。

```kotlin
when (foo) {
    true -> bar() // 良好
    false -> { baz() } // 不良
}
```

* **方法调用格式化**

在较长参数列表的左括号后添加一个换行符。按 4 个空格缩进参数。 将密切相关的多个参数分在同一行。

```kotlin
drawSquare(
    x = 10, y = 10,
    width = 100, height = 100,
    fill = true
)
```

在分隔参数名与值的 = 左右留空格。

* **链式调用换行**

当对链式调用换行时，将 . 字符或者 ?. 操作符放在下一行，并带有单倍缩进：

```kotlin
val anchor = owner
    ?.firstChild!!
    .siblings(forward = true)
    .dropWhile { it is PsiComment || it is PsiWhiteSpace }
```

调用链的第一个调用通常在换行之前，当然如果能让代码更有意义也可以忽略这点。

* **Lambda 表达式格式化**

在 lambda 表达式中，应该在花括号左右以及分隔参数与代码体的箭头左右留空格。 如果一个调用接受单个 lambda 表达式，应该尽可能将其放在圆括号外边传入。

```kotlin
list.filter { it > 10 }
```

如果为 lambda 表达式分配一个标签，那么不要在该标签与左花括号之间留空格：

```kotlin
fun foo() {
    ints.forEach lit@{
        // ……
    }
}
```

在多行的 lambda 表达式中声明参数名时，将参数名放在第一行，后跟箭头与换行符：

```kotlin
appendCommaSeparated(properties) { prop ->
    val propertyValue = prop.get(obj)  // ……
}
```

如果参数列表太长而无法放在一行上，请将箭头放在单独一行：

```kotlin
foo {
   context: Context,
   environment: Env
   ->
   context.configureEnv(environment)
}
```

* **Unit**

如果函数返回 Unit，那么应该省略返回类型：

```kotlin
fun foo() { // 这里省略了“: Unit”

}
```

* **分号**

尽可能省略分号。

* **字符串模版**

将简单变量传入到字符串模版中时不要使用花括号。只有用到更长表达式时才使用花括号。

```kotlin
println("$name has ${children.size} children")
```

#### 11.语言特性的惯用法

* **不可变性**

优先使用不可变（而不是可变）数据。初始化后未修改的局部变量与属性，总是将其声明为 val 而不是 var 。
总是使用不可变集合接口（Collection, List, Set, Map）来声明无需改变的集合。使用工厂函数创建集合实例时，尽可能选用返回不可变集合类型的函数：

```kotlin
// 不良：使用可变集合类型作为无需改变的值
fun validateValue(actualValue: String, allowedValues: HashSet<String>) { …… }

// 良好：使用不可变集合类型
fun validateValue(actualValue: String, allowedValues: Set<String>) { …… }

// 不良：arrayListOf() 返回 ArrayList<T>，这是一个可变集合类型
val allowedValues = arrayListOf("a", "b", "c")

// 良好：listOf() 返回 List<T>
val allowedValues = listOf("a", "b", "c")
```

* **默认参数值**

优先声明带有默认参数的函数而不是声明重载函数。

```kotlin
// 不良
fun foo() = foo("a")
fun foo(a: String) { /*……*/ }
// 良好
fun foo(a: String = "a") { /*……*/ }
```

* **类型别名**

如果有一个在代码库中多次用到的函数类型或者带有类型参数的类型，那么最好为它定义一个类型别名：

```kotlin
typealias MouseClickHandler = (Any, MouseEvent) -> Unit
typealias PersonIndex = Map<String, Person>
```

* **Lambda 表达式参数**

> 在简短、非嵌套的 lambda 表达式中建议使用 it 用法而不是显式声明参数。而在有参数的嵌套 lambda 表达式中，始终应该显式声明参数。

* **在 lambda 表达式中返回**

> 避免在 lambda 表达式中使用多个返回到标签。请考虑重新组织这样的 lambda 表达式使其只有单一退出点。 如果这无法做到或者不够清晰，请考虑将 lambda 表达式转换为匿名函数。
> 不要在 lambda 表达式的最后一条语句中使用返回到标签。

* **具名参数**

当一个方法接受多个相同的原生类型参数或者多个 Boolean 类型参数时，请使用具名参数语法， 除非在上下文中的所有参数的含义都已绝对清楚。

```kotlin
drawSquare(x = 10, y = 10, width = 100, height = 100, fill = true)
```

* **使用条件语句**

优先使用 try、if 与 when 的表达形式。例如：

```kotlin
return if (x) foo() else bar()

return when(x) {
    0 -> "zero"
    else -> "nonzero"
}
```

优先选用上述代码而不是：

```kotlin
if (x)
    return foo()
else
    return bar()
    
when(x) {
    0 -> return "zero"
    else -> return "nonzero"
}
```



* **if 还是 when**

二元条件优先使用 if 而不是 when。不要使用

```kotlin
when (x) {
    null -> // ……
    else -> // ……
}
```

而应使用 if (x == null) …… else ……
如果有三个或多个选项时优先使用 when。

* **在条件中使用可空的 Boolean 值**

如果需要在条件语句中用到可空的 Boolean, 使用 if (value == true) 或 if (value == false) 检测。

* **使用循环**

优先使用高阶函数（filter、map 等）而不是循环。例外：forEach（优先使用常规的 for 循环， 除非 forEach 的接收者是可空的或者 forEach 用做长调用链的一部分。）
当在使用多个高阶函数的复杂表达式与循环之间进行选择时，请了解每种情况下所执行操作的开销并且记得考虑性能因素。

* **区间上循环**

使用 until 函数在一个开区间上循环：

```kotlin
for (i in 0..n - 1) { /*……*/ }  // 不良
for (i in 0 until n) { /*……*/ }  // 良好
```

* **使用字符串**

优先使用字符串模板而不是字符串拼接。
优先使用多行字符串而不是将 \n 转义序列嵌入到常规字符串字面值中。
如需在多行字符串中维护缩进，当生成的字符串不需要任何内部缩进时使用 trimIndent，而需要内部缩进时使用 trimMargin：

```kotlin
assertEquals(
    """
    Foo
    Bar
    """.trimIndent(), 
    value
)

val a = """if(a > 1) {
          |    return a
          |}""".trimMargin()
```

* **函数还是属性**

在某些情况下，不带参数的函数可与只读属性互换。 虽然语义相似，但是在某种程度上有一些风格上的约定。
底层算法优先使用属性而不是函数：
•	不会抛异常
•	计算开销小（或者在首次运行时缓存）
•	如果对象状态没有改变，那么多次调用都会返回相同结果

* **使用扩展函数**

放手去用扩展函数。每当你有一个主要用于某个对象的函数时，可以考虑使其成为一个以该对象为接收者的扩展函数。为了尽量减少 API 污染，尽可能地限制扩展函数的可见性。根据需要，使用局部扩展函数、成员扩展函数或者具有私有可视性的顶层扩展函数。

* **使用中缀函数**

一个函数只有用于两个角色类似的对象时才将其声明为中缀函数。良好示例如：and、 to、zip。 不良示例如：add。
如果一个方法会改动其接收者，那么不要声明为中缀形式。

* **工厂函数**

如果为一个类声明一个工厂函数，那么不要让它与类自身同名。优先使用独特的名称， 该名称能表明为何该工厂函数的行为与众不同。只有当确实没有特殊的语义时， 才可以使用与该类相同的名称。
例如：

```kotlin
class Point(val x: Double, val y: Double) {
    companion object {
        fun fromPolar(angle: Double, radius: Double) = Point(...)
    }
}
```

如果一个对象有多个重载的构造函数，它们并非调用不同的超类构造函数，并且不能简化为具有默认参数值的单个构造函数，那么优先用工厂函数取代这些重载的构造函数。

*  **平台类型**

返回平台类型表达式的公有函数/方法必须显式声明其 Kotlin 类型：

```kotlin
fun apiCall(): String = MyJavaApi.getProperty("name")
```

任何使用平台类型表达式初始化的属性（包级别或类级别）必须显式声明其 Kotlin 类型：

```kotlin
class Person {
    val name: String = MyJavaApi.getProperty("name")
}
```

使用平台类型表达式初始化的局部值可以有也可以没有类型声明：

```kotlin
fun main() {
    val name = MyJavaApi.getProperty("name")
    println(name)
}
```

* **使用作用域函数 apply/with/run/also/let**

Kotlin 提供了一系列用来在给定对象上下文中执行代码块的函数：let、 run、 with、 apply 以及 also。 关于不同情况下选择正确作用域函数的准则

# 注意事项

####  1.类型的声明与使用

val与var
val->不可变引用，var->可变引用。
我们应该尽可能地使用val关键字来声明所有的kotlin变量 为什么呢？

1. 首先一个变量在声明时是不可变的，那就代表你在使用的时候不需要考虑其他地方会对它重新赋值和改变(对于对象注意只是引用不可变)，直接使用。

2. val声明的类型由于必须初始化,它是线程安全的。

3. kotlin为了保证类型安全，所有变量声明的地方必须要做初始化，即显示赋一个初值。

   #### 2.空与非空

kotlin对于可空类型和非空类型认为是两个完全不同的类型，比如Int与Int?,他们俩就不是相同的类型。利用这个特点和编译时检查，kotlin基本可以避免空指针异常。
上面已经说了kotlin的类型声明时必须要有初值，所以空与非空类型和val与var一组合就会变成4种情况，下面我们对一个Person对象进行声明:

```kotlin
val p:Person = Person()
val p:Person? = null    // 这个情况是没有意义的
var p:Person = Person()  // 如果这个对象是非空类型，那么初始化的时候必须赋一个非null初值
var p:Person? = null   //可以给一个null,也可以给一个对象实例
```

**上面这段代码基本解释了val与var和空与非空的关系。**

#### 3.空与非空的使用

kotlin对空与非空做了严格的限制，那我们在使用时不都要对可空类型类型做判断吗？为了避免这个问题，kotlin提供了许多运算符来使开发人员对于可空与非空编码更加愉快。

* **安全调用运算符 "?."**

比如我们声明了这样一个类型var p:Person? = null。如果我们直接使用p.name，kotlin编译器是会报错的无法编译通过，所有我们必须这么做:

``` if(p != null) p.name```

这种代码写多了实在是太没有意义了，所有kotlin提供了?.。上面代码我们可以直接这样代替p?.name。它的实际执行过程是这样的:如果p为空则不操作，如果p不为空则调用p.name。

* **Elvis 运算符 "?:"**

在kotlin中我们会写这种代码:

```val name = if(p != null) p.name else ""   //kotlin中的if是一个表达式```

不过使用?:可以更简单实现上面的逻辑 : val name = p?.name ?: "" 。 它的实际执行逻辑是如果p为null，p?.name就会为null， ?:会检查前面的结果，如果是null，那么则返回""。

- 安全转换 "as?"

在kotlin中类型转换关键字为as。不过类型转换会伴随着转换失败的风险。使用as?我们可以更优雅的写出转换无风险的代码:

``` kotlin
//Person类中的方法
fun equals(o:Any?):Boolean{
val otherPerson = o as? Person ?: return false
....
} 
```

即as?在转换类型失败时会返回null

* **非空断言 "!!"**

如果使用一个可空类型的方法，并且你不想做非空判断，那么你可以这样做: person!!.getAge()。 不过如果person为null，这里就会抛出空指针异常。
其实还是在蛮多case下可以使用它，但是不建议使用, 因为你完全可以使用?、?:来做更优雅的处理, 你也可以使用lateinit来避免编译器的可空提示

#### 4.val与bylazy、var与lateinit

- **bylazy**

它只能和val一块使用。

```kotlin
private val mResultView：View = bylazy{
initResultView()
}
```

使用bylazy我们可以对一个变量延迟初始化，即懒加载。它是线程安全的。具体原理是:当我们使用bylazy声明的变量时，如果这个变量为null,那么就会调用bylazy代码块来初始化这个变量。

* **lateinit**

它只能和var一块使用。并且不允许修饰可空类型。那它的使用场景是什么呢？ 在有些case下，比如一个构造复杂的对象，我们就是想把变量声明为非空类型并且就是不想给他一个初值(代价太大了),这时候我们就可以使用lateinit :

```kotlin
lateinit var p : Person  //Person的构造函数太复杂了，不想在这里给一个初值
 
fun refreshUI(p2:Person){
//p = p2
val name = p.name  //注意这个地方是可能会抛p为初始化异常的！！！如果你没有初始化
}
```

由于使用lateinit的时候我们要人工保证这个变量已经被初始化，并且kotlin在你每个使用这个变量的地方都会添加一个非null判断。所以lateinit尽量少用。

#### 5.when 与 if

* **if**

if在kotlin中不只是一个控制结构它也是一个表达式,即它是有返回结果的，我们可以利用它来代替java中的三目运算符:

```kotlin
val background = if(isBlcak) R.drawable.black_image else R.drawable.white_image
```

* **when**

它的使用方法有多种:
•	代替switch的功能

```kotlin
when(color){
"red","green" ->{ }
"blue"->{ }
}
```

kotlin中的when可以用来判断任何对象，它会逐一检查每一个分支，如果满足这个分支的条件就执行。
•	多条件判断
可以使用when来避免if..elseif..elseif..else的写法:

```kotlin
val a = 1
val b = 2
when{
a > 0 && b > 0 ->{}
a < 0 && b > 0 ->{}
a < 0 && b < 0 ->{}
else ->{ }
}
```

•	when是带有返回值的表达式
和if一样，when也是一个表达式:

```kotlin
val desColor = when(color){
"red", "gren" -> "red&green"
"blue" -> "blue"
else -> "black" // when作为表达式时必须要有else分支。
}
```



#### 6.类

**1.类的构造与主构造函数**

•	简单的声明一个 javabean
在kotlin中我们可以这样简单的定义一个类: class Person(val name:String = "", var age:Int = 0)
这样就定义了一个Person类，这个类有两个属性:name和age。并且他有一个两个参数的构造函数来对这两个属性初始化。可以看出kotlin将一个类的声明变的十分方便。
•	主构造函数
普通的java构造函数是有代码块的，即可以做一些逻辑操作，那按照kotlin上面的方式，我们怎么做构造函数的逻辑操作呢? kotlin提供了初始化代码块:

```kotlin
class Person(val name:String = "", var age:Int = 0){
init{
name = "susion"
age = 13
}
}
```

init代码块会在主构造函数之后运行，注意不是所有的构造函数。

**2.数据类 data class**

更方便的定义一个javabean，我们可以使用数据类:

```kotlin
data class Person(val name:String = "", val age:Int = 0)
```

使用data定义的Person会默认生成equals、hashCode、toString方法。需要注意的是数据类的属性我们应该尽量定义成val的。这是因为在主构造函数中声明的这些属性都会纳入到equals和hashCode方法中。如果某个属性是可变的，
那么这个对象在被加入到容器后就会是一个无效的状态。

**3.object 和 companion object**

在kotlin中没有静态方法，也没有静态类。不过kotlin提供了object与companion object

•	object 单例类
使用object我们可以很轻松的创建一个单例类 :

```kotlin
object LoginStatus{
var isLogin = false
 
fun login(){
isLogin = true
}
...
}
```

我们可以这样直接使用LoginStatus.isLogin()。 那这个单例在kotlin中是怎么实现的呢？我们可以反编译看一下它生成的java代码:

```kotlin
public final class LoginStatus {
private static boolean isLogin;
public static final LoginStatus INSTANCE; // for java调用
 
..... //省略不重要的部分
 
public final void login() {
isLogin = true;
}
 
private LoginStatus() {
INSTANCE = (LoginStatus)this;
}
 
static { //类加载的时候构造实例
new LoginStatus();
}
}
```

即kotlin object实现的单例是线程安全的。它的对象是在类创建的时候就产生了。

•	object的静态方法的使用
上面我们已经知道object创建单例的原理了。这在某些case下就很棒，但是某些时候我们不是想要单例，我们只是想要一些静态方法呢？比如我们经常创建的一些工具类(UIUtils、StringUtils)等。我们可以直接使用object来完成:

```kotlin
public object UIUtils{
...
...
...
..很多方法
}
```

按照kotlin单例的设计，我们只要一旦使用这些方法，那么一直有一个单例对象UIUtils存在于内存中。那么这样好吗? 我们是否可以这样写呢 :

```kotlin
public class UIUtils{
...
...
...
..很多方法
}
```

然后在使用的时候:UIUtils().dp2Px(1)。这样至少不会有一个对象一直在内存中。我想我们在某些case下可以这样使用我们的工具类。或者你可以使用kotlin的扩展函数或顶层函数来定义一些工具方法。所以对于kotlin的object的使用需要注意

- companion object

companion object主要是为了方便我们可以在一个类中创建一些静态方法而存在的，比如:

```kotlin
class Person(val name: String) {
companion object {
fun isMeal(p: Person) = false
}
 
}
```

即它也是生成了一个单例类Person.Companion。不过这个单例类是一个静态类。不允许构造。不过它的实现机制几乎和object相同。

* Sealed Classes

Sealed Classes 类似于 Java 中的 Enum 不过每个类都是一个普通的类。用来表示有限的子类型。

```kotlin
sealed class Expr
data class Const(val number: Double) : Expr()

data class Sum(val e1: Expr, val e2: Expr) : Expr()

object NotANumber : Expr()adadfas 
```

上面的 Expr 为 sealed class，则其所有的子类都定义在同一个文件中，这样编译器就知道 Expr 只有三个子类 Const、Sum 和 NotANumber，然后可以使用 when 表达式来判断种情况：

```kotlin
    fun eval(expr: Expr): Double = when(expr) {
    is Const -> expr.number
    is Sum -> eval(expr.e1) + eval(expr.e2)
    NotANumber -> Double.NaN
    // the `else` clause is not required because we've covered all the cases
} 
```

**4.Delegation**

Kotlin 在语言级别就支持代理, 很多 Listener 都定义了不只一个函数，如果我们只关心其中的一个，其他的函数就需要用空的函数体来实现，这样代码看起来不够简洁，使用代理可以解决这个问题：

``` kotlin
class MyListener : TransitionListener by EmptyTransitionListener {
  override fun onTransitionStart(transition: Transition) {

  }
 }
 object EmptyTransitionListener : TransitionListener {

  override fun onTransitionEnd(transition: Transition) {}

  override fun onTransitionResume(transition: Transition) {}

  override fun onTransitionPause(transition: Transition) {}

  override fun onTransitionCancel(transition: Transition) {}

  override fun onTransitionStart(transition: Transition) {}

 }
```

比如上面的 TransitionListener 定义了 5 个函数，MyListener 只关心其中一个，使用 object 来定义一个空实现对象 EmptyTransitionListener ，这个 EmptyTransitionListener 对象是单例，MyListener 中没有实现的函数都被代理到 EmptyTransitionListener 对象对应的函数。

**5.lambda**

kotlin中lambda的本质就是可以传递给其他函数的一小段代码。kotlin中lambda使用的最多的就是和集合一块使用。
•	lambad与java接口
比如我们经常给View设置onClickListener,在kotlin中我们可以很方便的实现这段代码:

```kotlin
userView.setOnClickListener{
 
}  // 如果lambda是函数的最后一个参数，那么是可以放在括号外面的。
```

即你可以直接传递给它一个lambda。kotlin在实际编译的时候会把这个lambda编译成一个匿名内部类。 那么所有java传对象的地方都可以这样使用吗？ 当然不是, 只有java参数满足下面条件才可以使用:
这个参数是一个接口，并且这个接口只有一个抽象方法。就可以这样使用，比如Runnable、Callable等。

•	with 与 apply
with个人感觉比较鸡肋，这里就不讲它了。不过apply是十分实用的。在kotlin中apply被实现为一个函数:

```kotlin
public inline fun <T> T.apply(block: T.() -> kotlin.Unit): T {  }
```

即它是一个扩展函数，接收一个lambda，并返回对象本身,并且他是一个内联的函数(下面会讲)
我最常用的一个case是给view设置参数:

```kotlin
val tvName = TextView(context).apply{
textSize = 10
textColor = xx
text = "susion"
...
}
```

##### 6.常用的库函数

* **内联函数**

kotlin集合库中很多函数可以接收lambda作为参数。但我们前面说了kotlin的lambda表达式会被编译为一个匿名内部类，即每一次lambda的调用都会创建一个匿名内部类，所以会带来运行时的开销。
kotlin集合库中函数时给我们使用的，如果有这种开销的话，这肯定是一个不好的设计，因此kotlin集合库中的大部分函数都是内联函数。

比如上面的apply声明 : public inline fun <T> T.apply(block: T.() -> kotlin.Unit): T { .... }

何为内联呢？: 当一个函数被声明为inline时，它的函数体是内联的，即函数体会被直接替换到函数被调用的地方。即不会存在运行时开销 下面要说的filter和map都是内联函数。
•	filter 与 map
filter接收一个返回Boolean的lambda，用于对一个集合做筛选工作,并返回一个新集合:

```kotlin
val wangList = userList.filter{it.firstName == "wang"}
```

map也是接收一个lambda，它的返回值是另一个类型的集合，即map可以把一个类型的集合变成另一个类型的集合。

```kotlin
val ageList = userList.map{ it.age }
```

这两个函数虽然好用，不过我们要注意他们的实现，以免带来不必要的性能损耗 : filter和map函数都会创建一个新集合，因此如果你是下面这种用法就可能出现不必要的集合创建:

```kotlin
val wangList = userList.map{ it.name }.filter{it == "wang"}
```

在这种情况下，userList集合非常大的话，那么map操作之后生成的中间集合也可能非常大 对于这种情况可以考虑使用 kotlin序列。

* **count与find**

count用于统计集合中满足某个条件的数量，可以对比下面这种写法

```kotlin
val wangCount= userList.filter{it.firstName == "wang"}.size
val wangCount2 = userList.count{it.firstName =="wang"}//很明显，这种写法远好于第一种
```

