# ln-为文件创建链接

ln 命令是 linux 系统中一个非常重要命令，英文全称是“link”，即链接的意思，它的功能是为某一个文件在另外一个位置建立一个同步的链接。 一种是 hard link，又称为硬链接；另一种是 symbolic link，又称为符号链接。

通俗一点理解，可以把硬链接当成源文件的副本，他和源文件一样的大小，但是事实上却不占任何空间。符号链接可以理解为类似 windows 一样的快捷方式。

**符号链接**

1. 符号链接以路径的形式存在，类似于 Windows 操作系统中的快捷方式。
2. 符号链接可以跨文件系统 ，硬链接不可以。
3. 符号链接可以对一个不存在的文件名进行链接，硬链接不可以。
4. 符号链接可以对目录进行链接，硬链接不可以。

**硬链接**

1. 硬链接以文件副本的形式存在，但不占用实际空间。
2. 硬链接不允许给目录创建硬链接。
3. 硬链接只有在同一个文件系统中才能创建。

**语法格式**： ln [参数][源文件或目录] [目标文件或目录]

## 常用参数

| 参数 | 注释                                                 |
| :--: | :--------------------------------------------------- |
|  -b  | 为每个已存在的目标文件创建备份文件                   |
|  -d  | 此选项允许“root”用户建立目录的硬链接                 |
|  -f  | 强制创建链接，即使目标文件已经存在                   |
|  -n  | 把指向目录的符号链接视为一个普通文件                 |
|  -i  | 交互模式，若目标文件已经存在，则提示用户确认进行覆盖 |
|  -s  | 对源文件建立符号链接，而非硬链接                     |
|  -v  | 详细信息模式，输出指令的详细执行过程                 |

## 参考实例

```shell
#为源文件file.txt创建硬链接file：
[root@xiaobei xiaobei]# ln file.txt file

#使用ln命令的“-s”参数来创建目录的符号链接，并使用ls命令来查看链接文件的详细信息：
[root@xiaobei home]# ln -s xiaobei file
[root@xiaobei home]# ls -l
total 0
lrwxrwxrwx. 1 root root   7 Jul  2 19:16 file -> xiaobei
drwxr-xr-x. 2 root root 119 Jul  2 19:14 xiaobei
prw-r--r--. 1 root root   0 Jun 30 16:39 xiaobei.pipe

#使用ln命令的“-v”参数来输出命令的详细执行过程：
[root@xiaobei xiaobei]# ln -v file.txt file_1
‘file_1’ => ‘file.txt’

#使用ln命令的“-b”命令来创建目标文件的备份文件，并使用ls命令来查看：
[root@xiaobei xiaobei]# ln -b file.txt file
[root@xiaobei xiaobei]# ls
file  file~  file_1  file1.txt  file2.txt  file3.txt  file4.txt  file5.txt  file.txt
```
