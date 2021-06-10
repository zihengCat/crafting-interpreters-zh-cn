# Lox 程序设计语言

<!--
> What nicer thing can you do for somebody than make them breakfast?
>
> <cite>Anthony Bourdain</cite>
-->
> 有什么能比得上为饥肠辘辘的人端上一份丰盛早餐更好的事呢？
>
> <cite>安东尼·波登<em>（美国知名大厨）</em></cite>

<!--
We'll spend the rest of this book illuminating every dark and sundry corner of
the Lox language, but it seems cruel to have you immediately start grinding out
code for the interpreter without at least a glimpse of what we're going to end
up with.
-->
本书接下来的部分将仔细探索 Lox 程序语言实现的角角落落，但是在真正动手编写解释器代码之前，还是让我们来好好地认识一下这个我们将要实现的程序语言吧。

<!--
At the same time, I don't want to drag you through reams of language lawyering
and specification-ese before you get to touch your text <span
name="home">editor</span>. So this will be a gentle, friendly introduction to
Lox. It will leave out a lot of details and edge cases. We've got plenty of time
for those later.
-->
在当下，你还没有真正上手编写<span name="home">代码</span>之前，我不想先让你陷入到 Lox 程序语言的各种细节与语言规范里头去。所以本章是对 Lox 程序语言的一个温和、友好的介绍，大量语法细节与边界条件让我们放到后面具体实现时再详细阐述。

<aside name="home">

如果都不能上手编写几行 Lox 代码运行起来看一看，那本篇的内容也太枯燥乏味了，可是可是，你手上还没有一支 Lox 解释器呀，你都还没有开始构建它呢！

不打紧，你可以用[我写好的 Lox 解释器][repo]上手尝试。

<!--
A tutorial isn't very fun if you can't try the code out yourself. Alas, you
don't have a Lox interpreter yet, since you haven't built one!

Fear not. You can use [mine][repo].
-->

[repo]: https://github.com/munificent/craftinginterpreters

</aside>

<!--
-- Hello, Lox
-->
## 你好，Lox

<!--
Here's your very first taste of <span name="salmon">Lox</span>:
-->
这是你首次<span name="salmon">接触</span> Lox 程序设计语言。

<aside name="salmon">

<!--
Your first taste of Lox, the language, that is. I don't know if you've ever had
the cured, cold-smoked salmon before. If not, give it a try too.
-->
这是你首次尝鲜 Lox 程序语言，不知道你之前有没有吃过腌制的冷熏三文鱼，没吃过可以尝试一下。

</aside>

```lox
// Your first Lox program!
print "Hello, world!";
```

<!--
As that `//` line comment and the trailing semicolon imply, Lox's syntax is a
member of the C family. (There are no parentheses around the string because
`print` is a built-in statement, and not a library function.)
-->
行注释`//`与句末分号`;`标示着 Lox 语法偏向 C 系，Lox 是一门类 C 的程序语言（字符串两边没有带小括号，这是因为`print`是一个内建语句，而不是一个库函数）。

<!--
Now, I won't claim that <span name="c">C</span> has a *great* syntax. If we
wanted something elegant, we'd probably mimic Pascal or Smalltalk. If we wanted
to go full Scandinavian-furniture-minimalism, we'd do a Scheme. Those all have
their virtues.
-->
我毫不否认，<span name="c">C</span> 有着令人惊叹的语法设计。如果我们想要更加优雅的语法风格，Lox 可能偏向 Pascal 或者 Smalltalk ；如果我们想要极简主义的语法风格，Lox 可能更偏向 Scheme 。这些程序语言都有各自的可取之处。

<aside name="c">

<!--
I'm surely biased, but I think Lox's syntax is pretty clean. C's most egregious
grammar problems are around types. Dennis Ritchie had this idea called
"[declaration reflects use][use]", where variable declarations mirror the
operations you would have to perform on the variable to get to a value of the
base type. Clever idea, but I don't think it worked out great in practice.
-->
我的想法一定带有些主观情绪，但是我认为 Lox 的语法相当简洁清晰。C 语法最大的问题在于类型，丹尼斯·里奇将 C 语言类型的设计想法称为[”声明反映用法“][use]：变量声明直接反映出如果对变量取值应该采取的操作。这真是一个绝妙的想法，但在具体实践中工作得并不是很好，许多变量声明不够清晰难以理解。

[use]: http://softwareengineering.stackexchange.com/questions/117024/why-was-the-c-syntax-for-arrays-pointers-and-functions-designed-this-way

<!--
Lox doesn't have static types, so we avoid that.
-->
Lox 并不是静态类型语言，所以我们可以很好地避免了这个问题。

</aside>

<!--
What C-like syntax has instead is something you'll often find more valuable
in a language: *familiarity*. I know you are already comfortable with that style
because the two languages we'll be using to *implement* Lox -- Java and C --
also inherit it. Using a similar syntax for Lox gives you one less thing to
learn.
-->
类 C 语法最大的优势在于，它可以为读者带来：*熟悉感*。我知道你已经对我们将要用来*实现*  Lox 的两门程序语言 Java、C 非常熟悉了，Lox 采用与 C、Java 一脉相承的类 C 语法，这将为读者减少很多心智负担。

<!--
-- A High-Level Language
-->
## 一门高阶的程序设计语言

