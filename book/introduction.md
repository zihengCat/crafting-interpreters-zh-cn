# 序言

<!--
> Fairy tales are more than true: not because they tell us that dragons exist,
> but because they tell us that dragons can be beaten.
>
> <cite>G.K. Chesterton by way of Neil Gaiman, <em>Coraline</em></cite>
-->
> 童话胜于现实，不仅是因为童话告诉我们巨龙存在，
> 更是因为它们告诉我们巨龙可以被打败。
>
> <cite>尼尔·盖曼，<em>卡洛琳（《鬼妈妈》）</em></cite>

<!--
I'm really excited we're going on this journey together. This is a book on
implementing interpreters for programming languages. It's also a book on how to
design a language worth implementing. It's the book I wish I'd had when I first
started getting into languages, and it's the book I've been writing in my <span
name="head">head</span> for nearly a decade.
-->
很高兴我们能一起携手开启这段旅程。这是一本关于如何实现程序语言解释器的书。
这也是一本关于如何设计一门程序语言的书。这是本我初入程序设计语言领域就想要拥有的书，
这是本我 10 年前就在我<span name="head">脑海</span>中构思的书。

<aside name="head">

<!--
To my friends and family, sorry I've been so absentminded!
-->
不好意思，朋友，我好像有点语无伦次了...

</aside>

<!--
In these pages, we will walk step-by-step through two complete interpreters for
a full-featured language. I assume this is your first foray into languages, so
I'll cover each concept and line of code you need to build a complete, usable,
fast language implementation.
-->
在这本书中，我们将一步一脚印地为一门五脏俱全程序设计语言实现两枚解释器（Interpreters）。
我会预设这是你初次涉足程序设计语言开发领域，所以你不需要有任何的心理负担。我将详细阐述构建
完整、可用、高效的语言解释器所需要的每一个基本概念及其代码实现。

<!--
In order to cram two full implementations inside one book without it turning
into a doorstop, this text is lighter on theory than others. As we build each
piece of the system, I will introduce the history and concepts behind it. I'll
try to get you familiar with the lingo so that if you ever find yourself at a
<span name="party">cocktail party</span> full of PL (programming language)
researchers, you'll fit in.
-->
为了将两枚解释器的完整实现塞到一本书中而又不使得该书变得臃肿不堪，我选择不在编译理论部分着墨过多。
当我们逐步搭建起编译系统的各个部分之时，我再向你介绍关于这部分的历史与其背后的概念。
我会努力让你理解那些在<span name="party">鸡尾酒晚会</span>上的程序设计语言研究者们所说的"行话"，相信你很快就能理解适应这些"术语"了。

<aside name="party">

<!--
Strangely enough, a situation I have found myself in multiple times. You
wouldn't believe how much some of them can drink.
-->
说实在话，在很多场合，我都发现那些家伙们有些是真的很能喝酒，你都没法想象。

</aside>

<!--
But we're mostly going to spend our brain juice getting the language up and
running. This is not to say theory isn't important. Being able to reason
precisely and <span name="formal">formally</span> about syntax and semantics is
a vital skill when working on a language. But, personally, I learn best by
doing. It's hard for me to wade through paragraphs full of abstract concepts and
really absorb them. But if I've coded something, run it, and debugged it, then I
*get* it.
-->
在大部分情况下，我们都将绞尽脑汁先让我们的程序运行起来。这并不意味着理论不重要，
能够正确且<span name="formal">形式化</span>地推导语法和语义是开发程序设计语言一项至关重要的技能。
但是，我个人还是喜欢边做边学。通读大段大段充斥抽象概念的段落并充分理解其含义对我来说实在是太过困难，
但如果让我写几行代码，跑一跑，跟踪 debug 一下，我很快就可以*掌握*它。

<aside name="formal">

<!--
Static type systems in particular require rigorous formal reasoning. Hacking on
a type system has the same feel as proving a theorem in mathematics.

It turns out this is no coincidence. In the early half of last century, Haskell
Curry and William Alvin Howard showed that they are two sides of the same coin:
[the Curry-Howard isomorphism][].

[the curry-howard isomorphism]: https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence
-->
静态类型系统尤其需要严格的形式推导。骇入静态类型系统与证明数学定理有着相似的感觉。
事实证明这不是巧合。在上世纪初，美国数学家哈斯凯尔·柯里和逻辑学家威廉·阿尔文·霍瓦德
就证明了它们正是同一枚硬币的一体两面：
[柯里-霍华德同构][]

