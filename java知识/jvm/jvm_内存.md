[TOC]
# jvm 内存解析
先来看图
![](https://raw.githubusercontent.com/Haiyang-coder/ImageRepository/main/image-20220913153513262.png)

字节码执行引擎是用来执行`.class`字节码文件的,每一个线程会有一个单独的程序计数器用来记录线程执行到字节码文件的哪个部分了.线程执行方法制会将执行的方法压栈到java虚拟机栈,java虚拟机栈中有方法的局部变量等方法信息.如果局部变量是引用类型,如new出的实例对象,这些实例对象存储在java的堆内存中,java虚拟机中会指向堆内存,方法区就是放类信息的.

* 方法区(放类的)
* 堆内存
* 程序计数器
* java虚拟机栈
* 方法运行区

所以你再new出一个类的时候,这个类会有两部分,一部分是在方法区中的唯一一份,你使用`.class`方法的得到的就是这东西,另一部分就是你new出来的对象实例,被放在了java的堆内存中

## 程序寄存器
>存放了执行的代码的地址
## java虚拟机栈
>也叫做帧栈,帧又叫做栈帧,这都是线程私有的,每一个线程有自己的一个帧栈,帧用来保存方法的局部变量,操作数栈.每一次调用一个方法都会创建一个栈帧压入栈中
## java堆
>存放系统创建的对象和数组,所有的线程都共享java的堆空间,对于GC来说,他管理的主要内容就是java堆.
>在运行时自动飞分配内存大小,自动进行垃圾回收,可以扩展和参数设置有关,堆也是分代的,是为了方便垃圾回收算法
- 对象的访问定位:以下是句柄方式
![](https://raw.githubusercontent.com/Haiyang-coder/ImageRepository/main/内存.png)
- 对象的访问定位:以下是指针模式
  ![](https://raw.githubusercontent.com/Haiyang-coder/ImageRepository/main/指针模式.png)

>句柄的运行速度慢(需要两次寻址),但是修改的话只需要修改一次,指针的话直接寻址一次就可以.
## 方法区
>方法区是线程共享的,用来保存装载的类的结构信息,如常量池,字段,方法等等你可以看一下Class字节码文件

