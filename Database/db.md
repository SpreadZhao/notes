# 数据库笔记

## 1. Introduction

Database-management system(DBMS): a collection of interrelated data and a set of programs to access the data.

Database: a collection of data, contains information relevant to an enterprise.

数据库管理的数据通常是这样的：

* 非常重要
* 非常多
* 经常被很多用户或者进程同时访问

### 1.1 File-processing System

以前用的老数据库通常都是一个文件，叫file-processing system。这样的方式有很多缺点：

* Data redundancy and inconsistency

  比如一个学生的主修课有数学和音乐，而其他的信息比如学号年龄啥的都是一样的。即使这样这个学生的信息也要被复制成两份文件。这样就会显得很冗余；另外如果想要修改数据，很有可能只改了一份，这样还会产生不一致的问题。

* Difficulty in accessing data

  如果辅导员想要调出所有学计算机学生的名单，这样的结构是办不到的。除非程序员预先写好了一个按着这种方式筛学生的程序，不然只能手工来弄。所以这种不能按既定要求查询的数据库也很麻烦。

* Data isolation(隔离)

  数据都是分散在不同文件里的，也就是都是隔离开的，这样找起来很麻烦。

* Integrity problems

  比如我的银行账户的余额永远不可能降到低于0，所以程序员写的时候就要单独为这个限制增加代码；后来又有了新的限制，如果要再加代码，这些刚才加的代码反而被破坏了，导致余额又会变到0以下了。这样**限制和限制之间起冲突**的问题也很难解决。万恶之源就是这个**consistency constraints**。

* Atomicity problems

  转账的时候，先从A那儿扣钱然后再加到B上面。要是在刚扣完钱的时候程序出错了，那钱呢？**所以要么做完，要么不做**。所以如果是上面的情况，一定要退回到转账还没发生，也就是还没扣钱的状态。在这种文本数据库里实现这个很困难。

* Concurrent-access anomalies

  俩人一起打怪，假设怪1000血。A读一下发现是1000血，打出2点伤害，变成998传回去；B在A还没打的时候也读了一下怪，发现还是1000血，打出3点伤害，变成997传回去。那这怪到底剩多少血？998还是997？显然都不对，所以解决这个问题也很重要。

* Security problems

  如果你是银行用户，你能随便查别人的余额吗？显然不能。但是**我也在这个数据库里**，我也要存钱取钱啊，如果不让我看别人的信息，**只给我看他们想让我看的**，这种文件格式也很难办到。

<img src="img/dbms.png" alt="img" style="zoom:43%;" />

> 我们看到，在DBMS的帮助下，应用程序能更好地使用数据库，同时还保证了数据库的安全性。这里**不同的应用通过DBMS后，看到的DB就是不一样的**。

### 1.2 Data

#### 1.2.1 Data Model

数据库里很重要的一个概念。说的就是这数据到底怎么去表述它，怎么去处理它们之间的关系，怎么解决consistency constraints的问题。这里有4种主流的模型：

* Relation Model
* Entity-Relationship Model
* Semi-structured Data Model
* Object-Based Data Model

这些模型之后都会详细讨论。

#### 1.2.2 Relational Data Model

<img src="img/rdm.png" alt="img" style="zoom:43%;" />

#### 1.2.3 Data Abstraction

为了普通用户使用，需要多次封装来让数据库看起来变得简单一些。

* Physical level

  存储的具体细节。是存在磁盘上还是存在云上；是用树形图存还是用其他的数据结构存等等

* Logical level

  我存的到底是个什么玩意儿。比如如果我要存一个instructor(大学导师)，可以用下面的方式：

  ```sql
  type instructor = record
  					ID: char(5);
  					name: char(20);
  					dept_name: char(20);
  					salary: numeric(8, 2);
                    end
  ```

* View level

  比如一个学校里的食堂大妈也在这个数据库里。它为什么要看到这个老师的工资呢？视图层的作用就是隐藏不需要的数据和不想让它看到的数据。这样也简化了数据库的使用。同样的数据库经过不同的试图层，就会在不同的人面前呈现出不同的样子。

这三层的关系就是这样的。

<img src="img/tl.png" alt="img" style="zoom:43%;" />

#### 1.2.4 DBMS Architecture

