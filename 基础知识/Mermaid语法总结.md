[TOC]

#### Mermaid语法总结

---

##### 一. 概念

mermaid是一个基于`JavaScript`的绘图工具，支持绘制流程图，时序图，状态图，类图，甘特图，实体关系图，用例图，饼图。

##### 二. 流程图

1. 声明关键字

   **graph**

2. 绘制方向

   - TB/TD：自上而下
   - BT：从下到上
   - RL：从右到左
   - LR：从左到右

3. 结点和形状

   id1[文字结点]

   id2(圆角结点)

   id3([体育场型结点])

   id4[[带子项的结点]]

   id5[(圆柱形结点)]

   id6((圆形结点))

   id7>不对称形结点]

   id8{菱形结点}

   id9{{六边形结点}}

   id10[/平行四边形结点/]

   id11[/梯形结点\]

   ```mermaid
   graph TB
   id1[文字结点]
   
   id2(圆角结点)
   
   id3([体育场型结点])
   
   id4[[带子项的结点]]
   
   id5[(圆柱形结点)]
   
   id6((圆形结点))
   
   id7>不对称形结点]
   
   id8{菱形结点}
   
   id9{{六边形结点}}
   
   id10[/平行四边形结点/]
   
   id11[/梯形结点\]
   ```

4. 实例

   ```mermaid
   graph TB
       A[Start] --> B{Is it?};
       B -->|Yes| C[OK];
       C --> D[Rethink];
       D --> B;
       B -->|No| E[End];
   ```

##### 三. 时序图

1. 关键字

   **sequenceDiagram**

2. 连线类型

   | Type | Description      |
   | ---- | ---------------- |
   | ->   | 无箭头实线       |
   | –>   | 无箭头虚线       |
   | -»   | 带箭头实线       |
   | –»   | 带箭头虚线       |
   | -x   | 带十字实线(异步) |
   | –x   | 带十字虚线(异步) |

3. 激活或禁用角色

   - 使用`activate/deactivate`关键字

     ```mermaid
     sequenceDiagram
     A ->> B : hello
     activate B
     B -->> A : hi
     deactivate B
     ```

   - 使用`+/-`符号

     ```mermaid
     sequenceDiagram
     A ->> +B : hello
     B -->> -A : hi
     ```

4. 循环表示

   ```
   loop text
     	statements
   end
   ```

   ```mermaid
   sequenceDiagram
   A -> B : hello
   loop every min
   	B --> A : hi
   end
   ```

5. 选择表示

   ```
   两种选择：
   alt text
   	statement
   else
   	statement
   end
   只有一种选择：
   opt text
   	statement
   end
   ```

   ```mermaid
   sequenceDiagram
   A ->> B : how are you?
   alt is sick
   	B ->> A : Not good
   else is well
   	B ->> A : I'm fine
   end
   ```

6. 平行关系表示

   ```
   par [action1]
   	statement
   and [action2]
   	statement
   end
   ```

   ```mermaid
   sequenceDiagram
   par A to B
   	A ->> B :hello
   and A to C
   	A ->> C : hello
   end
   ```

##### 四. 类图

1. 关键字

   **classDiagram**

2. 类结构组成

   由三部分组成,分别为头部的`名称`,中间的`属性`和底部的`方法`

   ```mermaid
   classDiagram
   class ClassName {
   +String arrtribute
   +method() List~String~
   }
   ```

   > 泛型使用`~`来表示

3. 元素可见性

   | 表示符号 | 描述                 |
   | -------- | -------------------- |
   | `+`      | **Public**           |
   | `-`      | **Private**          |
   | `#`      | **Protected**        |
   | `~`      | **Package/Internal** |

4. 特殊类表示

   - 种类

     | 表示符号,关键字   | 描述       |
     | ----------------- | ---------- |
     | `<<interface>>`   | **接口**   |
     | `<<abstract>>`    | **抽象类** |
     | `<<service>>`     | **服务类** |
     | `<<enumeration>>` | **枚举**   |

   - 表示方法

     1. 在外部进行修饰

        ```
        class Clazz
        <<interface>> Clazz
        ```

        ```mermaid
        classDiagram
        class Clazz
        <<interface>> Clazz
        ```

     2. 将修饰符内嵌到类内部

        ```
        class Clazz {
        <<interface>>
        }
        ```

        ```mermaid
        classDiagram
        class Clazz {
        <<interface>>
        }
        ```

5. 关系表示

   | 表示符号 | 描述           |
   | -------- | -------------- |
   | `<|-`    | **继承**       |
   | `*_`     | **组合**       |
   | `o–`     | **聚合**       |
   | `–>`     | **关联**       |
   | `-`      | **联系(实线)** |
   | `..>`    | **依赖**       |
   | `..|>`   | **实现**       |
   | `..`     | **联系(虚线)** |

6. 实例

##### 五. 状态图

1. 关键字

   **stateDiagram或stateDiagram-v2**

