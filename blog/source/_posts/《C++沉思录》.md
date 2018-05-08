---
title: C++ 沉思录阅读笔记
date: 2018/5/4
tags: [C++]
---

作者: [美]Andrew Konnig  Barbara Moo

## 序幕

    在C中扩展代码要比C++中难，因为在状态转移的过程中需要很多状态，C需要把这些状态保存在局部，C++可以直接保存到类里。

    C++类将动作和状态绑定在一起，新增代码会更加方便。

    C++在处理字符串时会更加方便。

### 为什么使用C++

    大型项目中的人更多地花时间在会议、沟通、处理规范等编程之外的事情。

    C++和C可以很好的共存。

### 类和继承

    继承是一种抽象，它允许程序员在某些时候忽略相似对象间的差异，又在其他时候利用这些差异。

## 第四章 类设计的核查表

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

## 第五章 代理类
代理类的好处是不用显式来管理内存
#### Vehicle基类定义
``` c++
class Vehicle {
public:
    virtual double weight() const = 0;
    virtual void start() = 0;
    // ...
};
class RoadVehicle: public Vehicle {/* .. */};
class AutoVehicle: public RoadVehicle {/* .. */};
class Aircraft: public Vehicle {/* .. */};
class Helicopter: public Aircraft {/* .. */};
```
目的是跟踪处理一系列不同的 `Vehicle` 
``` c++
    // 错误的，因为Vehicle是一个抽象基类
    Vehicle parking_lot[1000];
```
#### 经典做法
``` c++
    Vehicle* parking_lot[1000]; // 指针数组
    // 输入
    Automobile x = /* ... */;
    parking_lot[num_vehicles++] = &x;
    // 这个做法的问题：
    // 如果x变量没有了，parking_lot的指针就不知道指向什么内容了。
```
#### 变通的做法
``` c++
    Automobile x = /* ... */;
    parking_lot[num_vehicles++] = new Automobile(x);
    // 这个做法的问题：
    // 1. 需要动态管理内存
    // 2. 只能绑定静态的对象

```
#### 虚复制函数
``` c++
// 想要能够复制任何类型的Vehicle，需要给Vehicle类中添加一个合适的虚函数
    virtual Vehicle* copy() cosnt = 0;

// 如果Truck继承自类Vehicle,那么它的copy函数就类似于：
Vehicle* Truck::copy() const {
    return new Truck(*this);
};
```
#### 定义代理类
``` c++
class VehicleSurrogate {
public:
    VehicleSurrogate();
    VehicleSurrogate(const Vehicle&);
    ~VehicleSurrogate();
    VehicleSurrogate(const VehicleSurrogate&);
    VehicleSurrogate& operator=(const VehicleSurrogate&);
private:
    Vehicle* vp;
};

VehicleSurrogate::VehicleSurrogate(): vp(0) { }
VehicleSurrogate::VehicleSurrogate(const Vehicle& v): vp(v.copy()) {}
VehicleSurrogate::~VehicleSurrogate() {
    delete vp;
}
VehicleSurrogate::VehicleSurrogate(const VehicleSurrogate& v): vp(v.vp? v.vp->copy(): 0) { }
VehicleSurrogate::operator=(const VehicleSurrogate& v) {
    if(this != &v) {
        delete vp;
        vp = (v.vp ? v.vp->copy() : 0);
    }
    return *this;
}
```
令代理类支持类 `Vehicle` 所能支持的操作
``` c++
class VehicleSurrogate {
public:
    VehicleSurrogate();
    VehicleSurrogate(const Vehicle&);
    ~VehicleSurrogate();
    VehicleSurrogate(const VehicleSurrogate&);
    VehicleSurrogate& operator=(const VehicleSurrogate&);
    // 来自Vehicle的操作
    double weight() const;
    void start();
private:
    Vehicle* vp;
};

double VehicleSurrogate::weight() const {
    if(vp == 0) 
        throw "empty weigth";
    return vp->weight();
}

void VehicleSurrogate::start() {
    if(vp == 0)
        throw "empty start";
    vp->start();
}
```
代理类的做法
``` c++
    VehicleSurrogate parking_lot[1000];
    Automobile x;
    parking_lot[num_vehicles++] = x;
    // 最后一句等价于
    parking_lot[num_vehicles++] = VehicleSurrogate(X);
```

