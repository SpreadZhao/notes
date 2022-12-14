# 编译原理笔记

## 1. 基本概念

### 1.1 字母表

**字母表**$\Sigma$是一个有穷符号集合。比如：

* 二进制字母表：{0, 1}
* ASCII字符集
* Unicode字符集

字母表的运算

* 乘积：$\Sigma$~1~$\Sigma$~2~ = {ab | a $\in$ $\Sigma$~1~, b $\in$ $\Sigma$~2~}

  比如：{0, 1} {a, b} = {0a, 0b, 1a, 1b}

* n次幂：

  * $\Sigma$^0^ = { $\epsilon$ }
  * $\Sigma$^n^ = $\Sigma$^n-1^$\Sigma$, n $\geq$ 1

  比如：

  {0, 1}^3^ = {0, 1} {0, 1} {0, 1} = {000, 001, 010, 011, 100, 101, 110, 111}

  我们也能发现，几个字母表相乘，结果的每一项就是几个元素，比如这里三个相乘，结果中每一个元素比如000，001都是三个数

  **我们还能发现，字母表的n次幂，就是长度为n的符号串构成的集合。比如这里n = 3，那结果集合中每一个元素长度都是3。而0次幂就是长度为0的符号串，那就是空串，用$\epsilon$表示**

* 正闭包：$\Sigma$^+^ = $\Sigma$ $\cup$ $\Sigma$^2^ $\cup$ $\Sigma$^3^ $\cup$ ...

  比如{a, b, c, d}^+^ = {a, b, c, d, aa, ab, ac, ad, ba, bb, bc, bd, ... , aaa, aab, aac, aad, ...} 

* 克林闭包：$\Sigma$^*^ = $\Sigma$^+^ $\cup$ $\Sigma$^0^ = $\Sigma$^0^ $\cup$ $\Sigma$ $\cup$ $\Sigma$^2^ $\cup$ $\Sigma$^3^ $\cup$ ...

  比如{a, b, c, d}^*^ = {$\epsilon$, a, b, c, d, aa, ab, ac, ad, ba, bb, bc, bd, ... , aaa, aab, aac, aad, ...} 

  由此可见，克林闭包是任意符号串构成的集合，并且这个串的长度可以是0
  
  *由克林闭包，我们能推出**串**的概念：克林闭包中的任何一个元素都是字母表$\Sigma$上的一个**串***
  
  > 比如字母表$\Sigma$ = {a, b, c, d}，那么$\epsilon$, a, b, c, d, aa, ab, ac, ad, ba, bb, bc, bd等都是$\Sigma$的串，比如让串s = ab, 那么s的长度|s| = 2，**也就是串中符号的个数**。而**|$\epsilon$| = 0**

### 1.2 串的运算

**连接**

> x和y的连接运算记作**xy**
>
> x = dog, y = house, 那么xy = doghouse
>
> 空串是连接运算的**单位元**，相当于不动，对于任意串s有
>
> $\epsilon$s = s$\epsilon$ = s
>
> 设x, y, z是三个串，x = yz，则称y是x的**前缀**，z是x的**后缀**

**幂**

> 若s = ba，那么：
>
> s^2^ = baba, s^3^ = bababa, ...
>
> 特别地：s^0^ = $\epsilon$

## 2. 文法分析

### 2.1 Terminal Symbol

终结符：是文法所定义的语言的基本符号。有时也称为**Token**

比如：V~T~ = {apple, boy, cat, little}是一个终结符集，其中的元素是终结符，在这里是**单词**

理解终结的含义：单词在这里再继续推导没有意义，也就是字母它本身也不在里面，所以不用继续推导，达到终点

***重点：$\epsilon$不是Nonterminal，也不是Terminal！！！***

### 2.2 Nonterminal Symbol

非终结符：用来表示**语法成分**的符号，有时也称为**语法变量**

比如：V~N~ = {<句子>, <名词短语>, <动词短语>, <名词>, ...}是一个非终结符集

> *注意：对于同一个文法形式(文法形式是什么稍后再说)，终结符集和非终结符集是没有交集的，即：V~T~ $\cap$ V~N~ = $\Phi$*
>
> 另外，V~T~ $\cup$ V~N~ = <**文法符号集**>

非终结的意思就是能继续推导，也就是这个东西能够继续细分成更细的东西

### 2.3 Production

产生式：描述了将**Terminal**和**Nonterminal**组合成**串**的**方法**，也就是用来**产生串的式子**。一般形式：$\alpha$ -> $\beta$ ($\alpha$定义为$\beta$)

比如在上面的例子中，可以有以下产生式：

P = {<句子> $\rightarrow$ <名词短语><动词短语>, <名词短语> $\rightarrow$ <形容词><名词短语>, ...}

### 2.4 Start Symbol

开始符号：文法中最大的语法成分：比如上面的例子，最大的语法成分就是**S = <句子>**

### 2.5 Grammar

就是上面四项组成的集合：

G = (V~T~, V~N~, P, S)

> 例：G = ( { **id**, **+**, *****, **(**, **)** }, {**E**}, **P**, **E** )
>
> 那么:
>
> * V~T~ = {id, +, *, (, )}
> * V~N~ = {E}
> * P = P
> * S = E
>
> *注：其中E就是Expression表达式，非终结符只有表达式一种*
>
> 接下来讨论P到底是什么：因为V~N~只有E一个，所以，只需要定义E即可
>
> P = {E -> E + E, E -> E *  E, E -> (E), E -> id}
>
> 也就是：表达式和表达式加起来还是表达式；表达式和表达式乘起来还是表达式；把表达式用括号括起来还是表达式；每一个表达式都可以赋给它一个id
>
> 那既然有这么多种表达E的方法，太多了，简化以下：也就是说，**把表达式加起来，乘起来，括起来，给个id，得到的都是表达式**，那么可以用**或(|)**的方式来简写：
>
> E -> E + E | E * E | (E) | id
>
> 那么对于任意一个产生式P = {$\alpha$ -> $\beta$~1~, $\alpha$ -> $\beta$~2~, ... , $\alpha$ -> $\beta$~n~}，都可以简化为：
>
> P = $\alpha$ -> $\beta$~1~ | $\beta$~2~ | ... | $\beta$~n~

补充：

* $\alpha$ $\in$ (V~T~ $\cup$ V~N~)^+^，并且$\alpha$中至少得有一个V~N~的元素，称为产生式的**头部**或者**左部**
* $\beta$ $\in$ (V~T~ $\cup$ V~N~)^+^，称为产生式的**体**或者**右部**

### 2.6 Symbolic Convention

上面说了那么多Terminal和Nonterminal，怎么区分他们呢？

**以下是Terminal(终结符)：**

* 字母表中**排在前面的小写字母**，如a, b, c
* **运算符**，如+, *
* **标点符号**，如逗号，括号等
* **数字**，0, 1, ... 9
* 粗体字符串，如id, if等

**以下是Nonterminal(非终结符)：**

* 字母表中**排在前面的大写字母**，如A, B, C
* 字母S，因为它通常表示开始符号
* **小写，斜体的名字**，比如expr, stmt(statement)等
* **代表程序构造的大写字母**。比如E(表达式)、T(term，项)和F(factor，因子)

**其他的一些东西：**

* 字母表中排在**后面**的**大**写字母，比如X, Y, Z表示**文法符号**，既可以是Terminal也可以是Nonterminal
* 字母表中排在**后面**的**小**写字母，比如u, v, ... , z表示**终结符号串**(包括空串)
* 小写希腊字母，比如$\alpha$, $\beta$, $\gamma$，表示**文法符号串**(包括空串)
* **除非有特别说明，第一个产生式的左部就是开始符号**

### 2.7 Language

*给你一个句子：`little boy eats apple`，咋判断它符不符合语法？*

#### 2.7.1 Derivation & Reduction

比如一个**文法**中的**Production**是这样的(这里就不写成集合了)：

* <句子> $\rightarrow$ <名词短语><动词短语>
* <名词短语> $\rightarrow$ <形容词><名词短语>
* <名词短语> $\rightarrow$ <名词>
* <动词短语> $\rightarrow$ <动词><名词短语>
* <形容词> $\rightarrow$ little
* <名词> $\rightarrow$ boy | apple
* <动词> $\rightarrow$ eat

*问题：这里对于$\alpha$的概念：是不是$\alpha$表示的是左边所有东西构成的集合？还是任何一个比如<动词短语>都可以叫做$\alpha$变量？如果是前者的话，那为什么要用$\in$而不是$\subseteq$?*

> 那既然<句子>可以定义为<名词短语>和<动词短语>拼一起，那么就可以这么写：
>
> <句子> $\Rightarrow$ <名词短语><动词短语>
>
> 而<名词短语>又可以是<形容词>和名词短语拼一起，那么又可以这么写：
>
> <句子> $\Rightarrow$ <名词短语><动词短语>
>
> ​			$\Rightarrow$ <形容词><名词短语><动词短语>
>
> 对应上面"little boy eats apple"，一步步展开，就能得到：
>
> **<句子> $\Rightarrow$ <名词短语><动词短语>**
>
> ​			**$\Rightarrow$ <形容词><名词短语><动词短语>**
>
> ​			**$\Rightarrow$ <形容词><名词><动词><名词>**
>
> ​			**$\Rightarrow$ little boy eats apple**
>
> 以上过程，就是Derive(推导)的过程；而反着来，就是Reduce(规约)的过程