2. 状态表示

   ```
   第一种:直接声明
   s1
   第二种:使用state关键字声明带描述的状态
   state "state with discription" as s2
   第三种:使用冒号声明
   s3:state with colon
   特殊状态表示:start/end
   start/end都使用[*]表示,通过指向方向区分
   ```

   ```mermaid
   stateDiagram
   s1
   state "state with discription" as s2
   s3:state with colon
   [*] --> s4 : start to s4
   s4 --> [*] : s4 to end
   ```

3. 状态转换

   使用`-->`符号来表示两个状态之间的转换

   ```mermaid
   stateDiagram
   s1 --> s2
   s3 --> s4:with discription
   ```

4. 复合状态表示

   使用`state`声明状态,然后使用`{}`定义状态体,可以进行多次嵌套
   ```mermaid
   stateDiagram
   s1
   state s2 {
   	s3 --> s4
   }
   s1 --> s2
   ```
   
5. 分支表示
   使用`<<fork>>`和`<<join>>`来表示分支
   ```mermaid
   stateDiagram
      state fork_state <<fork>>
      [*] --> fork_state
      fork_state --> S2
      fork_state --> S3

      state join_state <<join>>
      S2 --> join_state
      S3 --> join_state
      join_state --> S4
      S4 --> [*]
   ```
   
6. 并发表示
   使用`-`来表示并发关系
   
   ```mermaid
   stateDiagram
   [*] --> Active
   state Active {
   [*] --> NumLockOff
   NumLockOff --> NumLockOn : EvNumLockPressed
   NumLockOn --> NumLockOff : EvNumLockPressed
   --
   [*] --> CapsLockOff
   CapsLockOff --> CapsLockOn : EvCapsLockPressed
   CapsLockOn --> CapsLockOff : EvCapsLockPressed
   }
   ```

##### 六. 实体关系图

1. 关键字

   **erDiagram**
   
2. 基本语法

   ```
   <实体> <关系> <另一实体> ：<关系描述>
   ```

3. 关系表示

   | Value (left) | Value (right) | Meaning                       |
   | :----------: | :-----------: | ----------------------------- |
   |    `\|o`     |     `o\|`     | Zero or one                   |
   |    `\|\|`    |    `\|\|`     | Exactly one                   |
   |     `}o`     |     `o{`      | Zero or more (no upper limit) |
   |    `}\|`     |     `\|{`     | One or more (no upper limit)  |

4. 实例

   ```
   erDiagram
       CUSTOMER ||--o{ ORDER : places
       ORDER ||--|{ LINE-ITEM : contains
       CUSTOMER }|..|{ DELIVERY-ADDRESS : uses
   ```

   

   ```mermaid
   erDiagram
       CUSTOMER ||--o{ ORDER : places
       ORDER ||--|{ LINE-ITEM : contains
       CUSTOMER }|..|{ DELIVERY-ADDRESS : uses
   ```

##### 七. 行程图

1. 关键字

   **journey**

2. 基本语法

   ```
   任务名称 ： 分数 ：任务对象
   ```

3. 实例

   ```
   journey
       title My working day
       section Go to work
         Make tea: 5: Me
         Go upstairs: 3: Me
         Do work: 1: Me, Cat
       section Go home
         Go downstairs: 5: Me
         Sit down: 5: Me
   ```

   

   ```mermaid
   journey
       title My working day
       section Go to work
         Make tea: 5: Me
         Go upstairs: 3: Me
         Do work: 1: Me, Cat
       section Go home
         Go downstairs: 5: Me
         Sit down: 5: Me
   ```

##### 八. 甘特图

1. 关键字

   **gantt**

2. 基本语法

   ```
   标题 title ****
   时间格式 dateFormat YYYY-MM-DD
   声明项目 section ***
   具体流程 task : date , time
   ```

3. 实例

   ```
   gantt
       title A Gantt Diagram
       dateFormat  YYYY-MM-DD
       section Section
       A task           :a1, 2014-01-01, 30d
       Another task     :after a1  , 20d
       section Another
       Task in sec      :2014-01-12  , 12d
       another task      : 24d
   ```

   

   ```mermaid
   gantt
       title A Gantt Diagram
       dateFormat  YYYY-MM-DD
       section Section
       A task           :a1, 2014-01-01, 30d
       Another task     :after a1  , 20d
       section Another
       Task in sec      :2014-01-12  , 12d
       another task      : 24d
   ```

九. 饼图

1. 关键字

   **pie**

2. 基本语法

   ```
   pie title 图名
   	子项 ： 权重
   	子项 ： 权重
   ```

3. 实例

   ```
   pie title Pets adopted by volunteers
       "Dogs" : 386
       "Cats" : 85
       "Rats" : 15
   ```

   

   ```mermaid
   pie title Pets adopted by volunteers
       "Dogs" : 386
       "Cats" : 85
       "Rats" : 15
   ```

   

