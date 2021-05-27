# 程序语言世界的地图
<!--
> You must have a map, no matter how rough. Otherwise you wander all over the
> place. In *The Lord of the Rings* I never made anyone go farther than he could
> on a given day.
>
> <cite>J. R. R. Tolkien</cite>
-->

> 你手中得握着一张地图，不管这张地图有多么简陋。要不然，你会在路途中迷失方向。在《魔戒》这部书里，我不会让任何人走超过他当天所能走过的最远的路。
> <cite>约翰·罗纳德·瑞尔·托尔金</cite>

<!--
We don't want to wander all over the place, so before we set off, let's scan
the territory charted by previous language implementers. It will help us
understand where we are going and the alternate routes others have taken.
-->
我们可不想在旅途中迷失方向，所以在正式启程出发之前，让我们好好翻看前辈们为我们绘制的地图，认清我们此行应去往何处，以及前辈们已经开拓出的道路。

<!--
First, let me establish a shorthand. Much of this book is about a language's
*implementation*, which is distinct from the *language itself* in some sort of
Platonic ideal form. Things like "stack", "bytecode", and "recursive descent",
are nuts and bolts one particular implementation might use. From the user's
perspective, as long as the resulting contraption faithfully follows the
language's specification, it's all implementation detail.
-->
首先，让我来简单明确几点。本书大部分内容都是关于一门程序设计语言的*具体实现*，这与*程序设计语言本身*是完全不同的。像“栈”、“字节码”、“递归下降”这些听起来唬人的词语，不过是实现某门程序语言可能会采用的的技术方法。从程序语言使用者的视角来看，只要他们手中的“奇妙装置”（编译器 / 解释器）产出的结果符合程序设计语言定义的规范即可，并不太关心语言实现的具体细节。

<!--
We're going to spend a lot of time on those details, so if I have to write
"language *implementation*" every single time I mention them, I'll wear my
fingers off. Instead, I'll use "language" to refer to either a language or an
implementation of it, or both, unless the distinction matters.
-->
我们将在程序语言实现的具体细节上花上很多时间，如果我每次在提及到它的时候都严格写下“程序设计语言的实现”一词，那我的手指可真受不了（字符实在是太长啦）。所以，除非有严格区分的必要，我会直接使用“语言”一词指代程序语言或是程序语言的实现。

<!--
-- The Parts of a Language
-->
## 程序语言面面观

<!--
Engineers have been building programming languages since the Dark Ages of
computing. As soon as we could talk to computers, we discovered doing so was too
hard, and we enlisted their help. I find it fascinating that even though today's
machines are literally a million times faster and have orders of magnitude more
storage, the way we build programming languages is virtually unchanged.
-->
早在计算机历史的洪荒时代，勤劳聪慧的计算机工程师们就开始着手构建计算机程序设计语言了。这是因为我们发现，直接与计算机沟通太过困难，计算机只认识二进制机器码，所以得寻求“程序语言”的帮助。现如今的计算机比起洪荒时代，运算速度快了百万倍千万倍，存储空间也有了几个数量级的提升，但构建计算机程序设计语言的方式却几乎没有太大变化，这真是很有趣。

<!--
Though the area explored by language designers is vast, the trails they've
carved through it are <span name="dead">few</span>. Not every language takes the
exact same path -- some take a shortcut or two -- but otherwise they are
reassuringly similar, from Rear Admiral Grace Hopper's first COBOL compiler all
the way to some hot, new, transpile-to-JavaScript language whose "documentation"
consists entirely of a single, poorly edited README in a Git repository
somewhere.
-->
尽管程序语言设计师们探索过的领域如此广阔，但被他们留下足迹证明此路可通的道路却<span name="dead">少之又少</span>。虽然不是每一门程序语言都会走完全相同的实现道路（有的语言会省略其中几步，有的还会抄近路）。但令人欣慰的是，从海军准将葛丽丝·霍普女士的第一支 COBOL 编译器到时下大热的编译到 JavaScript 程序语言（它们中某些仅有一份躺在 Git 仓库中的简陋 README 自述文档），都遵循着一脉相承的道路。

<aside name="dead">

<!--
There are certainly dead ends, sad little cul-de-sacs of CS papers with zero
citations and now-forgotten optimizations that only made sense when memory was
measured in individual bytes.
-->
其中很多都非常可惜地拐入了死胡同，比如说那些无人问津，论述极端内存情况下才可见效的程序语言优化方法的计算机论文。

</aside>

<!--
I visualize the network of paths an implementation may choose as climbing a
mountain. You start off at the bottom with the program as raw source text,
literally just a string of characters. Each phase analyzes the program and
transforms it to some higher-level representation where the semantics -- what
the author wants the computer to do -- become more apparent.
-->
我把程序语言的编译过程绘制成了一副路径图，看着就像登山一样。从山脚开始，开发者所写的程序只是一串字符序列构成的纯文本。之后的每一阶段，程序都会被分析、转化为某种更为高阶的表示形式，程序语义（即：程序员想让计算机做的事）也变得越来越明晰。