那么开头的问题就好办了，只要满足Derive或者Reduce的过程，那么就是一个符合语法的句子

补充：

* 如果$\alpha$~0~ $\Rightarrow$ $\alpha$~1~ $\Rightarrow$ $\alpha$~2~ $\Rightarrow$ ... $\Rightarrow$ $\alpha$~n~，则称串$\alpha$~0~经过n步推导出$\alpha$~n~，可以简记为：$\alpha$~0~ $\Rightarrow$^n^ $\alpha$~n~
* $\Rightarrow$^0^表示0步推导，也就是不推导
* $\Rightarrow$^+^表示经过正数步推导(>0)
* $\Rightarrow$^*^表示经过若干步推导($\geq$0)

#### 2.7.2 Sentential form & Sentence

在上面的例子中：

**<句子> $\Rightarrow$ <名词短语><动词短语>**

​			**$\Rightarrow$ <形容词><名词短语><动词短语>**

​			**$\Rightarrow$ <形容词><名词><动词><名词>**

​			**$\Rightarrow$ little boy eats apple**

或者这么写：

**<句子> $\Rightarrow$ <名词短语><动词短语>**

​			**$\Rightarrow$ <形容词><名词短语><动词短语>**

​			**$\Rightarrow$ little <名词短语><动词短语>**

​			**$\Rightarrow$ little <名词><动词短语>**

​			**$\Rightarrow$ little boy <动词短语>**

​			**$\Rightarrow$ little boy <动词><名词短语>**

​			**$\Rightarrow$ little boy eats <名词短语>**

​			**$\Rightarrow$ little boy eats <名词>**

​			**$\Rightarrow$ little boy eats apple**

整个所有的过程都叫做**Sentential form(句型)**，而只有最后一句`little boy eats apple`是**Sentence(句子)**

我们能从中发现一个特点：**句子中没有Nonterminal，全都是Terminal**

那么给出句型和句子的定义

* 如果S经过若干步推导得到了$\alpha$**(S $\Rightarrow$^*^ $\alpha$)**，并且$\alpha$ $\in$ (V~T~ $\cup$ V~N~)^*^，则称$\alpha$是G的一个**句型**。句型中包含Terminal Symbol和Nonterminal Symbol，还可能是空串$\epsilon$
* 如果S $\Rightarrow$^*^ $\omega$，并且$\omega$ $\in$ (V~T~ $\cup$ V~N~)^*^，则称$\omega$是G的一个句子。**句子**是不包含Nonterminal Symbol的句型

#### 2.7.3 Language

由文法G的Start Symbol**推导出的所有<u>句子</u>**构成的集合称为文法**G生成的语言**，记为L(G)

L(G) = {$\omega$ | S $\Rightarrow$^*^ $\omega$, $\omega$ $\in$ V~T~^*^} (*问题：这里V~T~是否加上星号存疑*)

> 例：文法G的Production如下：
>
> 1. S -> L | LT
> 2. T -> L | D | TL | TD
> 3. L -> a | b | c | ... | z
> 4. D -> 0 | 1 | ... | 9
>
> 该文法生成的语言是？
>
> 首先，通过观察可以得到：D表示的是数字Digit，L表示的是字母Letter。然后看S，因为**语言必须由S来推出**。S推出的东西必须由L也就是字母打头，然后后面是个T。而T可以是L, D, TL, TD中任何一种，那么就可以不断套娃进行代换：T $\Rightarrow$ TL $\Rightarrow$ TDL $\Rightarrow$ TDDL $\Rightarrow$ TLDDL $\Rightarrow$ ... $\Rightarrow$ DDDDD...LDDL(显然有超多代换方式，这里只是一种)。然后再和前面那个L拼一起。
>
> 那么可以得到结论：推出来的东西是**字母开头**，**后面有字母有数字**。很显然和变量名很像啊！
>
> 答案就是：标识符(identifier)

**语言上的运算**

| 运算                   | 定义和表示                                      |
| ---------------------- | ----------------------------------------------- |
| L和M的并(两种语言的并) | $L \cup M = \{s | s \in L || s \in M\}$         |
| L和M的连接             | LM = { st \| s $\in$ L && t $\in$ M }           |
| L的幂                  | L^0^ = {$\epsilon$}; L^n^ = L^n-1^L, n $\geq$ 1 |
| L的Kleene闭包          | L^*^ = $\cup^\infty_{i=0}L^i$                   |
| L的正闭包              | L^+^ = $\cup^\infty_{i=1}L^i$                   |

> 例：令L = {A, B, ..., Z, a, b, ..., z}，D = {0, 1, ..., 9}则L(L $\cup$ D)^*^表示的语言是？
>
> 语言也和串一样可以连接，**语言也是句子的集合**，因此**本题中默认A, B等都是句子**，D也是语言，0, ..., 9也都是句子。而L(L $\cup$ D)^*^表示一个字母开头，后面连接上L并上D的克林闭包，那么就是说是由字母和数字组成的一堆东西，然后还是字母开头，那表示的也是**标识符**

### 2.8 Grammatical Classification

#### 2.8.1 Type-0 Grammar: Unrestricted Grammar

比如前面说的那些Production，只要左边(左部)**至少有一个**Nonterminal，那就是0型文法

#### 2.8.2 Type-1 Grammar: Context-Sensitive Grammar(CSG)

Production的一般形式：$\alpha_1A\alpha_2\rightarrow\alpha_1\beta\alpha_2(\beta\neq\epsilon)$

从这里能看出定义的过程和上下文$\alpha_1, \alpha_2$有关

既然$\beta\neq\epsilon$，那么CSG中**不能**包含空产生式，也就是右部是空串。而之前说过，**左部至少要有一个Nonterminal**，那么左部的长度至少是1，那一个**长度是1或者以上的东西怎么可能由空串来定义呢**？显然右部的长度要$\geq$左部$\geq$1

#### 2.8.3 Type-2 Grammar: Context-Free Grammar(CFG)

现在定义：**A表示非终结符**

二型文法中规定：左部必须是一个Nonterminal。那很容易得到一般形式：$A\rightarrow\beta$。2型和0型的区别就是2型左部全是Nonterminal，0型有一个就行

比如之前的那个例子：

> 1. S -> L | LT
> 2. T -> L | D | TL | TD
> 3. L -> a | b | c | ... | z
> 4. D -> 0 | 1 | ... | 9

左边全是Nonterminal，那就是一个2型文法。**由上下文无关语法生成的语言叫做上下文无关语言**

#### 2.8.4 Type-3 Grammar: Regular Grammar(RG)

还是定义A为Nonterminal，现在B也是个Nonterminal，w是Terminal，RG分为两种：

##### 2.8.4.1 Right Linear

* $A \rightarrow wB$
* $A \rightarrow w$

也就是说，可以定义一步到位；也可以在右边挂个B，不停套娃

##### 2.8.4.2 Left Linear

* $A \rightarrow Bw$
* $A \rightarrow w$

显然差不多，就是在左边套娃

> 例：
>
> 1. S -> a | b | c | d
> 2. S -> aT | bT | cT | dT
> 3. T -> a | b | c | d | 0 | 1 | 2 | 3 | 4 | 5
> 4. T -> aT | bT | cT | dT | 0T | 1T | 2T | 3T | 4T | 5T
>
> 首先，**左部都是Nonterminal**，所以先是一个2型文法；然后右部中，要么是字母、数字这些Terminal，要么在Terminal的右边放个T，也就是Nonterminal，所以就是**Right Linear**
>
> 然后它生成的是什么语言呢？首先看T是啥：T要么是单纯的a ~ d, 0 ~ 5，要么是这些后面再套娃上T，所以T就是一个任意长度的字母和数字组成的串；然后再看S：S要么是单纯的abcd，要么是abcd打头，后面跟上个T，所以这个生成的还是**标识符**，只不过它只能由0 ~ 5和a ~ d构成

### 2.9 Grammatical Relationships

**0型**包含**1型**包含**2型**包含**3型**

### 2.10 CFG Tree

首先CFG：就是一堆$A \rightarrow \beta$

那这个树就是用来分析CFG的。比如：

> G:												    
>1. E -> E + E
> 2. E -> E * E
> 3. E -> -E
> 4. E -> (E)
> 5. E -> id

![img](img/cfgt.png)

* 这棵树的根节点就是Start Symbol
* 从根节点出发，是E。然后它的子节点是-和E，正好构成一个Production，父节点是左部，所有子节点从左到右构成右部
* 而这里叶节点中有E也有-+()，证明**叶节点既可以是Terminal也可以是Nonterminal**。从左到右排列叶节点得到的符号称为这棵树的产出(yield)或者边缘(frontier)

这树有啥用呢？我们来推导一下：

$E \Rightarrow -E \Rightarrow -(E) \Rightarrow -(E + E) \Rightarrow -(id + E) \Rightarrow -(id + id)$

那么根据分析的过程，我们能看到：

