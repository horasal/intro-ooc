# Introduction to OOC Programming language

`文/akisann @ cnblogs.com / zhaihj @ github.com`

本文同时发布在github上：[https://github.com/zhaihj/intro-ooc](https://github.com/zhaihj/intro-ooc)

`本文遵循CC-BY-NC。`

我想试一门新语言……但：

* 我希望这门语言能简洁易懂 —— 排除了Perl/Rust...
* 我不想自己管理内存 —— 排除了C/C++/Object Pascal...
* 最好它能跟C差不多快 —— 排除了Python/Ruby...
* 并且最好能在任何地方编译&运行 —— 排除了D
* 我不想带着一个数百兆的运行库 —— 排除了Java
* __我不怕Bug__ —— 欢迎来到OOC的世界

## Why OOC?

* Compile to C，所有的代码都会首先编译成C，然后由clang或者gcc编译成二进制代码
这意味着，只要你有一台能运行ooc编译器的电脑，那么你的代码就可以在几乎任何有C编译器的平台上运行。

* Class，Function overloading，Extend，Operator Overloading.... OOC拥有绝大部分你所期待的高级语言的功能。
ooc借鉴了很多语言，尝试这把这些语言里优秀（并且有趣）的元素融合到一起。

* Easy interface to c. OOC可以直接使用C的头文件，也可以在C里简单的使用ooc的函数。

OOC的[官方网站](http://ooc-lang.org/)里有更多介绍和参考资料。

## First Impression

一个求Fibonacci数的程序看起来是这样：

	fibonacci: func(n: Int) -> Int{
		n < 2 ? n : fibonacci(n-1) + fibonacci(n-2)
	}

	for(i in 0..30) "f(#{i})=#{fibonacci(i)}" println()

这段程序输出了前30个Fibonacci数。看起来是不是很容易懂？
函数通过`函数名: func(参数)->返回类型`来定义， 而它的内容则与C一模一样。同时，它不需要分号，不需要return，因为最后一个表达式的值会作为函数的返回值（当然也可以显式的使用return）。

## Getting Started

如果在看了上面的代码之后你有兴趣继续，那么是时候准备一下编译环境了。[OOC的一个实现可以在github上简单的获得](https://github.com/fasterthanlime/rock)。好的，让我们一步一步来：

* 首先，clone编译器(rock)的源代码：

	git clone https://github.com/fasterthanlime/rock

* 随后，至于要运行make即可

	cd rock && make rescue

rock是一个bootstrap的编译器，也就是说它本身是由OOC写成的。首先makefile会下载一套预编译的C源代码，用C编译得到的编译器来进一步编译现行代码。不出意外，make之后就会得到一个可以运行的ooc编译器了，它默认是`./bin/rock`，你可以用soft link把它放到任何地方。

使用rock编译ooc的程序非常简单，只需要简单的执行

	rock yourfile.ooc

就会得到可执行文件`yourfile`。

## Hello World

为了确认编译器是不是正常工作，首先来编译一个简单的`Hello World`

	"Hello world!" println()

把上面的代码保存在`hello.ooc`里，然后执行

	rock hello.ooc

你就会得到`hello`，执行它，看看结果吧。

当然，你也可以加一些佐料：

	rock --cc=clang --O3 --pr hello.ooc

`--cc`用来指定使用的编译器，`--O3`与gcc下的意思是一样的，代表了最高优化，而`--pr`则表示是发行版。同样，你可以指定`--pg`来获得调试版。

## Basic Elements

OOC来自与C，编译成C。因此绝大部分内容跟C十分类似，在这里仅仅介绍一些不同的元素。

###变量定义

	foo : Int
	bar : String
	cstyle : Float*
	p : Pointer

ooc的变量定义类似Pascal，用`变量名：类型`的格式，同时需要注意的是所有的类型首字母都是大写，并且c里面的`void*`在ooc里是`Pointer`类型。当然，在变量定义时也可以赋值：

	error: String = "System Error!"

不仅如此，在变量定义有初始值时，ooc还允许省略类型，这个特性跟Go或者IO语言里的特性是类似的：

	error := "System Error!"

OOC里，变量的类型跟C一一对应，并且有非常简单的特征，比如：


| ooc | c |
|-----|---|
| Char | char |
| UChar| unsigned char|
| Int | int |
| UInt| unsigned int|
|Float | float|
| ... | ... |

###循环与判断

if/while语句与c完全一样：

	if(c) { return true }
	else { renurn false }
	while(c){ c -= 1 }

稍微有些不同的是ooc的for：

	for(i in 0..100) { a += i }

ooc的for语句不支持c格式，for仅仅能够遍历一个范围，但这在普通情况下已经够用了。同时，对于c里的switch语句，ooc里使用的是match：

	match (token) {
		case "if" => 1
		case "case" => 2
		case "while" => 3
		case => -1
	}

这里每一个case执行完之后就会自动中断，并不会继续执行下一个case。同时，ooc的match可以用来配对几乎任何东西，即使对于对象（后述），只要它有`matches?`成员，就能够用来match-case里。

###函数

就像在最初所展示的一样，ooc的函数定义类似Pascal（或者Ada）：

	myfunc: func(argument: Int, argument2: String) -> Int{
		// do what you want
	}

函数的返回值不仅仅可以是单个的值，也可以是tuple：

	swap: func(a, b: Int) -> (Int, Int){ (b, a) }
	(a, b) := swap(10, 5) // result is a=5, b=10

当然，跟其他语言一样，ooc也支持First-class Function：

	foo: func(add: Func(Int,Int)->Int) -> Int{ add(10, 20) }
	bar: func(a,b: Int) -> Int{ a + b }
	foo(bar)
	foo(|x, y| x + y)

这里foo函数的参数add是一个函数（或者也可以说是“函数指针”）,需要注意这里的Func是大写——因为在ooc里，所有的类型的首字母都是大写。bar可以直接作为参数使用。同时，`|x, y| x + y`是lambda表达式，它定义了一个以参数x，y为输入的函数。当然，函数也可以作为变量(闭包）：

	fileFilter: func(name: String) -> Bool{
		getName := func(s: String) -> String{
			// get name ...
		}

		// use getName as function
	}

对于指针，ooc里几乎与c是一样的：

	mul2: func(v : Int*){ mul2@ *= 2 }
	mul2ref: func(v : Int@){ mul2 *= 2 }

	myvar := 3
	mul2(myvar&)
	mul2ref(myvar&)

这里定义了一个叫mul2的函数，它的它的定义与使用跟c并没有太大差别，唯一需要注意的就是在ooc里，访问指针不再是`*var`而是`var@`，类似的取地址也不是`&var`而是`var&`。在另外一个函数mul2ref里，我们用了Int@，它代表了变量v是一个参照——也就是说虽然它通过指针传递，但在使用时会被自动取值。

最后，函数是可以overloading的，比如下面的例子：

	foo: func (i: Int) -> Int{ i * 3 }
	foo: func ~withj (i: Int, j: Int) -> Int{ i * j }

ooc里，函数的特征（signature）并不仅仅是函数名和参数列表，你还可以给任何一个函数添加”后缀“，也就是这里`~withj`的地方，通过后缀，可以实现函数的overloading。后缀跟函数名不同的地方在于，在执行时，即使不加后缀，编译器会自动寻找最合适的函数去执行，而通过制定后缀，可以硬性的制定一个函数， 比如：

	foo(1) // 执行第一个
	foo(1,2) //执行第二个
	foo ~withj (1,2) //执行第二个


### 类与覆盖

虽然最终编译成C，与其他高级语言一样，ooc里有类的概念，类的定义十分简单：

	myclass: class{ init: func }

其中init是类的构造器，但它并不需要是static（静态）的，因为编译器会自动生成真正的构造器`new`，也就是说，在使用myclass，应该像下面这样：

	v := myclass new() // new is auto-generated static function

注意，每一个类必须有至少一个init，因为如果没有init，编译器就不会为它生成new，因此也就无法初始化。当然，你也可以自己定义`new`:

	myclass: class{
		new: static func -> This{
			// do initialization
		}
	}

我们之前已经说了很多次，ooc里所有的类型都是首字母大写，因此代表着当前对象的this首字母大写之后代表这当前类。不过需要注意，自己定义new并不是见好事情，因为class最终还是由c里的struct实现的，因此你需要自己分配内存，管理初始化……等等。静态的new只在包装c函数时有用处，比如包装SDL-ttf时：

	TTFFont: cover from TTF_Font*{
		new: static func(filename: String, ptSize: Int) -> This{
			TTF open(filename toCString(), ptSize)
		}

		new: static func ~rw (data: Pointer, freedata: Int, ptSize: Int) -> This{
			TTF openRW(data, freedata, ptSize)
		}

		....
	}

这样就可以非常自然的将c函数转换成了类。当然，在这里代码使用了cover（覆盖），它在地位上等同与c的struct，但可以拥有函数，也可以被扩展，你可以认为cover是一个仅仅在使用c代码时才会用到的特殊类（class）。对于普通的类，只能继承（extends）其他的类，但对于覆盖，它既可以来自其他覆盖，也可以来自c的struct。比如：

	Array: cover from _lang_array__Array {
		length: extern SizeT
		data: extern Pointer

		free: extern(_lang_array__Array_free) func
	}

这段代码里`_lang_array__Array`是定义在c的头文件里的struct，而length和data都是它的成员，运用cover，可以很简单的将c中struct转换成ooc里可用的类型。

一个类的函数成员可以是static，可以是final。当它是final时，你是不能继承它的，比如：

	foo: class{
		init: func
		a: final func
	}

	bar: class extends foo{
		init: func
		a: func
	}

这段代码会出现编译错误：

	$ rock ff.ooc 

	test.ooc:8:5 error Can not inherit from final function 'a'
		a: func
		~      

	test.ooc:3:5 info ...first definition was here:
		a: final func
		~            

	[FAIL]

最后，类与覆盖都是可以被扩展（extend）的，它类似与ruby的extend，允许你向已经存在的类或覆盖里追加新的成员函数：

	extend Int{
		isOdd?: func -> Bool{
			this % 2 == 1
		}
	}

这里我们给Int类型添加了一个`isOdd?`函数，这里问号没有特殊意义，仅仅是函数名的一部分，ooc允许函数名的最后一个字符是问号，用来表示这是一个“查询”函数，在编译成c是，问号会被翻译成字符串“query”。

好的现在我们可以是一下新定义的函数了，成员函数的访问与其他语言不一样，不是通过点(.)来实现， 而是简单的空格：

	1 isOdd?() toString() println()
	2 isOdd?() toString() println()

可以看到它们已经能够正常执行了。这里，isOdd?是Int的成员函数，而toString是Bool的成员函数，println()又是String（toString的返回类型）的成员函数，这点跟ruby非常接近。
同时，你可能已经注意到了，所有的函数调用都必须加上括号，否则编译器会抛出错误，那么有没有办法让不加括号呢？ 答案是有的，至于要定义属性即可：

	extend Int{
		isOdd : Bool {
			get { this % 2 == 1 }
		}
	}

然后就可以像普通成员变量一样使用它了：

	1 isOdd toString() println()
	2 isOdd toString() println()

在这里，我们定义了一个只读的属性，对于这种属性，ooc提供了一个更简单的定义方式：（pdfe）

	extend Int{
		isOdd ::= (this % 2 == 1)
	}

这段代码的效果跟上面是完全一样的。

最后，类还支持运算符重载， 比如：

	vector: class{
		operator + (v: This) -> This{
			// add this and v
		}
	}

### 泛型

ooc里存在泛型，但它完全不同与其他语言里的泛型，在ooc里，它可以“接受任何类型的变量”，而在其他语言里，泛型意味这“自动生成对应类型的实现”。这种差别决定了ooc的泛型远没有其他语言里强力。
你可以认为，ooc里的泛型是为集合（Collection）引入的，比如ArrayList和HashMap，对于普通函数，大量使用泛型不会有任何优势。

这里仅仅举一个简单的例子：

	foo: class <T>{
		data: T*
		length: Int

		init: func(=length){
			data = gc_malloc(T size * length)
		}
	}

	myfoo := foo<Int> new(10)

在这里，`=length`代表了参数直接赋值给成员，随后我们会根据参数大小分配一块内存给数据。这里的T可以是任何类型。

在ooc里，泛型是一个非常容易出错的部分，具体的设计思想可以参见[我翻译的一篇文章](http://www.cnblogs.com/akisan/p/4235763.html)。在这里不多描述。

### 库与头文件

ooc里使用库是非常简单的，所需要的就是简单的import而已：

	import structs/ArrayList // 使用ArrayList

编译器会在`../sdk`或者`$OOC_LIBS`里寻找所有的库，然后自动使用和编译它们。对于C语言的头文件，可以通过include来使用：

	include stdbool

这样就可以使用stdbool里面定义的内容了。

## 一个小例子

这个小例子是Computer Langugae Benchamark Game里BinaryTree的一个实现，可以拿它跟C语言的版本来做比较。

	Node: class{
		item: Int
		left,right: Node

		init: func(depth: Int, =item){
			if(depth<=0) return
			left = Node new(depth-1, 2*item-1);
			right = Node new(depth-1, 2*item);
		}

		itemCheck: func() -> Int{
			if(!left) return item
			return item+left itemCheck()-right itemCheck()
		}
	}

	mindep := 4
	main: func(argv: Int, argc: CString*) -> Int{
		depth: Int
		if(argv>1) depth = argc[1] toString() toInt()
		else return 1
		stretch := depth+1
		check := Node new(stretch,0) itemCheck()
		"stretch tree of depth %d\t check: %d" printfln(stretch, check)
		longlived := Node new(depth,0)
		i := mindep
		while(i<=depth){
			iterations := 1<<(depth-i+mindep)
			check: Int = 0
			for(j in 1..iterations+1){
				check += Node new(i,j) itemCheck()
				check += Node new(i,-j) itemCheck()
			}
			"%d\ttrees of depth %d\t check: %d" printfln(iterations*2, i, check);
			i+=2
		}
		"long lived tree fo depth %d\t check %d" printfln(depth, longlived.itemCheck());
		return 0
	}


至于这个程序的执行结果：(参见我的[Github Repo](https://github.com/zhaihj/rock-benchmark))

| ooc | c |
|-----|---|
|16.95|16.45|

可以看到，二者几乎没有差别。

## 结语

OOC是一个很不错的第二，或者第三语言。虽然有公司在用ooc做些事情，但我并不认为那很明智。的确，ooc兼具执行效率和开发效率，但目前它的编译器还远远没有完美。如果你有兴趣，那么不妨[fork一下](https://github.com/fasterthanlime/rock)，让ooc的编译器更加完善。