[柯里-霍华德同构]: https://en.wikipedia.org/wiki/Curry%E2%80%93Howard_correspondence

</aside>

<!--
That's my goal for you. I want you to come away with a solid intuition of how a
real language lives and breathes. My hope is that when you read other, more
theoretical books later, the concepts there will firmly stick in your mind,
adhered to this tangible substrate.
-->
这也是我对你的期望。我希望你通过动手实操，对一门真正程序设计语言的诞生过程留下一个坚实的印象，如此一来，以后当你在阅读其他更具理论性的书籍著作时，这些核心概念将深深镌刻进你的大脑，使得你可以更加充分地理解它们。

<!--
-- Why Learn This Stuff?
-->
## 为什么要学习这玩意儿？

<!--
Every introduction to every compiler book seems to have this section. I don't
know what it is about programming languages that causes such existential doubt.
I don't think ornithology books worry about justifying their existence. They
assume the reader loves birds and start teaching.

But programming languages are a little different. I suppose it is true that the
odds of any of us creating a broadly successful, general-purpose programming
language are slim. The designers of the world's widely used languages could fit
in a Volkswagen bus, even without putting the pop-top camper up. If joining that
elite group was the *only* reason to learn languages, it would be hard to
justify. Fortunately, it isn't.
-->

似乎每一本关于编译原理的书都会在开头简介部分来上这么一段，告诉读者为什么要学习编译原理。
我不知道造成这种"存在性怀疑"的程序设计语言是什么。
<!--
正如我不认为鸟类学书籍是为了证明鸟儿的存在性一样，作者会预设读者都是爱鸟之人，然后开始教学。
但是程序设计语言略有不同，
-->
我敢肯定地说，你们中的大多数人创造出一门被广为使用的通用程序设计语言的可能性微乎其微，
毕竟，即使把这个世界上被广为应用程序语言的设计者们都拉出来，还塞不满一辆大众巴士呢，
如果学习编译原理的**唯一**理由就是要加入这群精英，创造出世界流行的程序设计语言，这无疑令人费解。
幸运的是，事实并非如此。

<!--
--- Little languages are everywhere
-->
### 小众语言无处不在

<!--
For every successful general-purpose language, there are a thousand successful
niche ones. We used to call them "little languages", but inflation in the jargon
economy led to the name "domain-specific languages". These are pidgins
tailor-built to a specific task. Think application scripting languages, template
engines, markup formats, and configuration files.
-->
对于任何一支成功的通用程序设计语言，都会有上千种其他语言与其相辅相成。我们通常将它们称为"小众语言"。
但这一称呼其实并不准确，更为确切的称呼应该是：领域特定语言（Domain-specific Languages、DSL）。
领域特定语言被设计用以完成某些特定任务，如：应用程序脚本、模版引擎、标记格式、配置文件等。

<span name="little"></span><img src="image/introduction/little-languages.png" alt="A random selection of little languages." />

<aside name="little">

随机筛选了一些你可能遇到过的"小众语言"。
<!--
A random selection of some little languages you might run into.
-->

</aside>

<!--
Almost every large software project needs a handful of these. When you can, it's
good to reuse an existing one instead of rolling your own. Once you factor in
documentation, debuggers, editor support, syntax highlighting, and all of the
other trappings, doing it yourself becomes a tall order.
-->
几乎每个大型软件项目都需要用到其中的一个或多个。
如果可以的话，最好是复用一支已经存在的领域特定语言，而不是自己再创造一个，
因为一旦使用自己编写的小众语言替换，在文档撰写、程序调试、编辑器支持、语法高亮等方面，
你都将面临许多不必要的麻烦。

<!--
But there's still a good chance you'll find yourself needing to whip up a parser
or other tool when there isn't an existing library that fits your needs. Even
when you are reusing some existing implementation, you'll inevitably end up
needing to debug and maintain it and poke around in its guts.
-->
但如果当现有的领域特定语言代码库无法满足你的需求时，你还是需要勉为其难地手写一枚解析器或者类似的工具以满足需求。即使你想要重用一些现有的代码，你也将不可避免地对其进行调试和维护，深入研究其原理。

<!--
--- Languages are great exercise
-->
### 着实锻炼编程技艺

