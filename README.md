# RavenScheme简介

RavenScheme使用了一个运行『读入-求值-打印』循环的解释器。该解释器从标准输入反复的读入表达式，对得到的表达式求值，然后打印出结果。

如果用户键入：

【加 1 2】

解释器将打印出3

如果用户键入3

解释器会打印出3

&nbsp;

RavenScheme提供一个加载函数，它可以从文件读入并行求值：

【加载 『我的程序』】

&nbsp;

RavenScheme对表达式采用了剑桥波兰记法。中文括号表示函数应用。左括号的第一个表达式表示函数，随后的表达式是它的参数。假定我们键入：

【【加 1 2】】

当解释器看到内层的括号时，他就会将1和2作为参数来调用函数加。由于外面还有一层括号，解释器就会将3作为一个无参函数来调用，这是一个错误：

求值：3不是一个有效的函数

RavenScheme中额外的括号会改变程序的语义：

【加 1 2】 --&gt; 7

【【加 1 2】】 --&gt; 错误

&nbsp;

我们可以阻止解释器对括起的表达式求值，为此需要加上一个引用：

【引用 【加 1 2】】 --&gt; 【加 1 2】

一种特殊的简写方式：

「【加 1 2】 --&gt; 【加 1 2】

&nbsp;

虽然在RavenScheme中每个表达式都有类型，但通常必须到运行时才能确定这个类型。大多数预定义函数都会做动态检查，以保证它们的参数具有合适的类型。表达式：

【如果 【大于 变量 0】 【加 1 2】 【加 1 『字符串』】】

在变量为正时将求出值3，否则会产生运行时的动态类型冲突错误。

&nbsp;

参数对多个类型都有意义的函数，自然是多态的：

【设置 最小值 【函数 【变量一 变量二】 【如果 【小于 变量一 变量二】 变量一 变量二】】】

表达式【最小值 123 456】将求出123，而【最小值 3.14 2.7】将求出2.7。

&nbsp;

用户定义函数也可以利用预定义的类型谓词函数实现类似的检查：

【是布尔值吗&nbsp;变量】

【是字符吗 变量】

【是字符串吗 变量】

【是符号吗 变量】

【是数值吗 变量】

【是对吗 变量】

【是表吗 变量】

&nbsp;

RavenScheme中的符号类似于其他语言中被称为标识符的东西。标识符中允许包含很多标点符号。

【是符号吗 《￥#@+%&amp;*1】 --&gt; 真

真表示布尔真，布尔假用假表示。

&nbsp;

要在RavenScheme中创建函数，就需要对lambda表达式求值：

【函数 【参数一 参数二】 【乘 参数一&nbsp;参数二】】 --&gt; 函数

lambda表达式的第一个参数是函数的形式参数列表。其余参数构成函数的体。

&nbsp;

lambda表达式不给它的函数命名，这件事要由『使得』或『设置』做。

当一个函数被调用时，语言实现将恢复相应lambda表达式求值时的引用环境。然后扩充这个引用环境，加入形式参数的约数，在其中按顺序求值函数体中的各个表达式。最后的那个表达式的值成为函数的返回值。

【【函数 【参数】 【乘 参数 参数】】 3】 --&gt; 9

&nbsp;

简单的条件表达式用『如果』：

【如果 【小于 1 2】 3 4】 --&gt; 3

【如果 假 1 2】 --&gt; 2

一般来说RavenScheme的表达式按照应用序求值。如『函数』和『如果』这样的特殊型是这个规则的例外。『如果』的实现检测其第一个参数的求值是否得到『真』，如果是，就返回它的第二个参数的值，而且并不求值第三个参数；否则就返回第三个参数的值，但并不求值第二个参数。

&nbsp;

# 约束

通过引入嵌套的作用域，可以将名字约束到值：

【使得 【【甲 3】

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 【乙 4】

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 【平方 【函数 【参】 【乘 参 参】】】

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 【加法 加】】

&nbsp; &nbsp; 【加法 【平方 甲】 【平方 乙】】】 ---&gt; 25

特殊型『使得』有两个或多个参数，其中第一个参数是一些二元组的表，每个二元组的第一个元素是名字，第二个元素就是这个名字在『使得』的第二个参数中代表的值。其余的参数将按顺序求值，对于整个结构的求值将是最终参数的值。