* 一开始只有一个E，然后推导第一步出来-E，那么树就是E连出一个-和一个E。到此为止，树的边缘是-E，**正好就是我们当前推出来的<u>句型</u>**
* 然后连出来的这个E又被代换称(E)，所以这个E又连出来三个：(, E, )，这时树的边缘是-(E)，**还是我们当前推出来的句型**
* ...

![img](img/cfgt2.png)

这样就能发现，推导的过程中，**每次得到的句型，都是在构造分析树的同时树的边缘**

#### 2.10.1 Phrase

在以下的分析树中，有这么几个短语：

![img](img/cfgt.png)

* -(E + E) -> 这是整棵树的边缘
* (E + E) -> 这是拿掉根节点后的**子树**的边缘
* E + E -> 这是再拿掉子树的根节点后的子树的边缘

能发现：短语就是分析树中**每一棵子树的边缘**

而这里E + E比较特殊：它的树只有父子两代结点，因为除了E + E和他们仨的父节点E，剩下的都被拿掉了。所以这样的短语叫做**直接短语(immediate phrase)**

> 例：
>
> ![img](img/cfgt3.png)
>
> 这个文法的分析树：
>
> ![img](img/cfgt4.png)
>
> 直接短语：提高，人民，生活，水平
>
> 而短语可以是：水平，生活水平，生活，人民，人民生活水平..................
>
> 但是文法中还有这些：高人，民生，活水。它们是产生式的右部，但却不是当前分析树的直接短语。因此我们能发现：**直接短语一定是产生式的右部，但是产生式的右部不一定是分析树的直接短语**。因为这个文法用不同的推导方式还能生成其他分析树，这些右部可能是其他分析树的直接短语

#### 2.10.2 Ambiguous Grammar

在写面向对象编程的时候，隐式转换通常会导致**二义性**(<u>可以这么整，也可以那么整，那咋整更好呢？我编译器也不知道，给你报个错吧！</u>)问题。这里来解释一下：

现在有一个文法：

> **S** -> if **E** then **S**
>
> ​	  | if **E** then **S** else **S**
>
> ​	  | **other**

那么给下面一个句型来构造分析树：`if E1 then if E2 then S1 else S2`

* 首先识别一下这个句子符合S的定义中的那种：
* `if E1 then ...`正好符合if **E** then **S**的格式，那就这么来！
* 那这个套娃下来的S就是`if E2 then S1 else S2`
* 而`if E2 then S1 else S2`正好符合if **E** then **S** else **S**的格式，那就构造完了！

![img](img/cfgt5.png)

但是，还有另一种构造的方式：

* `if E1 then ... else ...`正好符合if **E** then **S** else **S**
* 那E就是E1，第一个S是`if E2 then S1`，第二个S是`S2`
* `if E2 then S1`正好符合if **E** then **S**，这样也能正确构造！

![img](img/cfgt6.png)

注：这里的**other**就是E1, E2, S1, S2等其他语句，没有if else这些的非条件东西

分析一下产生二义性的原因：`if E1 then if E2 then S1 else S2`

因为后面的else既可以和第一个if匹配，也可以和第二个if匹配，所以会产生二义性。而如果**规定else只能和最近的前一个if匹配**，就能够让第二种构造方法无效了。

## 3. 词法分析

### 3.1 Regular Expression

有如下一个句子：

$L=\{a\}\{a,b\}^*(\{\epsilon\}\cup(\{.,\_\}\{a,b\}\{a,b\}^*))$

注：这里要结合一下2.7.3中语言的并的运算

这个L可以解释为：a开头，然后连上任意长度的由a、b组成的串(可以是空串，因为是克林闭包)；然后再连上一个并集：要么是$\epsilon$空串，要么是连接上这个：点或者下划线、a或者b、a和b组成的任意长度串

那么这么写起来比较复杂，怎么简化呢？用的就是**正则表达式**(Regular Expression)：

$r=a(a|b)^*(\epsilon|(.|\_)(a|b)(a|b)^*)$

**每个正则表达式r可以表示一个语言，记为L(r)**

那么正则表达式怎么写呢？**由小到大！**，也就是由一个很小很小很简单的正则表达式，不断扩大变成大的正则表达式，最后变大到我们需要的为止

* 最小的当然是空串，如果$\epsilon$是一个RE，那么$L(\epsilon)=\{\epsilon\}$
* 然后是一个字母的，如果$a\in\Sigma$，则a是一个RE，$L(a)=\{a\}$
* 如果是俩呢？假设r和s都是RE，表示的语言分别是L(r)和L(s)，那么
  * r|s是一个RE，$L(r|s)=L(r)\cup L(s)$
  * rs是一个RE，$L(rs)=L(r)L(s)$ **(注意：这里不是交集！是连接！！！)**
  * r^*^是一个RE，L(r^*^) = (L(r))^*^
  * (r)是一个RE，L((r)) = L(r)

> 运算的优先级：克林闭包(^*^) > 连接 > 或(|)，括号还是最牛b

> 例：
>
> 1. 令$\Sigma = \{a,b\}$，求L(a|b)，L((a|b)(a|b))，L(a^*^)，L((a|b)^*^)，L(a|a^*^b)
>
>    因为a和b都是字母，那a和b自然都是一个正则表达式，表示的语言分别是L(a) = {a}，L(b) = {b}
>
>    接下来由小变大，就可以由这些规则开始了
>
>    * $L(a|b) = L(a) \cup L(b) = \{a\} \cup \{b\} = \{a,b\}$
>
>    * $L((a|b)(a|b)) = L(a|b)L(a|b) = \{a,b\}\{a,b\} = \{aa,ab,ba,bb\}$
>
>      *注意：这里是用的上面第三条中的第二条连接的规则*
>
>    * $L(a^*) = (L(a))^* = \{a\}^* = \{\epsilon,a,aa,aaa,...\}$
>
>    * $L((a|b)^*) = (L(a|b))^* = \{a,b\}^* = \{\epsilon,a,b,aa,ab,ba,bb,aaa,...\}$
>
>    * $L(a|a^*b) = L(a) \cup L(a^*b) = L(a) \cup L(a^*)L(b) = \{a\} \cup \{\epsilon,a,aa,aaa,...\}\{b\} = \{a,b,ab,aab,...\}$
>
> 2. 十进制整数的RE
>
>    (1|...|9)(0|...|9)^*^|0
>
>    *这里最后的0是表示整个就是个0*
>
> 3. 八进制整数的RE(C语言)
>
>    0(1|2|3|...|7)(0|1|2|...|7)^*^
>
> 4. 十六进制整数的RE(C语言)
>
>    0x(1|2|...|f)(0|1|...|f)^*^
>
>    *大写字母这里不写了先*

**正则表达式的运算定律**

| 定律                         | 描述 |
| ---------------------------- | ---- |
| $r|s=s|r$                    | 交换律 |
| $r|(s|t)=(r|s)|t$            | 或结合律 |
| $r(s|t)=rs|rt; (s|t)r=sr|tr$ | 分配律 |
| $\epsilon r=r\epsilon=r$     | 乘1 |
| $r^*=(r|\epsilon)^*$         	| 闭包中一定含$\epsilon$(没啥用) |
| $r^{**}=r^*$ | 呃呃 |
| $r(st)=(rs)t$ | 连接结合律 |

### 3.2 Regular Definition

给一些常用的正则表达式起个名字，比如：

* digit -> 0|1|2|...|9
* letter_ -> A|B|...|Z|a|b|...|z|_

那么这样起个名字，就很容易表示标识符了：

identifier -> letter_(letter\_|digit)^*^

> 例：
>
> * digit -> 0|1|2|...|9
>
> * digits -> digit digit^*^
>
> * optionalFraction(可选小数部分) -> .digits|$\epsilon$
>
> * optionalExponent(可选指数部分) -> (E(+|-|$\epsilon$)digits)|$\epsilon$
>
> * number -> digits optionalFraction optionalExponent
>
>   比如2.15E+3，2就是digits；.15是optionalF；E+3是optionalE
>
> *之所以是可选的，是因为最后的那个|$\epsilon$，最后这个number表达的就是所有整形或者浮点型的无符号整数*

### 3.3 Finite Automata

![img](img/fa.png)

* start表示开始，它指向的东西代表**初始状态**，也就是0
* 0，1，2，3每一个**结点**都表示FA的一个**状态**
* 最后3上有两个圈，表示3是**终止状态**，也叫**接收状态**
* **初始状态只有一个，但是终止状态可以有多个**
* 比如状态0，有**输入**a，可以变成状态0和状态1；输入b，可以变成状态0

那对于这个有穷自动机，如果给定串`abbaabb`的话：

它的状态可以是：

```mermaid
graph LR;
	start[0_start]--a-->state1[0]--b-->state2[0]--b-->state3[0]--a-->state4{0}--a-->state5[1]--b-->state6[2]--b-->state7[3]
```

菱形之前的过程一直在自旋，之后遇到`abb`才向下走。用这种走法正好能**从初始状态达到终止状态**。那么能满足这样条件的一个串就叫做**能被该FA接收**的串

那么这个FA肯定不会只能接收这一个串，还有好多串。看一下就能明白：**只要是以`abb`结尾，并且只由a和b构成的串**，就能被这个FA接收。那么这些串构成的集合也是一个语言。记作L(M)，M表示该FA

接下来看另一个FA：

![img](img/fa2.png)