<!--
Eventually we reach the peak. We have a bird's-eye view of the user's program
and can see what their code *means*. We begin our descent down the other side of
the mountain. We transform this highest-level representation down to
successively lower-level forms to get closer and closer to something we know how
to make the CPU actually execute.
-->
当我们终于到达到山顶的时候，我们就可以鸟瞰全局，完全理解程序代码想要表达的真正含义。随后，我们从另一侧下山。下山的过程即是把程序的高阶表示形式一步步转化为越来越靠近 CPU 的低阶表示形式，最终到达 CPU 可以执行的机器码。

<img src="image/a-map-of-the-territory/mountain.png" alt="The branching paths a language may take over the mountain." class="wide" />

<!--
Let's trace through each of those trails and points of interest. Our journey
begins on the left with the bare text of the user's source code:
-->
让我们通过一个实例来追踪程序编译的整个过程，看一看在每个编译阶段都会发生哪些有趣的事情。我们从下面这行用户程序源代码开始：

<img src="image/a-map-of-the-territory/string.png" alt="var average = (min + max) / 2;" />

### 词法扫描（Scanning）

<!--
The first step is **scanning**, also known as **lexing**, or (if you're trying
to impress someone) **lexical analysis**. They all mean pretty much the same
thing. I like "lexing" because it sounds like something an evil supervillain
would do, but I'll use "scanning" because it seems to be marginally more
commonplace.
-->
万里长征的第一步是**词法扫描（Scanning）**，也叫做**词法化（Lexing）**，或**词法分析（Lexical Analysis）**。它们都表示同一个意思，如果你想让旁人印象深刻，可以用听上去最为惊艳的“词法分析”。我个人比较喜欢“词法化”，这听上去就像是某个超级大恶魔会才会做的事情。但在这里，我还是使用“词法扫描”，因为“扫描（Scanning）”一词看起来更为平易近人。

<!--
A **scanner** (or **lexer**) takes in the linear stream of characters and chunks
them together into a series of something more akin to <span
name="word">"words"</span>. In programming languages, each of these words is
called a **token**. Some tokens are single characters, like `(` and `,`. Others
may be several characters long, like numbers (`123`), string literals (`"hi!"`),
and identifiers (`min`).
-->
**词法扫描器（Scanner）**，或者说**分词器（Lexer）**，读取一连串程序源代码字符流，按照一定的规则将代码字符分割为一系列的<span name="words">“词”</span>。在程序设计语言领域，我们把这些“词”称之为<span name="token">**词法单元（Token）**</span>。有些词法单元仅有单个字符，如：左小括号`(`、逗号`,`。有些词法单元则由多个字符组成，如：数字`123`、字符串字面量`"hi!"`、标识符`min`。

<aside name="word">

<!--
"Lexical" comes from the Greek root "lex", meaning "word".
-->
“Lexical”一词来源于希腊语“lex”，意为“单词”。

</aside>

<aside name="token">

> 译者注：本书接下来的篇章都将直接使用 token 一词而非其译文”词法单元“。

</aside>

<!--
Some characters in a source file don't actually mean anything. Whitespace is
often insignificant, and comments, by definition, are ignored by the language.
The scanner usually discards these, leaving a clean sequence of meaningful
tokens.
-->
程序源代码中的某些字符一般不表达任何含义，比如：空白字符、注释。词法扫描器通常简单的忽略它们，仅保留那些有意义的词法单元。

<img src="image/a-map-of-the-territory/tokens.png" alt="[var] [average] [=] [(] [min] [+] [max] [)] [/] [2] [;]" />

<!--
--- Parsing
-->
### 语法分析（Parsing）

<!--
The next step is **parsing**. This is where our syntax gets a **grammar** -- the
ability to compose larger expressions and statements out of smaller parts. Did
you ever diagram sentences in English class? If so, you've done what a parser
does, except that English has thousands and thousands of "keywords" and an
overflowing cornucopia of ambiguity. Programming languages are much simpler.
-->
接下来的一步叫做**语法分析（Parsing）**。通过分析程序的词法从而得到**语法（Grammar）**，将相互关联的几个词法单元拼装组合变成更加复杂的表达式和语句的能力。还记得小时候上语法课时的情景吗，标注一句话的语法成分（主、谓、宾、定、状、补），你那时做的事便是“语法分析”。只是，英语这门语言包含着成千上万的“关键字（Keywords）”，以及大量意思相近的歧义词。程序语言则简单很多。

