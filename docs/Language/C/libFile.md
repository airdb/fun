linux 中的.so和.a文件
===

http://blog.csdn.net/nieyinyin/article/details/6890557


Linux下的.so是基于Linux下的动态链接,其功能和作用类似与windows下.dll文件。

下面是关于.so的介绍:

一、引言

通常情况下，对函数库的链接是放在编译时期（compile time）完成的。所有相关的对象文件（object file）与牵涉到的函数库（library）被链接合成一个可执行文件（executable file）。程序在运行时，与函数库再无瓜葛，因为所有需要的函数已拷贝到自己门下。所以这些函数库被成为静态库（static libaray），通常文件名为“libxxx.a”的形式。


其实，我们也可以把对一些库函数的链接载入推迟到程序运行的时期（runtime）。这就是如雷贯耳的动态链接库（dynamic link library）技术。

二、动态链接库的特点与优势

首先让我们来看一下，把库函数推迟到程序运行时期载入的好处：

1. 可以实现进程之间的资源共享。

什么概念呢？就是说，某个程序的在运行中要调用某个动态链接库函数的时候，操作系统首先会查看所有正在运行的程序，看在内存里是否已有此库函数的拷贝了。如果有，则让其共享那一个拷贝；只有没有才链接载入。这样的模式虽然会带来一些“动态链接”额外的开销，却大大的节省了系统的内存资源。C的标准库就是动态链接库，也就是说系统中所有运行的程序共享着同一个C标准库的代码段。

2. 将一些程序升级变得简单。用户只需要升级动态链接库，而无需重新编译链接其他原有的代码就可以完成整个程序的升级。Windows 就是一个很好的例子。

3. 甚至可以真正坐到链接载入完全由程序员在程序代码中控制。

程序员在编写程序的时候，可以明确的指明什么时候或者什么情况下，链接载入哪个动态链接库函数。你可以有一个相当大的软件，但每次运行的时候，由于不同的操作需求，只有一小部分程序被载入内存。所有的函数本着“有需求才调入”的原则，于是大大节省了系统资源。比如现在的软件通常都能打开若干种不同类型的文件，这些读写操作通常都用动态链接库来实现。在一次运行当中，一般只有一种类型的文件将会被打开。所以直到程序知道文件的类型以后再载入相应的读写函数，而不是一开始就将所有的读写函数都载入，然后才发觉在整个程序中根本没有用到它们。

三、动态链接库的创建

由于动态链接库函数的共享特性，它们不会被拷贝到可执行文件中。在编译的时候，编译器只会做一些函数名之类的检查。在程序运行的时候，被调用的动态链接库函数被安置在内存的某个地方，所有调用它的程序将指向这个代码段。因此，这些代码必须实用相对地址，而不是绝对地址。在编译的时候，我们需要告诉编译器，这些对象文件是用来做动态链接库的，所以要用地址不无关代码（Position Independent Code （PIC））。

对gcc编译器，只需添加上 -fPIC 标签，如：

gcc -fPIC -c file1.c
gcc -fPIC -c file2.c
gcc -shared libxxx.so file1.o file2.o

注意到最后一行，-shared 标签告诉编译器这是要建立动态链接库。这与静态链接库的建立很不一样，后者用的是 ar 命令。也注意到，动态链接库的名字形式为 “libxxx.so” 后缀名为 “.so”

四、动态链接库的使用

使用动态链接库，首先需要在编译期间让编译器检查一些语法与定义。

这与静态库的实用基本一样，用的是 -Lpath 和 -lxxx 标签。如：

gcc file1.o file2.o -Lpath -lxxx -o program.exe

编译器会先在path文件夹下搜索libxxx.so文件，如果没有找到，继续搜索libxxx.a（静态库）。

在程序运行期间，也需要告诉系统去哪里找你的动态链接库文件。在UNIX下是通过定义名为 LD_LIBRARY_PATH 的环境变量来实现的。只需将path赋值给此变量即可。csh 命令为：

setenv LD_LIBRARY_PATH   your/full/path/to/dll

一切安排妥当后，你可以用 ldd 命令检查是否连接正常。

ldd program.exe