如果我输入的串是：`<=`，那么会发生什么？

* 首先，从0开始，检测到`<`，到达状态1
* 然后，**发现状态1是一个终止状态，那么还继续匹配吗？这个串是不是就直接被接收了？**
* 不会的，还会继续进行检测，`=`也匹配，到达状态2，这时候才是终止状态

也就是说，`<`和`<=`都是串`<=`的**前缀**，而这两个前缀都能匹配上这个FA的模式，那么我们**总是选择最长的这个前缀进行匹配**，这也叫做**Longest String Matching Principle**。在到达某个终止状态之后，只要输入带上还有符号，FA就会继续前进，以便寻找**尽可能长**的匹配

### 3.4 DFA & NFA

#### 3.4.1 Deterministic Finite Automata

看一个确定的有穷自动机：

![img](img/dfa.png)

* 这里面一共有四个状态：0, 1, 2, 3，它们构成了**状态集合S**

* 能输入的符号有a, b，它们构成了**字母表集合$\Sigma$**

* 然后我们画一张表格：

  | 状态\输入   | a    | b    |
  | ----------- | ---- | ---- |
  | 0           | 1    | 0    |
  | 1           | 1    | 2    |
  | 2           | 1    | 3    |
  | **3(终态)** | 1    | 0    |

  这个表格表示：状态0经过输入a会变成状态1，经过输入b会变成状态0；状态1经过输入a会变成状态1，经过输入b会变成状态2.........

  我们能发现：**即使不给图，只根据这个表，我们也能把转换图画出来。**也就是说，这个表格和转换图是等价的，这个表(**转换表**)也描述了一个**函数**，参数是当前状态和输入，返回值是下一个状态。这个函数记为$\delta$

* 这个DFA的开始状态是0，记为s~0~，s~0~$\in$S

* 这个DFA只有一个终止状态(接收状态)，记为F，F$\subseteq$S(能相等是因为有可能所有状态都是终止状态)

由以上我们可以总结：**一个DFA记为M，可以表示成：$M=(S,\Sigma,\delta,s_0,F)$**

#### 3.4.2 Nondeterministic Finite Automata

看一个NFA，其实和DFA就一点区别

![img](img/nfa.png)

比如状态0，此时检测到输入a后，**既能到状态0，也能到状态1**，因此是**不确定**的。

那么和上面的区别就是转换表的画法有不同：

| 状态\输入 | a      | b    |
| --------- | ------ | ---- |
| 0         | {0, 1} | {0}  |
| 1         | $\O$   | {2}  |
| 2         | $\O$   | {3}  |
| **3**     | $\O$   | $\O$ |

注意里面的大括号：因为这是NFA，不像DFA一样到达的状态就确定是一个。因此哪怕NFA到达的状态真就一个，也得加上大括号表示一个集合，只不过集合里就一种状态罢了

#### 3.4.3 Their Relation

对于这俩DFA和NFA：

<img src="img/dfa.png" alt="img" style="zoom:67%;" />|<img src="img/nfa.png" alt="img" style="zoom:67%;" />

NFA在3.3中分析过一模一样的，我们来分析一下DFA，对于任意一个由a，b构成的串，**只要不是以`abb`结尾，那就不能被这个DFA接收。**由此我们能发现，这两个东西是**等价的**。其实，对于任何一个NFA，都存在能和它识别同一个语言的DFA；对于任何一个DFA，也存在和它识别同一个语言的NFA。

补充：对于这个DFA，

* 状态1：串以a结尾
* 状态2：串以ab结尾
* 状态3：串以abb结尾

配合转换表来看会更加清晰

那么，以`abb`结尾的串的正则表达式是什么？来构建一下：

$\{a,b\}^*$是任意长度的ab串，也包括空串，那也就是说是(a|b)^*^，然后再拼上结尾abb，就是**(a|b)^*^abb**

其实，**正则表达式和正则文法还有FA都是等价的，可以互相构建**：正则文法$\Leftrightarrow$正则表达式$\Leftrightarrow$FA

#### 3.4.4 Other Things

**带有$\epsilon$边的NFA**

![img](img/nfa2.png)

也就是说，状态A不需要任何输入就能到达状态B，但是由接收0改为接收1

那么它接收的串的正则表达式？是由若干个0连接上若干个1连接上若干个2，即：$r=0^*1^*2^*$

那么，用不带空边的NFA是否也能表示这个RE呢？能！

![img](img/nfa3.png)

这里要注意：**三个状态都是终态**，如果在状态A结束表示若干个0；在状态B结束表示若干个0加上若干个1，因为一但跳到状态B，就不再接收0了；在状态C结束表示若干个0连上若干个1连上若干个2

*问题：线上写的是0,1；1,2；0,1,2，为啥要写3个数呢？只写上1；2；2不是也可以吗？可能是因为这里要构造NFA，所以硬加上去的？*

来看一下实现DFA的算法，DFA比NFA的实现要简单

```c
s = s0;					    // 将初始状态付给s
c = nextChar();				// c表示当前的输入符号，nextChar返回输入串x的下一个符号
while(c != EOF){			// 当当前输入不是EOF时循环
    s = move(s, c);			// 从s出发，沿着输入是c的边到达的状态，返回给s
    c = nextChar();			// c继续变成下一个符号
}
if(s in F) return TURE;		// 如果最后s是终态(F)的话，就返回"接收"
else return FALSE;			// 不是则不接收
```

![img](img/dfa.png)

比如，s一开始是s0(也就是状态0)，然后输入串`abbaabb`，则c是a，然后a不是EOF，则执行move，从状态0出发，沿着a走，到达状态1，将状态1返回给s，然后c变成了b；b不是EOF，执行move，从状态1开始沿着b走，到达状态2，将状态2返回给s，c变成了b；b不是EOF，执行move，从状态2开始沿着b走，到达状态3........

最后循环结束后，s会变成状态3。因为状态3是终态，返回TRUE

### 3.5 RE -> FA

由正则表达式直接构造DFA是比较难的，因此通常是：

```mermaid
graph LR;
	RE-->NFA-->DFA
	RE-.->DFA
```

#### 3.5.1 RE -> NFA

1. 还是从小到大来，单独对一个空串$\epsilon$，它的NFA是

![img](img/snfa.png)

2. 然后是字母表，字母表中的每一个字母，都是一个正则表达式，它对应的NFA是

![img](img/anfa.png)

3. 然后是俩表达式连到一起，对于r = r~1~r~2~，它对应的NFA是

![img](img/rrnfa.png)

4. 然后是两个表达式或运算，对于r = r~1~|r~2~，它对应的NFA是

![img](img/rnfa.png)

5. 最后是克林闭包，对于r = r~1~^*^，它的NFA是

![img](img/knfa.png)

由这些基本规则，我们就可以由正则表达式来构建NFA了。比如：r = (a|b)^*^abb

一开始，给一个初始状态和终止状态，然后把表达式写上去：

![img](img/z1.png)

很显然，这个表达式可以拆成连接的形式，根据第3个规则，就可以：

![img](img/z2.png)

然后，第一个是个克林闭包，也就是把**两个点合成一个点，然后自旋**

![img](img/z3.png)

最后把这个或也拆开，就变成了俩自旋

![img](img/z4.png)

#### 3.5.2 NFA -> DFA

比如给一个NFA：

![img](img/nd1.png)

画出它的转换表：

| 状态\输入 | a      | b      | c      |
| --------- | ------ | ------ | ------ |
| A         | {A, B} | $\O$   | $\O$   |
| B         | $\O$   | {B, C} | $\O$   |
| C         | $\O$   | $\O$   | {C, D} |
| **D**     | $\O$   | $\O$   | $\O$   |

构造DFA。首先是初始状态

![img](img/nd2.png)

这个初始状态在遇到a的时候，会进入A或者B，**那既然是俩状态，合到一起就可以了！**

![img](img/nd3.png)

那接下来咋办呢？看A和B：A和B收到a时，会进入{A, B}或者$\O$，因为空集不算，那还是自己

![img](img/nd4.png)

AB收到b时会进入$\O$和{B, C}，因此

![img](img/nd5.png)

然后，BC收到b会进入自己；收到c会进入CD

![img](img/nd6.png)

别忘了，CD还会接收c变成自己！

![img](img/nd7.png)

检验，NFA和DFA能接收的串都是：aa^*^bb^*^cc^*^(这里是因为至少要有一个a或b或c，也可以写成a^+^b^+^c^+^)

接下来，再看一个例子，这个例子带空边

![img](img/nd8.png)

还是画转换表，但是这个表比较难画：

| 状态\输入 | 0         | 1      | 2    |
| --------- | --------- | ------ | ---- |
| A         | {A, B, C} | {B, C} | {C}  |
| B         | $\O$      | {B, C} | {C}  |
| **C**     | $\O$      | $\O$   | {C}  |

比如A接收到0，可以接收完0不动，即状态A，也可以动一下变成B，或者动两下变成C；而A为啥能接收1呢？因为A可以先动一下变成B，这样就能接收1了。因为A变成B啥都不需要

然后来构造DFA，首先初始状态的时候就要注意：**因为A到B到C啥都不需要，所以这仨都能是初始状态，那么三合一就可以了**

![img](img/nd9.png)