<!--
Long distance runners sometimes train with weights strapped to their ankles or
at high altitudes where the atmosphere is thin. When they later unburden
themselves, the new relative ease of light limbs and oxygen-rich air enables
them to run farther and faster.
-->
长跑运动员时常会在脚踝绑上重物，或是去到在氧气稀薄的高海拔地区训练自身。
当他们卸下重负，来到富氧地区比赛时，他们的四肢变得轻盈，脚步变得稳健，跑得更快更远。

<!--
Implementing a language is a real test of programming skill. The code is complex
and performance critical. You must master recursion, dynamic arrays, trees,
graphs, and hash tables. You probably use hash tables at least in your
day-to-day programming, but do you *really* understand them? Well, after we've
crafted our own from scratch, I guarantee you will.
-->
实现一门程序设计语言是对编程技艺一次极好地磨练。代码实现复杂艰深，对代码性能也有着至关重要的要求。
你必须很好地掌握递归、动态数组、树、图、哈希表等关键技术。
你也许经常在日常编程中使用哈希表数据结构，但你是否**真正**理解它了呢？
当我们从头开始完成语言解释器之后，我保证你对这些基础算法数据结构的认知会更上一层楼。

<!--
While I intend to show you that an interpreter isn't as daunting as you might
believe, implementing one well is still a challenge. Rise to it, and you'll come
away a stronger programmer, and smarter about how you use data structures and
algorithms in your day job.
-->
尽管我想努力向你传达，实现一枚程序语言解释器并不如你想象的那般困难，但实际上这仍是一个巨大的挑战。
克服它，你会变得更加强大，在日常工作中，你也将更加聪明地运用算法与数据结构。

<!--
--- One more reason
-->
### 另一个原因

<!--
This last reason is hard for me to admit, because it's so close to my heart.
Ever since I learned to program as a kid, I felt there was something magical
about languages. When I first tapped out BASIC programs one key at a time I
couldn't conceive how BASIC *itself* was made.
-->
最后一个原因我其实有点难以启齿，因为这是我内心深处最为真切的想法。
在我还是一个孩童时，就觉得程序设计语言非常的神奇，我一个字符一个字符敲下了一支 BASIC 程序，
程序正常运行，但我却不知道 BASIC 语言是如何工作的。

<!--
Later, the mixture of awe and terror on my college friends' faces when talking
about their compilers class was enough to convince me language hackers were a
different breed of human -- some sort of wizards granted privileged access to
arcane arts.
-->
后来我上了大学，同上编译原理课的同学对我说，程序语言黑客简直是另一种人类，
就像会施展奥术的巫师一样，说这话时脸上还充满了敬畏之情。

<!--
It's a charming <span name="image">image</span>, but it has a darker side. *I*
didn't feel like a wizard, so I was left thinking I lacked some inborn quality
necessary to join the cabal. Though I've been fascinated by languages ever since
I doodled made-up keywords in my school notebook, it took me decades to muster
the courage to try to really learn them. That "magical" quality, that sense of
exclusivity, excluded *me*.
-->
那可真是幅迷人的<span name="image">画面</span>。但我并不觉得我自己像个巫师，也许我天生就缺少一些加入巫师集团的特质。
从那时起，我被程序设计语言深深吸引，我在学校笔记本上涂满了拼凑而成关于程序语言的关键字。
在往后数十年时间里，我都在学习它们，探索着程序设计语言"魔盒"背后的奥秘。

说不定，你也与我有同样的感受呢。

<aside name="image">

<!--
And its practitioners don't hesitate to play up this image. Two of the seminal
texts on programming languages feature a [dragon][] and a [wizard][] on their
covers.
-->
程序设计语言领域的奠基者们似乎从不吝啬饰演这一形象。
两本在程序设计语言领域极富开创性的教科书分别选用了“龙”和”巫师“形象作封面，
即是大名鼎鼎的[《龙书》][dragon]与[《巫师书》][wizard]。

[dragon]: https://en.wikipedia.org/wiki/Compilers:_Principles,_Techniques,_and_Tools
[wizard]: https://mitpress.mit.edu/sites/default/files/sicp/index.html

</aside>

