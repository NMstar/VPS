---
title: C++ 沉思录阅读笔记
date: 2018/4/20
tags: [C++]
---

作者: [美]Andrew Konnig  Barbara Moo

### 序幕

    在C中扩展代码要比C++中难，因为在状态转移的过程中需要很多状态，C需要把这些状态保存在局部，C++可以直接保存到类里。

    C++类将动作和状态绑定在一起，新增代码会更加方便。

    C++在处理字符串时会更加方便。

### 为什么使用C++

    大型项目中的人更多地花时间在会议、沟通、处理规范等编程之外的事情。

    C++和C可以很好的共存。

### 类和继承

    继承是一种抽象，它允许程序员在某些时候忽略相似对象间的差异，又在其他时候利用这些差异。

### 类设计的核查表

#### 数据成员是否是私有的？
用户可能直接修改成员
``` c++
template<class T> class Vector {
public:
    int length; //公有成员数据
};
```
使用函数来避免用户修改长度
``` c++
template<class T> class Vector {
public:
    int length() const;
};
```
通过引用只允许用户进行读取访问
``` c++ 
template<class T> class Vector {
public: 
    // 每个构造函数都将length绑定到true_length上
    const int& length;
    ... 
private:
    int true_length;
};
```

#### 是否需要无参的构造函数？
``` c++
class Point {
public:
    Point(int p,int q): x(p), y(q) {}
    ...
private:
    int x, y;
};
```
上述这种情况下有两种情况是非法的:<br>
1. ` Point P;`
2. ` Point pa[100];`

#### 类需要析构函数吗？
如果构造函数里有 `new` 表达式，一般在析构函数中需要包含 `delete` 表达式。

#### 类需要虚析构函数吗？
基类有可能需要虚析构函数
``` c++
class a {
public:
    virtual ~a() {
         cout<<"delete a"<<endl;
    };
};

class b : public a {
   ~b() {
       cout<<"delete b"<<endl;
   };
};


int main() {
    a *pa = new b;
    delete pa;
    return 0;
}
```
运行结果:<br>
delete b<br>
delete a<br>

删去virtual:<br>
delete a<br>

我们实际上需要调用 ` class b` 的构造函数，如果不适用virtual，会被基类的构造函数给覆盖掉。(virtual设计的意义就是如此)

#### 类需要复制构造函数吗？
如果类在构造函数内分配资源，可能需要显式的复制函数来管理资源。
``` c++
class String {
public:
    String();
    String(const char* s);
private:
    char *data;
};
```
复制 `String` 后两个对象的 `data` 成员指向同样的内存，两个对象进行操作会相互影响，对象销毁时，这块内存会被释放两次。

如果不想用户能够复制类的对象，就声明复制构造函数为私有的。
``` c++
class Thing {
public:
    // ...
private:
    Thing(const Thing&);
    Thing& operator=(const Thing&);
};
```

#### 赋值操作符的错误操作
```c++
class String {
public: 
    String& operator=(const String& s);
    ...
private:
    char *data;
}

// 很明显但是不正确的实现
// 将一个String对象赋给它本身，这个方法就会失败。因为 s == *this，所以 data 和 s.data 是一样的。
String& String::operator=(const String& s) {
    delete [] data;
    data = new char[strlen(s.data)+1];
    strcpy(data,s.data);
    return *this;
}

// 实现方法一:
String& String::operator=(const String& s) {
    if(&s != this) {
        delete [] data;
        data = new char[strlen(s.data)+1];
        strcpy(data,s.data);
    }
    return *this;
}

// 实现方法二:
String& String::operator=(const String& s) {
    char * newdata = new char[strlen(s.data)+1];
    strcpy(newdata, s.data);
    delete [] data;
    data = newdata;
    return *this;
}

```
#### 删除数组记得使用 delete[]
#### 适当地声明成员函数为const

### C++ 基础知识

类成员函数定义为const时，不允许修改类的数据成员