DBMS在内部做的事情其实是很复杂的，尽管我们看起来很简单。比如下面有三张表：

<img src="img/d1.png" alt="img" style="zoom:43%;" />

第一张是学生的学号，姓名，性别，年龄和专业；第二张是课程号，课程名等等；第三张是学生的学号，课程号和课程的成绩。那么如果我们在**物理层**把课程信息和成绩单信息存到一起，学生的信息单独存起来。

<img src="img/d2.png" alt="img" style="zoom:43%;" />

我们会发现，如果我们要打印一个这样的成绩单：

<img src="img/d3.png" alt="img" style="zoom:43%;" />

我们需要他同时具有这三个表的信息才可以。因为成绩单那张表只有课程号和成绩，并没有课程名，需要调用课程信息的表才可以。从外面看，我们只能看到这个成绩单；但是**在DBMS内部需要转化成对三张表的查询和对两个物理存储介质的增删改查**，因此DBMS的内部逻辑是很复杂的。

因此，我们将逻辑层的模式叫做Logical Schema，将外部暴露给用户的模式叫做External Schema，将物理层最裸露的处理模式叫做Physical Schema。**Logical Schema其实就对应着上面的Logical level，是用来表示我存的到底是个什么，整个数据拆开揉碎了都是怎么样的**。

<img src="img/le.png" alt="img" style="zoom:43%;" />

#### 1.2.5 Instances and Schemas

这俩的关系有点像实例和类型之间的关系。比如在C++里定义了一个Animal类，实例化出一个dog对象，那么dog就是Instance，而Animal就是Schema。

* Instance: the collection of information stored in the database **at a particular moment**.
* Database schema: the **overall** design of the database.

### 1.3 Database Language

#### 1.3.1 DDL

Data-definition language(DDL): 定义数据库的schema。比如：

```sql
create table department
(
    dept_name	char(20),
    building	char(15),
    budget		numeric(12,2)
);
```

这里面还有很多其他的功能，比如把`dept_name`定义成**主键**等等。

#### 1.3.2 DML

Data-manipulation language(DML): 数据操作语言，就是对数据库进行**增删改查**的语言。

这部分更多是上机练习。

### 1.4 Database Design

先来看一个例子。

<img src="img/dd.png" alt="img" style="zoom:43%;" />

这个数据库有什么问题吗？我们来想想：如果我们想把物理学改成应用物理，那么我们就要搜索整个表，把所有能找到的物理学都改成应用物理，否则就会出现不一致的问题。这种问题叫做**更新异常**；另外，这个表里现在只有一个音乐学院的老师。如果这个老师被调离了，我们当然要从数据库中删除它。而造成的问题就是：这个表里所有关于音乐学院的信息都丢失了。这种问题叫做**删除异常**；如果我们在删除了这个音乐老师之后，又来了一个新音乐老师，这时候插入也会产生**插入异常**。这些情况，都是由与我们的**数据冗余**导致的，也就是**一样的信息我们在表中存储了多份**，或者说**我们把本应该存在多张表里的东西都堆在了一起**。解决这个问题的办法也很显而易见，就是将这个表拆成多张表。具体怎么拆之后再说。

## 2. Relational Model

### 2.1 Concepts

Relation表示关系，关系的模型其实就是一个表格。比如下面的这个

<img src="img/rm.png" alt="img" style="zoom:43%;" />

每一列都代表着一个**属性(attribute)**，每一行都是一个**元组(tuple)**，其实就是一个对象。这里列其实描述的就是之前说的Schema，而每一行其实就在描述这个数据库的Instance。

对于每一个属性，都会有它的约束。比如ID必须是一个定长为5的字符串；name是一个最大长度为20的变长字符串等等，这些约束叫做属性的**值域(domain)**。

属性的值域需要满足一个要求，叫做**原子性(atomic)**。比如某个数据库里存了人的电话号。那么如果说这个电话号可以存很多个号，比如一个人有几张卡这种情况。这样存的话，这个数据库就不满足原子性。因为这个电话号属性是可以拆开的。如果一定要这么存的话，我们只能把所有电话号拆开，比如拆成个人手机、固定电话、亲属电话等等。**总之一个属性里必须只能存一个数据**。

