# 重定向

## 常用重定向快速预览

1. ls -l /usr/bin > ls-output.txt
2. ls -l /bin/usr 2> ls-error.txt
3. ls -l /bin/usr > ls-output.txt 2>&1
4. ls -l /bin/usr &> ls-output.txt
5. ls -l /bin/usr 2> /dev/null
6. ls -l /bin/usr &> /dev/null

## 关于标准输出和标准错误

这是个很小非常重要的概念，且看  `cat` ：

```
$ cat /etc/passwd file_not_exists
root:x:0:0:root:/root:/bin/bash
bin:x:1:1:bin:/bin:/sbin/nologin
...
cat: file_not_exists: 没有那个文件或目录 
```

示例中使用cat程序分别打开两个文件，其中第二个文件是不存在的。
回显中先输出第一份文件内容，这是程序的正常输出，准备输出第二份文件内容时报错。

注意，程序的正常输出和后面的报错内容其实是两个不同的东西，前者是cat程序正常工作内容，后者是出问题cat额外提示错误信息，或者是操作系统来帮忙显示错误信息。这两个信息分属不同的地方，只是恰巧都默认呈现在屏幕上而已。

所以，程序的输出内容其实是区分**标准输出**和**标准错误**的。

## 关于文件描述符

众所周知，Linux中一切都被抽象成了文件（文本、键盘、网卡等）。
对一些标准文件的引用，系统规定了默认值。

文件描述符	| STDIO	| POSIX	| 用途	 | 默认
----|----|----|----|----|----|
0	|stdin	|STDIN_FILENO	|标准输入	|键盘
1	|stdout	|STDOUT_FILENO	|标准输出	|屏幕
2	|stderr	|STDERR_FILENO	|标准错误	|屏幕

这些文件描述类似一个引用/句柄等，熟悉C语言的可以理解为使用 `&1` 就可以操纵标准输出了。

一些讲究的程序会将正常内容打印到标准输出（&1），将错误信息打印到标准错误（&2）。

至此你会不会突然理解了 ，例如：` cat file  > file  2>&1`的意思了。

## 关于重定向标准输出

```
$ ls /not-exists > result
ls: /not-exists: No such file or directory
```

上例中可以发现错误信息仍然在屏幕上呈现了，这是因为 ` > ` 仅仅是重定向标准输出，是不重定向标准出错的（而标准出错默认也是显示在屏幕上）。

## 关于重定向标准错误

让我们回到Linux正在诞生的那个莽荒年代，想象自己就是重定向功能的设计者。

一个程序的输出结果我们已经细分为标准输出（stdout）和标准错误（stderr），这些结果最后放哪呢：

	1. 对于标准输出默认就给输出到 `&1` （也就是引用/句柄为1的文件，一般都是屏幕）上；
	2. 用户可以使用例如 `command 1> file` 将标准输出指定位置，这就是标准输出的重定向了。（后面经过优化，可能发现基本都只重定向标准输出，于是把1直接给省略了）；
	3. 于是重定向stderr就类似于 `command 2> file` 一般容易了；
	4. 我还可以将stderr重定向进stdout里，这样两个结果就可以放一起了，如 `commad > file 2>&1`（先将stdout重定向到file，2>&1，将stderr输出到文件句柄为1的文件里，也就是标准输出了）。


## 参考
1. [Linux中的文件描述符与打开文件之间的关系](http://blog.csdn.net/cywosp/article/details/38965239)
2. [The Linux Command Line](http://linuxcommand.org/tlcl.php)
