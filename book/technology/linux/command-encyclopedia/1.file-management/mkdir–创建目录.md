# mkdir–创建目录

mkdir 命令是“make directories”的缩写，用来创建目录。

注意：默认状态下，如果要创建的目录已经存在，则提示已存在，而不会继续创建目录。 所以在创建目录时，应保证新建的目录与它所在目录下的文件没有重名。 mkdir 命令还可以同时创建多个目录。

**语法格式 :** mkdir [参数][目录]

## 常用参数

| 参数 | 注释                         |
| :--: | :--------------------------- |
|  -p  | 递归创建多级目录             |
|  -m  | 建立目录的同时设置目录的权限 |
|  -z  | 设置安全上下文               |
|  -v  | 显示目录创建过程             |

## 参考实例

```shell
# 在工作目录下，建立一个名为 dir 的子目录
[root@xiaobei xiaobei]# mkdir dir
# 在目录/home/xiaobei/dir下建立子目录dir，并且设置文件属主有读、写和执行权限，其他人无权访问
[root@xiaobei xiaobei]# mkdir -m 700 /home/xiaobei/dir/dir
# 同时创建子目录dir1，dir2，dir3
[root@xiaobei dir]# mkdir dir1 dir2 dir3
# 递归创建目录
[root@xiaobei dir]# mkdir -p /home/xiaobei/dir/dd/dir
```