<!--
While this book ended up bigger than I was hoping, it's still not big enough to
fit a huge language like Java in it. In order to fit two complete
implementations of Lox in these pages, Lox itself has to be pretty compact.
-->
当我写完这本书时，书本的篇幅超出了我的预期，但这本书仍然没有达到够容纳类似 Java 这样大型程序语言实现的厚度。为了能在本书有限的篇幅里包含两种 Lox 语言的完整实现，Lox 语言本身必须被设计得小巧紧凑。

<!--
When I think of languages that are small but useful, what comes to mind are
high-level "scripting" languages like <span name="js">JavaScript</span>, Scheme,
and Lua. Of those three, Lox looks most like JavaScript, mainly because most
C-syntax languages do. As we'll learn later, Lox's approach to scoping hews
closely to Scheme. The C flavor of Lox we'll build in [Part III][] is heavily
indebted to Lua's clean, efficient implementation.
-->
当我思考哪些程序语言既小巧又实用的时候，我脑海中浮现出像 <span name="js">JavaScript</span>、Scheme、Lua 这样的高级“脚本”语言。在这三者之中，Lox 与 JavaScript 最为相像，大多数类 C 语法的脚本语言都像 JavaScript。当我们继续深入，就会发现 Lox 的作用域界定与 Scheme 相似。而在本书[第三部分][part iii]，我们使用 C 语言编写的 Lox 解释器，在具体实现细节上大量参考借鉴了 Lua 清晰高效的代码实现。

[part iii]: a-bytecode-virtual-machine.html

<aside name="js">

<!--
Now that JavaScript has taken over the world and is used to build ginormous
applications, it's hard to think of it as a "little scripting language". But
Brendan Eich hacked the first JS interpreter into Netscape Navigator in *ten
days* to make buttons animate on web pages. JavaScript has grown up since then,
but it was once a cute little language.
-->
如今，JavaScript 已经统治了世界，被用以构建超大型应用程序，认为 JavaScript 只是一门“小型脚本语言”的认知已经不太合适了。但在 JavaScript 最初诞生的那会儿，布兰登·艾奇为了能让网页上的按钮动起来，仅花了*十天*时间就设计实现了第一支 JS 解释器，放进网景浏览器里。在那时，JS 的却称得上是一支小巧可爱的小脚本语言，但随着 Web 技术的发展，如今的 JavaScript 已经变得异常庞大了。

<!--
Because Eich slapped JS together with roughly the same raw materials and time as
an episode of MacGyver, it has some weird semantic corners where the duct tape
and paper clips show through. Things like variable hoisting, dynamically bound
`this`, holes in arrays, and implicit conversions.
-->
可能是因为艾奇当初设计 JavaScript 所花的时间与心思太少了，就像《玉面飞龙》电视连续剧那样，JS 语言在许多语法角落和实现细节方面都留下了不少坑，如：变量作用域、`this`动态绑定、数组方面的坑、隐式类型转换等等。

<!--
I had the luxury of taking my time on Lox, so it should be a little cleaner.
-->
我花了很多时间在 Lox 语言设计上，所以 Lox 应该会比 JavaScript 更清晰一些，嘻嘻。

</aside>

<!--
Lox shares two other aspects with those three languages:
-->
在以下两个方面，Lox 也与这三门程序语言有着同样的理念。

<!--
--- Dynamic typing
-->
### 动态类型

<!--
Lox is dynamically typed. Variables can store values of any type, and a single
variable can even store values of different types at different times. If you try
to perform an operation on values of the wrong type -- say, dividing a number by
a string -- then the error is detected and reported at runtime.
-->
Lox 是一门动态类型的程序语言。变量可以存储任意类型的数据，也可以在程序运行的任意时刻改变其存储的数据类型。如果你在不兼容数据类型上执行了一个错误操作，比如说：将一个数字除以一枚字符串，那么该错误将在运行时被捕获和抛出。

<!--
There are plenty of reasons to like <span name="static">static</span> types, but
they don't outweigh the pragmatic reasons to pick dynamic types for Lox. A
static type system is a ton of work to learn and implement. Skipping it gives
you a simpler language and a shorter book. We'll get our interpreter up and
executing bits of code sooner if we defer our type checking to runtime.
-->
<span name="static">静态类型</span>有着许多令人喜爱的理由，但在实践上，我还是为 Lox 选择了动态类型。实现一个完备的静态类型系统需要大量的背景知识、深厚的代码功底。忽略静态类型，程序语言会变得更加简单，这本书也会变得更薄一些。将类型检查下推到运行时，可以让我们快速构建起一支可以执行代码的语言解释器。

<aside name="static">

<!--
After all, the two languages we'll be using to *implement* Lox are both
statically typed.
-->
毕竟，两门我们用来*实现* Lox 的程序语言（Java、C）都是静态类型的。

</aside>

<!--
--- Automatic memory management
-->
### 自动内存管理

<!--
High-level languages exist to eliminate error-prone, low-level drudgery, and what
could be more tedious than manually managing the allocation and freeing of
storage? No one rises and greets the morning sun with, "I can't wait to figure
out the correct place to call `free()` for every byte of memory I allocate
today!"
-->
高阶程序语言诞生的目的之一便是为了消除易出错的底层操作，还有什么比手动管理内存的分配释放更令人烦心的事情呢？没有人愿意早晨起来这么互相打招呼：“我迫不及待想为我今天申请的每块内存找个合适的地方调用`free()`函数啦！”

