# userdel-删除用户

userdel 命令用于删除指定的用户及与该用户相关的文件，英文全称即“user delete”。其实 userdel 命令实际上是修改了系统的用户账号文件 /etc/passwd、/etc/shadow 以及/etc/group 文件。这与 Linux 系统”一切操作皆文件”的思想正好吻合。

值得注意的是，但是如果有该要删除用户相关的进程正在运行，userdel 命令通常不会删除一个用户账号。如果确实必须要删除，可以先终止用户进程，然后再执行 userdel 命令进行删除。但是 userdel 命令也提供了一个面对该种情况的参数，即”-f”选项。

**语法格式**：userdel [参数][用户名]

## 常用参数

| 参数 | 注释                           |
| :--: | :----------------------------- |
|  -f  | 强制删除用户账号               |
|  -r  | 删除用户主目录及其中的任何文件 |
|  -h  | 显示命令的帮助信息             |

## 参考实例

```shell
#删除用户，但不删除其家目录及文件：
[root@xiaobei ~]# userdel www

#删除用户，并将其家目录及文件一并删除：
[root@xiaobei ~]# userdel -r www

#强制删除用户：
[root@xiaobei ~]# userdel -f www

```
