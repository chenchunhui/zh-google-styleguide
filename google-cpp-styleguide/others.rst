5. 其他 C++ 特性
------------------------

5.1. 引用参数
~~~~~~~~~~~~~~~~~~~~

.. tip::
    所以按引用传递的参数必须加上 ``const``.
    
定义:
    在 C 语言中, 如果函数需要修改变量的值, 参数必须为指针, 如 ``int foo(int *pval)``. 在 C++ 中, 函数还可以声明引用参数: ``int foo(int &val)``.
    
优点:
    定义引用参数防止出现 ``(*pval)++`` 这样丑陋的代码. 像拷贝构造函数这样的应用也是必需的. 而且更明确, 不接受 ``NULL`` 指针.
    
缺点:
    容易引起误解, 因为引用在语法上是值变量却拥有指针的语义.
    
结论:
    函数参数列表中, 所有引用参数都必须是 ``const``:
        .. code-block:: c++
            
            void Foo(const string &in, string *out);
        
    事实上这在 Google Code 是一个硬性约定: 输入参数是值参或 ``const`` 引用, 输出参数为指针. 输入参数可以是 ``const`` 指针, 但决不能是 非 ``const`` 的引用参数.
    
    在以下情况你可以把输入参数定义为 ``const`` 指针: 你想强调参数不是拷贝而来的, 在对象生存周期内必须一直存在; 最好同时在注释中详细说明一下. ``bind2nd`` 和 ``mem_fun`` 等 STL 适配器不接受引用参数, 这种情况下你也必须把函数参数声明成指针类型.

.. _function-overloading:

5.2. 函数重载
~~~~~~~~~~~~~~~~~~~~

.. tip::
    仅在输入参数类型不同, 功能相同时使用重载函数 (含构造函数). 不要用函数重载模拟 `缺省函数参数 <http://code.google.com/p/google-gflags/>`_.
    
定义:
    你可以编写一个参数类型为 ``const string&`` 的函数, 然后用另一个参数类型为 ``const char*`` 的函数重载它:
        .. code-block:: c++
            
            class MyClass {
                public:
                void Analyze(const string &text);
                void Analyze(const char *text, size_t textlen);
            };
        
优点:
    通过重载参数不同的同名函数, 令代码更加直观. 模板化代码需要重载, 同时为使用者带来便利.
    
缺点:
    限制使用重载的一个原因是在某个特定调用点很难确定到底调用的是哪个函数. 另一个原因是当派生类只重载了某个函数的部分变体, 会令很多人对继承的语义产生困惑. 此外在阅读库的用户代码时, 可能会因反对使用 `缺省函数参数 <http://code.google.com/p/google-gflags/>`_ 造成不必要的费解.
    
结论:
    如果你想重载一个函数, 考虑让函数名包含参数信息, 例如, 使用 ``AppendString()``, ``AppendInt()`` 而不是 ``Append()``.


5.3. 缺省参数
~~~~~~~~~~~~~~~~~~~~

.. tip::
    我们不允许使用缺省函数参数.
    
优点:
    多数情况下, 你写的函数可能会用到很多的缺省值, 但偶尔你也会修改这些缺省值. 无须为了这些偶尔情况定义很多的函数, 用缺省参数就能很轻松的做到这点.
    
缺点:
    大家通常都是通过查看别人的代码来推断如何使用 API. 用了缺省参数的代码更难维护, 从老代码复制粘贴而来的新代码可能只包含部分参数. 当缺省参数不适用于新代码时可能会导致重大问题.
    
结论:
    我们规定所有参数必须明确指定, 迫使程序员理解 API 和各参数值的意义, 避免默默使用他们可能都还没意识到的缺省参数.


5.4. 变长数组和 alloca()
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::
    我们不允许使用变长数组和 ``alloca()``.

优点:
    变长数组具有浑然天成的语法. 变长数组和 ``alloca()`` 也都很高效.

缺点:
    变长数组和 ``alloca()`` 不是标准 C++ 的组成部分. 更重要的是, 它们根据数据大小动态分配堆栈内存, 会引起难以发现的内存越界 bugs: "在我的机器上运行的好好的, 发布后却莫名其妙的挂掉了".
    
结论:
    使用安全的内存分配器, 如 ``scoped_ptr`` / ``scoped_array``.


