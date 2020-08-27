## C语言内存布局

![image-20200827183344267](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200827183344267.png)

![image-20200827183355394](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200827183355394.png)

size test 可以查看内存布局

##### 代码段text

用来存放程序执行代码的一块内存区域，在运行前已经确定大小。

也包含字符串常量

##### 数据段data

用来存放已经初始化的全局变量和局部静态变量。

##### bss段

用来存放已经未初始化的全局变量和局部静态变量。

在运行前自动初始化为0

##### 堆

用于存放进程运行中被动态分配的内存段

##### 栈

函数执行的内存区域