# 复制与交换 (Copy-and-swap)
##目的
创建对异常安全的赋值运算符重载。

##别名
Create-Temporary-and-Swap

##动机
异常安全(Exception safety)是构建高度安全的C++软件的重要基石，它使用异常(exceptions)来表示不正常的的事件。至少有三种异常安全性级别:基本(basic)、强(strong)和no-throw。一般使用基本异常，毕间实现简单。强异常安全在一些情况下并不适用。本惯用法就是为了让赋值操作值能够以强异常安全的方式实现。

    关于异常安全性的三种类型 (《Effective C++》中Item 29):
    * 基本承诺
    如果异常抛出，程序内的任何事物仍然保持在有效状态下。没有任何对象或数据结构会因此而败坏，所有对象都处于一种内部前后一致的状态。但是程序的状态可能无法预料。
    * 强烈保证
    如果异常抛出，程序状态不改变。调用这样的函数需有这样的认知:如果函数成功，就是完全成功，如果函数失败，程序会恢复到调用函数之前的状态。
    * 不抛掷保证
    承诺绝不抛出异常，因为它们总是能够完成它们原先承诺的功能。作用于内置类型身上的所有操作都提供nothrow保证。


##解决方案及示例
使用本惯用法会在丢掉当前资源之前取得新资源。其中使用RAII惯用法获取资源，如果成功获取资源，再使用non-throwing swap idiom(无异常swap惯用法)。最终，旧资源在RAII的副作用下被释放掉。
```
class String
{
    char * str;
public:
    String & operator=(const String & s)
    {
        String temp(s); // 拷贝建构 -- RAII
        temp.swap(*this); // 无异常swap

        return *this;
    } // 在temp析构时，旧资源会被释放。
    void swap(String & s) throw() // 无异常swap的实现
    {
        std::swap(this->str, s.str);
    }
};
```

还有一些上面这个实现的变种。自赋值操作虽然不是强制需要的，但它在自赋值的情况下带来性能提升(虽然很少发生)。
```
class String
{
    char * str;
public:
    String & operator=(const String & s)
    {
        if (this != &s)
        {
            String(s).swap(*this); //拷贝建构，以及无异常的swap
        }

        // 在temp析构时，旧资源会被释放。
        return *this;
    }
    void swap(String & s) throw() // 无异常swap的实现
    {
        std::swap(this->str, s.str);
    }
};
```

**拷贝省略(copy ellsion)和copy-and-swap惯用法**
严格来说，在赋值操作中显示地创建临时对象是不需要的。赋值操作的右参形式的参数可以通过以值的形式传递到函数。这个参数本身就是临时变量。
```
String & operator = (String s) // 以值传递的参数s就是临时变量
{
   s.swap (*this); // 无异常swap
   return *this;
} // s的析构函数被调用时，旧资源会被释放。
```
不是为了方便才这么做的，因为它是实实在在的优化。如果参数绑定到左值(另一个非常量对象)，就会在创建参数时自动创建一个复本。如果绑定到右值(临时变量，常量)，这个复制会被忽略掉（拷贝省略），这样就节省了拷贝建构和析构的开销。在赋值运算符最先的版本里，接受的是常量引用（右值引用），不会发现拷贝省略，仍然会需要创建和释放额外的对象。

在C++11里，这类赋值运算符也被称为统一赋值运算符(unifying assignment operator),因为它免去写两个不同赋值运算符的麻烦:拷贝赋值运算符和move赋值运算符。只要一个类有move建构函数，C++11编译器通常会用它来优化从另一个临时变量拷贝的方式。非C++11编译器可以利用拷贝省略(Copy-elision)达到和C++11编译器相同的优化效果。
```
String createString(); // 一个返加String对象的函数。
String s;
s = createString();
// 右手边是一个右值。以值传递赋值运算符的赋值运算符比以常量引用传递的方式会更为有效
```

也不是所有的类都能从这样的赋值运算符上受益。考虑一个字串的赋值运算符，字串内部持有一个右值字串对象的复本，当内存不足时，它会释放旧的内存再分配的内存。如果应用之前讨论的优化，就必须写一个自定义的赋值运算符。因为新字串的拷贝会使得之前内存分配优化失效，这个自定义的赋值运算符就必须避免从一个临时字串对象拷贝，而且要接受常量引用的参数。

##已知的应用

##相关的惯用法
* RAII (Resource Acquisition Is Initialization)
* Not-throwing swap

##参考
