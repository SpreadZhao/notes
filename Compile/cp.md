# 编译原理笔记

## 词法分析

### 字母表

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

### 串的运算

**连接**

> x和y的连接运算极左xy
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

## 文法分析

### Terminal Symbol

终结符：是文法所定义的语言的基本符号。有时也称为**Token**

比如：V~T~ = {apple, boy, cat, little}是一个终结符集，其中的元素是终结符，在这里是**单词**

### Nonterminal Symbol

非终结符：用来表示语法成分的符号，有时也称为**语法变量**

比如：V~N~ = {<句子>, <名词短语>, <动词短语>, <名词>, ...}是一个非终结符集
