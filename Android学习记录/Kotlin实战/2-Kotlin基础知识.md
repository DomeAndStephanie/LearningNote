### Kotlin基础知识

1. ##### 函数

   1. 函数的基本构成

      ```kotlin
      fun 函数名称 (参数列表): 返回类型 {
        函数体
      }

      fun max(a:Int,b:Int):Int{
        return if(a>b) a else b
      }
      ```

   2. 表达式和语句

      表达式：有值，并且能作为另一个表达式的一部分使用

      语   句：总是包含着它的代码块中的顶层元素，并且没有自己的值

      > 在Java中所有的控制结构都是语句；
      >
      > 在Kotlin中除了循环(for,do和do/while)以外大多数控制结构都是表达式

   3. 省略返回值类型

      ```kotlin
      fun max ( a:Int,b:Int) = if(a>b) a else b
      ```
      
      > 如果函数体写在花括号中，则这个函数有代码块体，如果直接返回一个表达式，则它有表达式体

2. ##### 变量

   - 如果指定了初始化器，那么在不指定类型的情况下，编译器会分析初始化器表达式的值，并把它的类型作为变量的类型，

     ```kotlin
     fun test():Int={
       val xValue = 2.5
       val yValue = 3
       return xValue + yValue
     }
     ```

   - 如果没有指定初始化器，需要显示地指定它的类型，因为此时编译器无法推断出它的类型。

     ```kotlin
     fun test(){
       val eValue : Int
       println("eValue = $eValue")
     }
     ```

   1. 可变变量和不可变变量

      - Val(Value)-不可变引用：使用该关键字声明的变量不能在初始化后再次赋值．

        1. 对应**JAVA**中的final变量

        2. 如果编译器确保只有唯一一条语句会被执行，可以根据条件使用不同的值来初始化

           ```kotlin
           val message:String
           if(canPerformOperation()){
               message = "Success"
           }else{
               message = "Failed"
           }
           ```

        3. 虽然val引用自身不可变，但指向的对象是可变的

           ```kotlin
           val language = arrayListOf("Java")
           language.add("Kotlin")
           ```

      - Var(Variable)-可变引用：该关键字声明的变量的值可以改变．

        1. 对应**JAVA**中的普通变量
        2. var变量的类型是不可变的,编译器只会根据初始化器来推断变量的类型，如果需要则必须**手动转换**或者**强制转换**

3. ##### 字符串模板

   声明一个变量后，在变量名称前添加字符`$`,就可以在字符串字面值中引用该变量．

   1. 打印字符串变量

      ```kotlin
      fun main(args:Array<String>){
          val name = "Kotlin"
          printlin("Hello,$name!")
      }
      ```

   2. 复杂表达式

      ```kotlin
      fun main(args:Array<String>){
          if(args.size > 0){
              println("Hello,${args[0]!}")
          }
      }
      ```

   3. 双引号嵌套

      ```kotlin
      fun main(args:Array<String>){
          println("Hello,${if(args.size > 0 args[0] else "Kotln")}")
      }
      ```

4. ##### 类和属性

   ```java
   //Java中的类
   public class Person{
       private String mName;
       
       private void setName(String name){
           mName = name;
       }
       
       private String getName(){
           return mName;
       }
   }
   ```

   > 字段和其访问器的组合为属性

   ```kotlin
   //Kotlin中的类
   //Kotlin中属性默认为public
   Class Person(val name:String)
   ```

   > 1. Kotlin中属性默认为**public**
   >
   > 2. 声明属性时默认生成访问器，只读属性只生成**getter**,可写属性生成**getter**和**setter**
   >
   >    ```kotlin
   >    //自定义访问器
   >    class Rectangle(val height:Int,val width:Int){
   >        val isSquare:Boolean
   >        get(){
   >            return height == width
   >        }
   >    }
   >    ```

