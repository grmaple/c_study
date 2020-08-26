## C语言工程开发

### 工程开发常用命令

pwd

查看自己在操作系统中当前位置(路径)

/   根目录

cd

切换所在路径

ls -l

查看当前目录下信息

mv 移动文件

mv main.c old    

touch 创建文件

touch main.c

rm 删除文件

rm main.c

cat 查看文件内容

cat Makefile

更多相关命令的可附加参数及使用意义，可以通过 man + [空格] + [命令名] 的方式进行进一步的查询（查询后退出只需敲击键盘上的 q 即可）。

### 多模块程序

对于一个只完成一个特定的任务，只包含十几个函数的程序来说，单文件的组织方式还算可以接受，但是当程序越来越长，程序实现的功能越来越多，将他们全部都组织在一个文件里就会变得不那么容易让人接受了。

因此，我们需要学习如何在 C 语言中将不同功能在多个代码文件中分别实现，然后将它们看作多个模块组织在一起为同一个程序服务。

对单个文件进行编译生成可执行文件

gcc -o program main.c && ./program

当程序有多个模块时，问题就开始变得复杂了.

我们对每一个模块会首先编译出每个模块对应的*.o目标代码文件（relocatable object file），例如：

编译，生成目标文件

gcc -c -o set.o set.c

链接，生成可执行文件

gcc -o program main.o set.o others.o

当我们将一个程序写在多个文件中时，每一个文件中的变量和函数默认都是只有文件内的部分才可以访问的。但是有一些特殊的全局变量、类型定义、函数可能会需要在多个文件中被使用。

这时候，我们可以将这类的内容单独写成一个 头文件（header file），并且将全局变量、类型定义、函数声明写到头文件中。

由于头文件里也可以引入头文件，因此我们可能事实上多次引入同一个文件，比如我们引入1.h和2.h，且1.h也引入2.h，这时因为2.h被引入了两次，就有可能出现重复的声明。
为了解决这个问题，我们在2.h中定义一个宏，在2.h的最开始判断这个宏是否被定义过，如果被定义过，就跳过2.h整个文件的内容。

这里我们将会用到两个新的预处理指令#ifndef xxx和#endif，它们成对出现且#ifndef在前，作用是如果这时并未已定义xxx宏，则这对#ifndef xxx, #endif之间的内容有效。（其中xxx可以替换为任意宏名）

```c
//2.h
#ifndef _2_H
#define _2_H
typedef enum Status {Success, Fail};
typedef struct {
    char *name;
    int age;
} People;
Status go_to_Jisuanke(People);
#endif
```

实际上，我们一般会采用一个与头文件名相关的名字来代替xxx，比如一个常用的代码风格里，这个宏的名字形式为工程名\_路径名\_文件名\_H_。

### Makefile

在前面学习多模块程序的时候，我们需要先把每个模块的代码都生成为目标代码文件，然后再将目标代码文件联编成一个可执行文件。如果每一次编译都要输入这么多命令，是不是很复杂呢？如果每次修改一点点内容就需要重新编译整个工程，是不是很浪费时间呢？

为了解决所遇到的问题，方便开发，我们使用一个叫做make的命令，它可以读取Makefile文件，并且根据Makefile中的规则描述把源文件生成为可执行的程序文件。

最基本的Makefile中包含了一系列形式如下的规则。

目标: 依赖1 依赖2 ...

​	命令

```makefile
array.o: array.c array.h
    gcc -c -o array.o array.c
```

表示生成的文件是目标代码文件array.o，它依赖于array.c和array.h。当我们在命令行中执行make array.o时，根据这一规则，如果array.o不存在或者array.c与array.h至少之一比array.o更新，就会执行gcc -c -o array.o array.c。

我们把上述代码保存为Makefile，与array.c和array.h放在同一目录，在那个目录里执行make array.o就能看到效果。

注意：Makefile里的除当前目录隐藏文件外的第一个目标会成为运行make不指定目标时的默认目标。

