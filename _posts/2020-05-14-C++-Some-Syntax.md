---
layout: post
title: "C++各中知识点"
featured-img: shane-rounce-205187
categories: [C++]
---



# C++中的各种变量或者关键字以及符号的用法

#### C++中mutable关键字的用法

​	mutable的意思是“可变的，易变的”，跟constant（即C++中的const）是反义词。

​	在C++中，mutable也是为了突破const的限制而设置的。被mutable关键字修饰的变量，永远将处于可变的状态，即使在一个const函数中。

​	我们知道，被const关键字修饰的函数的一个重要作用就是为了能够保护类中的成员变量。即：该函数可以使用类中的所有成员变量，但是不能修改他们的值。然而，在某些特殊情况下，我们还是需要在const函数中修改类的某些成员变量，因为要修改的成员变量与类本身并无多少关系，即使修改了也不会对类造成多少影响。当然，你可以说，你可以去掉该函数的const关键字呀！但问题是，我只想修改某个成员变量，其余成员变量仍然希望被const保护。



#### C++中枚举类型没有定义类型名称

例如：

```c++
enum {
    STATION_IDLE = 0,
    STATION_CONNECTING,
    STATION_WRONG_PASSWORD,
    STATION_NO_AP_FOUND,
    STATION_CONNECT_FAIL,
    STATION_GOT_IP
}；
```

没有定义类型名称的枚举类型，相当于使用\#define定义了名称以及值。

```c++
#define     STATION_IDLE = 0;
#define     STATION_CONNECTING = 1;
#define     STATION_WRONG_PASSWORD = 2;
#define     STATION_NO_AP_FOUND = 3;
#define     STATION_CONNECT_FAIL = 4;
#define     STATION_GOT_IP = 5;
```



#### C++中CHECK_EQ等函数的用法

  CHECK_EQ(x,y)<<"x!=y"，EQ即equation，意为“等于”，函数判断是否x等于y，当x!=y时，函数打印出x!=y。

  CHECK_NE(x,y)<<"x=y"，NE即not equation，意为“不等于”，函数判断是否x不等于y，当x=y时，函数打印出x=y。

  CHECK_LE(x,y) <<"x<=y",LE即lower equation,意为小于等于，函数判断是否x小于等于y。当x<=y时，函数打印x<=y。

  CHECK_LT(x,y)<<"x<=y",LT即为lower to ，意为小于，函数判断是否x小于y，当x<y时，函数打印x<y。

  CHECK_GE(x,y) <<"x>=y",GE即为great equation，意为大于。判断意义根据上述可推导出。

  CHECK_GT(x,y) <<"x>=y",同理如上。



#### C++中explicit关键字的用法

​	在C++中，我们有时候可以将构造函数用作自动类型转换函数。但是这种自动转换属性并非总是合乎要求的，有时会导致意外的类型转换。因此，C++新增了关键字explicit，用于关闭这种自动特性。即被explicit关键字修饰的类构造函数，不能进行自动地隐式类型转换，只能显式地尽心类型转换。

​	**注意：**只有一个参数的构造函数，或者构造参数有n个参数，但有n—1个参数提供了默认值，这样的情况才能进行类型转换。



#### C++中strlen(char *s)函数的作用

​	计算字符串s的长度，不包括“\0”文件结束符号



#### C++中的 struct stat sb的含义  

​	在c++中，struct stat sb这个结构是用来描述一个linux系统文件中的文件属性的结构。 stat 函数获取文件的所有相关信息，一般情况下，我们关心的文件大小和创建时间、访问时间、修改时间。

##### 	1、struct stat 结构体介绍

​	首先是用到struct stat结构体的函数原型：

```C++
int stat(const char *path, struct stat *buf);
int lstat(const char *path, struct stat *buf);
int fstat(int filedes, struct stat *buf);
```

​	三个函数返回关于文件的信息。前两个函数的第一个参数都是文件的全路径，第二个参数是struct stat的指针。返回值为0，表示成功执行。最后一个函数的第一个参数一个“文件描述符”，文件描述符是需要我们用open系统调用后才能得到的，而全文件路径直接写就可以了。

​	stat和lstat的区别：当文件是一个符号链接时，lstat返回的是该符号链接本身的信息；而stat返回的是该链接指向的文件的信息。（似乎有些晕吧，这样记，lstat比stat多了一个l，因此它是有本事处理符号链接文件的，因此当遇到符号链接文件时，lstat当然不会放过。而 stat系统调用没有这个本事，它只能对符号链接文件睁一只眼闭一只眼，直接去处理链接所指文件喽）

​	struc stat信息如下：