5.5. 友元
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::
    我们允许合理的使用友元类及友元函数.
    
通常友元应该定义在同一文件内, 避免代码读者跑到其它文件查找使用该私有成员的类. 经常用到友元的一个地方是将 ``FooBuilder`` 声明为 ``Foo`` 的友元, 以便 ``FooBuilder`` 正确构造 ``Foo`` 的内部状态, 而无需将该状态暴露出来. 某些情况下, 将一个单元测试类声明成待测类的友元会很方便.

友元扩大了 (但没有打破) 类的封装边界. 某些情况下, 相对于将类成员声明为 ``public``, 使用友元是更好的选择, 尤其是如果你只允许另一个类访问该类的私有成员时. 当然, 大多数类都只应该通过其提供的公有成员进行互操作.

5.6. 异常
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::
    我们不使用 C++ 异常.
    
优点:
    - 异常允许上层应用决定如何处理在底层嵌套函数中 "不可能出现的" 失败, 不像错误码记录那么含糊又易出错;
    
    - 很多现代语言都使用异常. 引入异常使得 C++ 与 Python, Java 以及其它 C++ 相近的语言更加兼容.
    
    - 许多第三方 C++ 库使用异常, 禁用异常将导致很难集成这些库.
    
    - 异常是处理构造函数失败的唯一方法. 虽然可以通过工厂函数或 ``Init()`` 方法替代异常, 但他们分别需要堆分配或新的 "无效" 状态；
    
    - 在测试框架中使用异常确实很方便.
    
缺点:
    - 在现有函数中添加 ``throw`` 语句时, 你必须检查所有调用点. 所有调用点得至少有基本的异常安全保护, 否则永远捕获不到异常, 只好 "开心的" 接受程序终止的结果. 例如, 如果 ``f()`` 调用了 ``g()``, ``g()`` 又调用了 ``h()``, ``h`` 抛出的异常被 ``f`` 捕获, ``g`` 要当心了, 很可能会因疏忽而未被妥善清理.
    
    - 更普遍的情况是, 如果使用异常, 光凭查看代码是很难评估程序的控制流: 函数返回点可能在你意料之外. 这回导致代码管理和调试困难. 你可以通过规定何时何地如何使用异常来降低开销, 但是让开发人员必须掌握并理解这些规定带来的代价更大.
    
    - 异常安全要求同时采用 RAII 和不同编程实践. 要想轻松编写正确的异常安全代码, 需要大量的支撑机制配合. 另外, 要避免代码读者去理解整个调用结构图, 异常安全代码必须把写持久化状态的逻辑部分隔离到 "提交" 阶段. 它在带来好处的同时, 还有成本 (也许你不得不为了隔离 "提交" 而整出令人费解的代码). 允许使用异常会驱使我们不断为此付出代价, 即使我们觉得这很不划算.

    - 启用异常使生成的二进制文件体积变大, 延长了编译时间 (或许影响不大), 还可能增加地址空间压力;
    
    - 异常的实用性可能会怂恿开发人员在不恰当的时候抛出异常, 或者在不安全的地方从异常中恢复. 例如, 处理非法用户输入时就不应该抛出异常. 如果我们要完全列出这些约束, 这份风格指南会长出很多!

结论:
    从表面上看, 使用异常利大于弊, 尤其是在新项目中. 但是对于现有代码, 引入异常会牵连到所有相关代码. 如果新项目允许异常向外扩散, 在跟以前未使用异常的代码整合时也将是个麻烦. 因为 Google 现有的大多数 C++ 代码都没有异常处理, 引入带有异常处理的新代码相当困难.
    
    鉴于 Google 现有代码不接受异常, 在现有代码中使用异常比在新项目中使用的代价多少要大一些. 迁移过程比较慢, 也容易出错. 我们不相信异常的使用有效替代方案, 如错误代码, 断言等会造成严重负担.
    
    我们并不是基于哲学或道德层面反对使用异常, 而是在实践的基础上. 我们希望在 Google 使用我们自己的开源项目, 但项目中使用异常会为此带来不便, 因此我们也建议不要在 Google 的开源项目中使用异常. 如果我们需要把这些项目推倒重来显然不太现实.
    
    对于 Windows 代码来说, 有个 :ref:`特例 <windows-code>`.

