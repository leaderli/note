为当前用户创建cron服务

1.    键入 crontab  -e 编辑crontab服务文件

      例如 文件内容如下：

      ```css
       */2 * * * * /bin/sh  /home/admin/jiaoben/buy/deleteFile.sh 
       :wq #保存文件并并退出
      ```

      /bin/sh /home/admin/jiaoben/buy/deleteFile.sh 这一字段可以设定你要执行的脚本，这里要注意一下bin/sh 是指运行  脚本的命令  后面一段时指脚本存放的路径

2. 查看该用户下的crontab服务是否创建成功， 用 crontab  -l 命令  

3. 启动crontab服务

      一般启动服务用  /sbin/service crond start 若是根用户的cron服务可以用 sudo service crond start， 这里还是要注意  下 不同版本Linux系统启动的服务的命令也不同 ，像我的虚拟机里只需用 sudo service cron restart 即可，若是在根用下直接键入service cron start就能启动服务

4. 查看服务是否已经运行用 ps -ax | grep cron

5. crontab命令

      ```css
      cron服务提供crontab命令来设定cron服务的，以下是这个命令的一些参数与说明:

      crontab -u //设定某个用户的cron服务，一般root用户在执行这个命令的时候需要此参数 
      crontab -l //列出某个用户cron服务的详细内容
      crontab -r //删除某个用户的cron服务
      crontab -e //编辑某个用户的cron服务
      比如说root查看自己的cron设置:crontab -u root -l
      再例如，root想删除fred的cron设置:crontab -u fred -r
      在编辑cron服务时，编辑的内容有一些格式和约定，输入:crontab -u root -e
      进入vi编辑模式，编辑的内容一定要符合下面的格式:*/1 * * * * ls >> /tmp/ls.txt
      任务调度的crond常驻命令
      crond 是linux用来定期执行程序的命令。当安装完成操作系统之后，默认便会启动此任务调度命令。crond命令每分锺会定期检查是否有要执行的工作，如果有要执行的工作便会自动执行该工作。

      ```

6.    cron文件语法

      ```css
      分     小时    日       月      星期     命令
      0-59   0-23   1-31   1-12     0-6     command     (取值范围,0表示周日一般一行对应一个任务)

      记住几个特殊符号的含义:
      “*”代表取值范围内的数字,
      “/”代表”每”,
      “-”代表从某个数字到某个数字,
      “,”分开几个离散的数字
      ```