如果要是来了一个新老师，还没确定他是什么学院的，也不确定工资是多少，只知道ID和姓名，那么这个时候我们怎么赋值呢？我们允许属性有一个特殊的值叫做**null**。

### 2.2 Relational Algebra

#### 2.1.1 Relation Schema and Instance

现在有一个学院的关系，叫做department relation。

<img src="img/dr.png" alt="img" style="zoom:43%;" />

那么我们观察可得，这表里有三个属性：dept_name, building, budget。我们记作$A_1,\ A_2,\ A_3$，那么我们就可以写出这个relation的schema了：
$$
department(dept\_name,\ building,\ budget)
$$
这个schema就叫做**Relation Schema，记作R**。我们能发现，**R其实就是所有的属性构成的集合而已**。那么，我们就让属性中的一部分叫做**键(Key)，记作K**。也就是说$K \subseteq R$。

上面例子中的`dept_name`，我们能发现，在这个relation中，只要`dept_name`确定了，我们就能确定唯一的一行。像这样的Key，我们叫它**superkey**。又或者上面老师的例子中，`ID`就可以是一个superkey，而{`ID`, `name`}也可以是一个superkey。但是`name`自己就不是一个superkey了，因为老师可能有重名。这里要注意的是，**Key是一个集合而不是单独的属性**，只不过我们日常生活中Key总是单元素的集合罢了。

那么在这些superkey中，我们总要选合适的。所以我们选择**元素个数最少的集合**(基本上都是只有1个元素的)，将它叫做**candidate key**。比如各种网络游戏，会有千千万万玩家。在刚进入游戏的时候，会要求每个玩家起名字，并且不能重名。那么很显然这个`name`就可以成为candidate key；另外，我们还给每个玩家定义了一个`user_id`(因为玩家起的名字通常千奇百怪，尽管没有重名但是管理起来依然很复杂)，也要求所有的`user_id`不能重复。很显然`user_id`也可以成为candidate key。

在上面游戏的例子中，`user_id`比起`name`显然更加适合搜索，因为`user_id`是程序有秩序的赋予而`name`是玩家起的各种千奇百怪的名字。所以我们将`user_id`作为**primary key**。需要注意的是，**primary key可以不止有一个，可以同时有很多**。

现在那之前说的两张表来：

<img src="img/rm.png" alt="img" style="zoom: 50%;" />|<img src="img/dr.png" alt="img" style="zoom: 50%;" />

在右边的relation中，`dept_name`显然是primary key；而在左边的表中也出现了`dept_name`。如果这俩表是在同一个数据库系统中的，那么我们就说左边的这个`dept_name`是一个**foreign key**，它来自右边的表。这玩意儿有什么用呢？我们来想想：假设我们要插入一个老师，但是惊讶的发现它是电竞学院的，那对不起，肯定是插入失败，因为我们学校根本没有电竞学院。那么我们是怎么有底气说出这话的呢？靠的就是foreign key。我们`dept_name`的**取值**都是来自于其它relation中的primary key，是一定真实存在的。所以我们`dept_name`的值域已经被限制的死死的了，因此**我们只能插入学院是右边那张表里存在的东西**。这种方式也是给我们的值域增加了一种约束。因此我们有时候也把外键称为**foreign key constraint**。

**对于一个学校的教务系统，我们就可以画出这样一张表。**

![img](img/sd.png)

> 这里的箭头表示被引用的关系。比如takes中的ID指向了student里的ID，表示了takes中的ID是一个foreign key，它来自student。
>
> 下划线表示primary key。比如takes(选课) relation中，学生的ID，课程的id，上课节数的id，学期和年份共同作为选课的主键。

#### 2.1.2 Relational-algebra Operations

##### 2.1.2.1 Select

表达式看着很玄乎，其实简单的不得了。我们还是拿instructor relation来举例子。

<img src="img/rm.png" alt="img" style="zoom: 50%;" />

我们如果想选出所有物理学院的老师，我们很容易写出结果：

<img src="img/select.png" alt="img" style="zoom:43%;" />

那么想想我们整个操作的过程：**我们对一个relation进行了操作，这个操作叫做select，同时得到了一个结果，这个结果也是个relation，而这个结果是原来的relation的一部分**。这样的过程就是进行了选择的过程。我们可以将这个**得到的结果relation**记作：
$$
\sigma_{dept\_name="Physics"}(instructor)
$$
那么很显然可以总结：$\sigma(r)$就是对r这个relation进行选择操作，而“怎么选”就写在下标里。另外，我们还能发现，**select操作只改变了tuple的个数，并没有改变attribute的个数**。