(YuleFox 注: 对于异常处理, 显然不是短短几句话能够说清楚的, 以构造函数为例, 很多 C++ 书籍上都提到当构造失败时只有异常可以处理, Google 禁止使用异常这一点, 仅仅是为了自身的方便, 说大了, 无非是基于软件管理成本上, 实际使用中还是自己决定)


5.7. 运行时类型识别
~~~~~~~~~~~~~~~~~~~~

.. tip::
    我们禁止使用 RTTI.
    
定义:
    RTTI 允许程序员在运行时识别 C++ 类对象的类型.
    
优点:
    RTTI 在某些单元测试中非常有用. 比如进行工厂类测试时, 用来验证一个新建对象是否为期望的动态类型.
    
    除测试外, 极少用到.
    
缺点:
    在运行时判断类型通常意味着设计问题. 如果你需要在运行期间确定一个对象的类型, 这通常说明你需要考虑重新设计你的类.
    
结论:
    除单元测试外, 不要使用 RTTI. 如果你发现自己不得不写一些行为逻辑取决于对象类型的代码, 考虑换一种方式判断对象类型.
    
    如果要实现根据子类类型来确定执行不同逻辑代码, 虚函数无疑更合适. 在对象内部就可以处理类型识别问题.
    
    如果要在对象外部的代码中判断类型, 考虑使用双重分派方案, 如访问者模式. 可以方便的在对象本身之外确定类的类型.
    
    如果你认为上面的方法你真的掌握不了, 你可以使用 RTTI, 但务必请三思 :-) . 不要试图手工实现一个貌似 RTTI 的替代方案, 我们反对使用 RTTI 的理由, 同样适用于那些在类型继承体系上使用类型标签的替代方案.

    
5.8. 类型转换
~~~~~~~~~~~~~~~~~~~~

.. tip::
    使用 C++ 的类型转换, 如 ``static_cast<>()``. 不要使用 ``int y = (int)x`` 或 ``int y = int(x)`` 等转换方式;
    
定义:
    C++ 采用了有别于 C 的类型转换机制, 对转换操作进行归类.
    
优点:
    C 语言的类型转换问题在于模棱两可的操作; 有时是在做强制转换 (如 ``(int)3.5``), 有时是在做类型转换 (如 ``(int)"hello"``). 另外, C++ 的类型转换在查找时更醒目.
    
缺点:
    恶心的语法.
    
结论:
    不要使用 C 风格类型转换. 而应该使用 C++ 风格.
    
        - 用 ``static_cast`` 替代 C 风格的值转换, 或某个类指针需要明确的向上转换为父类指针时.
        - 用 ``const_cast`` 去掉 ``const`` 限定符.
        - 用 ``reinterpret_cast`` 指针类型和整型或其它指针之间进行不安全的相互转换. 仅在你对所做一切了然于心时使用.
        - ``dynamic_cast`` 测试代码以外不要使用. 除非是单元测试, 如果你需要在运行时确定类型信息, 说明有 `设计缺陷 <design flaw>`_.


5.9. 流
~~~~~~~~~~~~~~~~~~~~

.. tip::
    只在记录日志时使用流.
    
定义:
    流用来替代 ``printf()`` 和 ``scanf()``.
    
优点:
    有了流, 在打印时不需要关心对象的类型. 不用担心格式化字符串与参数列表不匹配 (虽然在 gcc 中使用 ``printf`` 也不存在这个问题). 流的构造和析构函数会自动打开和关闭对应的文件.
    
缺点:
    流使得 ``pread()`` 等功能函数很难执行. 如果不使用 ``printf`` 风格的格式化字符串, 某些格式化操作 (尤其是常用的格式字符串 ``%.*s``) 用流处理性能是很低的. 流不支持字符串操作符重新排序 (%1s), 而这一点对于软件国际化很有用.

结论:
    不要使用流, 除非是日志接口需要. 使用 ``printf`` 之类的代替.
    
    使用流还有很多利弊, 但代码一致性胜过一切. 不要在代码中使用流.

