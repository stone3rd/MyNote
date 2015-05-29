##OSAL消息队列2
上一次了解了消息队列的**入队和出队**操作。这次先来了解一下消息队列内存**分配和释放**是在哪里完成的。我的习惯做法是source insight中`Ctrl+/`找哪些地方调用了`osal_msg_enqueue`和`osal_msg_dequeue`这两个函数。  
有两个地方调用了`osal_msg_enqueue`，但是在_Znp_app.c_文件中涉及的是另一个队列，直接忽视了。但是奇怪的是却没有哪一处调用了`osal_msg_dequeue`，干想也想不出来就先看看那个有的吧！

	//这个函数在上一篇中已经出场过一次了，只不过上一篇他不是主角走了个过场
	uint8 osal_msg_send(uint8 destination_task, uint8 *msg_ptr)
	{
		//一开始就是一些出错处理，消息是否为空
		if(msg_ptr == NULL)
			return ( INVALID_MSG_POINTER );
		//任务号是否大于规定的最大任务号
		if( destination_task >= tasksCnt )
		{
			osal_msg_deallocate( msg_ptr );
			return ( INVALID_TASK );
		}
		//消息头是否满足要求
		if(OSAL_MSG_NEXT( msg_ptr) != NULL ||
		   OSAL_MSG_ID( msg_ptr) != TASK_NO_TASK )
		{
			osal_msg_deallocate( msg_ptr );
			return ( INVALID_MSG_POINTER );
		}
		//设置消息头中任务号
		OSAL_MSG_ID( msg_ptr ) = destiation_task;
		//入队
		osal_msg_enqueue( &osal_qHead, msg_ptr );
		//设置任务，SYS_EVENT_MSG是一个事件合集，这里不是重点。
		osal_set_event( destination_task, SYS_EVENT_MSG );

		return ( SUCCESS );
	}

>看完了，但是还是没找到内存分配的迹象，那就只能再往上找了。再往上找就又回到上一篇讲的内容了。这里就不重复了，附上一张图说明一下个函数调用的次序。  
![2](https://github.com/pikstone/mynote/blob/master/resource/2.jpg)

既然从出队函数处找不到消息释放的影子，那就去其他地方找。对消息的操作应该有创建、发送、处理这几个基本的步骤，创建和发送在前面的介绍中已经有涉及，关于处理可以从协议栈中提供的一个用户案例中查看。在文件GenericApp.c文件中。  
在该任务的事件处理函数`GenericApp_ProcessEvent(uint8 task_id, uint16 events)`中有这样一行代码：

	MSGpkt = (afIncomingMSGPacket_t *)osal_msg_receive( GenericApp_TaskID );
>在协议栈中有关于这个函数的说明：该函数被某一个任务调用以获取消息，这个任务在处理完消息之后一定要调用函数`osal_msg_deallocate`释放消息的缓存。首先来看一下`osal_msg_receive`函数内部做了哪些事情。

	uint8 *osal_msg_receive(uint8 task_id)
	{
		osal_msg_hdr_t *listHdr;
		osal_msg_hdr_t *prevHdr = NULL;
		osal_msg_hdr_t *foundHdr = NULL;
		halIntState_t intState;

		HAL_ENTER_CRITICAL_SECTION(intState);

		listHdr = osal_qHead;
		//遍历队列查找符合要求的任务信息
		while(listHdr != NULL)
		{
			if((listHdr - 1)->dest_id == task_id )
			{//找到了任务号对应的消息节点
				if( foundHdr == NULL)
				{
					//把找到的消息指针给foundHdr
					foundHdr = listHdr;
				}
				else
				{
					break;
				}
		}
		if( foundHdr == NULL)
		{
				prevHdr = listHdr;
		}
		listHdr = OSAL_MSG_NEXT(listHdr);
	}

		//一个任务号在队列中存在不只一个消息要处理，处理了一个但还是要保证任务标志位为1
		if( listHdr != NULL)
		{//找到了任务号对应的消息，清空任务标志位
			osal_set_event( task_id, SYS_EVENT_MSG );
		}
		else
		{//只有一个消息要处理
			osal_clear_event( task_id, SYS_EVENT_MSG );
		}
		//要找的消息存在，把那个消息节点从队列中脱离
		if( foundHdr != NULL)
		{
			//获取并删除一个消息节点。
			osal_msg_extract( &osal_qHead, foundHdr, prevHdr );
		}

		HAL_EXIT_CRITICAL_SECTION( intState );

		return ( (uint8*) foundHdr );
	}
>在调用`osal_msg_extract`之后要处理的消息节点会从队列上取出，但是并没有释放内存。真正释放内存要等任务处理函数对消息处理结束之后才会调用函数`osal_msg_deallocate`释放消息的缓存。附上一幅图说明一下函数调用过程：  
![pic2](https://github.com/pikstone/mynote/blob/master/resource/2-2.jpg)

>至此10个有关消息队列的函数大多已经介绍完了，还有剩下的几个下一篇再说明吧。