那么如果不是只按照一个标准选怎么办？再看一个例子。

<img src="img/select2.png" alt="img" style="zoom:43%;" />

如果这个是relation r，那么问：$\sigma_{A=B\ \and \ D>5}(r)$是多少？

一看就能看明白，是让我们找：A=B并且D>5的tuple。那么也很容易写出结果：

<img src="img/select3.png" alt="img" style="zoom:43%;" />

##### 2.1.2.2 Project

这里Project译为“投影”。和select相反，它不是横着筛选，而是竖着筛选。还是instructor的例子，如果我们不要`dept_name`，只要剩下三个属性，那么就能得到这样一个relation：

<img src="img/project.png" alt="img" style="zoom:43%;" />

那么我们也能写出这个结果：
$$
\Pi_{ID,\ name,\ salary}(instructor)
$$
**注意另一种情况：**

<img src="img/project2.png" alt="img" style="zoom:43%;" />

如果我们要写出$\Pi_{A,\ C}(r)$，会发现：

<img src="img/project3.png" alt="img" style="zoom:43%;" />

**有两行是重复的，因为这里我们不要的B其实是superkey。**那么我们得到的这个就不是最后的结果，**还要删除所有重复的tuple才行**。

<img src="img/project4.png" alt="img" style="zoom:43%;" />

##### 2.1.2.3 Union

前面两个操作都是对单个的relation，接下来就是对多个relation了。首先是“并”，也就是两个relation并起来形成一个新的relation。

<img src="img/union.png" alt="img" style="zoom:43%;" />

那么我们很容易能写出$r\cup s$：

<img src="img/union2.png" alt="img" style="zoom:43%;" />

需要注意的是，这里只写了4条而不是5条，是因为$\alpha\ 2$这个数据在两个relation中都有，所以还要**去重**。接下来再看一个复杂一点的例子。

<img src="img/union3.png" alt="img" style="zoom:43%;" />

假设这个是section relation。那么我们如果想要找出在Fall 2017**或者**在Spring 2018开设的**课的课程id**，应该怎么计算?一步步来：

* 选出所有Fall 2017的tuple。$\longrightarrow select$

* 选出所有Spring 2018的tuple。$\longrightarrow select$

* 分别将两个结果relation只选出`course_id`属性。$\longrightarrow project \times 2$

  *Fall 2017:*

  * *CS-101*
  * *CS-347*
  * *PHY-101*

  *Spring 2018:*

  * *CS-101*
  * *CS-315*
  * *CS-319*
  * ***CS-319***
  * *FIN-201*
  * *HIS-351*
  * *MU-199*

* 将两个选完的结果并起来。$\longrightarrow union$

其中，后面的两步是可以交换的。也就是说，可以先并起来再按着`course_id`去选也可以。

那么这整个过程的结果表达式可以写成：
$$
\Pi_{course\_id}(\sigma_{semester="Fall"\ \and\ year=2017}(section))\cup\Pi_{course\_id}(\sigma_{semester="Spring"\ \and\ year=2018}(section))
$$
最后的结果就是：

<img src="img/union4.png" alt="img" style="zoom:43%;" />

**这里依然要注意，CS-101和CS-319出现了2次，所以要去重**。

##### 2.1.2.4 Intersection

和上面几乎是一样的，所以这里不多说了。还是上面的例子，如果要计算：
$$
\Pi_{course\_id}(\sigma_{semester="Fall"\ \and\ year=2017}(section))\cap\Pi_{course\_id}(\sigma_{semester="Spring"\ \and\ year=2018}(section))
$$
结果就是:

<img src="img/in.png" alt="img" style="zoom:43%;" />

##### 2.1.2.5 Set-difference

还是上面的例子，如果我们要找在Fall2017开设而不在Spring 2018开设的课，应该怎么写？使用的就是Set-difference运算符。
$$
\Pi_{course\_id}(\sigma_{semester="Fall"\ \and\ year=2017}(section))-\Pi_{course\_id}(\sigma_{semester="Spring"\ \and\ year=2018}(section))
$$
其实结果一定是Fall 2017中的一部分。把在Spring 2018里也有份的那个刨掉就行了。