<!--
A **parser** takes the flat sequence of tokens and builds a tree structure that
mirrors the nested nature of the grammar. These trees have a couple of different
names -- **parse tree** or **abstract syntax tree** -- depending on how
close to the bare syntactic structure of the source language they are. In
practice, language hackers usually call them **syntax trees**, **ASTs**, or
often just **trees**.
-->
**语法解析器（Parser）** 读入 token 词法单元序列，构建出符合当前代码语义的树型结构，因为语法是嵌套的，所以要用树来表示。这些树型结构有着很多不同的名字：**解析树（Parse Tree）**、**抽象语法树（Abstract Syntax Tree）**，具体叫什么取决于这棵树与程序语言语法结构之间的对应程度。在实践中，程序语言开发者们习惯称呼它们为**语法树（Syntax Tree）** 或 **抽象语法树（AST）**，也有人为了简单起见直接叫**树（Tree）**。

<img src="image/a-map-of-the-territory/ast.png" alt="An abstract syntax tree." />

<!--
Parsing has a long, rich history in computer science that is closely tied to the
artificial intelligence community. Many of the techniques used today to parse
programming languages were originally conceived to parse *human* languages by AI
researchers who were trying to get computers to talk to us.
-->
在计算机科学领域，语法解析技术有着漫长且丰富的历史。语法解析还与人工智能有着千丝万缕的联系。现如今程序语言所用的很多语法解析技术，最初都是 AI 研究者们为解析*人类语言*而发明的。他们想让计算机能够与我们交流对话。

<!--
It turns out human languages were too messy for the rigid grammars those parsers
could handle, but they were a perfect fit for the simpler artificial grammars of
programming languages. Alas, we flawed humans still manage to use those simple
grammars incorrectly, so the parser's job also includes letting us know when we
do by reporting **syntax errors**.
-->
后来，研究者们发现，人类所使用的语言实在太过复杂，仅能处理严谨语法的语法解析器处理不了如此复杂的人类语言。但是语法解析器非常适合用来处理语法严谨、毫无歧义的程序语言，毕竟，程序语言可比人类语言简单多了。但即使是使用语法如此简单的程序语言，我们程序员还是会在写代码时犯各式各样的语法错误。所以语法解析器的职责又多了一条：报告程序员们所犯的**语法错误**，让他们改正。

<!--
--- Static analysis
-->
### 静态分析（Static analysis）

<!--
The first two stages are pretty similar across all implementations. Now, the
individual characteristics of each language start coming into play. At this
point, we know the syntactic structure of the code -- things like which
expressions are nested in which -- but we don't know much more than that.
-->
几乎所有的程序设计语言实现都会经历词法分析、语法分析这两个阶段。那么在此之后，不同的程序语言就要开始展现其各自实现特点了。在当下这个节点，我们仅知道程序代码的语法结构，如：表达式之间的嵌套关系等。除此之外就没有更多的信息了。

<!--
In an expression like `a + b`, we know we are adding `a` and `b`, but we don't
know what those names refer to. Are they local variables? Global? Where are they
defined?
-->
如有表达式`a + b`，我们知道需要对`a`和`b`执行加法运算，但是我们不知道`a`和`b`这两个名字到底指向哪里，指向本地变量还是全局变量？变量定义在何处？

<!--
The first bit of analysis that most languages do is called **binding** or
**resolution**. For each **identifier**, we find out where that name is defined
and wire the two together. This is where **scope** comes into play -- the region
of source code where a certain name can be used to refer to a certain
declaration.
-->
对于大多数程序语言而言，分析语法树的第一步叫做 **标识符绑定（Binding）** 或 **标识符解析（Resolution）** ：对于语法树中的每个标识符，找到这枚标识符被定义的地方，把它和语法树中标识符所在的位置串连起来。这个时候 **作用域（Scope）** 就开始发挥作用了，作用域规定了一枚既定标识符在源代码中所能被正确解析的区域。

<!--
If the language is <span name="type">statically typed</span>, this is when we
type check. Once we know where `a` and `b` are declared, we can also figure out
their types. Then if those types don't support being added to each other, we
report a **type error**.
-->
如果这是一门<span name="type">静态类型</span>的程序语言，此时也是我们做类型检查的时候。一旦我们找到了`a`和`b`在何处声明，我们就可以很容易地确定它们的类型。如果`a`、`b`类型之间无法相加，那么我们就可以抛出一个**类型错误**。

<aside name="type">

<!--
The language we'll build in this book is dynamically typed, so it will do its
type checking later, at runtime.
-->
在本书中我们要实现的程序语言是动态类型的，类型检查这步将被推后到运行时。

</aside>