&nbsp;

由『使得』产生的约束的作用域就是这个『使得』的第二个参数：

【使得 【【甲 3】】

&nbsp; &nbsp; 【使得 【【甲 4】

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 【乙 甲】】

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;【加 甲 乙】】】 --&gt; 7

其中乙取外层甲的值。这种在声明表最后『所有事情一起做』的语义使『使得』不能用于定义递归函数。

『递归使得』用于处理这个问题：

【递归使得&nbsp;【【搞 【函数 【参】

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 【如果 【等于 参 1】 1

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 【乘 参 【搞 【减 参 1】】】】】】】

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;【搞 5】】 --&gt; 120

&nbsp;

还有一种『依次使得』，其中的名字一个一个地编程可见的，因此在后面定义可以用前面的东西，但反过来不行。

&nbsp;

虽然『使得』和『递归使得』使用户可以创建嵌套的作用域，但他们不会影响全局名字的意义。RavenScheme引进了一个称为『设置』的特殊型，其副作用就是为名字创建全局约束：

【设置 距离&nbsp;

&nbsp; &nbsp; 【函数 【甲 乙】

&nbsp; &nbsp; &nbsp; &nbsp; 【开方 【加 【乘 甲 甲】 【乘 乙 乙】】】】】

【距离 3 4】 --&gt; 5

&nbsp;

# 表和数

与所有Lisp方言一样，RavenScheme也提供了很多表操作函数。

表是一种递归定义的结构，它或者是空表，或者是一个有序对，有序对由一个可以使表或原子的对象和另一个表组成。表对于函数式语言程序设计特别有用，因为大多数工作都是通过递归函数或高阶函数完成的。事实上，程序本身也是表，可以在运行中构造一个表并执行它，通过这种方式来扩展程序自身。

RavenScheme的表用【】括起，其中的元素用空白分割，如【甲 乙 丙 丁】。上述记法表示的是完全的表，其最后有序对包含一个元素和一个空表。

由于程序也是表，因此必须区分需要用来求值的表和那些作为一种数据结构、应该就是那样的表。

为了组织系统来求值作为文字量的表，要为表加上引用，写作【引用 【甲 乙 丙 丁】】，或简写为「【甲 乙 丙 丁】。

表的最基本操作就是构造表，以及提取出表中的成分：

【构造 「甲 「【乙 丙】】 --&gt; 【甲 乙 丙】

【取首 「【甲 乙 丙】】 --&gt; 甲

【取尾 「【甲 乙 丙】】 --&gt; 【乙 丙】

【取尾 「【甲】】 --&gt; 空

【取首 空】 --&gt; 空

【取尾 空】 --&gt; 空

【追加 「【甲 乙】 「【丙】】 --&gt; 【甲 乙 丙】

&nbsp;

谓词『是空吗』确定其参数是否为一个空表：

【是空吗 【取首 「【2】】】 --&gt; 真

&nbsp;

为了能快速访问一个序列中的任意元素，RavenScheme还提供了数组类型，它以整数作为下标，但其中元素的类型可以不同。

&nbsp;

# 相等检测和检索

RavenScheme提供两种不同的相等检测函数。对于数值比较，『等于』在需要时将执行类型转换。对于通常的使用，『相等』将进行深比较，『相同』将进行浅比较。

要在一个表中检索元素，函数『相同成员』『相等成员』以一个元素和一个表作为参数，返回以表中这个元素作为开头的最长后缀。

如果要找的元素不存在，就返回假。RavenScheme的条件表达式将一切不是假的东西都当做真。

&nbsp;

# 控制流和赋值

我们在前面已经看过特殊型『如果』的情况，与它差不多的还有『条件』，它与更常见的if&hellip;&hellip;elseif&hellip;&hellip;else&hellip;&hellip;很像：

【条件

&nbsp; &nbsp; 【【小于 3 2】 1】

&nbsp; &nbsp; 【【小于 4 3】 2】

&nbsp; &nbsp; 【否则 3】】 --&gt; 3

&nbsp;

『条件』的参数是一些二元组，它们将从头到尾按顺序考虑。如果第一个二元组的第一个元素求值得到真，那么整个表达式的值就是这个二元组的第二个元素的值。如果没有任何二元组的第一个元素求出真，那么整个表达式的值就是假。符号『否则』只能作为整个结构中最后一个二元组的第一个元素，它是『真』的语法糖。