<img src="img/sd1.png" alt="img" style="zoom:50%;" />

##### 2.1.2.6 Cartesian-product

笛卡尔积还是稍微复杂一点的。我们看一个例子。

<img src="img/dk1.png" alt="img" style="zoom:43%;" />

我们把$r\times s$叫做笛卡尔积的结果。先拿出来$\alpha\ 1$，把它和s中的所有tuple都分别拼到一块；然后再拿出$\beta\ 2$做相同的操作。最终得到的relation就是笛卡尔积的结果。

<img src="img/dk2.png" alt="img" style="zoom:43%;" />

之前的操作符都多多少少有重复的问题。那么这个笛卡尔积这么可劲儿乘，肯定是有很大的隐患的。我们还是看之前老师和课的例子。

<img src="img/rm.png" alt="img" style="zoom:50%;" />|<img src="img/teaches.png" alt="img" style="zoom:50%;" />

左边是instructor relation，右边是teaches relation。那么我们能写出一个超级长的笛卡尔积结果：

<img src="img/dk3.png" alt="img" style="zoom:43%;" />

> $instructor \times teaches$

这个结果很显然有一个很大的问题：存在大量的非法数据。它们都是什么？我们来举个例子：比如Srinivasan老师。他的教职ID是10101，而我们看右边的teaches可以了解到，只有前3门课是他交的。而我们在笛卡尔积运算中，**却把所有的课都和他匹配了一遍**。因此除了这三门课以外的所有课都不是他交的，自然也就属于**非法数据**了。所以我们如果想要得到有效的结果，需要对这个笛卡尔积的结果**再进行一次select操作**：选择出`instructor.ID`和`teaches.ID`相等的tuple，也就是**这个课真就是这个老师交的**。
$$
\sigma_{instructor.ID=teaches.ID}(instructor \times teaches)
$$
<img src="img/dk4.png" alt="img" style="zoom:43%;" />

##### 2.1.2.7 Rename

比如关系r：

<img src="img/rename1.png" alt="img" style="zoom:43%;" />

如果我们要算$r\times r$，会有一个问题：结果的属性应该是4个。但是这俩r的属性都是A和B，那么我们怎么区分结果中的A和B是**前面r的**还是**后面r的**呢？这个时候只需要改个名字就好了：
$$
r \times \rho_s(r)
$$
这表示把后面的r改名为s。那么结果也就很好区分了：

<img src="img/rename2.png" alt="img" style="zoom: 67%;" />

##### 2.1.2.8 Join

在之前笛卡尔积的例子中，我们看到了对于两个表之间建立关系，然后再从中进行选择的操作。包括下面的Exercise也有这样的操作。其实这样的操作经常会见到，所以我们不妨直接给他重新命名，叫做**join**。非常形象，就是把两张表合成一张表。比如还是笛卡尔积的老师的例子中，我们得到的结果是：
$$
\sigma_{instructor.ID=teaches.ID}(instructor \times teaches)
$$
那么我们可以将这个写成连接操作：
$$
instructor \bowtie_{instructor.ID=teaches.ID} teaches
$$
总结一下，有两张表r和s，如果我们要按照一定的要求$\theta$将它们连接起来(也就是先笛卡尔积，然后按照这个要求来选择)，所得到的结果就是$r \bowtie_\theta s$，即
$$
r \bowtie_\theta s=\sigma_\theta(r \times s)
$$
~~如果这个条件是**某些属性的等值**(比如上面的`instructor.ID=teaches.ID`)~~，我们可以称这个连接为**自然连接**。但是自然连接有时候也会遇到一些问题。我们看下面一个例子：

<img src="img/join.png" alt="img" style="zoom: 67%;" />

如果我们要给`course`和`prereq`做自然连接的话，会发现一个事情：CS-315和CS-347这两门课的信息被完全抹去了。因为我们一定是用`course.course_id=prereq.course_id`来进行选择，而这两门课是不可能匹配上的，所以它们也自然是要被删除的tuple。这就产生了问题：比如拿CS-315来举例子。这是Robotics课，它有可能根本就没有前置课程，也就是null。这种情况下我们其实是需要这样的信息的，但是自然选择却把它给抹去了。所以我们需要新的方式来保留住这个信息。