拓展讨论:
    对这一条规则存在一些争论, 这儿给出点深层次原因. 回想一下唯一性原则 (Only One Way): 我们希望在任何时候都只使用一种确定的 I/O 类型, 使代码在所有 I/O 处都保持一致. 因此, 我们不希望用户来决定是使用流还是 ``printf + read/write``. 相反, 我们应该决定到底用哪一种方式. 把日志作为特例是因为日志是一个非常独特的应用, 还有一些是历史原因.
    
    流的支持者们主张流是不二之选, 但观点并不是那么清晰有力. 他们指出的流的每个优势也都是其劣势. 流最大的优势是在输出时不需要关心打印对象的类型. 这是一个亮点. 同时, 也是一个不足: 你很容易用错类型, 而编译器不会报警. 使用流时容易造成的这类错误:
        .. code-block:: c++
            
            cout << this;   // Prints the address
            cout << *this;  // Prints the contents
    
    由于 ``<<`` 被重载, 编译器不会报错. 就因为这一点我们反对使用操作符重载.
    
    有人说 ``printf`` 的格式化丑陋不堪, 易读性差, 但流也好不到哪儿去. 看看下面两段代码吧, 实现相同的功能, 哪个更清晰?
        .. code-block:: c++
            
            cerr << "Error connecting to '" << foo->bar()->hostname.first
                 << ":" << foo->bar()->hostname.second << ": " << strerror(errno);
            
            fprintf(stderr, "Error connecting to '%s:%u: %s",
                    foo->bar()->hostname.first, foo->bar()->hostname.second,
                    strerror(errno));
    
    你可能会说, "把流封装一下就会比较好了", 这儿可以, 其他地方呢? 而且不要忘了, 我们的目标是使语言更紧凑, 而不是添加一些别人需要学习的新装备.
    
    每一种方式都是各有利弊, "没有最好, 只有更适合". 简单性原则告诫我们必须从中选择其一, 最后大多数决定采用 ``printf + read/write``.


5.10. 前置自增和自减
~~~~~~~~~~~~~~~~~~~~

.. tip::
    对于迭代器和其他模板对象使用前缀形式 (``++i``) 的自增, 自减运算符.
    
定义:
    对于变量在自增 (``++i`` 或 ``i++``) 或自减 (``--i`` 或 ``i--``) 后表达式的值又没有没用到的情况下, 需要确定到底是使用前置还是后置的自增 (自减).
    
优点:
    不考虑返回值的话, 前置自增 (``++i``) 通常要比后置自增 (``i++``) 效率更高. 因为后置自增 (或自减) 需要对表达式的值 ``i`` 进行一次拷贝. 如果 ``i`` 是迭代器或其他非数值类型, 拷贝的代价是比较大的. 既然两种自增方式实现的功能一样, 为什么不总是使用前置自增呢?
    
缺点:
    在 C 开发中, 当表达式的值未被使用时, 传统的做法是使用后置自增, 特别是在 ``for`` 循环中. 有些人觉得后置自增更加易懂, 因为这很像自然语言, 主语 (``i``) 在谓语动词 (``++``) 前.
    
结论:
    对简单数值 (非对象), 两种都无所谓. 对迭代器和模板类型, 使用前置自增 (自减).
    
5.11. ``const`` 的使用
~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::
    我们强烈建议你在任何可能的情况下都要使用 ``const``.
    
定义:
    在声明的变量或参数前加上关键字 ``const`` 用于指明变量值不可被篡改 (如 ``const int foo`` ). 为类中的函数加上 ``const`` 限定符表明该函数不会修改类成员变量的状态 (如 ``class Foo { int Bar(char c) const; };``).
    
优点:
    大家更容易理解如何使用变量. 编译器可以更好地进行类型检测, 相应地, 也能生成更好的代码. 人们对编写正确的代码更加自信, 因为他们知道所调用的函数被限定了能或不能修改变量值. 即使是在无锁的多线程编程中, 人们也知道什么样的函数是安全的.
    
缺点:
    ``const`` 是入侵性的: 如果你向一个函数传入 ``const`` 变量, 函数原型声明中也必须对应 ``const`` 参数 (否则变量需要 ``const_cast`` 类型转换), 在调用库函数时显得尤其麻烦.
    