```makefile
main: main.o array.o
    gcc -o main array.o main.o
main.o: main.c array.h
    gcc -c -o main.o main.c
array.o: array.c array.h
    gcc -c -o array.o array.c
```

在Makefile有多条规则时，如果我们希望只生成其中一个，我们可以在make命令后加上需要生成的目标的名称。

例如，在这里我们可以执行make main.o、make array.o或make main。当我们执行make main时，make命令发现array.o和main.o不存在，就会根据以它们为目标的规则先生成它们。

一个 Makefile 可以包含多个规则，我们既可以每次在make后说明执行哪个功能，也可以通过定义的all来执行一系列的规则。

很多时候，你会需要将.o为后缀的目标代码文件和可执行的程序文件删除，完全从头进行编译。那么我们可以写一条clean规则，例如：

```makefile
clean:
	rm -f array.o main.o main
```

按照预期，当我们执行make clean就可以删除array.o、main.o和main了。事实真的这样吗？

细心的同学已经发现，这时如果已经存在clean文件，rm命令就不会执行了。为了解决这个问题，我们通过一个特殊的方法告诉make这个名为clean的规则在clean存在的时候仍然有效。

```makefile
.PHONY: clean
    
clean:
	rm -f array.o main.o main
```

.PHONY用于声明一些伪目标，伪目标与普通的目标的主要区别是伪目标不会被检查是否存在于文件系统中而默认不存在且不会应用默认规则生成它。

在Makefile中我们还可以使用它的变量和注释。

```makefile
# 设置C语言的编译器
CC = gcc
# 标记编译参数
# -g 增加调试信息 -Wall 打开大部分警告信息
CFLAGS = -g -Wall
#整理一下main依赖哪些目标文件
MAINOBJS = main.o array.o
#声明一些伪目标
.PHONY: clean
main: $(MAINOBJS)
    $(CC) $(CFLAGS) -o main $(MAINOBJS)
main.o: main.c array.h
    $(CC) $(CFLAGS) -c -o main.o main.c
array.o: array.c array.h
    $(CC) $(CFLAGS) -c -o array.o array.c
clean:
	rm -f $(MAINOBJS) main
```

定义的变量可以直接通过$(变量名)进行使用。

Makefile其实描述了一系列转为对象文件、联编的过程，不使用make也是可以完成的。

### 命令行参数

我们的main函数一般都是没有参数的，对应的，在运行时，我们一般就是直接输入可执行的程序文件名（例如./main）。

但是，实际上main函数是可以有参数的。我们可以将任何过去无参数的main函数替换成下面这种有参数的main函数（不过考虑到我们并没有利用，不写是很正常的）。

```c
int main (int argc, char **argv) {
	//...
}
```

在这里，main函数有两个参数，第一个参数是整数型，会传入命令行参数的个数，程序运行时就可以接收到。第二个参数是char **，其中储存了用户从命令行传递进来的参数。

如果我们的程序可执行文件名为main，则在命令行中输入./main hello world我们会得到argc为 3，argv[0]为./main（可执行程序路径会被视为首个参数），argv[1]为hello，argv[2]为world。如果有更多参数也可以以此类推。

命令行参数默认都是空格分隔，但是如果我们希望包含空格的一个字符串作为参数，我们则需要在输入参数时用引号将其包裹起来。

如果我们的程序可执行文件名为main，则在命令行中输入./main "hello world" is my greet我们会得到argc为 5，argv[0]为./main，argv[1]为hello world，argv[2]为is，argv[3]为my，argv[4]为greet。

任何被接收到的argv参数都可以被当做正常的字符串在代码里使用。在很多程序的设计中，我们会需要根据接收到的参数来决定程序的执行方式，这时候，学会使用argc和argv就显得很重要了。

```c
#include <stdio.h>
int main(int argc, char **argv) {
    int i;
    printf("程序运行参数共 %d 个，分别是：\n", argc);
    for (i = 0; i < argc; i++) {
    	printf("argv[%d]: %s\n", i, argv[i]);
    }
    return 0;
}
```

