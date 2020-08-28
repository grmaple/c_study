## IO缓冲区

![image-20200828163231566](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200828163231566.png)

解决IO设备和CPU设备速度不匹配问题

和内存池原理类似

当调用fputs时，数据并没有写入设备，写入的是缓冲区里面，只有当fclose后才写入设备。

如果我们希望数据立刻写入设备呢？

fflush函数，刷新缓冲区，立刻写入设备。

标准IO提供三种类型的缓冲模式

按块缓存：全缓存，填满缓冲区后在进行设备读写

按行缓存：在接收到\n前，都是先缓存在缓冲区

不缓存：允许直接读写设备上的数据

setvbuf函数：修改一个数据流的缓冲模式

![image-20200828163945774](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200828163945774.png)

char buff[1024];

stvbuf(stdout, buff, _IOFBF, 1024);



getchar();//阻塞函数

C 库函数 **int getchar(void)** 从标准输入 stdin 获取一个字符（一个无符号字符）。这等同于 **getc** 带有 stdin 作为参数。

putchar(ch);

C 库函数 **int putchar(int char)** 把参数 char 指定的字符（一个无符号字符）写入到标准输出 stdout 中。