<!--
There are two main <span name="gc">techniques</span> for managing memory:
**reference counting** and **tracing garbage collection** (usually just called
**garbage collection** or **GC**). Ref counters are much simpler to implement --
I think that's why Perl, PHP, and Python all started out using them. But, over
time, the limitations of ref counting become too troublesome. All of those
languages eventually ended up adding a full tracing GC, or at least enough of
one to clean up object cycles.
-->
目前主要有两类技术用以管理内存：**引用计数（Reference Counting）** 和 **垃圾回收（Tracing Garbage Collection、Garbage Collection、GC）**。引用计数在实现上更加简便，我想这也是为什么 Perl、PHP、Python 起初都使用引用计数管理内存的原因。但是随着语言的发展，引用计数的局限性越来越明显，所以到最后，这些程序语言都添加上了完整的垃圾回收器，用于在对象全流程生命周期里做好内存清理回收工作。

<aside name="gc">

<!--
In practice, ref counting and tracing are more ends of a continuum than
opposing sides. Most ref counting systems end up doing some tracing to handle
cycles, and the write barriers of a generational collector look a bit like
retain calls if you squint.
-->
在具体实践中，引用计数和垃圾跟踪收集这两种技术更像是硬币的一体两面。大部分引用计数系统最终都会添加上一些追踪技术管理对象生命周期，而分代垃圾回收机制更像是一种最后的保留手段。

<!--
For lots more on this, see "[A Unified Theory of Garbage Collection][gc]" (PDF).
-->
更多关于这方面的讨论，参看这篇论文：[A Unified Theory of Garbage Collection][gc] (PDF)。

[gc]: https://researcher.watson.ibm.com/researcher/files/us-bacon/Bacon04Unified.pdf

</aside>

<!--
Tracing garbage collection has a fearsome reputation. It *is* a little harrowing
working at the level of raw memory. Debugging a GC can sometimes leave you
seeing hex dumps in your dreams. But, remember, this book is about dispelling
magic and slaying those monsters, so we *are* going to write our own garbage
collector. I think you'll find the algorithm is quite simple and a lot of fun to
implement.
-->
“垃圾回收（GC）”声名远播，让人心生畏惧，它需要开发者真正面向内存做一些细致的工作，调试垃圾回收器能让你做梦都能梦到程序的 16 进制转储调试信息。但是无需担心，本书就是为了揭开魔法、消灭怪兽的。我们将编写自己的垃圾回收器，你会发现，垃圾回收用到的算法其实非常简单，实现起来也非常有趣。

<!--
-- Data Types
-->
## 数据类型

<!--
In Lox's little universe, the atoms that make up all matter are the built-in
data types. There are only a few:
-->
在 Lox 的小小宇宙中，构成宇宙的原子即是内置数据类型。Lox 目前只有几种简单的内置数据类型：

<!--
*   **<span name="bool">Booleans</span>.** You can't code without logic and you
    can't logic without Boolean values. "True" and "false", the yin and yang of
    software. Unlike some ancient languages that repurpose an existing type to
    represent truth and falsehood, Lox has a dedicated Boolean type. We may
    be roughing it on this expedition, but we aren't *savages*.
-->
*   **<span name="bool">布尔类型（Boolean）</span>**。没有布尔类型，就无法表示逻辑，无法进行逻辑判断，就难以编写代码。“True” 与 “False” 就像是程序的“阴”与“阳”。许多古老的程序语言使用已经存在的数据类型表示逻辑真假（如：C 语言），Lox 没有选择这么做，Lox 有着专用的布尔类型，虽然实现上较为粗糙，但对比古老的程序语言们，我们足够先进，一点也不老土。

    <aside name="bool">

    <!--
    Boolean variables are the only data type in Lox named after a person, George
    Boole, which is why "Boolean" is capitalized. He died in 1864, nearly a
    century before digital computers turned his algebra into electricity. I
    wonder what he'd think to see his name all over billions of lines of Java
    code.
    -->
    布尔类型是唯一一种使用人名来命名的数据类型：乔治·布尔（George Boole），人们使用“Boolean”一词（人名，首字母大写）纪念他。乔治·布尔于 1864 年去世，过了一个多世纪，人们将布尔代数在电路上实现，造出了现代电子计算机。我很好奇，如果乔治·布尔生活在现今，看到成千上万的代码行里都镌刻着他的名字（特别是 Java 代码），会作何感想呢。

    </aside>

    <!--
    There are two Boolean values, obviously, and a literal for each one.
    -->
    显而易见的，布尔类型只有两个值：`true`和`false`。

    ```lox
    true;  // Not false.
    false; // Not *not* false.
    ```

<!--
*   **Numbers.** Lox has only one kind of number: double-precision floating
    point. Since floating-point numbers can also represent a wide range of
    integers, that covers a lot of territory, while keeping things simple.
-->
*   **数字类型（Numbers）**。Lox 程序语言只有一种数字类型：双精度浮点数。浮点数也可以覆盖很大范围的整数，为了让事情变得更加简单，我们只用这一种数。

    <!--
    Full-featured languages have lots of syntax for numbers -- hexadecimal,
    scientific notation, octal, all sorts of fun stuff. We'll settle for basic
    integer and decimal literals.
    -->
    功能全面的通用程序设计语言通常有着很多表示数字的语法：十六进制数、科学计数法、八进制数等等，这些数字表示法挺有趣的，但在这里，我们只支持最基本的十进制整数与小数。

    ```lox
    1234;  // An integer.
    12.34; // A decimal number.
    ```
