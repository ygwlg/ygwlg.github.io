---
title: c++数据类型
date: 2025-06-07 14:00:21
tags:
---

## 整型

short 2字节；int 4字节；long 4或8字节；long long 8字节 unsigned int 4字节

Q1: 现代计算机一般使用补码存储整型的原因：

+ 源码存储会面临+0和-0的问题，且加减法不方便运算；

+ 反码存储：正数不变，负数除符号位取反，依然有上述问题

+ 补码存储：正数不变，负数取反加一，解决上述问题

Q2: 为什么long的存储长度不统一？

取决于平台的体系结构，可通过sizeof函数查看

```c++
#include<iostream>
int main(){
    std::cout << sizeof(long);
}

// output: 8
```

## 浮点类型

float 4字节 double 8字节 long double 8或16字节（平台相关）

## 字符类型

char 1字节 ASCII字符

wchar_t 2/4字节，Unicode字符

char16_t/char32_t: 2/4字节，UTF-16和UTF-32编码

## 符合类型

数组

数组定义：

1. 静态数组

```c++
#include<iostream>
int main(){
    int array[] = {1,2,3,4,5};
    
    for (int i =0; i < 5; i++) {
        std::cout << array[i] << ' ';
    }

    int array2[5] = {1, 2, 3};
    for (int i = 0; i < 5; i++) {
        std::cout << array2[i] << ' ';
    }
}

```

2. 动态数组

```c++
#include<iostream>
int main() {
    int size=10;
    int* darr = new int[size];
    darr[0] = 100;
    for (int i=0; i<10; i++) {
        std::cout << darr[i] << ' ';
    }
}
```

## 指针

指针的本质是存放地址的变量，其值是分配的内存首个字节的地址

```c++
int *p;  //指针声明
int x = 10;
p = &x;  //指针赋值
*p = 20;  //通过指针修改地址值
```

```c++
// 在创建指针时初始化，否则可能导致程序崩溃
int *p = &x;
int *p = nullptr;
```

指针的算数运算：既然指针的含义是内存地址，那么可以通过指针来遍历数组中的值

```c++
// 一开始还以为要 + sizeof(int)，原来指针加法会自动计算sizeof
#include<iostream>
int main() {
    int size=10;
    int* darr = new int[size];
    darr[0] = 100;
    int *p = darr;
    for (int i=0; i<10; i++) {
        std::cout << *(p + i) << ' ';
    }
}
```
c++不给你保管内存，使用完了得自己释放，不然就会导致内存泄漏。
创建了不初始化导致野指针，释放了继续使用导致悬空指针

int* p和int *p没有区别。。。


## 引用
必须初始化，一旦绑定不可更改。修改引用即修改值。

```c++
#include<iostream>
int main() {
    int value = 10;
    int& ref = value;
    std::cout << value;
    ref = 20;
    std::cout << value;
}
```
引用本质上也是通过指针实现的，但是语法更安全，更直观。

这些和python的引用是啥关系：

1. c/cpp的指针是内存地址，py的引用是名称绑定关系

2. 对于不可变对象的操作，py会重新创建对象再绑定到新对象

## 结构体struct

```c++
#include <iostream>
#include <string>
struct Student{
    int id; 
    std::string name;
    bool gender;
};
int main() {

    Student a = {1, "aaa", false};
    std::cout << a.name;
}

```

## 联合体Union
和结构体的区别在于，联合体只维护一个成员值

```c++

```

## 枚举enum

```c++
enum Color {Red, Green, Yellow};
enum class Color2 {Red, Green, Yellow};
#include <iostream>
int main() {
    Color c1 = Red;
    Color2 c2 = Color2::Yellow;
    std::cout << c1 << c2;
}
```
Q1：为啥要多个class或者struct关键字？

为了解决不同枚举之间的重名问题。

Q2：为啥上述代码会报错？
enum class不支持std::cout。。。


## 容器类型

### vector