&nbsp;

RavenScheme也提供了赋值、顺序复合和迭代结构。赋值用特殊型『赋值』和函数『首赋值』『尾赋值』实现：

【使得 【【甲 1】

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 【乙 「【子 丑】】】

&nbsp; &nbsp; &nbsp;【赋值 甲 2】

&nbsp; &nbsp; &nbsp;【赋值首 乙 「【寅 卯】】

&nbsp; &nbsp; &nbsp;【赋值尾 乙 「【啊】】

&nbsp; &nbsp; &nbsp;&hellip;&hellip; 甲 --&gt; 2

&nbsp; &nbsp; &nbsp;&hellip;&hellip; 乙 --&gt; 【【寅 卯】 啊】

&nbsp;

『赋值』的返回值为空。

&nbsp;

顺序结构使用特殊型『执行』

【执行

&nbsp; &nbsp; 【输出 『子曰』】

&nbsp; &nbsp; 【输出 『诗云』】】

&nbsp;

迭代结构使用特殊型『做』和函数『对每个』实现：

【设置 斐波迭代器 【函数 【界】

&nbsp; &nbsp; ；输出第n+1个斐波那契的值

&nbsp; &nbsp; 【做 【【子 0 【加 子 1】】 &nbsp; &nbsp; &nbsp; &nbsp;；最初为0，每次迭代加1

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;【甲 0 乙】 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;；最初为0，每次迭代设为b

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;【乙 1 【加 甲 乙】】】 &nbsp; &nbsp;；最初为1，设置为a和b的和

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 【【等于 子 界】 乙】 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;；迭代终止测试和返回值

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 【输出 乙】 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;；函数主体

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp; 【输出 『 』】】】】 &nbsp; &nbsp; &nbsp; &nbsp; &nbsp;；函数主体

【对每个 【函数 【甲 乙】 【输出 【乘 甲 乙】】 【换行】】

&nbsp; &nbsp; &nbsp; &nbsp;「【1 2 3】

&nbsp; &nbsp; &nbsp; &nbsp;「【4 5 6】】

&nbsp;

『做』的第一个参数是一些三元组的表，每个三元组描述了一个新变量、这个变量的初始值以及一个表达式。在每次迭代最后求值这个表达式，并将得到的值赋给相应变量的新实例。第二个参数是个二元组，指定了结束条件和要返回的表达式。在每次迭代最后，所有循环变量用当前值计算，只有所有的新值都计算完后才执行赋值。

&nbsp;

函数『对每个』以一个函数和一系列的表作为参数。序列中表的个数必须与函数的参数个数相同，所有的表必须一样长。『对每个』反复调用作为其参数的那个函数，从作为另一个参数的那一组表中顺序取出实参传递给它。在上面的例子中，匿名函数由一个lambda表达式生成，它将分别用参数1和4，2和5，3和6调用，解释器将打印出：

4

10

18

空

最后一个是『对每个』的返回值，其中假定是空。

&nbsp;

# 惰性求值

RavenScheme通过内部函数『延迟』『出力』提供了可选的正则序求值功能。这两个函数提供了惰性求值的一种实现。如果没有副作用，惰性求值的语义与正则序完全相同，但语言的实现需要维护关于哪些表达式已经求值的轨迹。如果在特定引用环境中多次需要这些表达式的值，那么就可以重复使用已经求出的那些值。

&nbsp;

一个『延迟』的表达式有时也称为一个许诺，用于维护各个许诺是否已求过值的机制称为记忆器。因为应用序求值是RavenScheme的默认机制，所以必须用特殊的语法形式来传递未求值表达式，而且使用这种表达式也需要特殊语法形式。

&nbsp;

惰性求值的一种常见用途是创建无穷数据结构，或称为惰性数据结构（函数式线段树？），它们可以根据实际请求而逐渐现形。

下面的代码创建了一个表，表中是所有的自然数：

【设置 自然数们

&nbsp; &nbsp; 【递归使得 【【下一个 【函数 【参】 【构造 参 【延迟 【下一个 【加 参 1】】】】】】】

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;【下一个 1】】】

【设置 首 取首】

