# 复制与交换 (Copy-and-swap)
##目的
创建对异常安全的赋值操作符重载。

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

##已知的应用

##相关的惯用法

##参考