<!--
Take a deep breath. We have attained the summit of the mountain and a sweeping
view of the user's program. All this semantic insight that is visible to us from
analysis needs to be stored somewhere. There are a few places we can squirrel it
away:
-->
做个深呼吸，我们已经抵达了山顶，对用户程序也有了一个全面的认知。从静态语义分析中得到的讯息都需要被妥善存储起来。有一些地方可供我们存放这些讯息：

<!--
* Often, it gets stored right back as **attributes** on the syntax tree
  itself -- extra fields in the nodes that aren't initialized during parsing
  but get filled in later.
-->
* 通常，我们可以把这些讯息作为语法树节点的 **属性（Attributes）** 存储起来。为树节点添加一些额外属性字段，在构建语法树时将字段留空，后续再将静态语义分析得到的信息填充进去。

<!--
* Other times, we may store data in a lookup table off to the side. Typically,
  the keys to this table are identifiers -- names of variables and declarations.
  In that case, we call it a **symbol table** and the values it associates with
  each key tell us what that identifier refers to.
-->
* 另外，我们还可以将这些数据存储到一张查找表里，以标识符（变量名、函数名、变量声明、函数声明等等）作键，标识符语义信息为值。如此一来，我们就可以轻松找到某个标识符的引用信息了。我们把这张表称为：**符号表（Symbol table）**。

<!--
* The most powerful bookkeeping tool is to transform the tree into an entirely
  new data structure that more directly expresses the semantics of the code.
  That's the next section.
-->
* 最为强大的信息记录工具就是将语法树转化为一种全新的数据结构，这种数据结构可以更为直接地表达代码的语义信息。下一节就将介绍这部分的内容。

<!--
Everything up to this point is considered the **front end** of the
implementation. You might guess everything after this is the **back end**, but
no. Back in the days of yore when "front end" and "back end" were coined,
compilers were much simpler. Later researchers invented new phases to stuff
between the two halves. Rather than discard the old terms, William Wulf and
company lumped those new phases into the charming but spatially paradoxical name
**middle end**.
-->
到目前为止，我们所讲的词法分析、语法分析、语义分析三阶段，通常被归类为编译 **“前端（Front end）”** 。你也许会想，那之后的流程都是编译 **“后端（Back end）”** 了，可惜不是。在过去那个只有“前端”和“后端”的年代，编译器比现在简单许多。后来，程序语言研究者们在“前端”和“后端”之间插入了几个新阶段。威廉·伍尔夫教授和一众大企业将这些新阶段汇总打包，遵照古老的命名习惯，起了个魅力十足但又稍显矛盾的名字：**“中端（Middle end）”** 。

<!--
--- Intermediate representations
-->
### 中间代码表示（Intermediate representations）

<!--
You can think of the compiler as a pipeline where each stage's job is to
organize the data representing the user's code in a way that makes the next
stage simpler to implement. The front end of the pipeline is specific to the
source language the program is written in. The back end is concerned with the
final architecture where the program will run.
-->
你可以把编译器想象为一条流水线，流水线的每一道工序都将用户代码加工成某种特殊的数据表示形式，方便下一道工序使用。编译流水线“前端”主要关注程序的源代码结构（词法、语法、语义），“后端”则主要关注程序最终执行平台的体系架构。

<!--
In the middle, the code may be stored in some <span name="ir">**intermediate
representation**</span> (**IR**) that isn't tightly tied to either the source or
destination forms (hence "intermediate"). Instead, the IR acts as an interface
between these two languages.
-->
在编译“中端”，程序代码可能会被存储为某种<span name="ir">**中间代码（Intermediate representations、IR）**</span>形式，既不贴近源代码也不贴近机器码，处于两者之间（所以称为“中间代码表示”）。IR 中间代码更像是介于源代码与二进制机器码之间的接口。

<aside name="ir">

<!--
There are a few well-established styles of IRs out there. Hit your search engine
of choice and look for "control flow graph", "static single-assignment",
"continuation-passing style", and "three-address code".
-->
目前，已经有许多精巧的 IR 中间代码设计方法论存在了。你可以打开浏览器搜索：“控制流图（Control-flow Graph、CFG）”，“静态单赋值形式（Static single-assignment、SSA）”，“续体传递风格（Continuation-passing style、CPS）”，“三位址码（Three-address code、TAC、3AC）”。

</aside>

有了 IR 中间代码的帮助，在支持多程序源码解析与多目标平台编译方面，就可以省下很多力气。比如说，你想要实现 Pascal，C，Fortran 这三门程序语言的编译器，期望程序编译到的目标平台为 x86，ARM，SPARC。通常情况下，这意味着你得编写*9支*完整的编译器：Pascal&rarr;x86，C&rarr;ARM，以及其他排列组合。
<!--
This lets you support multiple source languages and target platforms with less
effort. Say you want to implement Pascal, C, and Fortran compilers, and you want
to target x86, ARM, and, I dunno, SPARC. Normally, that means you're signing up
to write *nine* full compilers: Pascal&rarr;x86, C&rarr;ARM, and every other
combination.
-->

