# 深入探索C++对象模型

[TOC]

## 第1章 关于对象

C++在布局以及存取时间上的主要额外负担有virtual引起的：

- 虚函数机制：用以支持一个有效率的”执行期绑定“
- 虚基类：用以实现“多次出现在继承体系中的base class，有一个单一而被共享的实例

### 1.1 C++对象模型

成员分类：

数据成员：static、non-static

成员函数：static、non-static、virtual

![image-20200412205149295](../../pics/image-20200412205149295.png)

每个类有一堆指向虚函数的指针，存放在表格中，这个表格称为**虚函数表**（vtbl），虚函数表的第一个slot中存放的是type_info object，用于动态绑定时的类型识别，支持runtime type identification，RTTI，通常存放在表格的第一个slot

该模型的优点：

- 空间和存取时间效率高

该模型的缺点：

- 类对象的非静态数据成员改变，需要重新编译

**成员，函数放置的位置**：

- 在类对象中：
  - 1.non-static数据对象在每个类对象中
  - 2.**虚函数指针（vptr）**：指向类的虚函数表，其设定和重置由类的构造函数，析构函数和拷贝赋值运算符自动完成
  - 3.**虚基指针（bptr）**：如果有虚继承的话，在**3.4**节深入讨论
- 在对象之外：
  - 1.虚函数表
  - 2.虚基类表
  - 非虚函数：包括static，non-static
  - 静态数据成员

<img src="../../pics/image-20200412210125191.png" alt="image-20200412210125191" style="zoom:67%;" />

在**虚拟继承**情况下，base class不管在继承串链中被派生多少次，永远只会存在一个实例；但是在普通继承下，则会存在**多个实例**

- 优点：
  - 每一个class object中对于继承都有一致的表现方式：每个class object都应该在固定的位置上安放一个base table指针，与base classes的大小和个数无关
  - 无需更改class objects本身，就可以放大缩小，或更改base class table

### 1.3 对象的差异

C++的多态只存在于一个个的public class继承体系中，以以下方法支持多态：

- 1.经由一组隐式的转化操作：把一个派生类的指针转化为一个指向public继承的基类指针
  - `shape *ps = new circle();`
- 2.经由虚函数机制
  - `ps->rotate();`
- 3.经由dynamic_cast和typeid运算符
  - `if( circle *pc = dynamic_cast<circle*>( ps ) )...`

多态的**主要用途**：经由一个共同的接口来影响类的封装，这个接口通常被定义在一个抽象的base class中

**sizeof一个类对象时需要考虑的部分：**

- 1.其non-static数据成员的总和
- 2.对齐导致填补的空间
- 3.支持虚函数和虚基类的指针

**类对象的布局：**

```c++
class ZooAnimal{
public:
    ZooAnimal();
    virtual ~ZooAnimal();
    virtual void rotate();
    
protected:
    int loc;
    string name;
};

ZooAnimal za( "Zoey" );
ZooAnimal *pza = &za;
```

za和pza的可能布局（注意string类型，string所存字符串长度与sizeof(对象)无关）：

![](../../pics/Pic_1_4_%E7%8B%AC%E7%AB%8Bclass%E7%9A%84object%E5%B8%83%E5%B1%80%E5%92%8Cpointer%E5%B8%83%E5%B1%80.png)

如果string的布局是按图所示，则==sizeof(ZooAnimal)==16字节（4+8+4），假定string是传统的8byte

- 但自己VS上测试的结果不一致
  - sizeof(stirng)==28
  - sizeof(za)==sizeof(ZooAnimal)==28+4+4==36
  - string vector的sizeof与个数无关，为固定大小，具体由编译器决定
- 转换（cast）其实是一种编译器指令，大部分情况下并不改变一个指针所含的真正地址，它只影响“被指出内存的大小和其内容”的解释方式

#### 1.3.1 指针的类型

“指针类型”的作用：告诉编译器如何解释某个特定地址中的内存内容及其大小

#### 1.3.2 加上多态之后

```c++
class bear : public ZoonAnimal {
public:
    Bear();
    ~Bear();
    void rotate();
    virtual void dance();
protected:
    enum Dance {...};
    
    Dance dances_know;
    int cell_block;
};

Bear b("Yogi");
Bear *pd = &b;
Bear &rb = *pb;
```

- 注意enum类型的布局

![image-20200412214513211](../../pics/image-20200412214513211.png)

```c++
Bear b;
ZooAnimal *pz = &b;
Bear *pb = &b;
```

**pz与pb的区别是：**

- 它们都指向Bear object的第一个byte

- pb所涵盖的地址包含整个Bear对象
- pz所涵盖的地址只包含Bear对象中的ZooAnimal子对象
- **不能通过pz来处理Bear的任何members，唯一的例外是通过虚函数**