结论:
    ``const`` 变量, 数据成员, 函数和参数为编译时类型检测增加了一层保障; 便于尽早发现错误. 因此, 我们强烈建议在任何可能的情况下使用 ``const``:
        
        - 如果函数不会修改传入的引用或指针类型参数, 该参数应声明为 ``const``.
        - 尽可能将函数声明为 ``const``. 访问函数应该总是 ``const``. 其他不会修改任何数据成员, 未调用非 ``const`` 函数, 不会返回数据成员非 ``const`` 指针或引用的函数也应该声明成 ``const``.
        - 如果数据成员在对象构造之后不再发生变化, 可将其定义为 ``const``.
    
    然而, 也不要发了疯似的使用 ``const``. 像 ``const int * const * const x;`` 就有些过了, 虽然它非常精确的描述了常量 ``x``. 关注真正有帮助意义的信息: 前面的例子写成 ``const int** x`` 就够了.
    
    关键字 ``mutable`` 可以使用, 但是在多线程中是不安全的, 使用时首先要考虑线程安全.

``const`` 的位置:
    有人喜欢 ``int const *foo`` 形式, 不喜欢 ``const int* foo``, 他们认为前者更一致因此可读性也更好: 遵循了 ``const`` 总位于其描述的对象之后的原则. 但是一致性原则不适用于此, "不要过度使用" 的声明可以取消大部分你原本想保持的一致性. 将 ``const`` 放在前面才更易读, 因为在自然语言中形容词 (``const``) 是在名词 (``int``) 之前.
    
    这是说, 我们提倡但不强制 ``const`` 在前. 但要保持代码的一致性! (yospaly 注: 也就是不要在一些地方把 ``const`` 写在类型前面, 在其他地方又写在后面, 确定一种写法, 然后保持一致.)
    
5.12. 整型
~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::
    C++ 内建整型中, 仅使用 ``int``. 如果程序中需要不同大小的变量, 可以使用 ``<stdint.h>`` 中长度精确的整型, 如 ``int16_t``.
    
定义:
    C++ 没有指定整型的大小. 通常人们假定 ``short`` 是 16 位, ``int``是 32 位, ``long`` 是 32 位, ``long long`` 是 64 位.
    
优点:
    保持声明统一.
    
缺点:
    C++ 中整型大小因编译器和体系结构的不同而不同.
    
结论:
    ``<stdint.h>`` 定义了 ``int16_t``, ``uint32_t``, ``int64_t`` 等整型, 在需要确保整型大小时可以使用它们代替 ``short``, ``unsigned long long`` 等. 在 C 整型中, 只使用 ``int``. 在合适的情况下, 推荐使用标准类型如 ``size_t`` 和 ``ptrdiff_t``.
    
    如果已知整数不会太大, 我们常常会使用 ``int``, 如循环计数. 在类似的情况下使用原生类型 ``int``. 你可以认为 ``int`` 至少为 32 位, 但不要认为它会多于 ``32`` 位. 如果需要 64 位整型, 用 ``int64_t`` 或 ``uint64_t``.
    
    对于大整数, 使用 ``int64_t``.
    
    不要使用 ``uint32_t`` 等无符号整型, 除非你是在表示一个位组而不是一个数值, 或是你需要定义二进制补码溢出. 尤其是不要为了指出数值永不会为负, 而使用无符号类型. 相反, 你应该使用断言来保护数据.
    
关于无符号整数:
    有些人, 包括一些教科书作者, 推荐使用无符号类型表示非负数. 这种做法试图达到自我文档化. 但是, 在 C 语言中, 这一优点被由其导致的 bug 所淹没. 看看下面的例子:
        .. code-block:: c++
            
            for (unsigned int i = foo.Length()-1; i >= 0; --i) ...
    
    上述循环永远不会退出! 有时 gcc 会发现该 bug 并报警, 但大部分情况下都不会. 类似的 bug 还会出现在比较有符合变量和无符号变量时. 主要是 C 的类型提升机制会致使无符号类型的行为出乎你的意料.
    
    因此, 使用断言来指出变量为非负数, 而不是使用无符号型!
    
5.13. 64 位下的可移植性
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::
    代码应该对 64 位和 32 位系统友好. 处理打印, 比较, 结构体对齐时应切记:
    
