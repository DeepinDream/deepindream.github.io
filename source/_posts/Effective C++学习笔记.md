---
title: Effective C++学习笔记
categories: C++
tags: C++
---

# 1. 基础部分

## 1.1 条款1： 视C++为一个语言联邦
C++语言的四个层次：

- C。没有C++的面向对象，没有模板，没有异常，没有重载等。
- Object-Oriented C++。也就是C with Classes。classes、封装、继承、多态、虚函数。是面向对象的特性。
- Template C++。C++的泛型编程部分, 也就是所谓的模板元编程。
- STL。STL是个template程序库。它对容器、迭代器、算法及函数对象的规约，并且是以templates及程序库的方式构建出来。

每个层次应该有自己的最佳实践。例如对于C层次，传入函数最佳的实践应该是传入值，而不是指针，而对于C with classes层次，则以传递引用为最佳的实践。

## 1.2 条款2：尽量以const、enum、inline替换define
- 对于全局的值，使用define定义，在预处理的时候会被替换成相应的值。宏是全局的，面向对象程序设计中破坏了封装。因此在C++中尽量避免它！

由于预处理器会直接替换的原因，宏定义最好用括号括起来。#define函数将会产生出乎意料的结果：
```c++
#define CALL_WITH_MAX(a, b) f((a) > (b) ? (a) : (b))
int a = 5, b = 0;
CALL_WITH_MAX(++a, b); //a被累加两次
CALL_WITH_MAX(++a, b+10); //a被累加一次
```
可以使用模板函数来替代：
```c++
template<typename T>
inline void callWithMax(const T &a, const T &b)
{
  f(a > b ? a : b);
}
```

## 1.3 条款3：尽可能使用const
这是防卫型（defensive）程序设计的原则， 尽量使用const，从而防止客户错误地使用你的代码。也可直观地让客户得知参数不会被修改。

const与指针的各种用法：
```c++
char greeting[] = "Hello";
 
char *p = greeting;                    // non-const pointer, non-const data
const char *p = greeting;              // non-const pointer, const data
char * const p = greeting;             // const pointer, non-const data
const char * const p = greeting;       // const pointer, const data 
```

## 1.4 条款4：确定对象被使用前已被初始化
- 对于内置类型（int，bool，float等），定义的时候一定要初始化，未初始化的值是undefined的。
- 对于非内置类型，需要在构造函数对每一个成员进行初始化。

初始化与赋值是不同的，初始化和赋值对内置类型的成员从效果上没有什么大的区别。对非内置类型成员变量，为了避免两次构造，推荐使用类构造函数初始化列表。const常量则必须在初始化列表中进行初始化：
```c++
Student::Student(const std::string& name, const std::string& address)
    :m_name(name)   // 初始化，效率高
{
    m_address = address;    // 赋值
}
```

# 2. 构造/析构/赋值运算
## 2.1 条款5：了解C++默默编写并调用哪些函数

- 如果没有自己编写构造函数，C++会生成一个不带参数的默认构造函数。

- 在非特殊情况下，C++会自动生成拷贝构造函数、赋值运算符以及析构函数

特殊情况指的是：如果类中有引用类型或者有const类型，此时由于引用类型和const类型不能重新赋值，所以编译器这个时候不会自动生成赋值运算符和拷贝构造函数。

四大函数的调用时机：
>- 构造函数：对象定义；使用其他兼容的类型初始化对象时（可使用 explicit 来避免这种情况）
>- 复制构造函数：用一个对象来初始化另一对象时；传入对象参数时；返回对象时；
>- 析构函数：作用域结束（包括函数返回）时；delete
>- =运算符：一个对象赋值给另一对象

```c++
Empty e1;               // 默认构造函数
Empty e2(e1);           // 拷贝构造函数
Empty e3 = e1;          // 拷贝构造函数
e2 = e1;                // = 运算符
 
void func(Empty e){     // 拷贝构造函数，拷贝一份参数对象
    return e;           // 拷贝构造函数，拷贝一份返回对象
                        // 析构函数，拷贝得到的参数对象被析构
}
 
e2 = func(e1);          // = 运算符
```

## 2.2 条款6：若不想使用编译器自动生成的函数，就该明确拒绝

通常在单例中使用会较多，单例不允许发生外部构造、拷贝和=操作。禁用方法是把函数声明为private的，这样，若尝试从外部调用拷贝函数，在编译就会报错。在c++11中，引入了delete关键字，更加严格的限制拷贝函数的生成。

```c++
class Uncopyable
{
protected:
    ~Uncopyable() {}
private:
    Uncopyable() {}
    Uncopyable(const Uncopyable &) = delete;
    Uncopyable &operator=(const Uncopyable) = delete;
};
```

## 2.3 条款7：为多态基类声明virtual析构函数

将基类的析构函数声明为virtual，目的在于基类指针调用析构函数时能够正确地析构子类部分的内存。 否则只会析构基类部分，子类部分的内存将会泄漏。

这里又可以引申出虚函数表指针的相关知识，参考链接：<https://jocent.me/2017/08/07/virtual-table.html>。

```c++
class Base
{
public:
    virtual ~Base();
    ...
};
class Derived: public Base
{
    ...
};
Base *p = new Derived();
delete p;
```

## 2.4 条款8：别让异常逃离析构函数