<!--
When I did finally start cobbling together my own little interpreters, I quickly
learned that, of course, there is no magic at all. It's just code, and the
people who hack on languages are just people.
-->
当我真正开始动手制作一枚小编译器的时候，我意识到，这其中并没有什么"神奇魔法"，仅是一堆代码罢了，
而创造程序语言的也仅是普通人而已。

<!--
There *are* a few techniques you don't often encounter outside of languages, and
some parts are a little difficult. But not more difficult than other obstacles
you've overcome. My hope is that if you've felt intimidated by languages and
this book helps you overcome that fear, maybe I'll leave you just a tiny bit
braver than you were before.
-->
当然，实现一门程序设计语言多少会涉及一些你在外头很少见到的技术，其中某些部分确实有点难度，
但也不会比你一路走来所克服的困难难上多少。如果你曾对程序设计语言和编译原理心生畏惧，
希望这本书可以帮你克服恐惧，为你带来些许勇气。

<!--
And, who knows, maybe you *will* make the next great language. Someone has to.
-->
说不定，下个知名程序设计语言的创造者就是你呢！

<!--
-- How the Book Is Organized
-->
## 本书是如何组织的

<!--
This book is broken into three parts. You're reading the first one now. It's a
couple of chapters to get you oriented, teach you some of the lingo that
language hackers use, and introduce you to Lox, the language we'll be
implementing.
-->
本书主要分为三个部分，你目前正在阅读的正是第一部分。第一部分通过几个章节带你快速进入程序设计语言的世界，为你讲解程序设计语言开发者们时常挂在嘴边的那些"术语"。第一部分还将带你认识 Lox ，
我们将要实现的程序设计语言。

<!--
Each of the other two parts builds one complete Lox interpreter. Within those
parts, each chapter is structured the same way. The chapter takes a single
language feature, teaches you the concepts behind it, and walks you through an
implementation.
-->
之后的两大部分，都将分别带你构建一支完整的 Lox 解释器。这两部分的章节组织也大同小异，每章取一个语言特性，讲解其背后的核心概念，带你完整实现一遍代码。

<!--
It took a good bit of trial and error on my part, but I managed to carve up the
two interpreters into chapter-sized chunks that build on the previous chapters
but require nothing from later ones. From the very first chapter, you'll have a
working program you can run and play with. With each passing chapter, it grows
increasingly full-featured until you eventually have a complete language.
-->
为了将两枚解释器的庞杂内容切分成各个内容适中的篇章，让各个篇章之间尽可能相对独立不相互影响，
可着实费了我一番功夫。从第一个篇章开始，你就可以写出一支可以运行的解释器程序，随着章节推进，
这支小解释器程序逐步成长，羽翼渐丰，直到你完整实现出全部功能。

<!--
Aside from copious, scintillating English prose, chapters have a few other
delightful facets:
-->
除了内容丰富、语句清晰的主体内容外，每个章节还会包含以下数个讨喜的部分：

<!--
--- The code
-->
### 代码

<!--
We're about *crafting* interpreters, so this book contains real code. Every
single line of code needed is included, and each snippet tells you where to
insert it in your ever-growing implementation.
-->
既然我们要开发一枚货真价实的程序语言解释器，代码部分自然是不可或缺的啦。任何一行重要的实现代码都将被包含其中详细阐释。贴出的代码片段还会贴心地告诉你应在何处插入这段代码。

<!--
Many other language books and language implementations use tools like [Lex][]
and <span name="yacc">[Yacc][]</span>, so-called **compiler-compilers**, that
automatically generate some of the source files for an implementation from some
higher-level description. There are pros and cons to tools like those, and
strong opinions -- some might say religious convictions -- on both sides.
-->
许多讲授程序设计语言开发的书籍都会借助一些类似<span name="yacc">[Lex][]、[Yacc][]</span>这样的**编译器编译程序（Compiler-compilers）**，让编译器编译程序帮忙自动生成一些源代码文件，
这么做有人说好，有人说不好，争论双方各执己见，各有各的道理。

<aside name="yacc">

<!--
Yacc is a tool that takes in a grammar file and produces a source file for a
compiler, so it's sort of like a "compiler" that outputs a compiler, which is
where we get the term "compiler-compiler".
-->
Yacc 是一个可以接收固定格式语法文件，为语法生成编译器源文件的辅助工具。这听起来像是可以生成"编译器"的编译器，所以我们就为这类工具起了个专业的名词：编译器编译程序。