**Natural left outer join**

这就是一种方式，叫做**自然左连接**。从上面的叙述能看出来，自然连接就是按照某些条件把两张表拼起来，所以我们可以先试试把两张表拼一拼：

| course_id | title       | dept_name | credits | prereq_id |
| --------- | ----------- | --------- | ------- | --------- |
| BIO-301   | Genetics    | Biology   | 4       | BIO-101   |
| CS-190    | Game Design | Comp.Sci. | 4       | CS-101    |
|           |             |           |         |           |

拼到这里的时候都是和自然连接一样的(因为两个course_id是重复的，所以删掉了)。但是接下来就出现了问题：本来下一行我们是要删掉的，但是我们却要留下来，所以这里我们采用自然左连接的方式。左就代表以左边的relation为基准，即**一定要留下它的所有信息**。比如我们这个表是这么构建的：
$$
(course)\ natural\ left\ outer\ join\ (prereq)
$$
那么我们就一定要留下course的所有信息：

| course_id | title       | dept_name | credits | prereq_id |
| --------- | ----------- | --------- | ------- | --------- |
| BIO-301   | Genetics    | Biology   | 4       | BIO-101   |
| CS-190    | Game Design | Comp.Sci. | 4       | CS-101    |
| CS-315    | Robotics    | Comp.Sci  | 3       | **null**  |

最后如果在另一张表里找不到，就写为null即可。

**Natural right outer join**

举一反三即可，以右边的为基准。还是刚才的例子，如果是这么构建的话：
$$
(course)\ natural\ right\ outer\ join\ (prereq)
$$
就保留prereq的所有信息即可：

| course_id | title       | dept_name | credits  | prereq_id |
| --------- | ----------- | --------- | -------- | --------- |
| BIO-301   | Genetics    | Biology   | 4        | BIO-101   |
| CS-190    | Game Design | Comp.Sci. | 4        | CS-101    |
| CS-347    | **null**    | **null**  | **null** | CS-101    |

**Natural full outer join**

既保留左边的，也保留右边的。还是一样的例子：

| course_id | title       | dept_name | credits  | prereq_id |
| --------- | ----------- | --------- | -------- | --------- |
| BIO-301   | Genetics    | Biology   | 4        | BIO-101   |
| CS-190    | Game Design | Comp.Sci. | 4        | CS-101    |
| CS-315    | Robotics    | Comp.Sci  | 3        | **null**  |
| CS-347    | **null**    | **null**  | **null** | CS-101    |

其实全外连接就是**先左，再右，最后并**。

**Inner join**

提一嘴就行。其实就是自然连接，只不过它**保留了相等的属性，没有删掉**。

![img](img/join2.png)

##### 2.1.2.9 Division

我们先来看一个复杂的例子。

![img](img/division.png)

> 图里最左边的表少画了一列，应该是customer_name和account_number，这张表叫depositor。

问：Find all customers who have an account at all branches located in Brooklyn city.

首先我们看Brooklyn，它对应的branch有Brighton和Downtown。所以我们的目标就是找到一个customer，**他同时拥有位于这两个branch的账户**。

按着上面的叙述来，第一步就是找出Brooklyn的所有账户：
$$
\sigma_{branch\_city="Brooklyn"}(branch)
$$
depositor那张表因为只有customer_name是有用的信息，account_number一看就是用来连接的。所以我们直接把depositor和account连接上。
$$
depositor \bowtie account
$$
然后问题来了。我们需要找的是Brooklyn中的branch，所以比较的重点就是这两张表中的branch_name属性。那么我们要怎么比？我们的目的是找到一个customer_name，他对应的branch_name会有很多个，**而这很多个一定要包括$\sigma_{branch\_city="Brooklyn"}(branch)$里全部的branch_name**。明确了这些，我们开始下面的操作。首先要去除无用的信息。在比较的过程中，我们发现只比较了$depositor \bowtie account$中的`customer_name`和`branch_name`，还有$\sigma_{branch\_city="Brooklyn"}(branch)$中的`branch_name`。所以我们要**分别把两张表做投影，把有用的属性筛出来**：
$$
\pi_{customer\_name,\ branch\_name}(depositor \bowtie account)\longrightarrow A1
$$