<!--
A <span name="gcc">shared</span> intermediate representation reduces that
dramatically. You write *one* front end for each source language that produces
the IR. Then *one* back end for each target architecture. Now you can mix and
match those to get every combination.
-->
通过借助一个共享的 IR 中间代码表示，我们的工作量将大大减少。只需要为每一门程序语言编写一个“前端”，将程序源代码编译为中间代码表示，再为每一个目标平台编写一个“后端”，将中间代码编译到目标平台的二进制机器码。之后，我们的任务就完成了，所有排列组合都覆盖得到。

<aside name="gcc">

现在你知道为什么 [GCC][] 可以支持那么多奇奇怪怪的程序语言和目标平台了吧（像是偏门的 Modula-3 程序语言和摩托罗拉 68k 目标平台）。语言“前端”先将程序源代码编译成各式各样的 IR 中间代码，比较常见的有[GIMPLE][] 和 [RTL][]，编译”后端”再将这些 IR 中间代码编译到目标平台的二进制机器码。
<!--
If you've ever wondered how [GCC][] supports so many crazy languages and
architectures, like Modula-3 on Motorola 68k, now you know. Language front ends
target one of a handful of IRs, mainly [GIMPLE][] and [RTL][]. Target back ends
like the one for 68k then take those IRs and produce native code.
-->

[gcc]: https://en.wikipedia.org/wiki/GNU_Compiler_Collection
[gimple]: https://gcc.gnu.org/onlinedocs/gccint/GIMPLE.html
[rtl]: https://gcc.gnu.org/onlinedocs/gccint/RTL.html

</aside>

<!--
There's another big reason we might want to transform the code into a form that
makes the semantics more apparent...
-->
另一个促使我们将代码转化为 IR 中间代码表示的重要原因是：使用 IR 中间代码可以使程序语义变得更加清晰...

<!--
--- Optimization
-->
### 代码优化（Optimization）

<!--
Once we understand what the user's program means, we are free to swap it out
with a different program that has the *same semantics* but implements them more
efficiently -- we can **optimize** it.
-->
当我们完全理解用户程序想要表达的含义之后，我们就可以放心地将用户源程序替换为一支*语义完全相同*、*运行效率更高*的新程序。也就是说，我们可以**优化**程序代码。

A simple example is **constant folding**: if some expression always evaluates to
the exact same value, we can do the evaluation at compile time and replace the
code for the expression with its result. If the user typed in this:

```java
pennyArea = 3.14159 * (0.75 / 2) * (0.75 / 2);
```

we could do all of that arithmetic in the compiler and change the code to:

```java
pennyArea = 0.4417860938;
```

Optimization is a huge part of the programming language business. Many language
hackers spend their entire careers here, squeezing every drop of performance
they can out of their compilers to get their benchmarks a fraction of a percent
faster. It can become a sort of obsession.

We're mostly going to <span name="rathole">hop over that rathole</span> in this
book. Many successful languages have surprisingly few compile-time
optimizations. For example, Lua and CPython generate relatively unoptimized
code, and focus most of their performance effort on the runtime.

<aside name="rathole">

If you can't resist poking your foot into that hole, some keywords to get you
started are "constant propagation", "common subexpression elimination", "loop
invariant code motion", "global value numbering", "strength reduction", "scalar
replacement of aggregates", "dead code elimination", and "loop unrolling".

</aside>

### Code generation

We have applied all of the optimizations we can think of to the user's program.
The last step is converting it to a form the machine can actually run. In other
words, **generating code** (or **code gen**), where "code" here usually refers to
the kind of primitive assembly-like instructions a CPU runs and not the kind of
"source code" a human might want to read.

Finally, we are in the **back end**, descending the other side of the mountain.
From here on out, our representation of the code becomes more and more
primitive, like evolution run in reverse, as we get closer to something our
simple-minded machine can understand.

We have a decision to make. Do we generate instructions for a real CPU or a
virtual one? If we generate real machine code, we get an executable that the OS
can load directly onto the chip. Native code is lightning fast, but generating
it is a lot of work. Today's architectures have piles of instructions, complex
pipelines, and enough <span name="aad">historical baggage</span> to fill a 747's
luggage bay.

Speaking the chip's language also means your compiler is tied to a specific
architecture. If your compiler targets [x86][] machine code, it's not going to
run on an [ARM][] device. All the way back in the '60s, during the
Cambrian explosion of computer architectures, that lack of portability was a
real obstacle.

<aside name="aad">