然后看ABC接收0之后只能变成自己，那么

![img](img/nd10.png)

ABC接收1之后会进入BC，并且BC中也含终态，所以

![img](img/nd11.png)

ABC接收2之后会进入C，C是终态

![img](img/nd12.png)

最终都分析完之后，DFA构建好了

![img](img/nd13.png)

检验可以得到，他俩接收的串都是r = 0^*^1^*^2^*^

### 3.6 Word DFA

#### 3.6.1 Identifier DFA

之前给过标识符的定义：

* digit -> 0|1|2|...|9
* letter_ -> A|B|...|Z|a|b|...|z|_
* **identifier -> letter_(letter\_|digit)^***^

那么构造NFA的话，首先就是要一个初始状态。这个状态要开始接收字符串，首先肯定是一个letter\_，然后即可以接收letter\_也可以接收digit，而且**不是必须的**，那么这个初始状态也是终止状态。然后这两种接收都会导致自旋，则

![img](img/w1.png)

因为这个NFA就是个DFA，所以也不用转成DFA了

#### 3.6.2 Unsigned DFA

然后是无符号数的定义：

* digit -> 0|1|2|...|9

* digits -> digit digit^*^

* optionalFraction(可选小数部分) -> .digits|$\epsilon$

* optionalExponent(可选指数部分) -> (E(+|-|$\epsilon$)digits)|$\epsilon$

* **number -> digits optionalFraction optionalExponent**

构造DFA：也是要有一个初始状态，然后这个状态接收的首先是digits部分，也就是digit digit^*^。此时其实已经可以结束了，因为后面两个都是可选的部分，那么此时的状态图是

![img](img/w2.png)

至于为什么不画成终态，之后再说

然后，添加可选的小数部分。首先是一个点，既然跟了点，就一定要再跟上后面的小数部分，其实还是个digits。不过除了`.digits`，也可以是个空串$\epsilon$(其实就是要把"我不选"给强调出来)。那么此时的状态图

![img](img/w3.png)

之后添加可选指数部分。首先一定要有个E，然后是连上3个(+, -, $\epsilon$)中任意一个，然后再加上digits；这整个过程也可以只变成一个空串。要注意的是，E后面之所以也可以加$\epsilon$是因为E+12和E12是一样的，为了体现出正号能省略

![img](img/w4.png)

这下能解释为啥状态1，状态3不化成终态了，因为没必要。最后通过空串都能到达状态6

然后来看一下：**状态0接收到d之后，可以直接不动变成状态1；也可以动一下变成状态3或者动两下变成状态6**，因此这是个NFA

转换的过程也不画状态表了，直接来：

首先有个初始状态，也就是状态0，状态0能接收的就是d，接收d之后会变成1, 3, 6也就是

![img](img/w5.png)

*因为{1, 3, 6}里有终态6，所以这个就是终态(其实也印证了之前说的1，3没必要画上终态)*

然后136能接收的有`d, E, .`。1接收d之后变成136；3接收d之后变成36；6接收d之后变成6。也就是说，136接收了d之后还会变成自己

![img](img/w6.png)

然后是`.`。1能接收点，进入状态2。所以整个136接收到点也会进入状态2

> 这里要说明一下。可能会有这样的疑问：为啥只要状态1能进入状态2，这仨就都能呢？是因为：之前进入状态136是接收了digit的结果。而**只要一直处于状态136中，就不可能还接收了除了digit以外其他的符号**。因此136状态中到底是1，3还是6是**可选的！**比如后面要是有个点，我就一直在1这儿待着不往3和6去，自然就能成功接收点并进入到状态2了

![img](img/w7.png)

然后是E。3能接收E，并进入状态4，因此整个136接收到E也会进入到状态4(这里的原因和上面一样哦！)

**另外，状态4不需要任何输入就能进入状态5，所以45也是捆起来的！**

![img](img/w8.png)

然后接着看：状态2能接收d，进入状态3，自然也能进入到状态6，这里可没有1了哦，所以要新开一个(终态)

![img](img/w9.png)

然后状态36接收到d还会变成自己；36遇到E会变成45；45遇到+-会变成5；45遇到d会变成6；5遇到d会变成6；6遇到d会变成6

![img](img/w10.png)

## 4. 语法分析

### 4.1 Top-Down Parsing

来看一个例子，也是之前表达式的例子

1. E -> E + E
2. E -> E * E
3. E -> (E)
4. E -> id

根据这个文法，我们能**自顶向下**构造出语法的分析树：

![img](img/td1.png)

显然，这个分析树不是唯一的，只是我们**随便**根据选的规则画出来的，下面来写一下这棵树对应的derive过程：

> $E \Rightarrow E + E$				将根节点E由规则1替换成E+E
>
> ​	$\Rightarrow E + (E)$			 将加号右边的E根据规则3替换成(E)
>
> ​	$\Rightarrow E + (E + E)$	 将括号里面的E根据规则1替换成E+E
>
> ​	$\Rightarrow E + (id + E)$	 将括号里面左边的E根据规则4替换成id
>
> ​	$\Rightarrow id + (id + E)$	 将最左边的E根据规则4替换成id
>
> ​	$\Rightarrow id + (id + id)$	 将最后一个E替换成id

我们能发现，**最后一个是句子，所有的符号都是Terminal。而且在这个句子中，所有的符号正好都是分析树的叶节点**

那么回顾我们推导的过程，在每步推导中，都要考虑两个事情：

* 我们要替换的**Nonterminal是谁**？这里其实替换的全是E，只不过是哪一个E不确定
* 我们要根据**什么规则**来进行替换？也就是这4个中的哪一个？

#### 4.1.1 Left-most Derivation

首先来回答上面的第一个问题。我们可以每次都选择**最左边**的来进行替换，只不过要是**Nonterminal**才行，否则也替换不了

![img](img/td2.png)

就向上图一样，每次都替换最左边的E，因为只有E才是Nonterminal。然后这整个推导的过程叫做**最左推导**，而如果反着来，显然就是**从右边开始**去合成大西瓜，就是**最右规约**

同时，推导出来的这个句型就叫做该文法的**最左句型**(left-sentential form)。要注意是句型不是句子，也就是这个过程中每一步都是最左句型；并且一定是要由**Start Symbol**开始推导

#### 4.1.2 Right-most Derivation

同理可得

![img](img/td3.png)

而在**自底向上**(**不是Top-Down!!!**)的分析过程中，我们总是采用最左规约的方式，因此把最左规约称为**规范规约**；相应的最右推导称为**规范推导**

**唯一性**

对于一个文法，它的最左推导和最右推导都是**唯一的**，因为推导的过程中要替换的那个一定是唯一的

**Top-Down一般采用Left-most Derivation**

下面来看一个例子，比如文法G：

1. E -> TE'
2. E' -> +TE' | $\epsilon$
3. T -> FT'
4. T' -> *FT' | $\epsilon$
5. F -> (E) | id

如果输入的串是`id + id * id`，那么会有：

首先，**输入指针**指向输入串的第一个字符，也就是id

![img](img/ld1.png)

然后从Start Symbol开始，也就是E，E只能按规则1替换

![img](img/ld2.png)

然后这时还没有id，所以要接着替换。从**最左边**开始，把T按着规则3替换

![img](img/ld3.png)

继续， 把F按着规则5替换。这时问题来了：F既可以替换成(E)也可以替换成id。那么替换哪个呢？我们**输入指针**指向的是id，那自然要就近来。所以要把F替换成id，自然也就匹配了

![img](img/ld4.png)

然后，**输入指针**自然也要往后移一位了

![img](img/ld5.png)

此时最左边的Nonterminal是T'，自然要替换它了。它能替换成*FT'或者$\epsilon$，而现在是个加号。很显然，T'不可能以\*开头，那么我们只能将T'替换成$\epsilon$了

![img](img/ld6.png)

此时最左边的是E'，它能替换成+TE'或者$\epsilon$，显然以加号开头的这个正是我们想要的，那么就这么替换

![img](img/ld7.png)

然后输入指针后移一位

![img](img/ld8.png)

然后开始替换T，只能替换成FT'

![img](img/ld9.png)

然后自然是把F替换成id了，并且后移一位输入指针

![img](img/ld10.png)|![img](img/ld11.png)

替换T'，因为这里是星号，所以替换成*FT'并后移输入指针

![img](img/ld12.png)|![img](img/ld13.png)

然后F替换成id往下走，因为已经完了，输入指针指向的是空串，所以只要能换成$\epsilon$就换了。最终

![img](img/ld14.png)|![img](img/ld15.png)

#### 4.1.3 Backtracking & Predictive Parsing

比如Production是这样的：A -> abb | abc，那么如果此时的输入指针指向的恰好就是个a的话，到底是替换成abb还是abc？这俩都是a开头的。因此我们要逐个尝试一下。如果不能匹配了，那就说明我们这条路走的不对，要回去重来。这个过程就叫做**回溯**。

显然，回溯会降低计算机的效率。因此我们要尽可能减少回溯(这里和KMP算法好像啊)，使用**预测分析**(Predictive Parsing)的方法。具体的做法就是，当前输入指针不是指向一个输入的符号吗，我们**再往前看看**，这样就能预测一下到底是哪一个Production更加合适，这样就能预测了。而往前看通常是看1个就够了