- 对于某些类型, ``printf()`` 的指示符在 32 位和 64 位系统上可移植性不是很好. C99 标准定义了一些可移植的格式化指示符. 不幸的是, MSVC 7.1 并非全部支持, 而且标准中也有所遗漏, 所以有时我们不得不自己定义一个丑陋的版本 (头文件 ``inttypes.h`` 仿标准风格):
    
    .. code-block:: c++
    
        // printf macros for size_t, in the style of inttypes.h
        #ifdef _LP64
        #define __PRIS_PREFIX "z"
        #else
        #define __PRIS_PREFIX
        #endif
        
        // Use these macros after a % in a printf format string
        // to get correct 32/64 bit behavior, like this:
        // size_t size = records.size();
        // printf("%"PRIuS"\n", size);
        #define PRIdS __PRIS_PREFIX "d"
        #define PRIxS __PRIS_PREFIX "x"
        #define PRIuS __PRIS_PREFIX "u"
        #define PRIXS __PRIS_PREFIX "X"
        #define PRIoS __PRIS_PREFIX "o"
    
    
    +-------------------+---------------------+--------------------------+------------------+
    | 类型              | 不要使用            | 使用                     | 备注             |
    +===================+=====================+==========================+==================+
    | ``void *``        |                     |                          |                  |
    | (或其他指针类型)  | ``%lx``             | ``%p``                   |                  |
    +-------------------+---------------------+--------------------------+------------------+
    | ``int64_t``       | ``%qd, %lld``       | ``%"PRId64"``            |                  |
    +-------------------+---------------------+--------------------------+------------------+
    | ``uint64_t``      | ``%qu, %llu, %llx`` | ``%"PRIu64", %"PRIx64"`` |                  |
    +-------------------+---------------------+--------------------------+------------------+
    | ``size_t``        | ``%u``              | ``%"PRIuS", %"PRIxS"``   | C99 规定 ``%zu`` |
    +-------------------+---------------------+--------------------------+------------------+
    | ``ptrdiff_t``     | ``%d``              | ``%"PRIdS"``             | C99 规定 ``%zd`` |
    +-------------------+---------------------+--------------------------+------------------+

    注意 ``PRI*`` 宏会被编译器扩展为独立字符串. 因此如果使用非常量的格式化字符串, 需要将宏的值而不是宏名插入格式中. 使用 ``PRI*`` 宏同样可以在 ``%`` 后包含长度指示符. 例如, ``printf("x = %30"PRIuS"\n", x)`` 在 32 位 Linux 上将被展开为 ``printf("x = %30" "u" "\n", x)``, 编译器当成 ``printf("x = %30u\n", x)`` 处理 (yospaly 注: 这在 MSVC 6.0 上行不通, VC 6 编译器不会自动把引号间隔的多个字符串连接一个长字符串).
    
- 记住 ``sizeof(void *) != sizeof(int)``. 如果需要一个指针大小的整数要用 ``intptr_t``.

- 你要非常小心的对待结构体对齐, 尤其是要持久化到磁盘上的结构体 (yospaly 注: 持久化 - 将数据按字节流顺序保存在磁盘文件或数据库中). 在 64 位系统中, 任何含有 ``int64_t``/``uint64_t`` 成员的类/结构体, 缺省都以 8 字节在结尾对齐. 如果 32 位和 64 位代码要共用持久化的结构体, 需要确保两种体系结构下的结构体对齐一致. 大多数编译器都允许调整结构体对齐. gcc 中可使用 ``__attribute__((packed))``. MSVC 则提供了 ``#pragma pack()`` 和 ``__declspec(align())`` (YuleFox 注, 解决方案的项目属性里也可以直接设置).
    
- 创建 64 位常量时使用 LL 或 ULL 作为后缀, 如:
    .. code-block:: c++
        
        int64_t my_value = 0×123456789LL;
        uint64_t my_mask = 3ULL << 48;
        
    
- 如果你确实需要 32 位和 64 位系统具有不同代码, 可以使用 ``#ifdef _LP64`` 指令来切分 32/64 位代码. (尽量不要这么做, 如果非用不可, 尽量使修改局部化)

.. _preprocessor-macros:

5.14. 预处理宏
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::
    使用宏时要非常谨慎, 尽量以内联函数, 枚举和常量代替之.
    
宏意味着你和编译器看到的代码是不同的. 这可能会导致异常行为, 尤其因为宏具有全局作用域.

