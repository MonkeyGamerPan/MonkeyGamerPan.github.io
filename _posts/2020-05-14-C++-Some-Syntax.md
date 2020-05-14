---
layout: post
title: "C++各中知识点"
featured-img: shane-rounce-205187
categories: [C++]
---



# C++中的各种变量或者关键字以及符号的用法

#### 1、C++中size_t的用法

 https://blog.csdn.net/chaoruizhe123/article/details/44589811/

​	

#### 2、C++中strlen(char *s)函数的作用

​	计算字符串s的长度，不包括“\0”文件结束符号



#### 3、C++中的status_t的含义；   

​	在Errors.h中定义的：// use this type to return error codestypedef int32_t status_t;



#### 4、C++const类型的引用参数

​	string类定义了一种char*到string的转换功能，这使得可以使用C-风格字符串来初始化string对象。
​	类型为const引用的形参其中一个属性表明：假设实参的参数类型与引用参数不匹配，但可以转换为引用类型，程序将创建一个正确类型的临时变量，使用转换后的实参值来初始化它，然后传递一个指向该临时变量的引用。
​	上面提到const引用为形参的属性，也就是说，如果引用的参数是const，则编译器在某些情况下会生成临时变量，比如下面这两种情况：
​	 实参类型不正确，但可以转换为正确类型。
​	 实参类型正确，但不是左值



#### 5、RefBase类

​	Android中通过引用计数来实现智能指针，并且实现有强指针与弱指针。由对象本身来提供引用计数器，但是对象不会去维护引用计数器的值，而是由智能指针来管理。

​	要达到所有对象都可用引用计数器实现智能指针管理的目标，可以定义一个公共类，提供引用计数的方法，所有对象都去继承这个公共类，这样就可以实现所有对象都可以用引用计数来管理的目标，在Android中，这个公共类就是RefBase，同时还有一个简单版本LightRefBase。

​	RefBase作为公共基类提供了引用计数的方法，但是并不去维护引用计数的值，而是由两个智能指针来进行管理：sp(Strong Pointer)和wp(Weak Pointer)，代表强引用计数和弱引用计数。 


#### 6、Mutex::AutoLock介绍(互斥类 Mutex)

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

