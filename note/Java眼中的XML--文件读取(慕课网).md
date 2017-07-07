###XML  

**目录：**  

* 初次邂逅xml  
* 在Java程序中如何获取xml文件的内容  
* 在Java程序中如何生成xml文件  

####初次邂逅xml  

表现：以“.xml”为文件扩展名的文件  
存储：树形结构（可以理解为一课倒着生成的树)  

eg:

	<?xml version="1.0" encoding="UTF-8">
	<bookstore>
		<book id="1">	//id为book的属性,同时id可以作为book的子节点
			<name>冰与火之歌</name>
			<author>乔治马丁</author>
			<year>2014</year>
			<price>89</price>
		</book>
		<book id="2">	//第二本书可以像上面一样定义,添加自己的属性
		<><>
		</book>
	</bookstore>

为什么要使用XML？  
1. 不同应用程序之间的通信(eg：订票软件和支付软件之间)
2. 不同平台的通信(eg：maos和windows之间)
3. 不同平台间数据的共享？(网站和APP之间)

第二课  
#####在Java程序中如何获取xml文件的内容  
在Java程序中读取xml文件的过程也称为----解析xml文件  
**解析的目的**：获取节点名、节点值、属性名、属性值  
四种解析方式：   
DOM  	//Java官方提供的接些方式  
SAX  	//Java官方提供的解析方式
DOM4J  
JDOM  