不要在析构中抛出异常，由于析构函数常常被自动调用，在析构函数中抛出的异常往往会难以捕获，引发程序非正常退出或未定义行为。

只需为不安全的语句提供一个新的函数，供客户调用和处理异常。

```c++
class DBConn{
public:
    ~DBConn{
        if(!closed){
            try{
                db.close(); // 假设这里发生异常
            }
            catch(...){
              cerr<<"数据库关闭失败"<<endl;
              // 吞掉异常，或者直接退出程序，但是客户也无法得知发生什么异常
                // std::abort();
            }
        }
    }

    void close(){   // 提供一个新的close函数，供客户调用和处理异常
        db.close();
    }

private:
    DBConnection db;
};
```

##2.5 条款9：不要在构造函数和析构函数中调用虚函数

- 在构造函数中，调用构造函数的顺序是基类->子类，当基类在构造的时候，子类的部分还没有开始构造，这时候，如果调用虚函数，只会调用基类版本的，不符合虚函数的语义。

- 在析构函数中，调用析构函数的顺序是子类->基类，当基类在析构的时候，子类的部分已经析构完成，这时候，如果调用虚函数，同样只会调用基类版本的，不符合虚函数的语义。

## 2.6 条款10：令operator=返回一直refrence to *this

用来支持链式的赋值语句：
```c++
int x, y, z;
x = y = z = 1;
```

同理，operator+=等也应该返回refrence to *this

## 2.7 条款11：在operator=中处理“自我赋值”

考虑到<strong>自赋值安全</strong>和<strong>异常安全</strong>。一个更加通用的技术便是复制和交换（copy and swap）

copy and swap策略: <https://www.bbsmax.com/A/x9J2o4nN56/>
```c++
class String{
    char * str;
public:
    String& String::operator=(String rhs){  // 传值而不是传引用
        swap(rhs);                // swap *this's data with
        return *this;             // the copy's
    }

    void swap(String & s) throw() // Non-throwing swap的实现
    {
        std::swap(this->str, s.str);
    }
}
```

## 2.8 条款12：复制对象时勿忘其每一个成分

实现拷贝函数时：
- 完整复制当前对象的数据（local data）
- 调用所有父类中对应的拷贝函数

不要让复制构造函数和赋值运算符相互调用，它们的语义完全不同。若不想代码重复，可以抽象到一个普通的方法中，比如init()。

# 3. 资源管理
## 3.1 条款13：使用对象来管理资源

比如以下代码：
```c++
void function()
{
  Widget *w = new Widget();
  if (xxx)
  {
    return;
  }
  delete w;
}
```

function 在退出时，需要手动释放 w 的资源，很难避免某些时候忘记释放。

可以使用 RAII ( Resource Acquisition Is Initialization ) —— 资源获取即是初始化，简单的说：
- 在构造时获取资源
- 在析构时释放资源

```c++
class Resource{};
class RAII{
public:
    RAII():r_(new Resource){} //获取资源
    ~RAII() {delete r_;} //释放资源
    Resource* get()    {return r_ ;} //访问资源
private:
    Resource* r_;
};
```

或者使用智能指针，无需手动管理资源，std::shared_ptr 本质也是一个RAII。

## 3.2 条款14：资源管理类要特别注意拷贝行为

使用 RAII 时，要特别注意 RAII 对象的拷贝行为，比如一个 RAII 的互斥锁：

```c++
class Lock {
public:
    explicit Lock(Mutex *pm):mutexPtr(pm){
        lock(mutexPtr);
    }
    ~Lock(){ unlock(mutexPtr); }
private:
    Mutex *mutexPtr;
};
```

该互斥锁的使用方法：

```c++
Mutex m;            // 定义互斥锁
{                   // 创建代码块，来定义一个临界区
    Lock m1(&m);    // 互斥锁加锁
    ...             // 临界区操作
}                   // m1退出作用域时被析构，互斥锁自动解锁
```

当 m1 被拷贝时，可能会发生死锁的情况，此时应当禁用 Lock 类的拷贝行为，详细参考条款6。

## 3.3 条款15：资源管理类需要提供对原始资源的访问

可参考 std::shared_ptr 的实现中，提供的 get() 方法，用于访问原始指针。同时也提供了operator->、operator*，让智能指针的表现与直接使用原始指针是一样的。

## 3.4 条款16： 使用new和delete时采取相同形式

一句话：当你使用 new 来申请内存时，要使用 delete 来释放内存，当你使用 new[] 来申请内存时，要使用 delete[] 来释放内存。

```c++
int* p = new int[2]{11, 22};
printf("%d, %d", *p, *(p+1));
delete[] p;
```
## 3.5 条款17：在单独的语句中将new的对象放入智能指针

考虑以下代码：

```c++
processWidget(shared_ptr<Widget>(new Widget), priority());
```

以上代码包含三个过程：
> 1. 执行 new Widget
> 2. 调用 priority()
> 3. 构造对象 shared_ptr<Widget>()

由于 c++ 编译器的不同，函数参数的调用顺序是不确定的。
若此函数的调用顺序如上述一致，并且在 2 中发生了异常，那么 1 中 new 的对象还没有加入到 shared_ptr 中，会发生内存泄漏。

正确做法：

```c++
std::shared_ptr<Widget> pWidget(new Widget);
processWidget(pWidget, priority());
```