### 文件操作

在读文件的时候我们需要先有一个可以让我们访问到文件的 文件指针（file pointer），它是一个FILE类型的指针。

FILE *fp;//声明一个文件指针

fp = fopen(文件路径，访问模式);//将文件指针和文件关联起来

第一个参数是一个字符串，对应了我们希望访问的文件路径。第二个参数是访问模式，它可以是表示只读模式的"r"，也可以是表示只写模式的"w"，还可以是在文件末尾追加的"a"。

通过fgetc(fp);获得当前指针之后位置的一个字符了，每获得一个字符，指针会向后移动一个字符（如果到达文件尾部则会返回EOF）。

通过fputc('c', fp);的方式将字符'c'写入到fp关联的文件内了。

实现将一个文件复制到另一个文件内的函数

```c
void filecopy(FILE *in_fp, FILE *out_fp) {
    char ch;
    while ((ch = fgetc(in_fp)) != EOF) {
        fputc(ch, out_fp);
    }
}
```

这个函数接收的两个参数都是文件指针。这个函数会通过一个可读模式的文件指针逐字符地读取，并且通过一个可写模式的文件指针逐字符地将所有字符写出，从而起到复制文件内容的作用。

你需要注意，在给文件指针进行命名的时候，要避开 stdin、stdout 和 stderr 这三个名称。因为这三个名称其实已经用于标准输入、标准输出、标准错误的文件指针。

你可能会问了，那我们看到的 stdin、stdout 和 stderr 的这三个文件指针可以直接使用吗？回答是肯定的。

我们是通过 fgetc(stdin); 获得来自标准输入的字符，也可以通过 fputc(ch, stdout); 或 fputc(ch, stderr); 将变量 ch 中的字符输出到标准输出或标准错误中的。

除了fgetc和fputc之外，我们还可以使用fscanf和fprintf函数。这两个函数都很像我们已经很熟悉的scanf和printf函数，只是不过，scanf和printf 可以被看作 fscanf和fprintf 的特例。

我们使用 fscanf 从文件指针in_fp进行读取时，可以写成：

fscanf(in_fp, "%c", &a);

而如果我们写

fscanf(stdin, "%c", &a);

这将完全与下面直接使用 scanf 的方式等价。

scanf ("%c", &a);

类似地，我们使用fprintf向文件指针out_fp进行写出时，可以写成：

fprintf(out_fp, "%c", a);

而如果我们写

fprintf(stdout, "%c", a);

这将完全与下面直接使用 printf 的方式等价。

printf(a);

在使用文件并且确定不再继续使用后，我们要通过下面所示的方式将文件指针fp与文件的关联断开。你可以将它视为和fopen相反的一个操作。

fclose(fp);

如果你不在程序中使用fclose，程序正常结束时，程序会为所有打开的文件调用fclose。

stdin、stdout 其实也是被打开的文件指针，如果你觉得用不到的话，其实也是可以使用 fclose 将他们关闭掉的。

```c
void filecopy(FILE *in_fp, FILE *out_fp) {
    char ch;
    while ((ch = fgetc(in_fp)) != EOF) {
        fputc(ch, out_fp);
    }
}
void filecopy2(FILE *in_fp, FILE *out_fp) {
    char ch;
    while (fscanf(in_fp, "%c", &ch) != EOF) {
        fprintf(out_fp, "%c", ch);
    }
}
int main() {
    FILE *in_fp, *out_fp;
	in_fp = fopen(文件路径, "r");
    out_fp = fopen(文件路径, "w");
    filecopy(in_fp, out_fp);
    fclose(in_fp);
    fclose(out_fp);
}
```

### 一个简单命令的实现