<!--
Yacc wasn't the first of its ilk, which is why it's named "Yacc" -- *Yet
Another* Compiler-Compiler. A later similar tool is [Bison][], named as a pun on
the pronunciation of Yacc like "yak".
-->
Yacc 不是其同类产品中的第一个，这也是为什么它被命名为 Yacc 的原因："*Yet Another* Compiler-Compiler"，在它之后还有[Bison][]这样的工具诞生。Yacc 的发音类似于："yak（牦牛）"。

<img src="image/introduction/yak.png" alt="A yak.">

[bison]: https://en.wikipedia.org/wiki/GNU_bison

<!--
If you find all of these little self-references and puns charming and fun,
you'll fit right in here. If not, well, maybe the language nerd sense of humor
is an acquired taste.
-->
如果你觉得这些小贴士和双关语很有意思，那你可真是我的同类。但如果你体会不到，也无伤大雅，
也许是程序语言书呆子们的幽默感一点也不好笑吧，哈哈。

</aside>

<!--
We will abstain from using them here. I want to ensure there are no dark corners
where magic and confusion can hide, so we'll write everything by hand. As you'll
see, it's not as bad as it sounds, and it means you really will understand each
line of code and how both interpreters work.
-->
在这本书中，我们不会使用这些辅助工具。我想确保在实现程序设计语言的道路上不会有任何黑暗的角落和神奇的魔法，我们将完完整整地写下全部代码，你也将充分理解写下的每一行代码是如何在解释器里工作的。

[lex]: https://en.wikipedia.org/wiki/Lex_(software)
[yacc]: https://en.wikipedia.org/wiki/Yacc

<!--
A book has different constraints from the "real world" and so the coding style
here might not always reflect the best way to write maintainable production
software. If I seem a little cavalier about, say, omitting `private` or
declaring a global variable, understand I do so to keep the code easier on your
eyes. The pages here aren't as wide as your IDE and every character counts.
-->
写在书中的代码与"真实世界"的代码总会有一些不同，某些代码风格也不是编写可维护软件代码的最佳方式。如果被你发现，比如说：漏写了个`private`，声明了一个全局变量，还请不要见怪。我力求代码简洁易懂，而且，这里书页的宽度也比不上集成开发环境（IDE）。

<!--
Also, the code doesn't have many comments. That's because each handful of lines
is surrounded by several paragraphs of honest-to-God prose explaining it. When
you write a book to accompany your program, you are welcome to omit comments
too. Otherwise, you should probably use `//` a little more than I do.
-->
另外，书中的代码不会带有太多的注释，这是因为每行代码的意义都会周围的几个段落被详细地解释清楚。如果以后你也提笔撰写一本自己的技术书，我也推荐使用这样的技术文风：少注释，多解释。

<!--
While the book contains every line of code and teaches what each means, it does
not describe the machinery needed to compile and run the interpreter. I assume
you can slap together a makefile or a project in your IDE of choice in order to
get the code to run. Those kinds of instructions get out of date quickly, and
I want this book to age like XO brandy, not backyard hooch.
-->
虽然书中包含了每一行需要实现的代码和对代码的解释，但我没有加入如何让解释器程序编译运行起来的描述性文本。这是因为我相信你有能力快速撰写一份`Makefile`或是使用 IDE 建立一个工程项目，从而让代码运行起来，相信我，这不是什么难事，而且，描述如何让程序编译执行的文字很快就会过时，新的代码工具层出不穷。我希望这本书能像白兰地酒那样，越老越醇香。

<!--
--- Snippets
-->
### 代码片段

<!--
Since the book contains literally every line of code needed for the
implementations, the snippets are quite precise. Also, because I try to keep the
program in a runnable state even when major features are missing, sometimes we
add temporary code that gets replaced in later snippets.
-->
除了包含实现解释器所需的代码之外，本书还会包含一些描述精准的代码片段。这是因为我需要让解释器程序在缺失某些主体功能的情况下也能正常运行，所以我会使用代码片段告诉你如何添加上临时性代码，以及后续如何替换掉它们。

<!--
A snippet with all the bells and whistles looks like this:
-->
一份典型的代码片段长这样：

<div class="codehilite"><pre class="insert-before">
      default:
</pre><div class="source-file"><em>lox/Scanner.java</em><br>
in <em>scanToken</em>()<br>
replace 1 line</div>
<pre class="insert">
        <span class="k">if</span> (<span class="i">isDigit</span>(<span class="i">c</span>)) {
          <span class="i">number</span>();
        } <span class="k">else</span> {
          <span class="t">Lox</span>.<span class="i">error</span>(<span class="i">line</span>, <span class="s">&quot;Unexpected character.&quot;</span>);
        }
</pre><pre class="insert-after">
        break;
</pre></div>
<div class="source-file-narrow"><em>lox/Scanner.java</em>, in <em>scanToken</em>(), replace 1 line</div>

<!--
In the center, you have the new code to add. It may have a few faded out lines
above or below to show where it goes in the existing surrounding code. There is
also a little blurb telling you in which file and where to place the snippet. If
that blurb says "replace _ lines", there is some existing code between the faded
lines that you need to remove and replace with the new snippet.
-->
可以看到，在代码片段的中心位置，有需要添加的代码，这部分代码被着重显示。新增代码上下环绕几行颜色偏淡已经存在的上下文代码。侧边会有个简短提示，告诉你应在哪个文件的哪个位置插入代码。如果提示文字中有诸如"替换_行"的描述，这就表示你需要先移除此处的旧代码，再替换上代码片段中的新代码。

<!--
--- Asides
-->
### 旁注

<!--
<span name="joke">Asides</span> contain biographical sketches, historical
background, references to related topics, and suggestions of other areas to
explore. There's nothing that you *need* to know in them to understand later
parts of the book, so you can skip them if you want. I won't judge you, but I
might be a little sad.
-->
<span name="joke">旁注</span>通常包含一些简单草图、章节历史背景、相关主题的参考内容以及对其他领域的建议，忽略这部分内容*完全不影响*你阅读接下来的章节内容。所以，你可以放心地忽略它们。忽略这部分的内容我也不会怪你，只是，我可能有点难过啦。

<aside name="joke">

<!--
Well, some asides do, at least. Most of them are just dumb jokes and amateurish
drawings.
-->
好吧，有些旁注确实如我所言那般，但更多的都是一些小笑话和随手涂鸦罢了。

</aside>

<!--
--- Challenges
-->
### 挑战

<!--
Each chapter ends with a few exercises. Unlike textbook problem sets, which tend
to review material you already covered, these are to help you learn *more* than
what's in the chapter. They force you to step off the guided path and explore on
your own. They will make you research other languages, figure out how to
implement features, or otherwise get you out of your comfort zone.
-->
在每个章节的结尾部分，会带有几道练习题。与教科书后的习题集不同，本书的练习题旨在帮助你复习你已经学到的内容，引导你去思考章节内容之外的东西。习题迫使你走出既定的引导路线去探索更多，这对你研究其他语言的功能实现大有裨益。不是有句话这么说嘛：你要勇于跳出舒适区。

<!--
<span name="warning">Vanquish</span> the challenges and you'll come away with a
broader understanding and possibly a few bumps and scrapes. Or skip them if you
want to stay inside the comfy confines of the tour bus. It's your book.
-->
努力<span name="warning">克服</span>这些挑战，你必然会有新的感悟与收获，亦或是，你也可以简单地跳过它们，继续前进。

<aside name="warning">

<!--
A word of warning: the challenges often ask you to make changes to the
interpreter you're building. You'll want to implement those in a copy of your
code. The later chapters assume your interpreter is in a pristine
("unchallenged"?) state.
-->
给你一点提示：这些习题挑战时常会需要你修改已经实现的语言解释器代码，你最好是拷贝一份代码来做题，后续章节会以未经修改的原始代码状态（没做挑战习题）为起点继续前进。

</aside>

<!--
--- Design notes
-->
### 设计笔记

<!--
Most "programming language" books are strictly programming language
*implementation* books. They rarely discuss how one might happen to *design* the
language being implemented. Implementation is fun because it is so <span
name="benchmark">precisely defined</span>. We programmers seem to have an
affinity for things that are black and white, ones and zeroes.
-->
大多数描述开发"程序设计语言"的书都将视线聚焦于如何按照定义*严格实现*一门语言，却很少谈及一个语言特性背后的设计思路，为什么这门语言要如此*设计*。实现程序语言的过程非常有趣，因为一门程序语言的语法规范是经过<span name="benchmark">严格定义</span>的。做我们程序员们的似乎总是很喜欢被严格定义清楚的东西，非黑即白，非零即一，泾渭分明，毫无歧义。

