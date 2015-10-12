# 取址 (Address Of)
##目的
针对一个重载了&运算符的类，取出其对象的地址。

##别名

##动机
C++允许一个类重载&运算符，并且返回值也未必是对象的地址。这样的类本身就是有问题的，但没办法，语言支持这样做。

取址(Address-of)惯用法就是无论是否重载了&运算符，以及它采用什么样的访问限制（access protection)，都可以正确地返回对象的真实地址。

下面的例子中，main函数会编译失败，首先是&运算符是私有的，即便把它改为公有的，但也无法从它返回的double型转为指针类型。
```
class nonaddressable
{
public:
    typedef double useless_type;
private:
    useless_type operator&() const;
};

int main()
{
  nonaddressable na;
  nonaddressable * naptr = &na; // 这里出现编译错误。
}
```

##解决方案及示例
取址(Address-of)惯用法可以通过一组的转换操作获取地对象地址:
```
template <class T>
T * addressof(T & v)
{
  return reinterpret_cast<T *>(& const_cast<char&>(reinterpret_cast<const volatile char &>(v)));
}
int main()
{
  nonaddressable na;
  nonaddressable * naptr = addressof(na); // No more compiler error.
}
```
##已知的应用
* [Boost addressof utility](http://www.boost.org/doc/libs/1_47_0/libs/utility/utility.htm#addressof)
* 在C++11中已经集成到<memory>头文件里。直接使用std::addressof。

##相关的惯用法

##参考