实现一个我们起名为ncat的命令，其作用是把输入的内容每行前加上行号输出。它可能有 2个、 3 个或 4 个命令行参数，第一个命令行参数是它本身的名字，在这节里我们可以忽略，第二个参数一定是-d（表示 decimal）或者-r（表示 roman）中的一个，若为-d则行号用阿拉伯数字表示，若为-r则行号用罗马数字表示，若存在第三个参数则其表示输入文件的路径，若不存在则表示输入来自标准输入，若存在第四个参数则其表示输出文件的路径，若不存在则表示输出到标准输出。注意：行号与输入文本的回显之间用\t隔开。

```c
#include <stdio.h>
#include <string.h>
#include "roman.h"
int main(int argc, char *argv[]) { 
    if (argc == 2) {//标准输入输出
        if (strcmp(argv[1], "-d") == 0) {//行号用阿拉伯数字表示
            char c;
            char buf[50][100];
            int i = 0;
            int j = 0;
            while (fscanf(stdin, "%c", &c) != EOF) {
                if (c != '\n') {
                    buf[j][i++] = c;
                } else {
                    buf[j][i] = '\0';
                    i = 0;
        	        j++;
                }              
            }
            for (int k = 0; k < j; k++) {
                int n = k + 1;
                fprintf(stdout, "%d\t%s\n", n, buf[k]);         
            }
        } else if (strcmp(argv[1], "-r") == 0) {//行号用罗马数字表示
            char c;
            char buf[50][100];
            int i = 0;
            int j = 0;
            while (fscanf(stdin, "%c", &c) != EOF) {
                if (c != '\n') {
                    buf[j][i++] = c;
                } else {
                    buf[j][i] = '\0';
                    i = 0;
        	        j++;
                }              
            }
            for (int k = 0; k < j; k++) {
                char* n;
                n = to_roman_numeral(n, k+1);
                fprintf(stdout, "%s\t%s\n", n, buf[k]);         
            }
        }
    } else if (argc == 3) {//文件输入，标准输出
        FILE *in_fp;
        in_fp = fopen(argv[2], "r");
        if (strcmp(argv[1], "-d") == 0) {//行号用阿拉伯数字表示
            char c;
            char buf[50][100];
            int i = 0;
            int j = 0;
            while (fscanf(in_fp, "%c", &c) != EOF) {
                if (c != '\n') {
                    buf[j][i++] = c;
                } else {
                    buf[j][i] = '\0';
                    i = 0;
        	        j++;
                }              
            }
            for (int k = 0; k < j; k++) {
                int n = k + 1;
                fprintf(stdout, "%d\t%s\n", n, buf[k]);         
            }
        } else if (strcmp(argv[1], "-r") == 0) {//行号用罗马数字表示
            char c;
            char buf[50][100];
            int i = 0;
            int j = 0;
            while (fscanf(in_fp, "%c", &c) != EOF) {
                if (c != '\n') {
                    buf[j][i++] = c;
                } else {
                    buf[j][i] = '\0';
                    i = 0;
        	        j++;
                }              
            }
            for (int k = 0; k < j; k++) {
                char* n;
                n = to_roman_numeral(n, k+1);
                fprintf(stdout, "%s\t%s\n", n, buf[k]);         
            }
        }
    } else if (argc == 4) {//文件输入输出
        FILE *in_fp, *out_fp;
        in_fp = fopen(argv[2], "r");
        out_fp = fopen(argv[3], "w");
        if (strcmp(argv[1], "-d") == 0) {//行号用阿拉伯数字表示
            char c;
            char buf[50][100];
            int i = 0;
            int j = 0;
            while (fscanf(in_fp, "%c", &c) != EOF) {
                if (c != '\n') {
                    buf[j][i++] = c;
                } else {
                    buf[j][i] = '\0';
                    i = 0;
        	        j++;
                }              
            }
            for (int k = 0; k < j; k++) {
                int n = k + 1;
                fprintf(out_fp, "%d\t%s\n", n, buf[k]);         
            }
        } else if (strcmp(argv[1], "-r") == 0) {//行号用罗马数字表示
            char c;
            char buf[50][100];
            int i = 0;
            int j = 0;
            while (fscanf(in_fp, "%c", &c) != EOF) {
                if (c != '\n') {
                    buf[j][i++] = c;
                } else {
                    buf[j][i] = '\0';
                    i = 0;
        	        j++;
                }              
            }
            for (int k = 0; k < j; k++) {
                char* n;
                n = to_roman_numeral(n, k+1);
                fprintf(out_fp, "%s\t%s\n", n, buf[k]);         
            }
        }
    }
    return 0;
}
```

