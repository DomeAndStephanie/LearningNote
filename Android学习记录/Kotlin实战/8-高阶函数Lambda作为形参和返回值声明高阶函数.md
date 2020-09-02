

#### 高阶函数:Lambda作为形参和返回值

---

###### 声明高阶函数

> 高阶函数即以另一个函数作为参数或返回值的函数.在Kotlin中,函数可以用lambda或者函数引用来表示.则任何以lambda或者函数引用作为参数或返回值的函数都是高阶函数.

- 函数类型

  Kotlin中的函数类型语法:箭头左侧为参数类型,右侧为返回类型

  ```mermaid
  graph LR
  A["(Int,String)"] --> B["Unit"]
  ```

  ```kotlin
  //声明返回值可空的函数类型
  var canReturnNull: (Int,Int) -> Int? = { null }
  //声明可空的函数类型的变量
  var funOrNull : ((Int,Int) -> Int)? = null
  ```

- 调用作为参数的函数

  ```kotlin
  fun twoAndThree(operation : (Int,Int) -> Int){
      println(operation(2,3))
  }
  ```

- 返回函数的函数

  > 声明一个返回另一个函数的函数,需要指定一个函数类型作为返回类型

  ```kotlin
  fun getShippingCostCalutor(delivery:Delivery) : (Order) -> Double{
      .....
      return {order -> 1.2*itemcount}
  }
  ```

###### 内联函数

- lambda表达式会被正常编译为匿名类，每调用一次lambda就会额外创建一个类，带来运行时的额外开销
- 使用`inline`修饰符标记一个函数,在函数被使用时编译器不会生成函数调用的代码,而是使用函数实现的真实代码替换每一次的函数调用
- 当一个函数被声明为`inline`时,函数体会被直接替换到函数被调用的地方,而不是正常调用
- 给内联函数传递一个函数类型的变量为参数而不是lambda时不会被内联.
- 如果在两个不同的位置使用同一个内联函数,但是用的是不同的lambda,那么内联函数会在每一个被调用的位置被分别内联.

###### 高阶函数中控制流

- lambda中的返回语句-从封闭函数返回

  1. 在lambda中使用**return**关键字,会从调用lambda的函数中返回,并不只是从lambda中返回
  2. 只有在以lambda作为参数的函数是内联函数的时候才能从更外层的函数返回

- 从lambda返回-使用标签返回

  从一个lambda表达式处返回需要在**return**关键字后面引用**标签**

  ```kotlin
  people.forEach label@{
      if(it.name == "Alice")
      	return@label   //返回表达式标签
  }
  ```

  可以自定义标签名或者使用函数名作为标签，但显式地指定了lambda表达式之后，再使用函数名作为标签会没有任何效果。一个lambda表达式的标签数量不能多于一个。

- 匿名函数-默认使用局部返回

  ```kotlin
  people.forEach(fun (person){
      if(person.name == "Alice") return 
      println("${person.name} is not Alice")
  })
  ```

  - 在匿名函数中,不带标签的**return**表达式会从匿名函数返回,而不是从包含匿名函数的函数返回:**return从最近的使用fun关键字声明的函数返回**
  - 尽管匿名函数看起来和普通函数很相似，但它其实是lambda表达式的另一种语法形式