For example, the [AAD][] ("ASCII Adjust AX Before Division") instruction lets
you perform division, which sounds useful. Except that instruction takes, as
operands, two binary-coded decimal digits packed into a single 16-bit register.
When was the last time *you* needed BCD on a 16-bit machine?

[aad]: http://www.felixcloutier.com/x86/AAD.html

</aside>

[x86]: https://en.wikipedia.org/wiki/X86
[arm]: https://en.wikipedia.org/wiki/ARM_architecture

To get around that, hackers like Martin Richards and Niklaus Wirth, of BCPL and
Pascal fame, respectively, made their compilers produce *virtual* machine code.
Instead of instructions for some real chip, they produced code for a
hypothetical, idealized machine. Wirth called this **p-code** for *portable*,
but today, we generally call it **bytecode** because each instruction is often a
single byte long.

These synthetic instructions are designed to map a little more closely to the
language's semantics, and not be so tied to the peculiarities of any one
computer architecture and its accumulated historical cruft. You can think of it
like a dense, binary encoding of the language's low-level operations.

### Virtual machine

If your compiler produces bytecode, your work isn't over once that's done. Since
there is no chip that speaks that bytecode, it's your job to translate. Again,
you have two options. You can write a little mini-compiler for each target
architecture that converts the bytecode to native code for that machine. You
still have to do work for <span name="shared">each</span> chip you support, but
this last stage is pretty simple and you get to reuse the rest of the compiler
pipeline across all of the machines you support. You're basically using your
bytecode as an intermediate representation.

<aside name="shared" class="bottom">

The basic principle here is that the farther down the pipeline you push the
architecture-specific work, the more of the earlier phases you can share across
architectures.

There is a tension, though. Many optimizations, like register allocation and
instruction selection, work best when they know the strengths and capabilities
of a specific chip. Figuring out which parts of your compiler can be shared and
which should be target-specific is an art.

</aside>

Or you can write a <span name="vm">**virtual machine**</span> (**VM**), a
program that emulates a hypothetical chip supporting your virtual architecture
at runtime. Running bytecode in a VM is slower than translating it to native
code ahead of time because every instruction must be simulated at runtime each
time it executes. In return, you get simplicity and portability. Implement your
VM in, say, C, and you can run your language on any platform that has a C
compiler. This is how the second interpreter we build in this book works.

<aside name="vm">

The term "virtual machine" also refers to a different kind of abstraction. A
**system virtual machine** emulates an entire hardware platform and operating
system in software. This is how you can play Windows games on your Linux
machine, and how cloud providers give customers the user experience of
controlling their own "server" without needing to physically allocate separate
computers for each user.

The kind of VMs we'll talk about in this book are **language virtual machines**
or **process virtual machines** if you want to be unambiguous.

</aside>

### Runtime

We have finally hammered the user's program into a form that we can execute. The
last step is running it. If we compiled it to machine code, we simply tell the
operating system to load the executable and off it goes. If we compiled it to
bytecode, we need to start up the VM and load the program into that.

In both cases, for all but the basest of low-level languages, we usually need
some services that our language provides while the program is running. For
example, if the language automatically manages memory, we need a garbage
collector going in order to reclaim unused bits. If our language supports
"instance of" tests so you can see what kind of object you have, then we need
some representation to keep track of the type of each object during execution.

All of this stuff is going at runtime, so it's called, appropriately, the
**runtime**. In a fully compiled language, the code implementing the runtime
gets inserted directly into the resulting executable. In, say, [Go][], each
compiled application has its own copy of Go's runtime directly embedded in it.
If the language is run inside an interpreter or VM, then the runtime lives
there. This is how most implementations of languages like Java, Python, and
JavaScript work.

[go]: https://golang.org/

## Shortcuts and Alternate Routes

That's the long path covering every possible phase you might implement. Many
languages do walk the entire route, but there are a few shortcuts and alternate
paths.

### Single-pass compilers

Some simple compilers interleave parsing, analysis, and code generation so that
they produce output code directly in the parser, without ever allocating any
syntax trees or other IRs. These <span name="sdt">**single-pass
compilers**</span> restrict the design of the language. You have no intermediate
data structures to store global information about the program, and you don't
revisit any previously parsed part of the code. That means as soon as you see
some expression, you need to know enough to correctly compile it.

<aside name="sdt">

[**Syntax-directed translation**][pass] is a structured technique for building
these all-at-once compilers. You associate an *action* with each piece of the
grammar, usually one that generates output code. Then, whenever the parser
matches that chunk of syntax, it executes the action, building up the target
code one rule at a time.

[pass]: https://en.wikipedia.org/wiki/Syntax-directed_translation

</aside>

Pascal and C were designed around this limitation. At the time, memory was so
precious that a compiler might not even be able to hold an entire *source file*
in memory, much less the whole program. This is why Pascal's grammar requires
type declarations to appear first in a block. It's why in C you can't call a
function above the code that defines it unless you have an explicit forward
declaration that tells the compiler what it needs to know to generate code for a
call to the later function.