【设置 尾 【函数 【流】 【出力 【取尾 流】】】】

有了这个定义之后，我们就可以根据需要取得任意多个自然数了：

【首 自然数们】 --&gt; 1

【首 【尾 自然数们】】 --&gt; 2

【首 【尾 【尾 自然数们】】】 --&gt; 3

这个表只占据了我们实际上探查的那些空间。更精妙的惰性数据结构（例如树）可以用于组合搜索问题，利用这种结构，巧妙的算法可以只探查潜在巨大的搜索空间中的某些路径。

&nbsp;

# 程序作为表

&nbsp;现在应该很清楚了，RavenScheme中的程序采用了表的形式。用技术术语说，是自表示的。括起的符号序列称为S-表达式，不管我们是将它们看作程序，还是看做表。事实上，未求值的程序也是表，也可以构造、析构，或用任何普通的表函数来操作它们。

&nbsp;

『引用』可以禁止对出现在函数调用中作为实参的表进行求值，RavenScheme提供了另一个与它对应的函数『求值』，利用它可以用来求值作为数据机构创建的表：

&nbsp;

【设置 组成

&nbsp; &nbsp; 【函数 【甲 乙】

&nbsp; &nbsp; &nbsp; &nbsp; &nbsp;【函数 【参】 【甲 【乙 参】】】】】

&nbsp;

【【组成 取首 取尾】 「【1 2 3】】 --&gt; 2

&nbsp;

【设置 组成二

&nbsp; &nbsp; 【函数 【甲 乙】

&nbsp; &nbsp; &nbsp; &nbsp; 【求值 【列表 「函数 「【参】 【列表 甲 【列表 乙 「参】】】】】】

【【组成二 取首 取尾】 「【1 2 3】】 --&gt; 2

&nbsp;

按照上面第一个声明，『组成』以一对函数『甲』『乙』做为参数，返回一个函数作为结果。返回的这个函数将以值『参』作为参数，先将『乙』作用于它，然后在应用【甲】，最后返回结果。第二个声明中的『组成二』具有相同的功能，但是用另一种完全不同的方式。函数『列表』返回一个表，这个表由它的经过求值的参数构成。在函数体中，这个表就是未求值的表达式【函数 【参】 【甲 【乙 参】】】。将这个表达式传给『求值』，就可以由这个表求出所需要的函数来。

&nbsp;

『求值』和『应用』，前面已经看到了第一个函数，第二个函数『应用』有两个参数：一个函数和一个表，其效果就相当于调用这个函数，以表中的元素作为函数的实际参数。

当传给『求值』的是数或字符串时，它直接返回这个数或字符串；如果传给它的是符号，它就在给定的环境中查找这个符号，并返回该符号的约束值。如果传给它的是表，『求值』就检查它，看看表中第一个元素是不是属于一小组构造在语言实现中的特殊名字，即那些称为原语的特殊型的名字。『求值』对这些特殊型（函数、如果、设置、赋值、引用等）提供了直接实现。对于其他表，『求值』将对其中的每个元素递归调用自己，然后调用『应用』，传给它的是第一个元素的值（它必须是个函数）和一个包含其他元素的值的表。最后，『求值』返回『应用』的返回值。

&nbsp;

在得到了函数和实参表之后『应用』先查看函数的内部表示，看它是不是一个原语函数。如果是的话就调用其内部实现，否则就从函数的表示中提取出函数的lambda表达式求值所在的引用环境，在这个环境中加入函数的形式参数的名字以及从实参表中取出的值。将这样的得到的环境与构成函数体的那些表达式传给『求值』。『应用』最终返回的是『求值』对函数体中的最后一个表达式求值的结果。

&nbsp;

# 高阶函数

如果一个函数以函数作为实际参数，或者返回函数作为值，那么它就是一个高阶函数。

RavenScheme中的『映射』与『对每个』类似，它以一个函数和一系列的表作为参数。表中的数目必须与这个函数所要求的实参个数相同，表的长度也必须一样。

『映射』将对这些表中的一组元素调用相应的函数：

【映射 乘 「【2 4 6】 「【3 5 7】】 --&gt; 【6 20 42】

其中『对每个』是为了副作用而执行的，其值由具体实现确定。而『映射』是纯函数式的，它返回一个表，其中包含对函数参数的调用返回的值。