值得庆幸的是, C++ 中, 宏不像在 C 中那么必不可少. 以往用宏展开性能关键的代码, 现在可以用内联函数替代. 用宏表示常量可被 ``const`` 变量代替. 用宏 "缩写" 长变量名可被引用代替. 用宏进行条件编译... 这个, 千万别这么做, 会令测试更加痛苦 (``#define`` 防止头文件重包含当然是个特例).

宏可以做一些其他技术无法实现的事情, 在一些代码库 (尤其是底层库中) 可以看到宏的某些特性 (如用 ``#`` 字符串化, 用 ``##`` 连接等等). 但在使用前, 仔细考虑一下能不能不使用宏达到同样的目的.

下面给出的用法模式可以避免使用宏带来的问题; 如果你要宏, 尽可能遵守:
    
    - 不要在 ``.h`` 文件中定义宏.
    - 在马上要使用时才进行 ``#define``, 使用后要立即 ``#undef``.
    - 不要只是对已经存在的宏使用#undef，选择一个不会冲突的名称；
    - 不要试图使用展开后会导致 C++ 构造不稳定的宏, 不然也至少要附上文档说明其行为.
    
5.15. 0 和 NULL
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::
    整数用 ``0``, 实数用 ``0.0``, 指针用 ``NULL``, 字符 (串) 用 ``'\0'``.
    
整数用 ``0``, 实数用 ``0.0``, 这一点是毫无争议的.

对于指针 (地址值), 到底是用 ``0`` 还是 ``NULL``, Bjarne Stroustrup 建议使用最原始的 ``0``. 我们建议使用看上去像是指针的 ``NULL``, 事实上一些 C++ 编译器 (如 gcc 4.1.0) 对 ``NULL`` 进行了特殊的定义, 可以给出有用的警告信息, 尤其是 ``sizeof(NULL)`` 和 ``sizeof(0)`` 不相等的情况.

字符 (串) 用 ``'\0'``, 不仅类型正确而且可读性好.

5.16. sizeof
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::
    尽可能用 ``sizeof(varname)`` 代替 ``sizeof(type)``.
    
使用 ``sizeof(varname)`` 是因为当代码中变量类型改变时会自动更新. 某些情况下 ``sizeof(type)`` 或许有意义, 但还是要尽量避免, 因为它会导致变量类型改变后不能同步.
    .. code-block:: c++
        
        Struct data;
        Struct data; memset(&data, 0, sizeof(data));
    
    .. warning::
        .. code-block:: c++
        
            memset(&data, 0, sizeof(Struct));

5.17. Boost 库
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. tip::
    只使用 Boost 中被认可的库.

定义:
    `Boost 库集 <http://www.boost.org/>`_ 是一个广受欢迎, 经过同行鉴定, 免费开源的 C++ 库集.
    
优点:
    Boost代码质量普遍较高, 可移植性好, 填补了 C++ 标准库很多空白, 如型别的特性, 更完善的绑定器, 更好的智能指针, 同时还提供了 ``TR1`` (标准库扩展) 的实现.
    
缺点:
    某些 Boost 库提倡的编程实践可读性差, 比如元编程和其他高级模板技术, 以及过度 "函数化" 的编程风格.
    
结论:
    为了向阅读和维护代码的人员提供更好的可读性, 我们只允许使用 Boost 一部分经认可的特性子集. 目前允许使用以下库:
        
        - `Compressed Pair <http://www.boost.org/libs/utility/compressed_pair.htm>`_ : ``boost/compressed_pair.hpp``
        
        - `Pointer Container <http://www.boost.org/libs/ptr_container/>`_ : ``boost/ptr_container`` (序列化除外)
        
        - `Array <http://www.boost.org/libs/array/>`_ : ``boost/array.hpp``
        
        - `The Boost Graph Library (BGL) <http://www.boost.org/libs/graph/>`_ : ``boost/graph`` (序列化除外)
        
        - `Property Map <http://www.boost.org/libs/property_map/>`_ : ``boost/property_map.hpp``
        
        - `Iterator <http://www.boost.org/libs/iterator/>`_ 中处理迭代器定义的部分 : ``boost/iterator/iterator_adaptor.hpp``, ``boost/iterator/iterator_facade.hpp``, 以及 ``boost/function_output_iterator.hpp``

我们正在积极考虑增加其它 Boost 特性, 所以列表中的规则将不断变化.
