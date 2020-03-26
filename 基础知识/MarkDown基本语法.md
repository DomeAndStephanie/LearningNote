#### MarkDown基本语法

---

1. 简介

   > **Markdown**是一种轻量级标记语言，创始人为约翰·格鲁伯（英语：John Gruber）。它允许人们使用易读易写的纯文本格式编写文档，然后转换成有效的XHTML（或者HTML）文档。这种语言吸收了很多在电子邮件中已有的纯文本标记的特性。
   >
   > 由于Markdown的轻量化、易读易写特性，并且对于图片，图表、数学式都有支持，当前许多网站都广泛使用Markdown来撰写帮助文档或是用于论坛上发表消息。如GitHub、Reddit、Diaspora、Stack Exchange、OpenStreetMap 、SourceForge、简书等，甚至还能被使用来撰写电子书。
   >
   > ​                                                                                                                            ---WikiPedia

2. 基本命令

   - 标题

     ```markdown
     # 一级标题
     ## 二级标题
     ### 三级标题
     #### 四级标题
     ##### 五级标题
     ###### 六级标题
     ```

   - 字体

     ```markdown
     **加粗**
     *斜体*
     ~~删除线~~
     <u>下划线</u>
     `注释`
     ==高亮==
     ```

   - 列表

     ```markdown
     ## 有序列表
     1. 
     2. 
     3. 
     ## 无序列表
     - 
     - 
     - 
     ## 任务列表
     - [ ] 任务1
     - [ ] 任务2
     - [x] 任务3 
     ```

   - 引用

     ```markdown
     > 这是引用部分内容
     ```

   - 代码块

     ```markdown
     ​```java
     public static void main(String[] args) {
      
     }
     ​```
     ```

   - 表格

     ```markdown
     | First Header  | Second Header |
     | ------------- | ------------- |
     | Content Cell  | Content Cell  |
     | Content Cell  | Content Cell  |
     ```

   - 目录

     ```markdown
     [TOC]
     ```

   - 链接

     - 超链接

       ```markdown
       [链接名称](链接地址)
       ```

     - 图片链接

       ```markdown
       ![图片名称](图片地址)
       ```

   - 脚注

     ```markdown
     You can create footnotes like this[^footnote].
     
     [^footnote]: Here is the *text* of the **footnote**.
     ```

   - Emoji表情

     ```markdown
     :smile:
     :kissing_smiling_eyes::kissing_smiling_eyes:
     ```

   - 视频

     ```markdown
     <video src="xxx.mp4"
     ```

   - HTML

     ```markdown
      <span style="color:red">红色的文本</span>
     ```
