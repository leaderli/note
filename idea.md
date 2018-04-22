1. jstl标签红色报错 是因为引入了jstl模版取消就可以了s

2. 控制台乱码 

   ```shell
   -Dfile.encoding=utf-8
   ```

3. 善用Postfix Completion这是IDEA 13.1 新增的功能。详细演示和说明看这里 Postfix Code Completion in IntelliJ IDEA 13.1 EAP 。简单来说就是你可以先输入语句的主体（例如要赋值的表达式，或者要循环的集合变量），然后输入用小数点分隔的模板名称后缀，IDE自动帮你智能展开。比如说，你想写 
   ```java
   for (User user:users) { ... } 
   ```
   只需要输入 users.for 再按tab就行了
   你想写 
   ```java
   Date birthday = user.getBirthday()
   ```
   只需要输入 user.getBirthday().var 再按tab就行了。IDEA会自动推断赋值类型和生成默认的变量名称。

4. 通过 `Alt + F1` + `1` 快捷键来定位当前文件所在 Project 组件窗口中的位置

5. soutv=System.out.println("变量名 = " + 变量);

6. soutm=System.out.println("当前类名.当前方法");




 7 . `Command + Shift + E` 来访问最近编辑的文件：

option tab 切换窗口

comand + ⬆️  上级包