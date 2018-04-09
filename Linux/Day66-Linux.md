# Linux

#### 内容摘要

Shell 概述，常用命令

------

#### Shell 概述

* 用户使用操作系统接口，命令语言，可以使用大部分系统内核的功能
* 主流 Shell
  * Bourne Shell （贝尔实验室）
  * BASH （GNU Bourne Again Shell）
  * Korn Shell （对 Bourne Shell 的发展、兼容）
  * C Shell （FreeBSD Shell，类 C 风格）

### 常用命令

#### cd

*change directory*，进入指定目录

* `..`  前往父目录
* `-`  前往上一次跳转目录
* `~` 前往 home/username 目录

#### mkdir

* `-p` 父目录不存在时会创建
* `-v` 显示创建成功信息

#### rmdir

* `-p` 递归删除空目录

```bash
rmdir -p a/b/c
# 等价于
rmdir a/b/c a/b a
```

#### ls

* `-a` 包括隐藏文件
* `-l` 详细信息，等价于 `ll`

#### ll

* `-h` 友好显示，将文件大小换算后显示 `4096 => 4kb`

#### pwd

*print working directory*，查看当前所在目录绝对路径

#### cat

*concat*，查看整个文件内容，输出到终端

#### more

分页显示

* space：下一屏
* enter ：下一行
* b ：上一页
* q ：退出

#### less

分页显示，可以左右移动

* PgUp、PgDn
* 方向键
* q ：退出

#### head

查看文件前几行

#### tail

查看文件后几行

* `-f` 当文件有追加写入新信息时，追加显示新信息，即动态查看文件

#### touch 

创建文件、或者修改时间

* `-c` 文件不存在时不创建新文件，文件存在时更新修改时间
* `-m` 更新文件修改时间
* `-t` 修改文件修改时间，[YYYY]MMDDhhmm[.ss]

#### cp

`cp source directory` 复制文件或目录

* `-r` 复制整个目录文件，递归复制
* `-f` 复制目录时，如果有同名目录，直接覆盖
* `-i` 复制目录时，如果有同名目录，会提示

####rm

删除文件或目录，不会进入回收站

* `-i` 删除前询问
* `-rf` 删除目录，递归删除 + 强制删除

#### mv

`mv source desc` 移动或重命名

* `desc` 是目录就移动，是文件名就重命名

#### tar

打包文件／目录或者解压缩

* `-c` 创建一个新的 **tar** 文件
* `-v` 显示运行过程信息
* `-f` 指定文件名
* `-z` 调用 `gzip` 压缩命名进行压缩
* `-t` 查看压缩文件内容
* `-x` 解压缩 **tar** 文件

```bash
# -cvf 打包
tar -cvf a.tar a
# -zcvf 打包并压缩 gzip
tar -zcvf b.tar b
# -xvf 解压缩到当前目录
tar -zcvf c.tar
# -xvf c.tar -C c 解压到 c 目录下（C 大写）
tar -xvf c.tar -C c
```

#### grep

`grep pattern filename` 查找文件中符合 *pattern* 内容，支持正则表达式

`|` 管道操作符，将左边的命令的标准输出作为右边命令的标准输入

```bash
grep "hello" hello.txt

ll | grep "^-"
```

#### find

* `-name` 按文件名查找，支持正则
* `-empty` 查找大小为 0 的目录或者文件
* `-type` 指定类型的文件或目录（d 目录，f 文件）
* `-user` 指定查找用户所属文件
* `-maxdepth` 限制文件查找范围，递归查找对大深度

```bash
# 注意命令顺序
find -maxdepth 1 -user parallels -type f -name "*.txt"
```

#### 重定向 `<` `<<` `>` `>>`

将某命令的标准输出重定向到另外地方，通常是文件里

* `>` 重定向到目标输出，会清空之前内容
* `>>` 重定向到目标输出，会追加新内容

将标准输入即键盘输入，重定向到另外地方，通常是文件里

* `<` 标准键盘输入
* `<<` 以结束符作为输入结束标志

```bash
# 将 ls 命令的输出重定向写入到 ls.txt 内，会覆盖原有内容
ls > ls.txt
# 将 ls 命令的输出重定向写入到 ls.txt 内，不会覆盖缘由内容，追加写入
ls > ls.txt

# 将键盘输入，以 eof 为结束之前的内容写入到 hello.txt 中
cat > hello.txt << eof
```

#### date

获取系统当前时间

#### ps -ef

*process status*，查看所有进程

```bash
 # 查看关键字为 root 的所有进程
 ps -ef | grep "root"
```

#### kill

结束进程号的进程

* `-9` 强制结束

```bash
# 结束进程号为 1399 的进程
kill -9 1399
```

#### ifconfig

查看网络设置

#### ping

测试网路连通性

#### sudo netstat -anp | grep port

查看端口号为 port 的网络端口被哪个应用程序占用