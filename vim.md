

```shell
:1,$ s/^/HELLO/g   	#替换行首
:1,$ s/$/WORLD/g	#替换行尾
% 					#匹配对应的括号
gu 					#转换小写到 需配合w,b等移动命令，类似t
gU 					#转换大写到 需配合w,b等移动命令，类似t
gd 					#高亮显示光标所属单词，"n" 查找！
:1,10d 				#命令解释：删除第一行到第10行
vi{					#选中大括号内,小括号等同理
＃				   #普通模式下输入＃寻找游标所在处的单词,＊是反向
D					#删除直到行尾
dt					#删除知道某个单词 仅限当前行,T反向
ct					#删除到某个单词并进入插入模式,T反向
caw					#更改当前光标所在字符的单词并进入插入模式
s					#删除当前字符并进入插入模式
w					#光标右移一个字至字首,跳过空格
b					#光标左移一个字至字首,跳过空格
e					#光标右移一个字至字尾,跳过空格
ge					#光标左移一个字至字尾,跳过空格
(					#光标移至句首,)反向
{ 					#转到上一个空行 }反向
H					#光标移至屏幕顶行
M					#光标移至屏幕中间行
L					#光标移至屏幕最后行
:s/vivian/sky/g 	#替换当前行所有 vivian 为 sky 
:%s/vivian/sky/g	#替换每一行中所有 vivian 为 sky 
:g/^s*$/d  			#删除所有空格
Ctrl+u	   			#向文件首翻半屏；
Ctrl+d	   			#向文件尾翻半屏；
Ctrl+f	   			#向文件尾翻一屏；
Ctrl+b	   			#向文件首翻一屏；
ZZ			   	    #命令模式下保存当前文件所做的修改后退出vi；
:f			        #在命令模式下，用于显示当前的文件名、光标所在行的行号以及显示比例；
```