### 4.2 Grammar Transformation

刚才刚说过回溯的问题，而自顶向下的分析方法正好就会产生这种问题。我们先来看一个例子

> G:
>
> S -> aAd | aBe
>
> A -> c
>
> B -> b
>
> 输入：`abc`

这样当指向a的时候，S的两个候选式都是a开头，就不知道到底用哪个了

除了回溯的问题，还有其他的问题

> G:
>
> E -> E + T | E - T | T
>
> T -> T * F | T / F | F
>
> F -> (E) | id
>
> 输入：`id + id * id`

这里的问题是，首先指向id，而Start Symbol右边哪几个没有一个是id开头的，所以只能一个一个试。首先来试试E + T，替换完了，如果采用Left-most的方式，那就是替换最左边那个E，而这个E再一替换还是E + T，这样就会无限循环下去

> $E \Rightarrow E + T$
>
> ​	$\Rightarrow E + T + T$
>
> ​	$\Rightarrow E + T + T + T$
>
> ​	$\Rightarrow \ ......$

那么怎么来消除这种情况呢？首先，这种情况可以统称为**左递归**，而只有一步推导就会产生的叫做**直接左递归**；两步即以上产生的叫做**间接左递归**。首先来看一下如何消除直接左递归

直接左递归包含这样的Production: $A \rightarrow A\alpha$

那么对于一个这样的Production: $A \rightarrow A\alpha | \beta\ (\alpha \neq \epsilon,\ \beta不以A开头)$，我们先来推导一下它能生成什么样的句子

> $A \Rightarrow A\alpha$
>
> ​	$\Rightarrow A\alpha\alpha$
>
> ​	$\Rightarrow A\alpha\alpha\alpha$
>
> ​	$\Rightarrow \ ...$
>
> ​	$\Rightarrow \beta\alpha\alpha...\alpha$

其实就是一个$\beta$开头，后面连上若干个$\alpha$(**可以是0个**)，即$r=\beta \alpha^*$

那么我们只需要再引入一个A'，让它形成这样的形式

> $A \rightarrow \beta A'$
>
> $A' \rightarrow \alpha A'|\epsilon$

这样在**Left-most Derivation**的过程中，首先替换A的时候就会直接采用带$\beta$的方式，也就不会有无限左递归的问题了。

有了一般的方法，我们怎么运用呢？看一开始的例子，这就是一个典型的直接左递归：

> G:
>
> E -> E + T | T
>
> T -> T * F | F
>
> F -> (E) | id
>
> *这里为了简便，删掉了一些*

那么我们要在里面找$A \rightarrow A\alpha$的形式，根据第一条就能看出，E就是A，后面的+T就是$\alpha$，T就是$\beta$。那么我们把这些带到上面化好的式子中：

> $E \rightarrow TE'$
>
> $E' \rightarrow +TE'|\epsilon$
>
> **别忘了，以上只是改造了第一条，还要以相同的方式改造第二条，因为它也是左递归！**
>
> $T \rightarrow FT'$
>
> $T' \rightarrow *FT'|\epsilon$
>
> **最后一条因为不含$A \rightarrow A\alpha$，所以不用改造**
>
> $F \rightarrow (E)|id$

那么如果是多个，应该怎么办呢？这里给出消除直接左递归的一般形式：

如果Production是：$A \rightarrow A\alpha_1|A\alpha_2|...|A\alpha_n|\beta_1|\beta_2|...|\beta_m\ (\alpha_i \neq \epsilon,\ \beta_j不以A开头)$

就可以换成这样的形式：

> $A \rightarrow \beta_1A'|\beta_2A'|...|\beta_mA'$
>
> $A' \rightarrow \alpha_1A'|\alpha_2A'|...|\alpha_nA'|\epsilon$

**其实就是，本来是一堆能套娃的串，但是你总得有个头，所以给了个$\beta$来结束。也就是说肯定是要以$\beta$开头的。很显然要是不想在左边套娃，就先把这个$\beta$放上去，然后在右边连接一堆能套娃的式子，也就是将递归从左边转移到了右边**

***另外需要注意的是，这里之所以n和m可以不相等，是因为如果n多的话，那么不够的$\beta$可以用$\epsilon$来凑；而m多的话就更简单，挪出去单独写就可以了***

然后再看一下如何消除间接左递归。其实，间接左递归中一般都会包含直接左递归，以下是一个例子：

> S -> Aa | b
>
> A -> Ac | Sd | $\epsilon$

这里第二条很显然是个直接左递归，因为有A -> Ac。那么何来间接左递归呢？我们看以下第一个式子的推导过程

> $S \Rightarrow Aa$
>
> ​	$\Rightarrow Sda$

经过了两步推导，我们又发现了S开头的句型。也就是说，这个东西每两次会套一下娃，因此是间接左递归

消除的方法，就是将**S带入A的Production中**：

> $A \rightarrow Ac|Aad|bd|\epsilon$

然后再**消除这个式子的直接左递归**即可。这里按照那个一般形式来转换就可以了

> 这里m和n都是2，那么$\alpha_1 = c,\ \alpha_2 = ad,\ \beta_1 = bd,\ \beta_2 = \epsilon$
>
> 带入上面的式子就能得到：
>
> $A \rightarrow bdA'|A'$
>
> $A' \rightarrow cA'|adA'|\epsilon$

### 4.3 LL(1) Grammar

#### 4.3.1 S_Grammar

**S文法**就是简单的(Simple)、确定的(Specific)文法。它要求：

* 每一个Production的右边都要以**Terminal**开始
* 对于同一个**Nonterminal**，它的所有候选式打头的**Terminal**都要不一样

根据定义可以看出，既然要求要以Terminal开始，那么肯定是不能含空产生式$\epsilon$的。为什么呢？来看个例子

> G:
>
> 1. S -> aBC
> 2. B -> bC
> 3. B -> dB
> 4. B -> $\epsilon$
> 5. C -> c
> 6. C -> a
> 7. D -> e
>
> 输入：
>
> * `ada`
> * `ade`

先来写一下`ada`的Derive过程，很顺利的

> $S \Rightarrow aBC$
>
> ​	$\Rightarrow adBC$		把B按照3换成dB
>
> ​	$\Rightarrow ad\epsilon C$		 没有a开头的，所以换成空串
>
> ​	$\Rightarrow ada$			最后把C换成a

**但是写`ade`的时候，会出现问题**

> $S \Rightarrow aBC$
>
> ​	$\Rightarrow adBC$		把B按照3换成dB
>
> ​	$\Rightarrow ad\epsilon C$		 没有e开头的，换成空串
>
> ​	$\Rightarrow ad?$

此时发现，最后的C没有以e开头的候选式，所以报错了

因为是和B相关的空产生式，所以来讨论它：空产生式的使用取决于在它的后面到底有什么。这里B的后面是C。由于C只能替换成c和a，所以**能紧跟着B后面出现的Terminal只能是c和a**

另外，在匹配第二个字符时，也就是上面两个输入中的`d`。当前的Nonterminal是B，那么如果这个d不是d而是f的话，也就是说**和当前的Nonterminal的候选式都不匹配的时候**，这时本应该报错的，但是还有个空产生式，那么到底用不用呢？这取决于它后面的东西。**如果这个f能紧跟着B后面出现**，那么把B替换成$\epsilon$是没有问题的，否则就应该报错了

#### 4.3.2 Follow Set

就像刚才说的，这个B后面只能紧跟着出现c和a。那么如果对于一个在**Sentential From**中出现的**Nonterminal**，它后面能紧跟着出现的这些**Terminal**可以形成一个集合，这个集合就叫做**Follow集**。
$$
FOLLOW(A) = \{a|S\Rightarrow^*\alpha Aa\beta,\ a\in V_T,\ \alpha,\beta\in(V_T\cup V_N)^*\}
$$
这个公式其实很好理解。首先S经过若干步推导，那得到的一定是个句型。然后右边就是左边一个$\alpha$，右边一个$\beta$，中间夹着的就是定义里的东西，其中$\alpha$和$\beta$都是文法中符合规定的任意一个串。

那还有个问题，如果A后面没东西了咋整？难道Follow集里还有空串？前面强调过，**$\epsilon$既不是Terminal也不是Nonterminal**，所以很显然是不能加到这个集合里的。所以我们引入一个结束符**$**，也就是说，**如果句型中如果最右边那个东西是个Nonterminal，那它的Follow集就是{$}**

由以上所说可以推出这个集合的用处了。就比如之前那个例子中的B，如果它后面出现的Terminal正好就在Follow(B)中，那很显然就应该选择这个候选式了。

> G:
>
> 1. S -> aBC
> 2. **B -> bC**
> 3. **B -> dB**
> 4. **B -> $\epsilon$**
> 5. C -> c
> 6. C -> a
> 7. D -> e

Follow集就是用在4号这种式子上面的。如果当前的Nonterminal是B，那么我们就计算Follow(B) = {a, c}。计算方法就是往后看，B后面是C，C能替换成c和a，所以这俩的集合就是B的Follow集。然后看当前的输入符号是啥：如果是b，就选2；如果是d，就选3；如果是a或者c(**Follow集中的元素**)，就选4。