$$
\pi_{branch\_name}(\sigma_{branch\_city="Brooklyn"}(branch))\longrightarrow A2
$$

最后就是开始寻找：遍历A1中的customer_name，对于每个name，看其对应的**一个或多个**branch_name是否完全包括了A2中所有的branch_name。如果有，那么就将这个name添加到结果的relation中。

上面进行的操作，就叫做**division**。而这道题最终的结果也可以记为：
$$
\pi_{customer\_name,\ branch\_name}(depositor \bowtie account) \div \pi_{branch\_name}(\sigma_{branch\_city="Brooklyn"}(branch))
$$
然后来说一下division的定义。

我们有两个relation: r和s。

<img src="img/div1.png" alt="img" style="zoom: 67%;" />

被除的是s，而除它的是r。s中有B，而r中有A和B。我们规定，$r \div s$中的属性是**r中的属性中把s的扣掉**。那么在这个例子中，就是AB扣掉B，自然就剩A了。

然后我们要在r中去**按照A来进行遍历**。对于每一个A中的值，我们要看**它对应的所有B是否全部包括了s中的B**，如果包括，就将它添加到结果relation中。这里$\alpha$很显然在遍历1，2行的时候就已经包括了所有的s中的B，所以$\alpha$就是结果relation中的一个。另外经过完整遍历，我们发现$\beta$也满足，所以结果就是：

![img](img/div2.png)

通过这两个例子也能看出，division操作是用来解决relation中的包含关系的。也就是**第一个relation中的哪些tuple具有==包括第二个relation完整信息==的能力**。

**接下来再看一个例子：**

<img src="img/div3.png" alt="img" style="zoom: 67%;" />

这里要计算$r \div s$，还是按照之前的叙述来。首先结果中应该只有ABC三个属性，也就是ABCDE扣掉DE。

然后按着ABC开始逐行遍历，比如第一行$\alpha\ a\ \alpha$，看它对应的DE能不能包括所有的DE。而我们发现这整个表里只有一行是$\alpha\ a\ \alpha$，所以毫无疑问匹配失败。

然后是第二行$\alpha\ a\ \gamma$。它在2，3行正好就全包括了a l和b l，所以这个是结果relation中的一员。

经过所有遍历，我们也很容易写出结果：

![img](img/div4.png)

**还有一种特殊情况，就是两个relation的属性并非完全包括关系。**

![img](img/div5.png)

如果要计算$R \div S$的话，我们首先发现：XY扣YF扣不掉。但是没有关系，我们只需要**能扣多少扣多少**。也就是把Y给扣掉就行了，**所以结果中只有一个X**。

然后还是按着X对R进行逐行遍历。这里只有X1和X2。首先是X1，而整个R中只有这一个组合，所以它肯定不是结果。

然后是X2，发现这里有三个，而也完全包括了S中的Y1和Y2(**这里不需要管F了，它和除法操作无关**)，所以它就是结果。

![img](img/div6.png)

#### 2.1.3 Exercise

##### 2.1.3.1 Ex1

现在我们有一个银行的relation：

| account_number | Branch_name | balance |
| -------------- | ----------- | ------- |
| A-101          | Downtown    | 500     |
| A-215          | Mianus      | 700     |
| A-102          | Perryridge  | 400     |
| ...            | ...         | ...     |

现在有一个需求：找出余额最多的银行账户有多少**余额**。很显然，一个号在这里就是一个tuple。所以我们要选tuple，很容易想到select操作。但是我们再仔细想想：之前所有的select操作都是和一个人为的定值比较，比如大于5、小于等于100等等……我们这里的要求是要**余额和余额来比较**，显然单独的选择操作不能满足我们的要求。那么我们又能想到：笛卡尔积正好是构造两个relation的关系，那么我们就可以让这个relation和自己来一个笛卡尔积，就能将所有的balance之间构造好关系了。另外，为了区分，我们将两个relation命名为$A_1和A_2$(这里表格画不下，所以省略了一些东西)：

