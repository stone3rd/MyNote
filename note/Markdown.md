Markdown语法  
===

区块元素  
---

###标题  
markdown支持两种标题的语法，类`Setext`和类'atx'形式。  

* 类Setext，利用`=`（最高阶标题）和`-`（第二阶标题），eg： 
 
		This is an H1
		=============
   	
		This is an H2
		-------------  

任何数量的`=`和`-`都可以有效果。  

* 类Atx，在行首插入1到6个`#`，eg：  

		#This is an H1
		##This is an H2
		###This is an H3

###区块引用Blockquotes  
在文字前加`>`  
>This is blockquotes.
>>this is another blockquotes.  

###列表  
markdown支持有序列表和无序列表。  
无序列表使用`*`或`+`或`-`加空格；有序列表使用数字加英文字母的`.`  

* red
- green
+ blue  
	+ red
	* green 
	- blue  
		- red
		+ green 
		* blue  

---
1. red  
2. green   
3. blue  

注：类表项目可以包含多个段落，每个项目下的段落都必须缩进4个空格或者一个制表符。  
包含引用的话也需要添加4个空格或者一个制表符。
代码块也是一样，额外添加4个空格或者一个制表符。  

###代码区块  
只需添加4个空格或者一个制表符就可以了。  

	#include <stdio.h>
	void main()
	{
		return 0;
	}   

###分隔线  
使用三个以上的星号、减号、底线来建立一个分隔线。  

*** 
--- 
___  

区段元素  
---
###链接  

[titl](www.baidu.com)  

###强调  
使用星号（`*`）和底线（`_`）作为标记  

**加粗**  *斜体*  
__加粗__ _斜体_  

###代码  
使用符合（`）   


###图片 
类似于链接，在前面加一个（`!`）  
![picture](...)  

---
其他
---
###自动链接
markdown支持比较简短的形式处理email，使用尖括号包起来。
<email@email.com>

###反斜杠  
markdown可以利用反斜杠来插入一些在语法中有其他意义的符合，一下符号前加反斜杠来表示普通符号：  

	\  反斜杠
	`  反引号
	*  星号
	_  底线
	{} 花括号
	[] 方括号
	() 括号
	#  井号
	+  加号
	-  减号
	.  英文点
	!  感叹号