3. 绘制结构图

   - 流程图

     ```markdown
     ## Flowchart.js
     ​```flow
     st=>start: Start
     op=>operation: Your Operation
     cond=>condition: Yes or No?
     e=>end
     
     st->op->cond
     cond(yes)->e
     cond(no)->op
     ​```
     ## Mermaid
     ​```mermaid
     graph LR
     A[Hard edge] -->B(Round edge)
         B --> C{Decision}
         C -->|One| D[Result one]
         C -->|Two| E[Result two]
     ​```
     ```

   - 时序图

     ```markdown
     ## js-sequence
     ​```sequence
     Alice->Bob: Hello Bob, how are you?
     Note right of Bob: Bob thinks
     Bob-->Alice: I am good thanks!
     ​```
     ## Mermaid
     ​```mermaid
     %% Example of sequence diagram
       sequenceDiagram
         Alice->>Bob: Hello Bob, how are you?
         alt is sick
         Bob->>Alice: Not so good :(
         else is well
         Bob->>Alice: Feeling fresh like a daisy
         end
         opt Extra response
         Bob->>Alice: Thanks for asking
         end
     ​```
     ```

   - 甘特图

     ```markdown
     ## Mermaid
     ​```mermaid
     %% Example with selection of syntaxes
             gantt
             dateFormat  YYYY-MM-DD
             title Adding GANTT diagram functionality to mermaid
     
             section A section
             Completed task            :done,    des1, 2014-01-06,2014-01-08
             Active task               :active,  des2, 2014-01-09, 3d
             Future task               :         des3, after des2, 5d
             Future task2               :         des4, after des3, 5d
     
     ​```
     ```

   - 类图

     ```markdown
     ## Mermaid
     ​```mermaid
     classDiagram
           Animal <|-- Duck
           Animal <|-- Fish
           Animal <|-- Zebra
           Animal : +int age
           Animal : +String gender
           Animal: +isMammal()
           Animal: +mate()
           class Duck{
               +String beakColor
               +swim()
               +quack()
           }
           class Fish{
               -int sizeInFeet
               -canEat()
           }
           class Zebra{
               +bool is_wild
               +run()
           }
     ​```
     ```

   - 状态图

     ```markdown
     ​```mermaid
     stateDiagram
         [*] --> Still
         Still --> [*]
     
         Still --> Moving
         Moving --> Still
         Moving --> Crash
         Crash --> [*]
     ​```
     ```

   - 饼图

     ```markdown
     ​```mermaid
     pie
         title Pie Chart
         "Dogs" : 386
         "Cats" : 85
         "Rats" : 150 
     ​```
     ```
4. 数学公式

   ```markdown
   ## 行内公式
   $ a^2 + b^2 = c^2 $
   ## 行间公式
   $$
    y = x ^ {2}
    x_{a + b} 
    x^{a + b}
   $$
   ```

   | 符号                                           | 代码                                             | 具体含义   |
   | ---------------------------------------------- | ------------------------------------------------ | ---------- |
   | $\sum$                                         | `$\sum$`                                         | 求和公式   |
   | $\sum_{i=0}^n$                                 | `$\sum_{i=0}^n$`                                 | 求和上下标 |
   | $\times$                                       | `$\times$`                                       | 乘号       |
   | $\pm$                                          | `$\pm$`                                          | 正负号     |
   | $\div$                                         | `$\div$`                                         | 除号       |
   | $\mid$                                         | `$\mid$`                                         | 竖线       |
   | $\cdot$                                        | `$\cdot$`                                        | 点         |
   | $\circ$                                        | `$\circ$`                                        | 圈         |
   | $\ast $                                        | `$\ast $`                                        | 星号       |
   | $\bigotimes$                                   | `$\bigotimes$`                                   | 克罗内克积 |
   | $\bigoplus$                                    | `$\bigoplus$`                                    | 异或       |
   | $\leq$                                         | `$\leq$`                                         | 小于等于   |
   | $\geq$                                         | `$\geq$`                                         | 大于等于   |
   | $\neq$                                         | `$\neq$`                                         | 不等于     |
   | $\approx$                                      | `$\approx$`                                      | 约等于     |
   | $\prod$                                        | `$\prod$`                                        | N元乘积    |
   | $\coprod$                                      | `$\coprod$`                                      | N元余积    |
   | $\cdots$                                       | `$\cdots$`                                       | 省略号     |
   | $\int$                                         | `$\int$`                                         | 积分       |
   | $\iint$                                        | `$\iint$`                                        | 双重积分   |
   | $\oint$                                        | `$\oint$`                                        | 曲线积分   |
   | $\infty$                                       | `$\infty$`                                       | 无穷       |
   | $\nabla$                                       | `$\nabla$`                                       | 梯度       |
   | $\because$                                     | `$\because$`                                     | 因为       |
   | $\therefore$                                   | `$\therefore$`                                   | 所以       |
   | $\forall$                                      | `$\forall$`                                      | 任意       |
   | $\exists$                                      | `$\exists$`                                      | 存在       |
   | $\not=$                                        | `$\not=$`                                        | 不等于     |
   | $\not>$                                        | `$\not>$`                                        | 不大于     |
   | $\leq$                                         | `$\leq$`                                         | 小于等于   |
   | $\geq$                                         | `$\geq$`                                         | 大于等于   |
   | $\not\subset$                                  | `$\not\subset$`                                  | 不属于     |
   | $\emptyset$                                    | `$\emptyset$`                                    | 空集       |
   | $\in$                                          | `$\in$`                                          | 属于       |
   | $\notin$                                       | `$\notin$`                                       | 不属于     |
   | $\subset$                                      | `$\subset$`                                      | 子集       |
   | $\subseteq$                                    | `$\subseteq$`                                    | 真子集     |
   | $\bigcup$                                      | `$\bigcup$`                                      | 并集       |
   | $\bigcap$                                      | `$\bigcap$`                                      | 交集       |
   | $\bigvee$                                      | `$\bigvee$`                                      | 逻辑或     |
   | $\bigwedge$                                    | `$\bigwedge$`                                    | 逻辑与     |
   | $\biguplus$                                    | `$\biguplus$`                                    | 多重集     |
   | $\bigsqcup$                                    | `$\bigsqcup$`                                    |            |
   | $\hat{y}$                                      | `$\hat{y}$`                                      | 期望值     |
   | $\check{y}$                                    | `$\check{y}$`                                    |            |
   | $\breve{y}$                                    | `$\breve{y}$`                                    |            |
   | $\overline{a+b+c+d}$                           | `$\overline{a+b+c+d}$`                           | 平均值     |
   | $\underline{a+b+c+d}$                          | `$\underline{a+b+c+d}$`                          |            |
   | $\overbrace{a+\underbrace{b+c}_{1.0}+d}^{2.0}$ | `$\overbrace{a+\underbrace{b+c}_{1.0}+d}^{2.0}$` |            |
   | $\uparrow$                                     | `$\uparrow$`                                     | 向上       |
   | $\downarrow$                                   | `$\downarrow$`                                   | 向下       |
   | $\Uparrow$                                     | `$\Uparrow$`                                     |            |
   | $\Downarrow$                                   | `$\Downarrow$`                                   |            |
   | $\rightarrow$                                  | `$\rightarrow$`                                  | 向右       |
   | $\leftarrow$                                   | `$\leftarrow$`                                   | 向左       |
   | $\Rightarrow$                                  | `$\Rightarrow$`                                  | 向右箭头   |
   | $\Longleftarrow$                               | `$\Longleftarrow$`                               | 向左长箭头 |
   | $\longleftarrow$                               | `$\longleftarrow$`                               | 向左单箭头 |
   | $\longrightarrow$                              | `$\longrightarrow$`                              | 向右长箭头 |
   | $\Longrightarrow$                              | `$\Longrightarrow$`                              | 向右箭头   |
   | $\alpha$                                       | `$\alpha$`                                       |            |
   | $\beta$                                        | `$\beta$`                                        |            |
   | $\gamma$                                       | `$\gamma$`                                       |            |
   | $\Gamma$                                       | `$\Gamma$`                                       |            |
   | $\delta$                                       | `$\delta$`                                       |            |
   | $\Delta$                                       | `$\Delta$`                                       |            |
   | $\epsilon$                                     | `$\epsilon$`                                     |            |
   | $\varepsilon$                                  | `$\varepsilon$`                                  |            |
   | $\zeta$                                        | `$\zeta$`                                        |            |
   | $\eta$                                         | `$\eta$`                                         |            |
   | $\theta$                                       | `$\theta$`                                       |            |
   | $\Theta$                                       | `$\Theta$`                                       |            |
   | *ϑ*                                            | `$\vartheta$`                                    |            |
   | *ι*                                            | `$\iota$`                                        |            |
   | *π*                                            | `$\pi$`                                          |            |
   | *ϕ*                                            | `$\phi$`                                         |            |
   | Φ                                              | `$\Phi$`                                         |            |
   | *ψ*                                            | `$\psi$`                                         |            |
   | Ψ                                              | `$\Psi$`                                         |            |
   | *ω*                                            | `$\omega$`                                       |            |
   | Ω                                              | `$\Omega$`                                       |            |
   | *χ*                                            | `\chi`                                           |            |
   | *ρ*                                            | `$\rho$`                                         |            |
   | *ο*                                            | `$\omicron$`                                     |            |
   | *σ*                                            | `$\sigma$`                                       |            |
   | Σ                                              | `$\Sigma$`                                       |            |
   | *ν*                                            | `$\nu$`                                          |            |
   | *ξ*                                            | `$\xi$`                                          |            |
   | *τ*                                            | `$\tau$`                                         |            |
   | *λ*                                            | `$\lambda$`                                      |            |
   | Λ                                              | `$\Lambda$`                                      |            |
   | *μ*                                            | `\mu`                                            |            |
   | ∂                                              | `$\partial$`                                     |            |
   | {}                                             | `$\lbrace \rbrace$`                              |            |
   | *a*                                            | `$\overline{a}$`                                 |            |


​     

   