5. ##### 枚举类

   ```kotlin
   //声明枚举类
   enum class Color{
       RED,ORANGE,YELLOW,GREEN,BLUE,INDIGO,VIOLET
   }
   ```

   > Kotlin中，*enum*是**软关键字**,在class前面才有特殊意义，其他地方只是普通名称
   >
   > 和Java一样，枚举不是值的列表，可以给枚举类声明属性和方法
   >
   > ```kotlin
   > enum class Color (
   >  val r : Int, val g : Int, val b : Int  //声明常量属性
   > ) {
   >     RED(255, 0, 0), ORANGE(255, 165, 0);  //必须添加分号结束
   >     fun rgb() = (r * 256 + g) * 256 + b  //定义方法
   > }
   > ```
   >
   > 如果要在枚举类中定义任何方法，就要使用**分号**把枚举常量列表和方法定义分开。

6. ##### **When**表达式

   ```kotlin
   fun getWarmth(color:Color) = when(color){
       Color.RED,Color.ORANGE,Color.YELLOW -> "warm"
       Color.GREEN -> "neutral"
       Color.BLUE,Color.INDIGO,Color.VIOLET -> "cold"
   }
   ```

   > `when`表达式的实参可以是任何对象

   ```kotlin
   fun mix(c1:Color,c2:Color) = when(setOf(c1,c2)){
       setOf(Color.RED,Color.YELLOW) -> Color.ORANGE
       setOf(Color.YELLOW,Color.BLUE) -> Color.GREEN
       setOf(Color.BLUE,Color.VIOLET) -> Color.INDIGO
       else -> throw Exception("Dirty Color")
   }
   ```

   计算`(1+2)+4`

   ```kotlin
   //类结构
   interface Expr
   class Num(val value:Int) : Expr
   class Sum(val left:Expr,val right:Expr) : Expr
   ```

   ```mermaid
   graph TB
   A[Sum] --> B[Sum]
   A[Sum] --> C["Num(4)"]
   B[Sum] --> D["Num(1)"]
   B[Sum] --> E["Num(2)"]
   ```

   ```kotlin
   fun eval(e:Expr) : Int = when(e){
       is Num -> e.value
       is Sum -> eval(e.right)+eval(e.left)
       else -> throw IllegalArgumentException("Unknown expression")
   }
   ```

7. **While**循环

   1. `while`循环

      ```kotlin
      //当codition为true时执行循环
      while(codition){
          //循环体
      }
      ```

   2. `do-while`循环

      ```kotlin
      //循环体第一次无条件执行，此后，当codition为true时才执行
      do{
          //循环体
      } while(codition)
      ```

8. **for**循环

   kotlin使用**区间**的概念：［起始值．．结束值］

   ```kotlin
   for(i in 1 until 100 step 2){
       
   }
   for(i in 100 downTo 1 step 2){
   
   }
   ```

9. 迭代**map**

   ```kotlin
   val binaryReps = TreeMap<Char,String>()
   for(c in 'A'..'F'){
       val binary = Integer.toBinaryString(c.toInt())
       binaryReps[c] = binary
   }
   for((letter,binary) in binaryReps){
       println("$letter = $binary")
   }
   ```

10. Ktolin中的异常

    1. 捕获异常

       ```kotlin
       fun readNumber(reader:BufferedReader) : Int?{
           try{
               val line = reader.readLine()
               return Interger.parseInt(line)
           }catch(e:NumberFormatException){
               return null
           }finally{
               reader.close()
           }
       }
       fun readNumber(reader:BufferedReader){
           val number = try{
               Integer.parseInt(reader.readLine())
           }catch(e:NumberFormatException){
               null
           }
           println(number)
       }
       ```

       > **try**结构也可以作为表达式

    2. 抛出异常

       ```kotlin
       val percentage = 
       if(number in 0..100){
           number
       }else{
           throw IllegalArgumentException("....")
       }
       ```

       > 与*Java*中不同的是，*kotlin*中**throw**结构是一个表达式．