## 第六章 句柄：第一部分
#### 引用计数型句柄
``` c++
class Point {
public:
    Point(): xval(0), yval(0) { }
    Point(int x, int y): xval(x), yval(y) { }
    int x() const { return xval; }
    int y() const { return yval; }
    Point& x(int xv) { xval = xv; return *this; }
    Point& y(int yv) { yval = yv; return *this; }
private:
    int xval, yval;
};

class UPoint {
    // 所有成员都是私有的
    friend class Handle
    Point p;
    // 引用计数标志
    int u;

    UPoint()： u(1) {}
    UPoint(int x, int y): p(x, y), u(1) {}
    UPoint(const Point &p0): p(p0), u(1) {}
};

class Handle {
public:
    Handle();
    Handle(int, int);
    Handle(const Point&);
    Handle(const Handle&);
    Handle& operator=(const Handle&);
    ~Handle();
    int x() const;
    Handle& x(int);
    int y() const;
    Handle& y(int);
private:
    UPoint* up;
};

Handle::Handle(): up(new UPoint()) {}

Handle::Handle(int x, int y): up(new UPoint(x,y)) {}

Handle::Handle(const Point& p): up(new UPoint(p)) {}

// 析构函数,在删除最后一个引用的时候删除up
Handle::~Handle() {
    if(--up->u == 0) 
        delete up;
}

// 复制构造函数
Handle::Handle(const Handle& h): up(h.up) { ++up->u; }

// 赋值操作符
Handle& Handle::operator=(const Handle& h) {
    ++h.up->u;
    if(--up->u == 0) 
        delete up;
    up = h.up;
    return *this;
}

// 读取操作
int Handle::x() const {
    return up->p.x();
}

int Handle::y() const {
    return up->p.y();
}
```
#### 写时复制 (copy on write)
``` c++
    Handle h(3,4);
    Handle h2 = h;
    h2.x(5);
    int n = h.x();`
```
在指针的语义下，`n == 5`。<br>
在值的语义下，`n == 3`。<br>

有时候我们需要值的语义，所以产生了写时复制(copy on write)的技术，优点是只有在绝对必要的时候才进行复制，从而避免了不必要的复制。

1. 指针语义
``` c++
Handle& Handle::x(int x0) {
    up->p.x(x0);
    return *this;
}

Handle& Handle::y(int y0) {
    up->p.y(y0);
    return *this;
}
``` 
2. 值语义
``` c++
Handle& Handle::x(int x0) {
    if (up->u != 1) {
        --up->u;
        up = new UPoint(up->p);
    }
    up->p.x(x0);
    return *this;
}

Handle& Handle::y(int y0) {
    if (up->u != 1) {
        --up->u;
        up = new UPoint(up->p);
    }
    up->p.y(y0);
    return *this;
}

// 可以看到上述函数有一部分是重复的，而且会在任何改变UPoint对象的成员函数中重复。所以可以写一个private函数。

// 恕我起不出好名字 
void Handle::OneCounter() {
    if (up->u != 1) {
        --up->u;
        up = new UPoint(up->p);
    }
}
``` 
``` c++
// 在值语义下：
    p.x(42);
// 相当于
    p = Point(42,p.y());
// 区别是: 后者在有很多数据成员的情况下会产生更大代价
```

### C++ 基础知识

类成员函数定义为const时，不允许修改类的数据成员

构造函数的调用细节:

``` c++
#include <iostream> 
using namespace std;

class Test {
public:
    Test();
    Test(const Test& t);
    Test& operator=(const Test& t);

private:
    int t1;
};

Test::Test() {
    cout<<"调用构造函数"<<endl;
}

Test::Test(const Test& t) {
    cout<<"调用复制构造函数"<<endl;
}

Test& Test::operator =(const Test& t) {
    cout<<"调用赋值构造函数"<<endl;
    t1 = t.t1;
    return *this;
}

int main() {
    Test t1;
    Test t2 = t1;
    Test t3;
    t3 = t1;
    return 0;
}

/* 运行结果：
    调用构造函数
    调用复制构造函数
    调用构造函数
    调用赋值构造函数 
*/
```