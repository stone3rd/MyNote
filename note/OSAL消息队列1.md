##OSAL消息队列(一)
>在osal中有关消息队列的函数共有10个存放在OSAL.c文件下。既然是队列那首先从**入队**和**出队**这两个函数开始。
>
    void osal_msg_enqueue(osal_msg_q_t *q_ptr, void *msg_ptr);//入队  
    void * osal_msg_dequeue(osal_msg_q_t *q_ptr);//出队  

在说明这两个函数内部具体实现之前，先介绍一个全局变量、一个结构体和一个宏。  

    //消息头指针,在系统初始化时设置为NULL.这里osal_msg_q_t为void *类型
    osal_msg_q_t osal_qHead;

    typedef struct  
    {  
        void *next;  
        uint16 len;//消息中数据的长度  
        uint8 dest_id;//任务号,表示该消息属于哪个任务
    }osal_msg_hdr_t;  

    #define OSAL_MSG_NEXT(msg_ptr) ((osal_msg_hdr_t *)(msg_ptr)-1)->next  

>下面是**入队**操作的源代码：
>
    void osal_msg_enqueue(osal_msg_q_t *q_ptr, void *msg_ptr)
    {
        void *list;
        halIntState_t intState;//用来保存EA的内容，等开中断时恢复，unsigned char 类型  
		//关中断，和协议栈有关暂不考虑
        HAL_ENTER_CRITICAL_SECTION(intState);
>
		OSAL_MSG_NEXT(msg_ptr) = NULL;
		if(*q_ptr == NULL)
		{//如果原来队列为空，则这个消息作为头结点
			*q_ptr = msg_ptr;
		}
		else
		{
			//遍历到队列尾部
			for(list = *q_ptr;OSAL_MSG_NEXT(list)!=NULL;list = OSAL_MSG_NEXT(list));、
			//把新的数据挂到队列尾部
			OSAL_MSG_NEXT(list) = msg_ptr;
		}
		//开中断
		HAL_EXIT_CRITICAL_SECTION(intState);
	}
>代码看上去比较简单，但是还不知道那两个参数代表的意思，只能从上面的宏中猜想应该是链表的结构应该是链表头加数据这样的结构。查找后可以发现另一个消息队列有关的函数调用了上面这个函数。
>
    //向目标任务发送消息
	uint8 osal_msg_send(uint8 destination_task, uint8 *msg_ptr)
	{
		//...这里我想知道入队函数的两个参数的具体意义，其他代码暂时忽略了。
		osal_msg_enqueue(&osal_qHead, msg_ptr);
		//...
	}
>这里知道了第一个参数`q_ptr`的意思，**队列的头指针**，但是第二个参数还在上一层，继续找。调用这个函数的地方比较多，毕竟任务和任务之间都要通过它来发送消息，对这个函数的用法基本相同：定义一个自己的结构体来存储数据，然后调用下面的函数osal\_msg\_allocate()来分配空间，最后调用osal\_msg\_send()发送信息。还不清楚的第二个参数就是调用下面这个函数的返回值。看完下面的函数就基本上了解这个消息队列是怎么回事了。下面用一张图描述一下消息的存储结构。
>
	uint8 *osal_msg_allocate(uint16 len)
	{
		osal_msg_hdr_t *hdr;  
>
		if(len == 0)
			return (NULL);
>
		//分配的大小为队列结构体+消息的大小
		hdr = (osal_msg_hdr_t *)osal_mem_alloc((short)(len + sizeof(osal_msg_hdr_t)));
		if(hdr)
		{
			hdr->next = NULL;
			hdr->len = len;
			hdr->dest_id = TASK_NO_TASK;
			//返回的是消息开始的地址
			return ((uint8 *)(hdr + 1));
		}
		else
			return (NULL);
	}

消息队列的头指针`osal_qHead`指向数据域，而消息队列结构体的`next`指针指向下一个消息的数据域。当然数据域的类型可以更具实际要传的内容而改变，所以图片中用msg1、msg2、msg3表示说明数据类型不一定相同。  
我认为这是一种以消息为中心的做法，尽量把链表的结构隐藏起来了。虽然也可以像**Linux内核链表**一样建立一个通用的链表而不关心数据域是什么情况，要使用数据域时通过指针的偏移找到数据域的入口处。但是OSAL这样实现的队列使得对用户来说我的指针永远指向我的数据域，而我不必要知道这些数据是怎么连接起来的。我关心的内容是消息本身，所以我只实现对消息的操作就可以了。  
![tu1](https://github.com/pikstone/mynote/blob/master/resource/1.PNG)

>对于入队操作的分析之后对osal消息队列的整体结构也大致了解了。下面就直接贴上**出队**操作的代码了：
>
	void *osal_msg_dequeue(osal_msg_q_t *q_ptr)
	{
		void *msg_ptr = NULL;
		halIntState_t intState;
>
		HAL_ENTER_CRITICAL_SECTION(intState);
>
		//队列不为空
		if(*q_ptr != NULL)
		{
			//返回队列头数据指针域
			msg_ptr = *q_ptr;
			//队列头指向下一个节点
			*q_ptr = OSAL_MSG_NEXT(msg_ptr);
			OSAL_MSG_NEXT(msg_ptr) = NULL;
			OSAL_MSG_ID(msg_ptr) = TASK_NO_TASK;
		}
>
		HAL_EXIT_CRITICAL_SECTION(intState);
>
		return msg_ptr;
	}
>这里又出现了一个新的宏`OSAL_MSG_ID`。和`OSAL_MSG_NEXT`一样它们把对消息队列头的操作都隐藏在了宏中，做了进一步的分层。像这样的宏共有6个，一看就明白的那种。下面把`OSAL_MSG_ID`的代码也写下来吧：
>
	#define OSAL_MSG_ID(msg_ptr) ((osal_msg_hdr_t *) (msg_ptr) - 1)->dest_id

###小结：
>1. `osal_msg_enqueue`和`osal_msg_dequeue`这两个函数做的是对消息队列的入队和出队操作，它们并没有对队列节点的空间的分配和释放。
>2. 对消息队列头的操作都包含在了一系列宏中，在这里我们关心的是消息包含的数据而非链表的实现。

**ps:**消息队列总共有10个函数，刚开始写这类笔记写到这里有点累了，这篇就先写到这里了，下次继续。  
2015-1-24 13:49:42
