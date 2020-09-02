#### Kotlin的类型系统

---

1. ##### 可空性

   - 可空类型

     `Kotlin`对*可空类型*显示的支持.问号`?`可以加在任何类型后面表示这个类型的变量是可空的。

   - 类型的含义

     类型:类型就是数据的分类,决定了该类型可能的值,以及在该类型的值上可以完成的操作.

   - 安全调用运算符`?.`

     ```mermaid
     graph LR
     A[foo?.bar] --> B[foo.bar]
     A --> C[null]
     ```

     如图所示,使用安全调用运算符时,如果`foo`对象不为`null`则会调用;否则不会调用`bar`方法

     ```kotlin
     //使用安全调用处理可空属性
     class Employee(val name:String,val manager:Employee?)
     
     fun managerName(employee:Employee):String? = employee,manager?.name
     ```

   - Elvis运算符(null合并运算符)`?:`

     ```mermaid
     graph LR
     A[foo?:bar] --> B[foo]
     A --> C[bar]
     ```

     如上图所示,当`foo`不为`null`时,结果为`foo`对象;否则为`bar`对象.

     ```kotlin
     fun strLenSafe(s:String?):Int = s?.length?:0
     ```

   - 安全转换`as?`

     该运算符会尝试将值转换为指定的类型,如果值不是合适的类型就返回`null`

     ```mermaid
     graph LR
     A[foo as? Type] --> B[foo as Type]
     A --> C[null]
     ```

   - 非空断言`!!`

     ```mermaid
     graph LR
     A[foo!!] --> B[foo]
     A --> C[NullPointerException] 
     ```

     `!!`可以将任何值转换为非空类型,但如果对`null`做非空断言,则会抛出`NPE`

   - `let`函数

     `let`函数会把一个调用它的对象变成`lambda`表达式的参数

     ```kotlin
     //let函数只会在email的值非空时调用
     email?.let{
         email -> sendEmailTo(email)
     }
     ```

   - 延迟初始化属性

     使用`lateinit`修饰符来声明延迟初始化属性,因为需要在构造方法之外修改值,所以延迟初始化的属性都是`var`.
   
     ```kotlin
     private lateinit var myService: MyService
     ```
   
   - 可空类性的扩展
   
     为可空类型定义扩展函数来接收和处理可能为`null`的情况.
   
     ```kotlin
     fun String?.isNullOrBlank(): Boolean = 
     	this == null || this.isBlank()
     ```
   
     > 扩展函数中的`this`可能为`null`,所以必须显式地检查. 
   
   - 类型参数的可空性
   
     1. Kotlin中所有**泛型类和泛型函数的类型参数**默认都是可空的.
   
        ```kotlin
        fun <T> printHashCode(t:T){
            println(t?.hashCode()) //因为`t`可能为null,所以必须使用安全调用
        }
        //为类型参数声明非空上界来拒绝可空值作为实参
        fun <T:Any> printHashCode(t: T){
            println(t.hashCode())
        }
        ```
   
   - 可空性和Java
   
     > - Java中的@Nullable注解表达的参数被Kotlin当作可空类型,即Any?;而@NotNull注解表达的参数被Kotlin当作Any
   
2. **基本数据类型和其他基本类型**

   - 基本数据类型:Int,Boolean及其他

     Java中分为**基本数据类型**和**引用类型**.基本数据类型的变量直接存储了值,可以高效的存储和传递,但是不能调用方法;引用类型的变量存储了指向包含该对象的内存地址的引用.因此Java中增加了包装类型

   - 数字转换

     Kotlin不会自动把数字从一种类型转换为另一种,必须显式的进行转换.

     ```kotlin
     val x = 1
     println(x.toLong() in listOf(1L,2L,3L))
     ```

   - ‘Any’和‘Any?’:根类型

     `Any`类型是Kotlin所有**非空类型**的超类型;`Any?`是可空类型的超类型

   - Unit类型:Kotlin中的‘Void’

     - Unit可以作为类型参数,void不可以

        ```kotlin
       interface Processor<T>{
           fun process():T
       }
       class NoResultProcessor : Processor<Unit>{
           override fun process(){
               //不需要显式的return
           }
       }
       ```

   - Nothing类型

     Nothing类型没有任何值,只有被当作函数返回值使用,或被当作泛型函数返回值的类型使用才有意义.

3. 集合与数组

   - 可空性和集合

     与变量一样,可以使用`?`标记是否可以为null

     ![可空性与集合](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/可空性与集合.png)

     > 声明一个包含可空类型的可空列表:List<Int?>?

   - 只读集合与可变集合

     - 只读集合(Collection):只能对集合中的数据进行读取操作

     - 可变集合(MutableCollection):可以修改集合中的数据

     - 只读集合不一定是不可变的,不总是线程安全的.

       | 集合类型 | 只读   | 可变                                           |
       | -------- | ------ | ---------------------------------------------- |
       | List     | listOf | mutableListOf,arrayListOf                      |
       | Set      | setOf  | mutableSetOf,hashSetOf,linkedSetOf,sortedSetOf |
       | Map      | mapOf  | mutableMapOf,hashMapOf,linkedMapOf,sortedMapOf |

   - 对象和基本数据类型的数组

     Kotlin中的一个数组是一个带有参数类型的类,其元素类型被指定为相应的类型参数;

     数组类型的类型参数始终会变成对象类型.

     创建数组: 

     - arrayOf:创建的数组包含的元素是指定为该函数的实参

     - arrayOfNulls:创建包含null元素的数组

     - Array构造方法:接收数组的大小和lambda表达式来创建数组

       ```kotlin
       val letters = Array<String>(26){ i -> ('a' + i).toString()}
       ```

       