```C++
struct stat {

        mode_t     st_mode;       //文件对应的模式，文件，目录等
        ino_t      st_ino;       //inode节点号
        dev_t      st_dev;        //设备号码
        dev_t      st_rdev;       //特殊设备号码
        nlink_t    st_nlink;      //文件的连接数
        uid_t      st_uid;        //文件所有者
        gid_t      st_gid;        //文件所有者对应的组
        off_t      st_size;       //普通文件，对应的文件字节数
        time_t     st_atime;      //文件最后被访问的时间
        time_t     st_mtime;      //文件内容最后被修改的时间
        time_t     st_ctime;      //文件状态改变时间
        blksize_t st_blksize;    //文件内容对应的块大小
        blkcnt_t   st_blocks;     //文件内容对应的块数量
      };
```







#### C++const类型的引用参数

​	string类定义了一种char*到string的转换功能，这使得可以使用C-风格字符串来初始化string对象。
​	类型为const引用的形参其中一个属性表明：假设实参的参数类型与引用参数不匹配，但可以转换为引用类型，程序将创建一个正确类型的临时变量，使用转换后的实参值来初始化它，然后传递一个指向该临时变量的引用。
​	上面提到const引用为形参的属性，也就是说，如果引用的参数是const，则编译器在某些情况下会生成临时变量，比如下面这两种情况：
​	 实参类型不正确，但可以转换为正确类型。
​	 实参类型正确，但不是左值



#### RefBase类

​	Android中通过引用计数来实现智能指针，并且实现有强指针与弱指针。由对象本身来提供引用计数器，但是对象不会去维护引用计数器的值，而是由智能指针来管理。

​	要达到所有对象都可用引用计数器实现智能指针管理的目标，可以定义一个公共类，提供引用计数的方法，所有对象都去继承这个公共类，这样就可以实现所有对象都可以用引用计数来管理的目标，在Android中，这个公共类就是RefBase，同时还有一个简单版本LightRefBase。

​	RefBase作为公共基类提供了引用计数的方法，但是并不去维护引用计数的值，而是由两个智能指针来进行管理：sp(Strong Pointer)和wp(Weak Pointer)，代表强引用计数和弱引用计数。 


#### Mutex::AutoLock介绍(互斥类 Mutex)

​	（1）、Mutex是互斥类，用于多线程访问同一个资源的时候，保证一次只有一个线程能访问该资源。对于这种互斥有一个很形象的比喻：想象你在飞机上如厕，这时卫生间的信息牌上显示“有人”，你必须等里面的人出来后才可以进去。这就是互斥的含义。

​	关于Mutex的使用，除了初始化外，最重要的是lock和unlock函数的使用，它们的用法如下：
 要想独占卫生间，必须先调用Mutex的lock函数。这样，这个区域就被锁住了。如果这块区域之前已被别人锁住，lock函数则会等待，直到可以进入这块区域为止。系统保证一次只有一个线程能lock成功。
 当你“方便”完毕，记得调用Mutex的unlock以释放互斥区域。这样，其他人的lock才可以成功返回。
 	另外，Mutex还提供了一个trylock函数，该函数只是尝试去锁住该区域，使用者需要根据trylock的返回值来判断是否成功锁住了该区域。
​	注意　以上这些内容都和Raw API有关，不了解它的读者可自行学习相关知识。在Android系统中，多线程也是常见和重要的编程手段，务必请大家重视。Mutex类确实比Raw API方便好用，不过还是稍显麻烦。

​	（2）、AutoLock类是定义在Mutex类内部的一个类，它的使用比较简单。

​		使用Mutex的时候，要先显示调用Mutex的lock。然后不使用了之后再调用Mutex的unlock，这两个操作必须一一对应，否则会出现死锁。

​		AutoLock的使用还是很简单

​			先定义一个Mutex，如Mutex xlock，

​			在使用xlock的地方，定义一个AutoLock，如Mutex：AutoLock autoLock（xlock）

​			由于C++对象的构造方法和析构函数都是自动被调用的，所以在AutoLock的生命周期内，xlock的lock和unlock也就自动被调用了这样就省去了重复书写unlock的麻烦，而且lock和unlock的调用肯定是一一对应的，这样也绝对不会出错了。

```
class Autolock {
    public:
        //构造的时候调用lock。
        inline Autolock(Mutex& mutex) : mLock(mutex)  { mLock.lock(); }
        inline Autolock(Mutex* mutex) : mLock(*mutex) { mLock.lock(); }
        //析构的时候调用unlock。
        inline ~Autolock() { mLock.unlock(); }
    private:
        Mutex& mLock;
    };

```

