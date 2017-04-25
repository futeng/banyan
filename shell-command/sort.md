# Sort

## 你怎么来实现sort程序？

1. 逐行，从第一个字符开始，安装ASCII码进行比较，从小到大的升序；
2. 定位，使用选项指定从哪个位置开始比较；

## 学前案例

1. 有数字的使用 -n number ：`sort -n file`
2. 重定义到本文件 -o output ：（直接使用 `>` 文件会被掏空） `sort -n file -o file`
3. 降序排列（从大到小）：`sort -n -r file`
4. 按照第8列的顺序排列：`sort -n -r -k 8 file`
5. 指定使用竖线分割 ： `sort -n -k 8 -t \| a -file`
6. 先按照指定位置排序，再按照另外一个位置排序，两个k：`sort -n -k 5 -k 8 -t \| FILE -r `
7. 呵呵：`sort -t \| -k 5n -k8nr FILE`
8. 组合：`ps aux | sort -k3nr | more`

## 参考

1. [http://roclinux.cn/?p=1350](http://roclinux.cn/?p=1350)
2. [http://roclinux.cn/?p=1472](http://roclinux.cn/?p=1472)

