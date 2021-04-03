---
title: "Linux"
date: 2020-10-06T16:33:07+08:00
draft: false
toc: false
images:
tags: ["",""]
---
-  POSIX 标准
- 可移植的操作系统接口
---
-  ECS 虚拟机
- 镜像选择 ubuntu 
- 版本号 xx.xx 年份.版本 
- 公网IP
---
- Kernel 内核 --> 控制操作系统和调度的代码
- Shell 用于控制kernel 的方式 ,访问内核的交互式命令行 
---
- 通过一个终端连接Linux主机，可以再任何地方使用
- 
---
- 镜像
- - 相当于光盘使用后能够安装处光盘里面的东西

---
- SSH (Secure Shell)
- -  加密访问，保证安全
- `$ ssh  root@公IP`
- 登录后默认在/home 
---
- Linux 用户
- 登录后就拥有一个用户名
- `$ whoami` --> 当前登录的权限 Eg：root……user
- - root 拥有所有权限，日常工作尽量不使用，但是安装系统等需要root/非常高权限的用户完成
- `sudo`切换成root 用户完成命令
- - `sudo username xxx` 生成新的用户
- - - 生成用户后并没有同时生成目录等，需要额外配置
- `$ exist` 退出
- - `su xxx`切换成该用户
- `pwd` 工作目录
- usergroup 用户组 用于权限管理，用户组能够共享权限
---
- `$ list (ls)`列出清单
- - `-l -h` 详情
- - `-a ` 查看包括隐藏的文件
- - - d 目录 时间后面就是文件夹的名字 文件夹的大小是固定的
>- 目录能够区分权限
>- dxxx xxx xxx -->d所有者权限 用户组权限 其他人  
>- r  read
>- w write
>- x execute
- - - l 快捷方式 -> 实际的地址 是一个软链接，执行能够通过软连接实际执行程序
>- 创造快捷方式
>- - `ln -s 目标地址 软连接`
> - - - 软连接必须在PATH 环境变量里面
> - `alias 程序快捷名=实际地址`
- - - `-` 文件 可读可写可执行
---
cat 查看文本文件内容
- `cat xxx.txt |less` `| more` 能够一页一页地查看
---
chmod 调整可执行的权限
- r = 4
- w = 2
- x = 1
- - `chmod 774 xxxfile` 就能够对文件修改权限
- 这样会模糊程序和文件的界限，因为只要有x权限就能够执行
- 查看目录需要 `x` 权限
---
环境变量
- 运行程序在环境变量里面执行，而不是当前目录
- 如果运行程序找不到的话，就需要添加`./`表示在当前目录运行，再执行程序
---
ssh 登录
- 私钥 rw-------
- - ssh-keygen -t rsa 生成私钥 需要输入密码或者留空 每次使用都需要加入密码
- 公钥 rw-r--r--
- 如果通过chmod 改变私钥的权限，就会被警告
---
~#
命令行四要素
1. 环境变量
- `export` 能够查看当前的环境变量
- - `HOME = /root ` --> `cd $HOME` == `cd /root`
- 把目录添加到环境变量
- - export 导出旧的变量，然后在后面添加`:需要的目录`
- - export PATH=$PATH`:/root`  直接在后面添加新的目录
- 定义环境变量
- - `export AAA=hello ` --> `echo AAA` ==> `hello`
- 删除环境变量
- - `unset` --> `unset AAA`
2. 可执行程序
- 内置 --> 命令行不会直接解释这个程序，会直接解析成一条指令，
- 可执行程序是默认在path 环境变量里面找的，如果一个程序找不到，就是不存在path环境变量里面，或者环境变量里面的PATH被更改了导致找不到
3. 参数
4. 工作目录
- 用来计算相对路径
---
`*` 能够匹配一个或多个字符，命令行会把这个字符展开进行匹配
- 如果要真正地使用* 就要添加单引号，需要原封不动地把参数传给程序都需要用单引号完成，这样程序才能够加载* 这个匹配符号 
- - 双引号会执行展开
- 命令行展开就不用程序执行的时候再展开，避免执行程序需要都需要展开
---
安装
- 软件
- - 绿色版 
---
- bin 存放程序
- lib 存放包
---
cp 拷贝
- scp 远程拷贝
- - scp ~/目标文件 user@Ip
---
tar xf 解压 

---
maven 运行需要在java里面完成，所以运行maven的前提是`JAVA_HOME`的环境变量配置好 

---
sshd 接受到链接请求后就会运行程序，返回一个ssh会话
- ssh 就会从sshd 里面获得一个窗口，ssh 里面对环境变量进行的修改就只是对当前窗口有用，关闭后配置就会失去，重新获取窗口也不会有变更
- 要把变更记录下来就要把变更记录在一个文件里面，保存在服务器里面，之后每个窗口都会有这个新的变更
---
永久环境配置
>- `/etc/` 全局
>- `~/` 下的配置只是当前用户
- `vi ~/.bash_profile` 把需要的命令记录到里面
- - 但只是记录到里面，并没有生效，只有新开的ssh才会生效
- - 立刻生效 需要`source ~/.bash_profile` 或者`. ~/bash_profile`，因为[`.` == source]
- 很多的txt都可以的
---
Vi / Vim
- 如果新建编辑一个文件没有保存会呈现`.xxx.txt`,是一个临时文件
---
pstree 
- 显示当前程序中的所有进程
---
- 通过`java -cp . classmainname `是通过交互式命令行启动的，一旦退出或者关闭进程就会退出
- 需要长久运行就需要`nohup` + java 运行 这样就能够后台运行
- tmux （terminal multiplexer） 新的终端。常驻后台程序，所以不能是bash 的子进程，不然已退出就没有了
- Service
- - 先建立一个service文件来存储配置
>- Aftre 在某个服务启动后就启动
>- user 最好不要使用root
>- exeStart 最好使用绝对路径
- docker run -d 
---
杀 进程 kill -9 进程

---
进程的输入输出
- 进程输出的重定向
- stdout
- - java -cp . mainname 1>out.txt 2>err.txt 
>- 0 标准输入 1 标准输出 2 标准错误输出
>- `>` 覆盖 `>>`追加
- 标准输出重定向
---
grep
- xxx关键字 等待从 stdin 里面输出数据， 再从stdout输出
---
pipe 管道
- ps aux 把输入的命令传给grep java 里面，让java 运行