# 编译原理笔记

## 1. 词法分析

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

  ==我们也能发现，几个字母表相乘，结果的每一项就是几个元素，比如这里三个相乘，结果中每一个元素比如000，001都是三个数==

  **我们还能发现，字母表的n次幂，就是长度为n的符号串构成的集合。比如这里n = 3，那结果集合中每一个元素长度都是3。而0次幂就是长度为0的符号串，那就是==空串==，用$\epsilon$表示**

* 正闭包：$\Sigma$^+^ = $\Sigma$ $\cup$ $\Sigma$^2^ $\cup$ $\Sigma$^3^ $\cup$ ...

  比如{a, b, c, d}^+^ = {a, b, c, d, aa, ab, ac, ad, ba, bb, bc, bd, ... , aaa, aab, aac, aad, ...} 

* 克林闭包：$\Sigma$^*^ = $\Sigma$^+^ $\cup$ $\Sigma$^0^ = $\Sigma$^0^ $\cup$ $\Sigma$ $\cup$ $\Sigma$^2^ $\cup$ $\Sigma$^3^ $\cup$ ...

  比如{a, b, c, d}^*^ = {$\epsilon$, a, b, c, d, aa, ab, ac, ad, ba, bb, bc, bd, ... , aaa, aab, aac, aad, ...} 

  ==由此可见，克林闭包是任意符号串构成的集合，并且这个串的长度可以是0==
  
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

### 2.5 文法

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

### 2.6 符号约定

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

## 3. 语言

*给你一个句子：`little boy eats apple`，咋判断它符不符合语法？*

### 3.1 Derivation & Reduction

比如一个**文法**中的**Production**是这样的(这里就不写成集合了)：

* <句子> $\rightarrow$ <名词短语><动词短语>
* <名词短语> $\rightarrow$ <形容词><名词短语>
* <名词短语> $\rightarrow$ <名词>
* <动词短语> $\rightarrow$ <动词><名词短语>
* <形容词> $\rightarrow$ little
* <名词> $\rightarrow$ boy | apple
* <动词> $\rightarrow$ eat

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

### 3.2 Sentential form & Sentence

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

### 3.3 Language

由文法G的Start Symbol**推导出的所有<u>句子</u>**构成的集合称为文法**G生成的语言**，记为L(G)

L(G) = {$\omega$ | S $\Rightarrow$^*^ $\omega$, $\omega$ $\in$ V~T~^*^} (*问题：这里V~T~是否加上星号存疑*)

