##do{}while(0)的几个用法

###**1**. 代替goto语句
>假设有这样一种场景：有好几个if语句用来表示出错处理，而且它们要去处理的内容都是一样的，比如说释放上面申请的内存等等。  
>我为了防止代码的冗余，所以我只写一份出错处理用一个_goto_语句将程序指向我实现出错处理的地方。这样做虽然没有什么错误，但是它确违背了软件模块化的思想，而且_goto_语句不便于代码的维护。
>####我可以用do{}while(0)这么做：
>    
	void fun()
	{
	...
	do
	{
		//假如这里判断出错了
		if()
		{
			break;
		}
	}while(0)
	//加个判断，判断一下是否要执行执行出错处理.
	...
	}

###**2**. 在宏中的应用
>在查看内核链表的时候看到了这个_do{}while(0)_的用法，第一眼看到的时候感觉也没什么，但是再看时就发现这不都是执行一次吗？干嘛还要加个_do{}while(0)_呢？而且**while(0)后面没有';'**(我查了一下Linux内核里面宏中涉及到_do{}while(0)_的都没有带分号)。
>
	#define INIT_LIST_HEAD(ptr) do{\
			(ptr)->next = (ptr); (ptr)->prev = (ptr);\
	}while(0)
这样做的原因是**辅助定义复杂的宏，避免引用的时候出错**。以上面的宏为例子设想我有一个_if_语句判断，如何成立则调用上面的宏，如果不成立就进行其他操作。代码类似如下：
>
	if(a)
		INIT_LIST_HEAD();	//注意这里有分号
	else
		...
如果没有_do{}while(0)_这个结构，上面这种调用宏肯定是不对的。除非**每次使用_if_语句时都使用大括号**。  
我们还可以发现，上面宏定义中_while(0)_后面没有分号，假如它自带分号的话，我们在调用它时再添加一个分号，那么_else_这里就会报错了。

###**3**. 避免空宏引起的warning
>内核中由于不同架构的限制，很多时候会用到空宏，在编译的时候，空宏会给出_warning_，为了避免这样的_waring_，就可以使用_do{}while(0)_来定义空宏：
>
	#define EMPTYMICRO do{}while(0)

###**4**. 定义一个单独的函数块来实现复杂的操作
>当你的功能很复杂，变量很多你又不愿意增加一个函数的时候，使用_do{}while(0)_，将你的代码写在里面，里面可以定义变量而不用考虑变量名会同函数之前或者之后的重复。

**注：**关于上面提到的第3、4点暂时不是很明白。看到网上提到就先摘录了下来，等有机会看到实例的时候应该能更透彻的理解吧。