```makefile
CC = gcc
CFLAGS = -std=c99 -Wall -Werror
MAINOBJS = ncat.o roman.o
ncat: $(MAINOBJS)
	$(CC) $(CFLAGS) -o ncat $(MAINOBJS)
ncat.o: ncat.c roman.h
	$(CC) $(CFLAGS) -c -o ncat.o ncat.c
roman.o: roman.c roman.h
	$(CC) $(CFLAGS) -c -o roman.o roman.c
clean:
	$(RM) -- *.o ncat

```

```c
#include "roman.h"

static const char *symbol[] = { "I", "V", "X", "L", "C", "D", "M",
                                "-V", "-X", "-L", "-C", "-D", "-M",
                                "=V", "=X", "=L", "=C", "=D", "=M" };

static char* addsymbol(char* dst, int index)
{
	for (int i = 0; *(symbol[index] + i) != '\0'; i++) {
		*(dst++) = *(symbol[index] + i);
	}
	return dst;
}

static char* _roman(char* dst, int src, int index)
{
	if (src != 0) {
		dst = _roman(dst, src / 10, index + 1);
		src %= 10;
		if (src >= 9) {
			dst = addsymbol(dst, index * 2);
			dst = addsymbol(dst, index * 2 + 2);
			src -= 9;
		}
		if (src >= 5) {
			dst = addsymbol(dst, index * 2 + 1);
			src -= 5;
		}
		if (src >= 4) {
			dst = addsymbol(dst, index * 2);
			dst = addsymbol(dst, index * 2 + 1);
			src -= 4;
		}
		while (src > 0) {
			dst = addsymbol(dst, index * 2);
			src--;
		}
	}
	return dst;
}

char* to_roman_numeral(char* dst, int src)
{
	*(_roman(dst, src, 0)) = '\0';
	return dst;
}

```

### 调试代码

gdb

在编译时使用-g作为一个编译参数(否则你将看不到函数名，变量名，而只能看到运行时的内存地址)

gcc -o program -g main.c

然后我们通过

gdb ./program

将程序在gdb提供的一个受控制的环境中运行起来

l	gdb会列出带着行号的10行程序，l之后加行号作为参数，列出这一行开始的10行

b	设置程序运行的断点，程序运行到断点时就会暂停运行，b之后既可以加函数名作为参数，使程序在调用某一函数时暂停，也可以加行号作为参数，使程序在运行某一行时暂停。

r	程序会开始运行，并且暂停在断点位置

运行暂停时

​	p 表达式	表示我们希望在当前断点运行这个表达式并查看他的值，例如 p ++age[0]表示我们希望让age[0]+1并查看之后的值，注意这个表达式会对之后的程序继续运行造成影响。

​	n	程序会执行暂停位置后的下一条语句并再次暂停。

​	c	程序会继续执行到下一断点再暂停。

### 简易OJ

首先用gdb调试并修正一个关于判断三角形的程序。然后编写一个简单的 OJ 核心判断这个程序或者其它一个仅操作于标准输入输出的程序是否正确。

```makefile
CC = gcc
CFLAGS = -std=c99 -g -Wall -Wextra

OBJS = oj.o run.o

.PHONY: clean

oj: $(OBJS)
	$(CC) $(CFLAGS) -o oj $(OBJS)
oj.o: oj.c run.h
	$(CC) $(CFLAGS) -c -o oj.o oj.c
run.o: run.c run.h
	$(CC) $(CFLAGS) -c -o run.o run.c 
clean:
	$(RM) -- $(OBJS) oj 
```



