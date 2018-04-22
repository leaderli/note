#shell

常用linux命令

```shell
du -d 1 -h  #命令查看当前目录下所有文件夹的大小 -d 指深度，后面加一个数值
history		#查看命令历史
ls -a **/*.log #使用**/来递归搜索 
man jad|col -b > helo.txt			#输出man文档
nl			#将输出的每一行加上行号。例如：'cat 1.txt | nl'，输出1.txt的文件并加上行号
sort        #将文件内容排序输出
tree		#以树的形式显示当前文件
wget -r -p -np -k  #整站下载  wget可以用来下载网页
```

mac shell软件

```shell
ag 									#文本搜索工具
bwm-ng 								#实时网速显示
enca								#字符集转码
fzf 								#模糊搜索工具,进入fzf模式
ksdiff								#diff工具 mergetool 用于合并代码
mdfind								#spotlight搜索
more test.txt|pbcopy				#将文件内容复制到剪切板
ncdu								#查看当前目录文件大小
pbpaste >> tasklist.txt				#将剪切板内容复制到文件
```

zsh

```
启用了历史纪录功能
自动灰色提示 按➡️用默认的 按⬆️⬇️搜索历史
zsh可自定义函数
```