另外我们能发现，2号对应的是b开头；3好对应的是d开头；4号对应的是a或者c开头，**它们是不相交的**。

#### 4.3.3 Selection Set

以上说的都是看当前的输入符号，去选择哪个Production。那么我们也可以反着来，就是说看**当前的某一个Production，它能适用于哪些输入符号呢**？这就是**可选集**的由来。

比如上面例子中的第二条：B -> bC。我们知道，只有当前输入符号是b的时候才能选这个。那么也就是说对于这个产生式，它的可选集就是{b}。而如果是第四条的话，就和前一节说的一样，**它的可选集就是Follow集**
$$
\begin{align}
&SELECT(A\rightarrow a\beta)=\{a\}\\
&SELECT(A\rightarrow\epsilon)=FOLLOW(A)
\end{align}
$$

#### 4.3.4 q_Grammar

相对于S文法，q文法更加强大。上面的例子中就是一个q文法

* 每个Production的右边要么是$\epsilon$，要么以Terminal打头
* 如果Production的左部相同，那么这些Production的可选集不能有交集

这个第二条正好就把S文法的第二条也覆盖了。因为如果是这种：

> A -> abc
>
> A -> acc

那么这两条Production的可选集一定有交集，也就是元素a。自然打头的Terminal一定要不一样

*问题：这里仅仅是个人猜想，有待证明*

我们根据q文法的规则也能推测出，q文法中产生式的右部一定不能以Nonterminal打头。

那么我就要以Nonterminal打头该咋办？这样计算可选集的难度会大很多。因为打头的这个还要继续套娃替换，甚至会套娃很多次，所以我们要再了解一些其他的东西。

#### 4.3.5  First Set

现在给定一个串$\alpha$，那很显然，它既可以以Terminal开头，也可以以Nonterminal开头，还可以是空串。