<!--
*   **Strings.** We've already seen one string literal in the first example.
    Like most languages, they are enclosed in double quotes.
-->
*   **字符串类型（String）**。在本章开头部分的第一个程序示例中，我们就已经见过字符串类型了。与大部分程序语言一样，字符串字面量由两个双引号包裹。

    ```lox
    "I am a string";
    "";    // The empty string.
    "123"; // This is a string, not a number.
    ```

    <!--
    As we'll see when we get to implementing them, there is quite a lot of
    complexity hiding in that innocuous sequence of <span
    name="char">characters</span>.
    -->
    当我们真正开始着手实现字符串类型时就会发现，在看似简单的<span name=char>字符序列</span>下，隐藏着许多复杂的实现细节。

    <aside name="char">

    <!--
    Even that word "character" is a trickster. Is it ASCII? Unicode? A
    code point or a "grapheme cluster"? How are characters encoded? Is each
    character a fixed size, or can they vary?
    -->
    甚至是“字符”一词都极具欺骗性。字符是 ASCII 码还是 Unicode 万国码？是字符码点（Code Point）还是字形簇（Grapheme Cluster）？字符如何进行编码？单字符是定长还是变长？

    </aside>

<!--
*   **Nil.** There's one last built-in value who's never invited to the party
    but always seems to show up. It represents "no value". It's called "null" in
    many other languages. In Lox we spell it `nil`. (When we get to implementing
    it, that will help distinguish when we're talking about Lox's `nil` versus
    Java or C's `null`.)
-->
*   **空类型（Nil）**。 最后一个内置数据类型是空类型，虽然我们经常会忘记它，但空类型总是会在不经意间跳出来。空类型的含义是：没有值。在很多其他的程序语言里，空类型使用`null`表示，在 Lox 中，我们使用`nil`表示空类型（用以实现 Lox 的两门语言 Java、C 都保留了`null`关键字，所以我们换个词加以区分）。

    <!--
    There are good arguments for not having a null value in a language since
    null pointer errors are the scourge of our industry. If we were doing a
    statically typed language, it would be worth trying to ban it. In a
    dynamically typed one, though, eliminating it is often more annoying
    than having it.
    -->
    有许多合理的理由告诉我们，不应该在程序语言中引入空值`null`，空指针异常（Null Pointer Errors）给工业界带来了巨大的灾难。如果我们构建的是一门静态类型语言，倒是可以尝试着消除空值`null`。但对于一门动态类型程序语言来说，消除空值`null`会令你陷入更大的麻烦之中，得不偿失，不如引入空值`null`。

<!--
-- Expressions
-->
## 表达式

If built-in data types and their literals are atoms, then **expressions** must
be the molecules. Most of these will be familiar.

### Arithmetic

Lox features the basic arithmetic operators you know and love from C and other
languages:

```lox
add + me;
subtract - me;
multiply * me;
divide / me;
```

The subexpressions on either side of the operator are **operands**. Because
there are *two* of them, these are called **binary** operators. (It has nothing
to do with the ones-and-zeroes use of "binary".) Because the operator is <span
name="fixity">fixed</span> *in* the middle of the operands, these are also
called **infix** operators (as opposed to **prefix** operators where the
operator comes before the operands, and **postfix** where it comes after).

<aside name="fixity">

There are some operators that have more than two operands and the operators are
interleaved between them. The only one in wide usage is the "conditional" or
"ternary" operator of C and friends:

```c
condition ? thenArm : elseArm;
```

Some call these **mixfix** operators. A few languages let you define your own
operators and control how they are positioned -- their "fixity".

</aside>

One arithmetic operator is actually *both* an infix and a prefix one. The `-`
operator can also be used to negate a number.

```lox
-negateMe;
```

All of these operators work on numbers, and it's an error to pass any other
types to them. The exception is the `+` operator -- you can also pass it two
strings to concatenate them.

### Comparison and equality

Moving along, we have a few more operators that always return a Boolean result.
We can compare numbers (and only numbers), using Ye Olde Comparison Operators.

```lox
less < than;
lessThan <= orEqual;
greater > than;
greaterThan >= orEqual;
```

We can test two values of any kind for equality or inequality.

```lox
1 == 2;         // false.
"cat" != "dog"; // true.
```

Even different types.

```lox
314 == "pi"; // false.
```

Values of different types are *never* equivalent.

```lox
123 == "123"; // false.
```

I'm generally against implicit conversions.

### Logical operators

The not operator, a prefix `!`, returns `false` if its operand is true, and vice
versa.

```lox
!true;  // false.
!false; // true.
```

The other two logical operators really are control flow constructs in the guise
of expressions. An <span name="and">`and`</span> expression determines if two
values are *both* true. It returns the left operand if it's false, or the
right operand otherwise.

```lox
true and false; // false.
true and true;  // true.
```

And an `or` expression determines if *either* of two values (or both) are true.
It returns the left operand if it is true and the right operand otherwise.

```lox
false or false; // false.
true or false;  // true.
```

<aside name="and">

I used `and` and `or` for these instead of `&&` and `||` because Lox doesn't use
`&` and `|` for bitwise operators. It felt weird to introduce the
double-character forms without the single-character ones.

I also kind of like using words for these since they are really control flow
structures and not simple operators.

</aside>

The reason `and` and `or` are like control flow structures is that they
**short-circuit**. Not only does `and` return the left operand if it is false,
it doesn't even *evaluate* the right one in that case. Conversely
(contrapositively?), if the left operand of an `or` is true, the right is
skipped.

### Precedence and grouping

All of these operators have the same precedence and associativity that you'd
expect coming from C. (When we get to parsing, we'll get *way* more precise
about that.) In cases where the precedence isn't what you want, you can use `()`
to group stuff.

```lox
var average = (min + max) / 2;
```

Since they aren't very technically interesting, I've cut the remainder of the
typical operator menagerie out of our little language. No bitwise, shift,
modulo, or conditional operators. I'm not grading you, but you will get bonus
points in my heart if you augment your own implementation of Lox with them.

Those are the expression forms (except for a couple related to specific features
that we'll get to later), so let's move up a level.

## Statements

Now we're at statements. Where an expression's main job is to produce a *value*,
a statement's job is to produce an *effect*. Since, by definition, statements
don't evaluate to a value, to be useful they have to otherwise change the world
in some way -- usually modifying some state, reading input, or producing output.

You've seen a couple of kinds of statements already. The first one was:

```lox
print "Hello, world!";
```

A <span name="print">`print` statement</span> evaluates a single expression
and displays the result to the user. You've also seen some statements like:

<aside name="print">

Baking `print` into the language instead of just making it a core library
function is a hack. But it's a *useful* hack for us: it means our in-progress
interpreter can start producing output before we've implemented all of the
machinery required to define functions, look them up by name, and call them.

</aside>

```lox
"some expression";
```

An expression followed by a semicolon (`;`) promotes the expression to
statement-hood. This is called (imaginatively enough), an **expression
statement**.

If you want to pack a series of statements where a single one is expected, you
can wrap them up in a block.

```lox
{
  print "One statement.";
  print "Two statements.";
}
```

Blocks also affect scoping, which leads us to the next section...

## Variables

You declare variables using `var` statements. If you <span
name="omit">omit</span> the initializer, the variable's value defaults to `nil`.

<aside name="omit">

This is one of those cases where not having `nil` and forcing every variable to
be initialized to some value would be more annoying than dealing with `nil`
itself.

</aside>

```lox
var imAVariable = "here is my value";
var iAmNil;
```

Once declared, you can, naturally, access and assign a variable using its name.

<span name="breakfast"></span>

```lox
var breakfast = "bagels";
print breakfast; // "bagels".
breakfast = "beignets";
print breakfast; // "beignets".
```

<aside name="breakfast">

Can you tell that I tend to work on this book in the morning before I've had
anything to eat?

</aside>

I won't get into the rules for variable scope here, because we're going to spend
a surprising amount of time in later chapters mapping every square inch of the
rules. In most cases, it works like you would expect coming from C or Java.

## Control Flow

It's hard to write <span name="flow">useful</span> programs if you can't skip
some code or execute some more than once. That means control flow. In addition
to the logical operators we already covered, Lox lifts three statements straight
from C.

<aside name="flow">

We already have `and` and `or` for branching, and we *could* use recursion to
repeat code, so that's theoretically sufficient. It would be pretty awkward to
program that way in an imperative-styled language, though.

Scheme, on the other hand, has no built-in looping constructs. It *does* rely on
recursion for repetition. Smalltalk has no built-in branching constructs, and
relies on dynamic dispatch for selectively executing code.

</aside>

An `if` statement executes one of two statements based on some condition.

```lox
if (condition) {
  print "yes";
} else {
  print "no";
}
```

A `while` <span name="do">loop</span> executes the body repeatedly as long as
the condition expression evaluates to true.

```lox
var a = 1;
while (a < 10) {
  print a;
  a = a + 1;
}
```

<aside name="do">

I left `do while` loops out of Lox because they aren't that common and wouldn't
teach you anything that you won't already learn from `while`. Go ahead and add
it to your implementation if it makes you happy. It's your party.

</aside>

Finally, we have `for` loops.

```lox
for (var a = 1; a < 10; a = a + 1) {
  print a;
}
```

This loop does the same thing as the previous `while` loop. Most modern
languages also have some sort of <span name="foreach">`for-in`</span> or
`foreach` loop for explicitly iterating over various sequence types. In a real
language, that's nicer than the crude C-style `for` loop we got here. Lox keeps
it basic.

<aside name="foreach">

This is a concession I made because of how the implementation is split across
chapters. A `for-in` loop needs some sort of dynamic dispatch in the iterator
protocol to handle different kinds of sequences, but we don't get that until
after we're done with control flow. We could circle back and add `for-in` loops
later, but I didn't think doing so would teach you anything super interesting.

</aside>

## Functions

A function call expression looks the same as it does in C.

```lox
makeBreakfast(bacon, eggs, toast);
```

You can also call a function without passing anything to it.

```lox
makeBreakfast();
```

Unlike in, say, Ruby, the parentheses are mandatory in this case. If you leave them
off, it doesn't *call* the function, it just refers to it.

A language isn't very fun if you can't define your own functions. In Lox, you do
that with <span name="fun">`fun`</span>.

<aside name="fun">

I've seen languages that use `fn`, `fun`, `func`, and `function`. I'm still
hoping to discover a `funct`, `functi`, or `functio` somewhere.

</aside>

```lox
fun printSum(a, b) {
  print a + b;
}
```

Now's a good time to clarify some <span name="define">terminology</span>. Some
people throw around "parameter" and "argument" like they are interchangeable
and, to many, they are. We're going to spend a lot of time splitting the finest
of downy hairs around semantics, so let's sharpen our words. From here on out:

*   An **argument** is an actual value you pass to a function when you call it.
    So a function *call* has an *argument* list. Sometimes you hear **actual
    parameter** used for these.

*   A **parameter** is a variable that holds the value of the argument inside
    the body of the function. Thus, a function *declaration* has a *parameter*
    list. Others call these **formal parameters** or simply **formals**.

<aside name="define">

Speaking of terminology, some statically typed languages like C make a
distinction between *declaring* a function and *defining* it. A declaration
binds the function's type to its name so that calls can be type-checked but does
not provide a body. A definition declares the function and also fills in the
body so that the function can be compiled.

Since Lox is dynamically typed, this distinction isn't meaningful. A function
declaration fully specifies the function including its body.

</aside>

The body of a function is always a block. Inside it, you can return a value
using a `return` statement.

```lox
fun returnSum(a, b) {
  return a + b;
}
```

If execution reaches the end of the block without hitting a `return`, it
<span name="sneaky">implicitly</span> returns `nil`.

<aside name="sneaky">

See, I told you `nil` would sneak in when we weren't looking.

</aside>

### Closures

Functions are *first class* in Lox, which just means they are real values that
you can get a reference to, store in variables, pass around, etc. This works:

```lox
fun addPair(a, b) {
  return a + b;
}

fun identity(a) {
  return a;
}

print identity(addPair)(1, 2); // Prints "3".
```

Since function declarations are statements, you can declare local functions
inside another function.

```lox
fun outerFunction() {
  fun localFunction() {
    print "I'm local!";
  }

  localFunction();
}
```

If you combine local functions, first-class functions, and block scope, you run
into this interesting situation:

```lox
fun returnFunction() {
  var outside = "outside";

  fun inner() {
    print outside;
  }

  return inner;
}

var fn = returnFunction();
fn();
```

Here, `inner()` accesses a local variable declared outside of its body in the
surrounding function. Is this kosher? Now that lots of languages have borrowed
this feature from Lisp, you probably know the answer is yes.

For that to work, `inner()` has to "hold on" to references to any surrounding
variables that it uses so that they stay around even after the outer function
has returned. We call functions that do this <span
name="closure">**closures**</span>. These days, the term is often used for *any*
first-class function, though it's sort of a misnomer if the function doesn't
happen to close over any variables.

<aside name="closure">

Peter J. Landin coined the term "closure". Yes, he invented damn near half the
terms in programming languages. Most of them came out of one incredible paper,
"[The Next 700 Programming Languages][svh]".

[svh]: https://homepages.inf.ed.ac.uk/wadler/papers/papers-we-love/landin-next-700.pdf

In order to implement these kind of functions, you need to create a data
structure that bundles together the function's code and the surrounding
variables it needs. He called this a "closure" because it *closes over* and
holds on to the variables it needs.

</aside>

As you can imagine, implementing these adds some complexity because we can no
longer assume variable scope works strictly like a stack where local variables
evaporate the moment the function returns. We're going to have a fun time
learning how to make these work correctly and efficiently.

## Classes

Since Lox has dynamic typing, lexical (roughly, "block") scope, and closures,
it's about halfway to being a functional language. But as you'll see, it's
*also* about halfway to being an object-oriented language. Both paradigms have a
lot going for them, so I thought it was worth covering some of each.

Since classes have come under fire for not living up to their hype, let me first
explain why I put them into Lox and this book. There are really two questions:

### Why might any language want to be object oriented?

Now that object-oriented languages like Java have sold out and only play arena
shows, it's not cool to like them anymore. Why would anyone make a *new*
language with objects? Isn't that like releasing music on 8-track?

It is true that the "all inheritance all the time" binge of the '90s produced
some monstrous class hierarchies, but **object-oriented programming** (**OOP**)
is still pretty rad. Billions of lines of successful code have been written in
OOP languages, shipping millions of apps to happy users. Likely a majority of
working programmers today are using an object-oriented language. They can't all
be *that* wrong.

In particular, for a dynamically typed language, objects are pretty handy. We
need *some* way of defining compound data types to bundle blobs of stuff
together.

If we can also hang methods off of those, then we avoid the need to prefix all
of our functions with the name of the data type they operate on to avoid
colliding with similar functions for different types. In, say, Racket, you end
up having to name your functions like `hash-copy` (to copy a hash table) and
`vector-copy` (to copy a vector) so that they don't step on each other. Methods
are scoped to the object, so that problem goes away.

### Why is Lox object oriented?

I could claim objects are groovy but still out of scope for the book. Most
programming language books, especially ones that try to implement a whole
language, leave objects out. To me, that means the topic isn't well covered.
With such a widespread paradigm, that omission makes me sad.

Given how many of us spend all day *using* OOP languages, it seems like the
world could use a little documentation on how to *make* one. As you'll see, it
turns out to be pretty interesting. Not as hard as you might fear, but not as
simple as you might presume, either.

### Classes or prototypes

When it comes to objects, there are actually two approaches to them, [classes][]
and [prototypes][]. Classes came first, and are more common thanks to C++, Java,
C#, and friends. Prototypes were a virtually forgotten offshoot until JavaScript
accidentally took over the world.

[classes]: https://en.wikipedia.org/wiki/Class-based_programming
[prototypes]: https://en.wikipedia.org/wiki/Prototype-based_programming

In class-based languages, there are two core concepts: instances and classes.
Instances store the state for each object and have a reference to the instance's
class. Classes contain the methods and inheritance chain. To call a method on an
instance, there is always a level of indirection. You <span name="dispatch">look
up the instance's class and then you find the method *there*:

<aside name="dispatch">

In a statically typed language like C++, method lookup typically happens at
compile time based on the *static* type of the instance, giving you **static
dispatch**. In contrast, **dynamic dispatch** looks up the class of the actual
instance object at runtime. This is how virtual methods in statically typed
languages and all methods in a dynamically typed language like Lox work.

</aside>

<img src="image/the-lox-language/class-lookup.png" alt="How fields and methods are looked up on classes and instances" />

Prototype-based languages <span name="blurry">merge</span> these two concepts.
There are only objects -- no classes -- and each individual object may contain
state and methods. Objects can directly inherit from each other (or "delegate
to" in prototypal lingo):

<aside name="blurry">

In practice the line between class-based and prototype-based languages blurs.
JavaScript's "constructor function" notion [pushes you pretty hard][js new]
towards defining class-like objects. Meanwhile, class-based Ruby is perfectly
happy to let you attach methods to individual instances.

[js new]: http://gameprogrammingpatterns.com/prototype.html#what-about-javascript

</aside>

<img src="image/the-lox-language/prototype-lookup.png" alt="How fields and methods are looked up in a prototypal system" />

This means that in some ways prototypal languages are more fundamental than
classes. They are really neat to implement because they're *so* simple. Also,
they can express lots of unusual patterns that classes steer you away from.

But I've looked at a *lot* of code written in prototypal languages -- including
[some of my own devising][finch]. Do you know what people generally do with all
of the power and flexibility of prototypes? ...They use them to reinvent
classes.

[finch]: http://finch.stuffwithstuff.com/

I don't know *why* that is, but people naturally seem to prefer a class-based
(Classic? Classy?) style. Prototypes *are* simpler in the language, but they
seem to accomplish that only by <span name="waterbed">pushing</span> the
complexity onto the user. So, for Lox, we'll save our users the trouble and bake
classes right in.

<aside name="waterbed">

Larry Wall, Perl's inventor/prophet calls this the "[waterbed theory][]". Some
complexity is essential and cannot be eliminated. If you push it down in one
place, it swells up in another.

[waterbed theory]: http://wiki.c2.com/?WaterbedTheory

Prototypal languages don't so much *eliminate* the complexity of classes as they
do make the *user* take that complexity by building their own class-like
metaprogramming libraries.

</aside>

### Classes in Lox

Enough rationale, let's see what we actually have. Classes encompass a
constellation of features in most languages. For Lox, I've selected what I think
are the brightest stars. You declare a class and its methods like so:

```lox
class Breakfast {
  cook() {
    print "Eggs a-fryin'!";
  }

  serve(who) {
    print "Enjoy your breakfast, " + who + ".";
  }
}
```

The body of a class contains its methods. They look like function declarations
but without the `fun` <span name="method">keyword</span>. When the class
declaration is executed, Lox creates a class object and stores that in a
variable named after the class. Just like functions, classes are first class in
Lox.

<aside name="method">

They are still just as fun, though.

</aside>

```lox
// Store it in variables.
var someVariable = Breakfast;

// Pass it to functions.
someFunction(Breakfast);
```

Next, we need a way to create instances. We could add some sort of `new`
keyword, but to keep things simple, in Lox the class itself is a factory
function for instances. Call a class like a function, and it produces a new
instance of itself.

```lox
var breakfast = Breakfast();
print breakfast; // "Breakfast instance".
```

### Instantiation and initialization

Classes that only have behavior aren't super useful. The idea behind
object-oriented programming is encapsulating behavior *and state* together. To
do that, you need fields. Lox, like other dynamically typed languages, lets you
freely add properties onto objects.

```lox
breakfast.meat = "sausage";
breakfast.bread = "sourdough";
```

Assigning to a field creates it if it doesn't already exist.

If you want to access a field or method on the current object from within a
method, you use good old `this`.

```lox
class Breakfast {
  serve(who) {
    print "Enjoy your " + this.meat + " and " +
        this.bread + ", " + who + ".";
  }

  // ...
}
```

Part of encapsulating data within an object is ensuring the object is in a valid
state when it's created. To do that, you can define an initializer. If your
class has a method named `init()`, it is called automatically when the object is
constructed. Any parameters passed to the class are forwarded to its
initializer.

```lox
class Breakfast {
  init(meat, bread) {
    this.meat = meat;
    this.bread = bread;
  }

  // ...
}

var baconAndToast = Breakfast("bacon", "toast");
baconAndToast.serve("Dear Reader");
// "Enjoy your bacon and toast, Dear Reader."
```

### Inheritance

Every object-oriented language lets you not only define methods, but reuse them
across multiple classes or objects. For that, Lox supports single inheritance.
When you declare a class, you can specify a class that it inherits from using a less-than
<span name="less">(`<`)</span> operator.

```lox
class Brunch < Breakfast {
  drink() {
    print "How about a Bloody Mary?";
  }
}
```

<aside name="less">

Why the `<` operator? I didn't feel like introducing a new keyword like
`extends`. Lox doesn't use `:` for anything else so I didn't want to reserve
that either. Instead, I took a page from Ruby and used `<`.

If you know any type theory, you'll notice it's not a *totally* arbitrary
choice. Every instance of a subclass is an instance of its superclass too, but
there may be instances of the superclass that are not instances of the subclass.
That means, in the universe of objects, the set of subclass objects is smaller
than the superclass's set, though type nerds usually use `<:` for that relation.

</aside>

Here, Brunch is the **derived class** or **subclass**, and Breakfast is the
**base class** or **superclass**.

Every method defined in the superclass is also available to its subclasses.

```lox
var benedict = Brunch("ham", "English muffin");
benedict.serve("Noble Reader");
```

Even the `init()` method gets <span name="init">inherited</span>. In practice,
the subclass usually wants to define its own `init()` method too. But the
original one also needs to be called so that the superclass can maintain its
state. We need some way to call a method on our own *instance* without hitting
our own *methods*.

<aside name="init">

Lox is different from C++, Java, and C#, which do not inherit constructors, but
similar to Smalltalk and Ruby, which do.

</aside>

As in Java, you use `super` for that.

```lox
class Brunch < Breakfast {
  init(meat, bread, drink) {
    super.init(meat, bread);
    this.drink = drink;
  }
}
```

That's about it for object orientation. I tried to keep the feature set minimal.
The structure of the book did force one compromise. Lox is not a *pure*
object-oriented language. In a true OOP language every object is an instance of
a class, even primitive values like numbers and Booleans.

Because we don't implement classes until well after we start working with the
built-in types, that would have been hard. So values of primitive types aren't
real objects in the sense of being instances of classes. They don't have methods
or properties. If I were trying to make Lox a real language for real users, I
would fix that.

## The Standard Library

We're almost done. That's the whole language, so all that's left is the "core"
or "standard" library -- the set of functionality that is implemented directly
in the interpreter and that all user-defined behavior is built on top of.

This is the saddest part of Lox. Its standard library goes beyond minimalism and
veers close to outright nihilism. For the sample code in the book, we only need
to demonstrate that code is running and doing what it's supposed to do. For
that, we already have the built-in `print` statement.

Later, when we start optimizing, we'll write some benchmarks and see how long it
takes to execute code. That means we need to track time, so we'll define one
built-in function, `clock()`, that returns the number of seconds since the
program started.

And... that's it. I know, right? It's embarrassing.

If you wanted to turn Lox into an actual useful language, the very first thing
you should do is flesh this out. String manipulation, trigonometric functions,
file I/O, networking, heck, even *reading input from the user* would help. But we
don't need any of that for this book, and adding it wouldn't teach you anything
interesting, so I've left it out.

Don't worry, we'll have plenty of exciting stuff in the language itself to keep
us busy.

<div class="challenges">

## Challenges

1. Write some sample Lox programs and run them (you can use the implementations
   of Lox in [my repository][repo]). Try to come up with edge case behavior I
   didn't specify here. Does it do what you expect? Why or why not?

2. This informal introduction leaves a *lot* unspecified. List several open
   questions you have about the language's syntax and semantics. What do you
   think the answers should be?

3. Lox is a pretty tiny language. What features do you think it is missing that
   would make it annoying to use for real programs? (Aside from the standard
   library, of course.)

</div>

<div class="design-note">

## Design Note: Expressions and Statements

Lox has both expressions and statements. Some languages omit the latter.
Instead, they treat declarations and control flow constructs as expressions too.
These "everything is an expression" languages tend to have functional pedigrees
and include most Lisps, SML, Haskell, Ruby, and CoffeeScript.

To do that, for each "statement-like" construct in the language, you need to
decide what value it evaluates to. Some of those are easy:

*   An `if` expression evaluates to the result of whichever branch is chosen.
    Likewise, a `switch` or other multi-way branch evaluates to whichever case
    is picked.

*   A variable declaration evaluates to the value of the variable.

*   A block evaluates to the result of the last expression in the sequence.

Some get a little stranger. What should a loop evaluate to? A `while` loop in
CoffeeScript evaluates to an array containing each element that the body
evaluated to. That can be handy, or a waste of memory if you don't need the
array.

You also have to decide how these statement-like expressions compose with other
expressions -- you have to fit them into the grammar's precedence table. For
example, Ruby allows:

```ruby
puts 1 + if true then 2 else 3 end + 4
```

Is this what you'd expect? Is it what your *users* expect? How does this affect
how you design the syntax for your "statements"? Note that Ruby has an explicit
`end` to tell when the `if` expression is complete. Without it, the `+ 4` would
likely be parsed as part of the `else` clause.

Turning every statement into an expression forces you to answer a few hairy
questions like that. In return, you eliminate some redundancy. C has both blocks
for sequencing statements, and the comma operator for sequencing expressions. It
has both the `if` statement and the `?:` conditional operator. If everything was
an expression in C, you could unify each of those.

Languages that do away with statements usually also feature **implicit returns**
-- a function automatically returns whatever value its body evaluates to without
need for some explicit `return` syntax. For small functions and methods, this is
really handy. In fact, many languages that do have statements have added syntax
like `=>` to be able to define functions whose body is the result of evaluating
a single expression.

But making *all* functions work that way can be a little strange. If you aren't
careful, your function will leak a return value even if you only intend it to
produce a side effect. In practice, though, users of these languages don't find
it to be a problem.

For Lox, I gave it statements for prosaic reasons. I picked a C-like syntax for
familiarity's sake, and trying to take the existing C statement syntax and
interpret it like expressions gets weird pretty fast.

</div>