| A1.number | A1.name    | A1.balance | A2.number | A2.name    | A2.balance |
| --------- | ---------- | ---------- | --------- | ---------- | ---------- |
| A-101     | Downtown   | 500        | A-101     | Downtown   | 500        |
| A-101     | Downtown   | 500        | A-215     | Mianus     | 700        |
| A-101     | Downtown   | **500**    | A-102     | Perryridge | **400**    |
| A-215     | Mianus     | **700**    | A-101     | Downtown   | **500**    |
| A-215     | Mianus     | 700        | A-215     | Mianus     | 700        |
| A-215     | Mianus     | **700**    | A-102     | Perryridge | **400**    |
| A-102     | Perryridge | 400        | A-101     | Downtown   | 500        |
| A-102     | Perryridge | 400        | A-215     | Mianus     | 700        |
| A-102     | Perryridge | 400        | A-102     | Perryridge | 400        |
| ...       | ...        | ...        | ...       | ...        | ...        |

看`A1.balance`和`A2.balance`这两列，我们就能发现：所有的比较都已经含在这两列中，那么接下来我们也自然而然就能想到，按照`A1.balance > A2.balance`来进行select操作，就能选出大的：
$$
\sigma_{A1.balance>A2.balance}(\rho_{A1}(account) \times \rho_{A2}(account))
$$
但是注意看表格中加粗的部分：这三个tuple都满足`A1.balance > A2.balance`，这代表我们一次选择并不能选出最大的balance。我们再想一想我们到底选了个什么：当然，我们最后要筛出A1那一列，因为选择的时候我们是假定A1大的。但是我们在比较的时候，并不是比最大的，而是**只要我比他大，我就入选了**。那么其实我们并不是选了最大的，而是**只是没有选最小的，其他的都选了**。所以我们不妨反着来：将>改成<，这样我们就是只是没有选最大的，然后再用整个集合减去它，就把最大的留了下来，注意别忘了筛选出balance那列，我们要的只是余额而已：
$$
\Pi_{balance}(account)-\Pi_{A1.balance}(\sigma_{A1.balance<A2.balance}(\rho_{A1}(account) \times \rho_{A2}(account)))
$$

##### 2.1.3.2 Ex2

现在有两个relation:

![img](img/ex2.png)

问：Find all customers who have at least two deposits in different branch.

首先来分析一下。要求是同一个人，但是要是不同的账户，而且分支还得不一样。总结起来就是customer_name相同，account_number不同，并且branch_name也不同。和上面的Ex1一样涉及到了自己和自己比的操作(**比customer_name和branch_name**)，我们也能想到自然连接。所以先把这两个表连接起来：
$$
depositor \bowtie account
$$
这里的连接条件很显然，就是account_number相等。连接之后我们就有了一个拥有4个attribute的relation：
$$
depositor \bowtie account(customer\_name,\ account\_number,\ branch\_name,\ balance)
$$
接下来我们要想一想：这里是自己和自己比的操作，而现在只有这一张表。很显然我们要进行一遍笛卡尔积才可以。所以我们要让这个结果和自己来个笛卡尔积，并且使用不同的命名：
$$
\rho_{D1}(depositor \bowtie account)\times\rho_{D2}(depositor \bowtie account)
$$
这样我们又得到了一个新的relation，表示$depositor \bowtie account$这个relation里面所有的tuple之间建立的关系。接下来，我们要从中筛选出符合题意的tuple，也就是那三个条件。
$$
\sigma_{D1.customer\_name=D2.customer\_name\ \and\ D1.account\_number \neq D2.account\_number\ \and\ D1.branch\_name\neq D2.branch\_name}\\
(\rho_{D1}(depositor \bowtie account)\times\rho_{D2}(depositor \bowtie account))
$$
最后题目要求的是找出customer即可，所以要投影出名字。
$$
\pi_{D1.customer\_name}(\\
\sigma_{D1.customer\_name=D2.customer\_name\ \and\ D1.account\_number \neq D2.account\_number\ \and\ D1.branch\_name\neq D2.branch\_name}\\
(\rho_{D1}(depositor \bowtie account)\times\rho_{D2}(depositor \bowtie account))\\
)
$$
本题和Ex1很像，只不过最后的比较条件由一个变成了三个。另外，这道题用了两次"笛卡尔积+选择"，而Ex1只用了一次。**这两次一次是为了将两张表和成一张；另一次是在这张合成的表中进行自我比较**。

***剩下的练习题见录播第二集和第三集***

##  3. SQL