<aside name="benchmark">
<!--
I know a lot of language hackers whose careers are based on this. You slide a
language spec under their door, wait a few months, and code and benchmark
results come out.
-->
据我了解，有一大批程序语言开发者们将他们的毕生精力倾注于此。你随手把一个语言规范贴到他们门框上，几个月后，他们就带着实现代码与测试报告出来了。

</aside>

<!--
Personally, I think the world needs only so many implementations of <span
name="fortran">FORTRAN 77</span>. At some point, you find yourself designing a
*new* language. Once you start playing *that* game, then the softer, human side
of the equation becomes paramount. Things like which features are easy to learn,
how to balance innovation and familiarity, what syntax is more readable and to
whom.
-->
就我个人而言，我觉得这个世界只需要实现<span name="fortran"> FORTRAN 77 </span>这一门语言就足够了。如果有一天，你自己开始着手设计一门*全新*的程序语言，这时候，一些考虑“人”的因素就会显示出来，比如：哪些语言特性简单易学，如何平衡语言的创新性与相似性，语法上是否更应该注重可读性，这门语言的目标人群是哪些等等等等。

<aside name="fortran">

<!--
Hopefully your new language doesn't hardcode assumptions about the width of a
punched card into its grammar.
-->
哦，希望你设计的新语言不要把打孔纸带的宽度也编入到语言规范里头去。

</aside>

<!--
All of that stuff profoundly affects the success of your new language. I want
your language to succeed, so in some chapters I end with a "design note", a
little essay on some corner of the human aspect of programming languages. I'm no
expert on this -- I don't know if anyone really is -- so take these with a large
pinch of salt. That should make them tastier food for thought, which is my main
aim.
-->
上述的这些都是你的新语言能否成功的重要因素。我也希望你的语言能大获成功，所以在部分章节的结尾，我会留下“设计笔记”，一篇从程序语言使用者视角出发，探讨程序语言设计的小短文。当然，我个人并不是这方面的专家，写得不好也请别见怪，糊上一把盐吞下去吧，味道能好不少。如果我写的这些内容能够发人深思，那我的目的也就达到了。

<!--
-- The First Interpreter
-->
## 第一支解释器

We'll write our first interpreter, jlox, in <span name="lang">Java</span>. The
focus is on *concepts*. We'll write the simplest, cleanest code we can to
correctly implement the semantics of the language. This will get us comfortable
with the basic techniques and also hone our understanding of exactly how the
language is supposed to behave.

<aside name="lang">

The book uses Java and C, but readers have ported the code to [many other
languages][port]. If the languages I picked aren't your bag, take a look at
those.

[port]: https://github.com/munificent/craftinginterpreters/wiki/Lox-implementations

</aside>

Java is a great language for this. It's high level enough that we don't get
overwhelmed by fiddly implementation details, but it's still pretty explicit.
Unlike in scripting languages, there tends to be less complex machinery hiding
under the hood, and you've got static types to see what data structures you're
working with.

I also chose Java specifically because it is an object-oriented language. That
paradigm swept the programming world in the '90s and is now the dominant way of
thinking for millions of programmers. Odds are good you're already used to
organizing code into classes and methods, so we'll keep you in that comfort
zone.

While academic language folks sometimes look down on object-oriented languages,
the reality is that they are widely used even for language work. GCC and LLVM
are written in C++, as are most JavaScript virtual machines. Object-oriented
languages are ubiquitous, and the tools and compilers *for* a language are often
written *in* the <span name="host">same language</span>.

<aside name="host">

A compiler reads files in one language, translates them, and outputs files in
another language. You can implement a compiler in any language, including the
same language it compiles, a process called **self-hosting**.

You can't compile your compiler using itself yet, but if you have another
compiler for your language written in some other language, you use *that* one to
compile your compiler once. Now you can use the compiled version of your own
compiler to compile future versions of itself, and you can discard the original
one compiled from the other compiler. This is called **bootstrapping**, from
the image of pulling yourself up by your own bootstraps.

<img src="image/introduction/bootstrap.png" alt="Fact: This is the primary mode of transportation of the American cowboy.">

</aside>