### Tree-walk interpreters

Some programming languages begin executing code right after parsing it to an AST
(with maybe a bit of static analysis applied). To run the program, the
interpreter traverses the syntax tree one branch and leaf at a time, evaluating
each node as it goes.

This implementation style is common for student projects and little languages,
but is not widely used for <span name="ruby">general-purpose</span> languages
since it tends to be slow. Some people use "interpreter" to mean only these
kinds of implementations, but others define that word more generally, so I'll
use the inarguably explicit **tree-walk interpreter** to refer to these. Our
first interpreter rolls this way.

<aside name="ruby">

A notable exception is early versions of Ruby, which were tree walkers. At 1.9,
the canonical implementation of Ruby switched from the original MRI (Matz's Ruby
Interpreter) to Koichi Sasada's YARV (Yet Another Ruby VM). YARV is a
bytecode virtual machine.

</aside>

### Transpilers

<span name="gary">Writing</span> a complete back end for a language can be a lot
of work. If you have some existing generic IR to target, you could bolt your
front end onto that. Otherwise, it seems like you're stuck. But what if you
treated some other *source language* as if it were an intermediate
representation?

You write a front end for your language. Then, in the back end, instead of doing
all the work to *lower* the semantics to some primitive target language, you
produce a string of valid source code for some other language that's about as
high level as yours. Then, you use the existing compilation tools for *that*
language as your escape route off the mountain and down to something you can
execute.

They used to call this a **source-to-source compiler** or a **transcompiler**.
After the rise of languages that compile to JavaScript in order to run in the
browser, they've affected the hipster sobriquet **transpiler**.

<aside name="gary">

The first transcompiler, XLT86, translated 8080 assembly into 8086 assembly.
That might seem straightforward, but keep in mind the 8080 was an 8-bit chip and
the 8086 a 16-bit chip that could use each register as a pair of 8-bit ones.
XLT86 did data flow analysis to track register usage in the source program and
then efficiently map it to the register set of the 8086.

It was written by Gary Kildall, a tragic hero of computer science if there
ever was one. One of the first people to recognize the promise of
microcomputers, he created PL/M and CP/M, the first high-level language and OS
for them.

He was a sea captain, business owner, licensed pilot, and motorcyclist. A TV
host with the Kris Kristofferson-esque look sported by dashing bearded dudes in
the '80s. He took on Bill Gates and, like many, lost, before meeting his end in
a biker bar under mysterious circumstances. He died too young, but sure as hell
lived before he did.

</aside>

While the first transcompiler translated one assembly language to another,
today, most transpilers work on higher-level languages. After the viral spread
of UNIX to machines various and sundry, there began a long tradition of
compilers that produced C as their output language. C compilers were available
everywhere UNIX was and produced efficient code, so targeting C was a good way
to get your language running on a lot of architectures.

Web browsers are the "machines" of today, and their "machine code" is
JavaScript, so these days it seems [almost every language out there][js] has a
compiler that targets JS since that's the <span name="js">main</span> way to get
your code running in a browser.

[js]: https://github.com/jashkenas/coffeescript/wiki/list-of-languages-that-compile-to-js

<aside name="js">

JS used to be the *only* way to execute code in a browser. Thanks to
[WebAssembly][], compilers now have a second, lower-level language they can
target that runs on the web.

[webassembly]: https://github.com/webassembly/

</aside>

The front end -- scanner and parser -- of a transpiler looks like other
compilers. Then, if the source language is only a simple syntactic skin over the
target language, it may skip analysis entirely and go straight to outputting the
analogous syntax in the destination language.

If the two languages are more semantically different, you'll see more of the
typical phases of a full compiler including analysis and possibly even
optimization. Then, when it comes to code generation, instead of outputting some
binary language like machine code, you produce a string of grammatically correct
source (well, destination) code in the target language.

Either way, you then run that resulting code through the output language's
existing compilation pipeline, and you're good to go.

### Just-in-time compilation

This last one is less a shortcut and more a dangerous alpine scramble best
reserved for experts. The fastest way to execute code is by compiling it to
machine code, but you might not know what architecture your end user's machine
supports. What to do?

You can do the same thing that the HotSpot Java Virtual Machine (JVM),
Microsoft's Common Language Runtime (CLR), and most JavaScript interpreters do.
On the end user's machine, when the program is loaded -- either from source in
the case of JS, or platform-independent bytecode for the JVM and CLR -- you
compile it to native for the architecture their computer supports. Naturally
enough, this is called **just-in-time compilation**. Most hackers just say
"JIT", pronounced like it rhymes with "fit".

