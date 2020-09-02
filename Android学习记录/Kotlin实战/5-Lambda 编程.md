#### Lambda 编程

---

1. Lambda 简介

   Lambda表达式(Lambda),本质上是**可以传递给其他函数的一小段代码**

2. Lambda表达式语法

   ![Lambda语法](https://gitee.com/domeofheaven2017/Image/raw/master/BlogImage/Lambda表达式语法.png)

   - 可以把`Lambda`表达式存储在一个变量中,把这个变量当作**普通函数**对待.

     ```kotlin
     //定义
     val sum = {x:Int,y:Int -> x + y }
     //调用
     println(sum(1,2))
     ```

   - `Kotlin`中当`Lambda`表达式是函数调用的最后一个实参时,可以将表达式放在括号外;且当`Lambda`式函数唯一的实参时,可以去掉调用代码中的空括号.

     ```kotlin
     //作用等价
     people.maxBy({ p:Person -> p.age})
     people.maxBy(){ p:Person -> p.age}
     people.maxBy{ p:Person -> p.age}
     people.maxBy{ p -> p.age} //编译器能推导出参数类型
     people.maxBy{ it.age }
     ```

   - `Lambda`也可以包含多条语句

     ```kotlin
     val sum = {
         x:Int,y:Int ->
         println("Computing the sum of $x and $y")
         x + y
     }
     ```

3. 成员引用

   使用`::`运算符,将函数转换成一个值传递给函数

   ```kotlin
   val getAge = Person :: age
   
   //引用顶层函数
   val action = { person : Person,message : String ->
                 sendEmail(person,message)
                }
   val nextAction = ::sendEmail
   ```

   > 不管引用的是函数还是属性，都不要在成员引用的名称后面加括号 

4. 集合函数API

   - `filter`:过滤掉不满足给定lambda表达式的元素,结果是一个新集合

     ```kotlin
     //filter函数定义
     /**
      * Returns a list containing only elements matching the given [predicate].
      */
     public inline fun <T> Iterable<T>.filter(predicate: (T) -> Boolean): List<T>{
         return filterTo(ArrayList<T>(), predicate)
     }
     //filter函数使用
     val list = listOf(1,2,3,4)
     println(list.filter { it % 2 == 0 })  //[2,4]
     ```

   - `map`:对集合中的每个元素应用给定的函数,并把结果收集到一个新集合

     ```kotlin
     //map函数定义
     /**
      * Returns a list containing the results of applying the given [transform] function
      * to each element in the original collection.
      */
     public inline fun <T, R> Iterable<T>.map(transform: (T) -> R): List<R> {
         return mapTo(ArrayList<R>(collectionSizeOrDefault(10)), transform)
     }
     //map函数使用
     val list = listOf(1,2,3,4)
     println(list.map{ it * 2}) //[2,4,6,8]
     ```

   - `all`:当所有元素都满足给定判别式时返回true

     ```kotlin
     //all函数定义
     /**
      * Returns `true` if all elements match the given [predicate].
      * 
      * @sample samples.collections.Collections.Aggregates.all
      */
     public inline fun <T> Iterable<T>.all(predicate: (T) -> Boolean): Boolean {
         if (this is Collection && isEmpty()) return true
         for (element in this) if (!predicate(element)) return false
         return true
     }
     //all函数使用
     val people = listOf(Person("Stephanie",26),Person("Dome",24))
     println(people.all{ it.age < 27 }) //true
     ```

   - `any`:检查集合所有元素中是否至少存在一个匹配的元素

     ```kotlin
     //any函数定义
     /**
      * Returns `true` if collection has at least one element.
      * 
      * @sample samples.collections.Collections.Aggregates.any
      */
     public fun <T> Iterable<T>.any(): Boolean {
         if (this is Collection) return !isEmpty()
         return iterator().hasNext()
     }
     //any函数使用
     val people = listOf(Person("Stephanie",26),Person("Dome",24))
     println(people.any{ it.age < 27 }) //true
     ```

   - `count`:返回集合中满足给定判断式的元素个数

     ```kotlin
     //count函数定义
     /**
      * Returns the number of elements matching the given [predicate].
      */
     public inline fun <T> Iterable<T>.count(predicate: (T) -> Boolean): Int {
         if (this is Collection && isEmpty()) return 0
         var count = 0
         for (element in this) if (predicate(element)) count++
         return count
     }
     //count函数使用
     val people = listOf(Person("Stephanie",26),Person("Dome",24))
     println(people.count{ it.age > 25 }) //1
     ```

   - `groupBy`:将集合按照给定的规则分成组,结果是一个`map`

     ```kotlin
     /**
      * Groups elements of the original collection by the key returned by the given [keySelector] function
      * applied to each element and returns a map where each group key is associated with a list of corresponding elements.
      * 
      * The returned map preserves the entry iteration order of the keys produced from the original collection.
      * 
      * @sample samples.collections.Collections.Transformations.groupBy
      */
     public inline fun <T, K> Iterable<T>.groupBy(keySelector: (T) -> K): Map<K, List<T>> {
         return groupByTo(LinkedHashMap<K, MutableList<T>>(), keySelector)
     }
     //groupBy使用
     val list = listOf(1,2,3,4)
     println(list.groupBy { it > 2 }) //{false=[1, 2], true=[3, 4]}
     ```

   - `flayMap`:对集合中的每个元素根据给定的函数作**映射**,然后将多个列表合并成一个

     ```kotlin
     /**
      * Returns a single list of all elements yielded from results of [transform] function being invoked on each element of original collection.
      */
     public inline fun <T, R> Iterable<T>.flatMap(transform: (T) -> Iterable<R>): List<R> {
         return flatMapTo(ArrayList<R>(), transform)
     }
     //flatMap函数使用 
     val list = listOf("Stephanie","DomeOfHeaven")
     println(list.flatMap { it.toList()})
     //[S, t, e, p, h, a, n, i, e, D, o, m, e, O, f, H, e, a, v, e, n]
     ```

5. 惰性集合操作:序列

   链式集合函数会及早地创建中间集合，每一步的中间结果都被存储在一个临时列表。序列中的元素求值是惰性的，可以使用序列(Sequence)更高效地执行链式操作，不会创建中间结果。

   - 执行序列操作：中间和末端操作

     1. 序列操作分为`中间的`和`末端的`两类，一次中间操作返回的是另一个序列；一个末端操作返回的是一个结果。

     2. 中间操作始终都是惰性的，末端操作触发执行了所有延期运算。

     3. 序列中所有操作都是按顺序应用在每一个元素上，处理完第一个(先映射再过滤),然后完成下面的元素，依次类推。

   - 创建序列

     1. 在集合上调用`asSequence`方法

        ```kotlin
        val people = listOf(Person("mym", 26), Person("zzc", 28))
        people.asSequence().map(Person.age).filter{it >= 27}.toList()
        ```

     2. 使用`generateSequence`函数

        ```kotlin
        //给定序列的前一个元素，该函数会计算出下一个元素
        val naturalNum = generateSequence(0){it + 1}
        ```

6. 使用Java函数式接口

   - 把lambda当作参数传递给Java方法

   - SAM构造方法：显式地把lambda转换成函数式接口

     SAM代表单抽象方法，SAM构造方法是编译器生成的函数。

     ```kotlin
     fun createAllDone(): Runnable {
         return Runnable {println("All Done")}
     }
     ```

7. `with`函数

   ```kotlin
   /**
    * Calls the specified function [block] with the given [receiver] as its receiver and returns its result.
    */
   @kotlin.internal.InlineOnly
   public inline fun <T, R> with(receiver: T, block: T.() -> R): R {
       contract {
           callsInPlace(block, InvocationKind.EXACTLY_ONCE)
       }
       return receiver.block()
   }
   //with函数使用
   with(obj){ //this:obj
       ...
   }
   ```

8. `apply`函数

   ```kotlin
   /**
    * Calls the specified function [block] with `this` value as its receiver and returns `this` value.
    */
   @kotlin.internal.InlineOnly
   public inline fun <T> T.apply(block: T.() -> Unit): T {
       contract {
           callsInPlace(block, InvocationKind.EXACTLY_ONCE)
       }
       block()
       return this
   }
   //apply函数使用
   obj.apply{ //this:obj
       ...
   }
   ```

