# touch-创建文件

touch 命令有两个功能：一是创建新的空文件，二是改变已有文件的时间戳属性。

touch 命令会根据当前的系统时间更新指定文件的访问时间和修改时间。如果文件不存在，将会创建新的空文件，除非指定了”-c”或”-h”选项。

注意：在修改文件的时间属性的时候，用户必须是文件的属主，或拥有写文件的访问权限。

**语法格式**：touch [参数][文件]

## 常用参数

|    参数     | 注释                                       |
| :---------: | :----------------------------------------- |
|     -a      | 改变档案的读取时间记录                     |
|     -m      | 改变档案的修改时间记录                     |
|     -r      | 使用参考档的时间记录，与 --file 的效果一样 |
|     -c      | 不创建新文件                               |
|     -d      | 设定时间与日期，可以使用各种不同的格式     |
|     -t      | 设定档案的时间记录，格式与 date 命令相同   |
| --no-create | 不创建新文件                               |
|   --help    | 显示帮助信息                               |
|  --version  | 列出版本讯息                               |

## 参考实例

```shell
# 创建空文件
[root@xiaobei xiaobei]# touch file.txt

# 批量创建文件
[root@xiaobei xiaobei]# touch file{1..5}.txt
[root@xiaobei xiaobei]# ls
file1.txt  file2.txt  file3.txt  file4.txt  file5.txt  file.txt

# 修改文件的access(访问)时间
[root@xiaobei xiaobei]# stat file.txt
  File: ‘file.txt’
  Size: 0         	Blocks: 0          IO Block: 4096   regular empty file
Device: fd00h/64768d	Inode: 16847736    Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:home_root_t:s0
Access: 2020-07-02 18:56:34.099010059 +0800
Modify: 2020-07-02 18:56:34.099010059 +0800
Change: 2020-07-02 18:56:34.099010059 +0800
 Birth: -

[root@xiaobei xiaobei]# touch -a file.txt

[root@xiaobei xiaobei]# stat file.txt 
  File: ‘file.txt’
  Size: 0         	Blocks: 0          IO Block: 4096   regular empty file
Device: fd00h/64768d	Inode: 16847736    Links: 1
Access: (0644/-rw-r--r--)  Uid: (    0/    root)   Gid: (    0/    root)
Context: unconfined_u:object_r:home_root_t:s0
Access: 2020-07-02 18:58:24.913003454 +0800
Modify: 2020-07-02 18:56:34.099010059 +0800
Change: 2020-07-02 18:58:24.913003454 +0800
 Birth: -

```
