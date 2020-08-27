## typedef

### 给数据类型起别名

第一门高级语言：Fortran

任何让C语言使用和Fortran相同的关键字定义变量呢？

<img src="C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200827221510466.png" alt="image-20200827221510466" style="zoom: 50%;" />

因此有了typedef关键字

typedef int INTEGER;

并不等价于 #define INTEGER int

一个是无符号整型的区别，一个是指针的区别。

相比起宏定义的直接替换，typedef是对类型的封装

typedef还可以一次性起多个别名。

typedef int INTEGER, *PIRINT;

### 给结构体起别名

typedef struct Date{

​	int year;

​	int month;

​	int day;

} DATE, *PDATE;

### 进阶typedef

在编程中使用typedef目的一般有二个：

一个是给变量起一个容易记住且意义明确的别名

一个是简化一些比较复杂的类型声明

一些比较恐怖的声明语句

```c
//数组指针
//int (*ptr)[3]
typedef int (*PTR_TO_ARRAY)[3];
int array[3] = {1, 2, 3};
PTR_TO_ARRAY ptr_to_array = &array;
//int (*ptr_to_array)[3];	ptr_to_array = &array;
//(*ptr_to_array)[0]
```

```c
//函数指针
//int (*ptr)()
typedef int (*PTR_TO_FUN)();
int fun() {return 520;}
PTR_TO_FUN ptr_to_arfun = &fun;
//int (*ptr_to_fun)();	ptr_to_fun = &fun;
//(*ptr_to_fun)()
```

```c
//指针数组，数组里的每个指针都是指针函数
//int *(*array[3])(int)
typedef int *(*PTR_TO_FUN)(int);
int *funA(int num) {return &num;}
int *funB(int num) {return &num;}
int *funC(int num) {return &num;}
PTR_TO_FUN array[3] = {&funA, &funB, &funC};
```

```c
//funA是指针函数，参数一是整型，参数二是函数指针，返回值是函数指针
//void (*funA(int, void (*funB)(int)))(int)
typedef void (*PIR_TO_FUN)(void);
PIR_TO_FUN funA(int, PIR_TO_FUN);
```

实际代码开发这样写真的有这么意义吗？

![image-20200827225712449](C:\Users\xuyingfeng\AppData\Roaming\Typora\typora-user-images\image-20200827225712449.png)