参考学习网站：[http://www.runoob.com/w3cnote/git-guide.html](http://www.runoob.com/w3cnote/git-guide.html)



**1.安装git for windows**  
[git for windows](https://git-for-windows.github.io/)  

**2.配置Git**  

* 在本地创建ssh key;  
`$ ssh-keygen -t rsa -C "821403614@qq.com"`  

* 在默认路径下会生成.ssh文件夹，打开里面的文件id_rsa.pub，复制里面的key。回到github上，可以在账户设置里面添加SSH Key。  

* 验证是否成功，在git bash下输入：  
`$ ssh -T git@github.com`  
第一次会提示是否continue，输入yes就会看到：You've successfully authenticated, but GitHub does not provide shell access.这就表示已成功连上github。  

* 设置username和email，因为github每次commit都会记录他们  
`$ git config --global user.name "your name"`  
`$ git config --global user.email "your_email@youremail.com"`  

**3.检出仓库**  

* 下载github上的文件  
`git clone xxx.xxx(文件地址，例如我笔记的ssh地址： git@github.com:stone3rd/MyNote.git）`

**4.工作流**  
你的本地仓库由git维护的三棵“树”组成。第一个是你的工作目录；第二个是暂存区(Index)，它像个缓存区域，临时保存你的改动；最后是HEAD，它指向你最后一次提交的结果。本地文件更新后可以使用下面指令：  

	git add <filename>
	git add *	             //由working dir上传到Index
	git commit -m "注释信息"  //由Index上传到HEAD
通过上面的指令，可以将改动提交到HEAD，但此时还没有上传到云端仓库。  