那么我对于这个串，如果它是以Terminal开头的话，很显然它的**首个符号是唯一确定的**。那么此时它的**串首终结符集**就是这个字符了：
$$
\begin{align}
&若S\rightarrow aABe\\
&则FIRST(S)=\{a\}
\end{align}
$$
然而如果这个串是以Nonterminal开头的话，它的首个符号还要看这个Nonterminal能替换成什么，直到是Terminal为止。并且这个替换方式还可能会有多种，因此这个串的**首个符号不是唯一确定的**。也就是说开头有多种可能，那么所有这些可能组成的集合就是它的First集：
$$
\begin{align}
&若：\\
&1.\ T \rightarrow FT'\\
&2.\ F \rightarrow (E)|id\\
&...\\
&那么FIRST(T)=\{\ (,\ id\ \}
\end{align}
$$
然后是空串的情况。这里其实有一个小问题。看下面的例子：

> 1. A -> BC
> 2. B -> $\epsilon$
> 3. C -> $\epsilon$

这里1号是以Nonterminal开头的，但是这个Nonterminal却只能被替换成空串。因此空串的情况应该并到上面当中。**实际上，如果$\alpha \Rightarrow^*\epsilon$，那么$\epsilon$也在$FIRST(\alpha)$中**。

有了这些，我们结合一下4.3.3中说的可选集的概念。

*这里补充一下。可选集表示的是**被选择**的关系。也就是这个产生式能在输入什么的时候被选择呢？就是可选集中的元素。*
$$
\begin{align}
&SELECT(A\rightarrow a\beta)=\{a\}\\
&SELECT(A\rightarrow\epsilon)=FOLLOW(A)
\end{align}
$$
看第一条，是不是和FIRST集有很大的相似？没错！如果我们把右边的$a\beta$看成一个大串$\alpha$，那么很显然$\epsilon\notin FIRST(\alpha)$。因为这个大串根本推导不出来空串(其实$FIRST(\alpha)=\{a\}$根据上面说的就能算出来)。那么这整个Production的可选集就是这个右部大串的FIRST集：
$$
SELECT(A\rightarrow\alpha)=FIRST(\alpha)\ (\epsilon\notin FIRST(\alpha))
$$
然后再看第二条。之前说过，当右边是空串的时候，要往后再看看，后面会出现什么，也就是FOLLOW集中的东西。那么如果对于任意一个大串$\alpha$，它如果能推导出空串的话，显然$\epsilon\in FIRST(\alpha)$。那么此时的可选集不但要包含大串的FIRST集，也要包含左部的FOLLOW集：
$$
SELECT(A\rightarrow\alpha)=[FIRST(\alpha)-\{\epsilon\}]\cup FOLLOW(A)\ (\epsilon\in FIRST(\alpha))
$$
这里要减掉{$\epsilon$}的原因是，可选集的右边表示的是输入的符号，总不可能是空串吧！

#### 4.3.6 Left-Right Left-most (1) Grammar

先来回顾下，整个4.3都是为了干什么。4.1.3中我们提到了Backtracking和Predictive Parsing。预测分析就是用来避免掉回溯的。那么我们怎么做到预测分析呢？就是**让每个输入符号只能对应一个候选式**。那么怎么做到呢？正好可选集就表示了被选择的关系。那么我们只需要**让所有的可选集都不相交**，自然就不会有**"两个候选式被同一个输入符号选择"**的情况了。而像S文法和q文法中的第二条，也正是为了达到这样的效果。而我们的LL(1)文法也是为了这个。那么它和前两个有啥区别呢？前两个文法很鸡肋啊！比如S文法，它只能以Terminal开头，限制就已经很大了。根本不允许在开始就套娃；而q文法也只是在S的基础上加了一个空串。而LL(1)文法就不会有这样的限制，并且也能做到每个输入符号只对应一个候选式。

首先，既然是文法，肯定要有产生式(这不废话吗...)。那么如果有这样的：$A\rightarrow\alpha|\beta$，右部这两个就是俩串。

* 当这俩串都不能推导出空串时，我们想要保证：

  * $A\rightarrow\alpha$
  * $A\rightarrow\beta$

  这俩产生式的可选集不能相交，因为在不能推导出空串时前面也说了，可选集就是FIRST集。自然只需要保证
  $$
  FIRST(\alpha)\cap FIRST(\beta)=\O
  $$
  FIRST集不相交，自然可选集也不相交

* 当$\alpha$能推出空串，但是$\beta$不能时，我们先分别写一下他们的可选集
  $$
  \begin{align}
  &SELECT(A\rightarrow\alpha)=[FIRST(\alpha)-\{\epsilon\}]\cup FOLLOW(A)\\
  &SELECT(A\rightarrow\beta)=FIRST(\beta)
  \end{align}
  $$
  我们既然要让等号左边不相交，让等号右边不相交就行了！那么我们就需要保证
  $$
  \begin{align}
  &FIRST(\beta)\cap FOLLOW(A)=\O\\
  &FIRST(\beta)\cap FIRST(\alpha)=\O
  \end{align}
  $$
  *问题：PPT中只给了第一条，这第二条不需要吗难道？*

* 如果反过来，也就是$\beta$能推出空串但是$\alpha$不能，只需要把这俩换一下就行了
  $$
  \begin{align}
  &FIRST(\alpha)\cap FOLLOW(A)=\O\\
  &FIRST(\alpha)\cap FIRST(\beta)=\O
  \end{align}
  $$

* 如果它俩都能推出空串，这种情况不存在。因为如果都能，它们的可选集里都有FOLLOW(A)，那本身就已经相交了

#### 4.3.7 Calculation

文法中有如下产生式：

* E -> TE'
* E' -> +TE' | $\epsilon$
* T -> FT'
* T' -> *FT' | $\epsilon$
* F -> (E) | id

计算E, E', T, T', F的FIRST集和FOLLOW集，并计算每个产生式的SELECT集

> ==**1. 计算FIRST集**==
>
> 首先我们从Terminal打头的入手
>
> * E -> TE'							FIRST(E) = {}
> * E' -> +TE' | $\epsilon$ 			 	   FIRST(E') = {}
> * T -> FT'							FIRST(T) = {}
> * T' -> *FT' | $\epsilon$						FIRST(T') = {}
> * F -> (E) | id						FIRST(F) = {}
>
> 也就是E'，T'和F。它们分别以+，*和(打头，同时F还有一个id，那么直接写进去
>
> * E -> TE'							FIRST(E) = {}
> * E' -> +TE' | $\epsilon$ 			 	   FIRST(E') = {+}
> * T -> FT'							FIRST(T) = {}
> * T' -> *FT' | $\epsilon$						FIRST(T') = {\*}
> * F -> (E) | id						FIRST(F) = {(, id}
>
> 然后再来逐个给Nonterminal打头的套娃。首先是E，它以T开头，而T此时还没确定，所以先来T
>
> T以F开头，而F以(或者id开头，所以T肯定也是以(或者id开头
>
> * E -> TE'							FIRST(E) = {}
> * E' -> +TE' | $\epsilon$ 			 	   FIRST(E') = {+}
> * T -> FT'							FIRST(T) = {(, id}
> * T' -> *FT' | $\epsilon$						FIRST(T') = {\*}
> * F -> (E) | id						FIRST(F) = {(, id}
>
> 那么既然E以T开头，E就也会以(或者id开头
>
> * E -> TE'							FIRST(E) = {(, id}
> * E' -> +TE' | $\epsilon$ 			 	   FIRST(E') = {+}
> * T -> FT'							FIRST(T) = {(, id}
> * T' -> *FT' | $\epsilon$						FIRST(T') = {\*}
> * F -> (E) | id						FIRST(F) = {(, id}
>
> **千万别忘了空串。空串是可以在FIRST集中的！**
>
> * E -> TE'							FIRST(E) = {(, id}
> * E' -> +TE' | $\epsilon$ 			 	   FIRST(E') = {+, $\epsilon$}
> * T -> FT'							FIRST(T) = {(, id}
> * T' -> *FT' | $\epsilon$						FIRST(T') = {\*, $\epsilon$}
> * F -> (E) | id						FIRST(F) = {(, id}

> **==2. 计算FOLLOW集==**
>
> 既然FOLLOW集是紧跟着某一个Nonterminal后面出现的，那么肯定是根据后面那个东西的FIRST集来的。所以我们要先计算它们的FIRST集
>
> * E -> TE'							FIRST(E) = {(, id}				FOLLOW(E) = {}
> * E' -> +TE' | $\epsilon$ 			 	   FIRST(E') = {+, $\epsilon$}				FOLLOW(E') = {}
> * T -> FT'							FIRST(T) = {(, id}				FOLLOW(T) = {}
> * T' -> *FT' | $\epsilon$						FIRST(T') = {\*, $\epsilon$}				FOLLOW(T') = {}
> * F -> (E) | id						FIRST(F) = {(, id}				FOLLOW(F) = {}
>
> 首先从E开始，也就是FOLLOW(T)。T的后面是E'，那么T后面紧跟着出现的就是E'的FIRST集中的内容。**然而E'可以推导出空串，也就是T之后可以是$\epsilon$，则T的FOLLOW集中也应该有'$'符号**。
>
> **另外，这个E作为开始符号，本身也是个句型，它的后面本身就啥也没有，所以也要加上$**
>
> 对于这个最后的E'，它后面没东西。所以**能加在E后面的也能加在E'后面**，也就是把E的FOLLOW集里的全家在E'的FOLLOW集中。虽然就是一个$
>
> * E -> TE'							FIRST(E) = {(, id}				FOLLOW(E) = {$}
> * E' -> +TE' | $\epsilon$ 			 	   FIRST(E') = {+, $\epsilon$}				FOLLOW(E') = {$}
> * T -> FT'							FIRST(T) = {(, id}				FOLLOW(T) = {+, $}
> * T' -> *FT' | $\epsilon$						FIRST(T') = {\*, $\epsilon$}				FOLLOW(T') = {}
> * F -> (E) | id						FIRST(F) = {(, id}				FOLLOW(F) = {}
>
> 然后是第二个产生式。因为这里T和E'和第一个位置一样，所以分析结果也一样。
>
> 然后是第三个。这里计算的显然就是FOLLOW(F)和FOLLOW(T')。F后面出现的就是T'的FIRST集，所以F的FOLLOW集里要加上\*；另外同理，因为T'能推出空串，所以也要加上$
>
> **漏了！因为T'能推出空串，所以这个产生式可以变成T -> F。代表能加在T后面的也能加在F后面。所以F的FOLLOW集应该有T的FOLLOW集里所有的东西，即+和$**
>
> 对于T'，它后面首先啥也没有，所以肯定有$；**另外能加在T后面的也能加在T'后面，所以T'也应该有T的FOLLOW集**
>
> * E -> TE'							FIRST(E) = {(, id}				FOLLOW(E) = {$, )}
> * E' -> +TE' | $\epsilon$ 			 	   FIRST(E') = {+, $\epsilon$}				FOLLOW(E') = {$}
> * T -> FT'							FIRST(T) = {(, id}				FOLLOW(T) = {+, $}
> * T' -> *FT' | $\epsilon$						FIRST(T') = {\*, $\epsilon$}				FOLLOW(T') = {+, $}
> * F -> (E) | id						FIRST(F) = {(, id}				FOLLOW(F) = {\*, $, +}
>
> 之后是第四条。因为第四条里F和T'和第三条一样，所以也不用来了。
>
> 最后是第五条。这里只有E需要看。因为E后面只能有)是唯一确定的，所以加进去就行了。
>
> **算完了吗？没有！我们再回头看，会发现少了许多东西**
>
> 第一句时，给出T的FOLLOW集中应该有E的FOLLOW集(**因为E'能推出空串**)。而这时E的FOLLOW集已经被更新了，所以要再改一下；另外，E'的FOLLOW集也要有E的FOLLOW集，所以也要跟着改。
>
> * E -> TE'							FIRST(E) = {(, id}				FOLLOW(E) = {$, )}
> * E' -> +TE' | $\epsilon$ 			 	   FIRST(E') = {+, $\epsilon$}				FOLLOW(E') = {$, )}
> * T -> FT'							FIRST(T) = {(, id}				FOLLOW(T) = {+, $, )}
> * T' -> *FT' | $\epsilon$						FIRST(T') = {\*, $\epsilon$}				FOLLOW(T') = {+, $}
> * F -> (E) | id						FIRST(F) = {(, id}				FOLLOW(F) = {\*, $, +}
>
> 第三句中，T'后面出现的应该包括T后面出现的所有；另外T'能变空串，所以F后面出现的也应该包括T后面出现的所有
>
> * E -> TE'							FIRST(E) = {(, id}				FOLLOW(E) = {$, )}
> * E' -> +TE' | $\epsilon$ 			 	   FIRST(E') = {+, $\epsilon$}				FOLLOW(E') = {$, )}
> * T -> FT'							FIRST(T) = {(, id}				FOLLOW(T) = {+, $, )}
> * T' -> *FT' | $\epsilon$						FIRST(T') = {\*, $\epsilon$}				FOLLOW(T') = {+, $, )}
> * F -> (E) | id						FIRST(F) = {(, id}				FOLLOW(F) = {\*, $, +, )}
>
> 然后第四句和第五句都分析不出啥，所以第二轮结束
>
> **算完了吗？可能还没有！因为FOLLOW集还是有更新，要算到"在某一轮中，一个FOLLOW集都没更新"这种情况出现时，才能算圆满完成！**
>
> 再来一轮，发现所有FOLLOW集都没更新，这才真算完了！

> **==3. 计算SELECT集==**
>
> 首先要把产生式给全都分开
>
> 1. E -> TE'
> 2. E' -> +TE'
> 3. E' -> $\epsilon$
> 4. T -> FT'
> 5. T' -> *FT'
> 6. T' -> $\epsilon$
> 7. F -> (E)
> 8. F -> id
>
> 然后列出所有Nonterminal的FIRST集和FOLLOW集
>
> * FIRST(E) = {(, id}				FOLLOW(E) = {$, )}
> * FIRST(E') = {+, $\epsilon$}				FOLLOW(E') = {$, )}
> * FIRST(T) = {(, id}				FOLLOW(T) = {+, $, )}
> * FIRST(T') = {\*, $\epsilon$}				FOLLOW(T') = {+, $, )}
> * FIRST(F) = {(, id}				FOLLOW(F) = {\*, $, +, )}
>
> 然后从第一条Production开始，运用这样的想法：**我在输入什么的时候，才会选到1号产生式呢？**很显然，在我当前输入的符号是以T的FIRST集中的元素开头的时候，我才会选择这个产生式来进行替换。所以**1号的可选集就是T的FIRST集**。
> $$
> SELECT(1)=\{(,\ id\}
> $$
> 然后是第二条。因为这是以Terminal开头的，所以别无选择，只有输入符号是+的时候才会选它。
> $$
> SELECT(2)=\{+\}
> $$
> 第三条中，因为E'可以变空串，所以E'后面紧跟着出现的东西正好是我当前输入的符号的话，我也能选它。这正对应了E'的FOLLOW集。
> $$
> SELECT(3)=\{$,\ )\}
> $$
> 第四条中，类比第一条，这条产生式的可选集自然要有F的FIRST集中的内容。
> $$
> SELECT(4)=\{(,\ id\}
> $$
> 第五条中和第二条一样，只有一个星号。
> $$
> SELECT(5)=\{*\}
> $$
> 第六条和第三条差不多，对应T'的FOLLOW集。
> $$
> SELECT(6)=\{+,\ $,\ )\}
> $$
> 然后是第七条，不用多说
> $$
> SELECT(7)=\{(\}
> $$
> 第八条也是一样
> $$
> SELECT(8)=\{id\}
> $$
> **这道题有一种特殊情况没涉及到。比如说第四条：有F的FIRST集自当天经地义；但是如果F能推出空串的话，也就是F压根不存在，那么开头就变成了F之后的东西，也就是F的FOLLOW集中的东西也应该有。**

总结：通过这道题，和之前的概念，我们也能发现，FOLLOW集是用在单个的Nonterminal上的，而FIRST集是用在串上的。因为Nonterminal本身也是一个串，所以自然都可以。而SELECT集是用在产生式上的，因此才会由小到大的计算。另外，FIRST集中要么是Terminal，要么是$\epsilon$；而FOLLOW集本身就是Terminal的集合，自然不能有$\epsilon$，而是用$代替；而SELECT集中因为表示的是输入符号，肯定要有意义，所以元素种类和FOLLOW集中的是一样的。