The most sophisticated JITs insert profiling hooks into the generated code to
see which regions are most performance critical and what kind of data is flowing
through them. Then, over time, they will automatically recompile those <span
name="hot">hot spots</span> with more advanced optimizations.

<aside name="hot">

This is, of course, exactly where the HotSpot JVM gets its name.

</aside>

## Compilers and Interpreters

Now that I've stuffed your head with a dictionary's worth of programming
language jargon, we can finally address a question that's plagued coders since
time immemorial: What's the difference between a compiler and an interpreter?

It turns out this is like asking the difference between a fruit and a vegetable.
That seems like a binary either-or choice, but actually "fruit" is a *botanical*
term and "vegetable" is *culinary*. One does not strictly imply the negation of
the other. There are fruits that aren't vegetables (apples) and vegetables that
aren't fruits (carrots), but also edible plants that are both fruits *and*
vegetables, like tomatoes.

<span name="veg"></span>

<img src="image/a-map-of-the-territory/plants.png" alt="A Venn diagram of edible plants" />

<aside name="veg">

Peanuts (which are not even nuts) and cereals like wheat are actually fruit, but
I got this drawing wrong. What can I say, I'm a software engineer, not a
botanist. I should probably erase the little peanut guy, but he's so cute that I
can't bear to.

Now *pine nuts*, on the other hand, are plant-based foods that are neither
fruits nor vegetables. At least as far as I can tell.

</aside>

So, back to languages:

* **Compiling** is an *implementation technique* that involves translating a
  source language to some other -- usually lower-level -- form. When you
  generate bytecode or machine code, you are compiling. When you transpile to
  another high-level language, you are compiling too.

* When we say a language implementation "is a **compiler**", we mean it
  translates source code to some other form but doesn't execute it. The user has
  to take the resulting output and run it themselves.

* Conversely, when we say an implementation "is an **interpreter**", we mean it
  takes in source code and executes it immediately. It runs programs "from
  source".

Like apples and oranges, some implementations are clearly compilers and *not*
interpreters. GCC and Clang take your C code and compile it to machine code. An
end user runs that executable directly and may never even know which tool was
used to compile it. So those are *compilers* for C.

In older versions of Matz's canonical implementation of Ruby, the user ran Ruby
from source. The implementation parsed it and executed it directly by traversing
the syntax tree. No other translation occurred, either internally or in any
user-visible form. So this was definitely an *interpreter* for Ruby.

But what of CPython? When you run your Python program using it, the code is
parsed and converted to an internal bytecode format, which is then executed
inside the VM. From the user's perspective, this is clearly an interpreter --
they run their program from source. But if you look under CPython's scaly skin,
you'll see that there is definitely some compiling going on.

The answer is that it is <span name="go">both</span>. CPython *is* an
interpreter, and it *has* a compiler. In practice, most scripting languages work
this way, as you can see:

<aside name="go">

The [Go tool][go] is even more of a horticultural curiosity. If you run `go
build`, it compiles your Go source code to machine code and stops. If you type
`go run`, it does that, then immediately executes the generated executable.

So `go` *is* a compiler (you can use it as a tool to compile code without
running it), *is* an interpreter (you can invoke it to immediately run a program
from source), and also *has* a compiler (when you use it as an interpreter, it
is still compiling internally).

[go tool]: https://golang.org/cmd/go/

</aside>

<img src="image/a-map-of-the-territory/venn.png" alt="A Venn diagram of compilers and interpreters" />

That overlapping region in the center is where our second interpreter lives too,
since it internally compiles to bytecode. So while this book is nominally about
interpreters, we'll cover some compilation too.

## Our Journey

That's a lot to take in all at once. Don't worry. This isn't the chapter where
you're expected to *understand* all of these pieces and parts. I just want you
to know that they are out there and roughly how they fit together.

This map should serve you well as you explore the territory beyond the guided
path we take in this book. I want to leave you yearning to strike out on your
own and wander all over that mountain.

But, for now, it's time for our own journey to begin. Tighten your bootlaces,
cinch up your pack, and come along. From <span name="here">here</span> on out,
all you need to focus on is the path in front of you.

<aside name="here">

Henceforth, I promise to tone down the whole mountain metaphor thing.

</aside>

<div class="challenges">

## Challenges

1. Pick an open source implementation of a language you like. Download the
   source code and poke around in it. Try to find the code that implements the
   scanner and parser. Are they handwritten, or generated using tools like
   Lex and Yacc? (`.l` or `.y` files usually imply the latter.)

1. Just-in-time compilation tends to be the fastest way to implement dynamically
   typed languages, but not all of them use it. What reasons are there to *not*
   JIT?

1. Most Lisp implementations that compile to C also contain an interpreter that
   lets them execute Lisp code on the fly as well. Why?

</div>