And, finally, Java is hugely popular. That means there's a good chance you
already know it, so there's less for you to learn to get going in the book. If
you aren't that familiar with Java, don't freak out. I try to stick to a fairly
minimal subset of it. I use the diamond operator from Java 7 to make things a
little more terse, but that's about it as far as "advanced" features go. If you
know another object-oriented language, like C# or C++, you can muddle through.

By the end of part II, we'll have a simple, readable implementation. It's not
very fast, but it's correct. However, we are only able to accomplish that by
building on the Java virtual machine's own runtime facilities. We want to learn
how Java *itself* implements those things.

## The Second Interpreter

So in the next part, we start all over again, but this time in C. C is the
perfect language for understanding how an implementation *really* works, all the
way down to the bytes in memory and the code flowing through the CPU.

A big reason that we're using C is so I can show you things C is particularly
good at, but that *does* mean you'll need to be pretty comfortable with it. You
don't have to be the reincarnation of Dennis Ritchie, but you shouldn't be
spooked by pointers either.

If you aren't there yet, pick up an introductory book on C and chew through it,
then come back here when you're done. In return, you'll come away from this book
an even stronger C programmer. That's useful given how many language
implementations are written in C: Lua, CPython, and Ruby's MRI, to name a few.

In our C interpreter, <span name="clox">clox</span>, we are forced to implement
for ourselves all the things Java gave us for free. We'll write our own dynamic
array and hash table. We'll decide how objects are represented in memory, and
build a garbage collector to reclaim them.

<aside name="clox">

I pronounce the name like "sea-locks", but you can say it "clocks" or even
"clochs", where you pronounce the "x" like the Greeks do if it makes you happy.

</aside>

Our Java implementation was focused on being correct. Now that we have that
down, we'll turn to also being *fast*. Our C interpreter will contain a <span
name="compiler">compiler</span> that translates Lox to an efficient bytecode
representation (don't worry, I'll get into what that means soon), which it then
executes. This is the same technique used by implementations of Lua, Python,
Ruby, PHP, and many other successful languages.

<aside name="compiler">

Did you think this was just an interpreter book? It's a compiler book as well.
Two for the price of one!

</aside>

We'll even try our hand at benchmarking and optimization. By the end, we'll have
a robust, accurate, fast interpreter for our language, able to keep up with
other professional caliber implementations out there. Not bad for one book and a
few thousand lines of code.

<div class="challenges">

## Challenges

1.  There are at least six domain-specific languages used in the [little system
    I cobbled together][repo] to write and publish this book. What are they?

1.  Get a "Hello, world!" program written and running in Java. Set up whatever
    makefiles or IDE projects you need to get it working. If you have a
    debugger, get comfortable with it and step through your program as it runs.

1.  Do the same thing for C. To get some practice with pointers, define a
    [doubly linked list][] of heap-allocated strings. Write functions to insert,
    find, and delete items from it. Test them.

[repo]: https://github.com/munificent/craftinginterpreters
[doubly linked list]: https://en.wikipedia.org/wiki/Doubly_linked_list

</div>

<div class="design-note">

## Design Note: What's in a Name?

One of the hardest challenges in writing this book was coming up with a name for
the language it implements. I went through *pages* of candidates before I found
one that worked. As you'll discover on the first day you start building your own
language, naming is deviously hard. A good name satisfies a few criteria:

1.  **It isn't in use.** You can run into all sorts of trouble, legal and
    social, if you inadvertently step on someone else's name.

2.  **It's easy to pronounce.** If things go well, hordes of people will be
    saying and writing your language's name. Anything longer than a couple of
    syllables or a handful of letters will annoy them to no end.

3.  **It's distinct enough to search for.** People will Google your language's
    name to learn about it, so you want a word that's rare enough that most
    results point to your docs. Though, with the amount of AI search engines are
    packing today, that's less of an issue. Still, you won't be doing your users
    any favors if you name your language "for".

4.  **It doesn't have negative connotations across a number of cultures.** This
    is hard to be on guard for, but it's worth considering. The designer of
    Nimrod ended up renaming his language to "Nim" because too many people
    remember that Bugs Bunny used "Nimrod" as an insult. (Bugs was using it
    ironically.)

If your potential name makes it through that gauntlet, keep it. Don't get hung
up on trying to find an appellation that captures the quintessence of your
language. If the names of the world's other successful languages teach us
anything, it's that the name doesn't matter much. All you need is a reasonably
unique token.

</div>
