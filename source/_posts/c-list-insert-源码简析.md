---
layout: '[layout]'
title: C++ list insert源码简析
tags: 数据结构
categories: C++
date: 2017-07-27 22:46:31
---


### stl::list是一个双向链表，list本身和list节点是不同的结构，stl::lilst中的节点（node）结构如下：

##### Tips：以下代码摘自《STL源码剖析--侯捷》

```c++
template <class T>
struct __list_node {
    typedef void* void_pointer;
    void_pointer prev; // 型别为void*。其实可设为__list_node<T>*
    void_pointer next;
    T data;
}
```

### list::insert() 是一个重载函数，其中最简单的一种如下。首先配置并构造一个节点，然后在尾端进行适当的指针操作，将新节点插入进去:

```c++
iterator insert(iterator position, const T& x)
{
    linke_type tmp = creat_node(x); // 产生一个节点（设内容为x） 
    
    // 调整双向指针，使tmp插入进去
    tmp->next = position.node;
    tmp->prev = position.node->prev; // 与当前节点的上一节点建立连接
    (link_type(position.node->prev))->next = tmp; // 将tmp插入到上一节点之后
    position.node->prev = tmp; // 被插入的位置往后移动
    return tmp;
}
```

```c++
typedef __list_node<T>* linke_type // from __list_iterator
```

```c++
// 产生（配置并构造）一个节点，
link_type create_node(const T& x)
{
    link_type p = get_node();
    construct(&p->data, x); // 全局函数， 构造/析构基本工具
}
```

```c++
// 配置一个节点并传回
link_type get_node()
{
    return list_node_allocator::allocate();
}
```

### stl::push_back()调用insert()将元素插入于list尾端：

```c++
void push_back(const T& x)
{
    insert(end(), x);
}
```