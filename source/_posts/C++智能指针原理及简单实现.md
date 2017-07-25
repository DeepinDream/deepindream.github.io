---
title: C++智能指针原理及简单实现
categories: C++
tags: C++
---

#### 指针是C++ 的一大利器，但是用得不好又容易引发灾难。幸而C++ 引入了智能指针，简直是我等手残患者的福音。

#### STL中有几种智能指针类型，包括auto_ptr（已弃用），share_ptr，unique_ptr，和weak_ptr。其中，后三者是C++11才加入到标准中的。三者有何不同，请看[维基百科：智能指针](https://www.wikiwand.com/zh-hans/%E6%99%BA%E8%83%BD%E6%8C%87%E9%92%88)。

## 裸指针（normal/raw/naked pointers）的问题

如果程序员不够细心，裸指针总能引发许多问题，比如指针所指向对象的生命周期，挂起引用和内存泄漏等。

如果一块内存被多个指针引用，其中一个指针释放了该内存空间而其他指针不知道，就发生了挂起引用。当从堆（heap）中申请了内存，使用完毕后忘记释放，就会引发内存泄漏。

## 什么是智能指针？

智能指针是一个RAII（Resource Acquisition is initialization）类模型，用来动态的分配内存。它提供所有普通指针提供的接口，却很少发生异常。在构造中，它分配内存，当离开作用域时，它会自动释放已分配的内存。这样的话，程序员就从手动管理动态内存的繁杂任务中解放出来了。

##### Tips： 上文链接：http://www.jianshu.com/p/e4919f1c3a28。

## C++ 智能指针使用

最简单的实现智能指针的方法是引用计数算法，实现简单，判定效率高。然而Java和C#都没有采用引用计数来进行内存管理，而是通过可达性分析来判定对象存活的。最主要的原因是它很难解决对象之间的相互循环引用的问题。

不谈其他，这里还是以引用计数来实现一个智能指针（share_ptr也是使用引用计数）。

智能指针的作用有如同指针，但会记录有多少个shared_ptrs共同指向一个对象。这便是所谓的引用计数。一旦最后一个这样的指针被销毁，也就是一旦某个对象的引用计数变为0，这个对象会被自动删除。

##### Tips: [share_ptr 介绍](http://zh.cppreference.com/w/cpp/memory/shared_ptr)

使用share_ptr创建一个动态分配内存的字符串对象：

```c++
#include <memory>
using std::share_ptr;
```

```c++
//新创建一个对象，引用计数器为1
shared_ptr<string> pstr(new string("abc"));
```

可以像操作普通指针一样调用string方法：

```c++
if (pstr && pstr->empty()) {
        *pstr = "hello";
}
```

当有另外一个智能指针对当前智能指针进行拷贝时，引用计数器加1：

```c++
shared_ptr<string> pstr(new string("abc")); //pstr指向的对象只有一个引用者
shared_ptr<string> pstr2(pstr); //pstr跟pstr2指向相同的对象，此对象有两个引用者
```

当两个智能指针进行赋值操作时，左边的指针指向的对象引用计数减1，右边的加1:

```c++
shared_ptr<string> pstr(new string("abc"));
shared_ptr<string> pstr2(new string("hello"));
pstr2 = pstr; //给pstr2赋值，令他指向另一个地址，递增pstr指向的对象的引用计数，递减pstr2原来指向的对象引用计数
```

指针离开作用域范围时，引用计数减1。当引用计数为0时，对象被回收。

## C++ 智能指针简单实现

```c++
template<typename T>
class SmartPrt{
public:
    SmartPtr(T*); // 使用普通指针初始化智能指针
    SmartPtr(SmartPtr&); // 拷贝构造函数
    
    T* operator->(); // 重写指针运算符
    T& operator*();  // 重写解引用运算符
    SmartPtr& operator=(SmartPrt&); // 重写赋值运算符
    
    ~SmartPrt();
    
private:
    int *count_; // 引用计数
    T *point_; // 底层指针
}
```

初始化时，将引用计数初始化为1：

```c++
template<typename T>
SmartPrt<T>::SmartPrt(T *p): point_(p), count_(new int(1))
{
}
```

定义拷贝构造函数，引用计数加1：

```c++
template<typename T>
SmartPtr<T>::SmartPtr(SmartPtr &sp): point_(sp.point_), count_(&(++*sp.count_))
{
}
```

定义指针运算符：

```c++
template<typename T>
T* SmartPtr<T>::operator->()
{
    return point_;
}
```

定义解引用运算符，直接返回指针的引用：

```c++
template<typename T>
T& SmartPtr<T>::operator*()
{
    return *point_;
}
```

定义赋值运算符，左边指针引用计数减1，右边指针引用计数加1，当左边引用计数为0时，释放内存：

```c++
template<typename T>
SmartPtr<T>& SmartPtr<T>::operator=(SmartPtr &sp)
{
    *sp.count_++;
    if(--*count_ == 0){
        delete count_;
        delete point_;
    }
    
    this->point_ = sp.point_;
    this->count_ = sp.count_;
    
    return *this;
}
```

定义析构函数：

```c++
template<typename T>
SmartPtr<T>::~SmartPtr()
{
    if(--*count_ == 0){
        delete count_;
        delete point_;
    }
}
```

Ok！接下来可以测试代码了：

```c++
SmartPtr<string> sptr(new string("abc"));
SmartPtr<string> sptr2(sptr);
SmartPtr<string> sptr3(new string("efg"));
sptr3 = sptr2;
```

## 总结

C++ 中的智能指针还是相当好用的，一定程度解放了程序员，可以不再拘泥于繁琐的内存管理之中。

嗯...且行且珍惜O(∩_∩)O