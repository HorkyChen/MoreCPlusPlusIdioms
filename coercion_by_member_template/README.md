# 通过成员模板支持隐式转换 (Coercion by Member Template)
##目的
允许模板类中的模板参数进行隐性的（或corecion）类型转换，以提高模板类的弹性。

##别名

##动机
将两个类的关系扩展到使用它们特化的模板类上还是比较有用的。例如，有一个类D继承自B, C++是支持将一个指向D对象的指针赋值给一个指向B对象的指针的。但是包含它们的类型是无法共享这种关系的，比如模板类。所以Helper<D>对象不能指定给一个Helper<B>对象。
```
class B {};
class D : public B {};
template <class T>
class Helper {};

B *bptr;
D *dptr;
bptr = dptr; // OK; C++允许

Helper<B> hb;
Helper<D> hd;
hb = hd; // 虽然有价值，但是禁止这样做
```
在很多情况下，这种转换是有用的。比如从`std::auto_ptr<D>`转换到`std::auto_ptr<B>`，这看起来很自然，但是C++不支持。这也是本惯用法所要解决的。

##解决方案及示例
在模板类中定义成员模板(member template)方法用来完成参数类型所需要的隐式类型转换。在下面的例子中，模板建议函数和重载的赋值操作可以支持任何类型的U, 它支持从`T*`初始化或赋值给一个`U*`。
```
template <class T>
class Ptr
{
  public:
    Ptr () {}

    Ptr (Ptr const & p)
      : ptr (p.ptr)
    {
      std::cout << "Copy constructor\n";
    }

    // 通过成员模板支持隐性类型转换。
    // 这不是copy建构函数，只是有点相似。
    template <class U>
    Ptr (Ptr <U> const & p)
      : ptr (p.ptr) // 需要隐式地从U转到T。
    {
      std::cout << "Coercing member template constructor\n";
    }

    // 拷贝赋值操作符重载
    Ptr & operator = (Ptr const & p)
    {
      ptr = p.ptr;
      std::cout << "Copy assignment operator\n";
      return *this;
    }

    // 通过成员模板的赋值操作符支持隐式转换。
    // 这个也不是拷贝赋值操作符重载函数，只是相似。
    template <class U>
    Ptr & operator = (Ptr <U> const & p)
    {
      ptr = p.ptr; // 需要从U到T的隐式转换。
      std::cout << "Coercing member template assignment operator\n";
      return *this;
    }

    T *ptr;
};

int main (void)
{
   Ptr <D> d_ptr;
   Ptr <B> b_ptr (d_ptr); // 支持了
   b_ptr = d_ptr;         // 支持了
}
```

另一个应用是允许将一个派生类指针数据赋值到指向基类的指针数组。再假如D继承自B, 一个D对象也是一个B对象，它们是is-a的关系。但是，一个D对象的数组并不是B对象的数组。C++不支持这种情况。如果能解开这个限制当然也是非常有用的。只是要特别注意将基类的指针数组复制给其派生类的数组。同样使用成员模板函数的特化可以解决这个问题。
下面的例子中使用了参数为`Array<U *>`的模板建构函数和赋值操作符重载函数。
```
template <class T>
class Array
{
  public:
    Array () {}
    Array (Array const & a)
    {
      std::copy (a.array_, a.array_ + SIZE, array_);
    }

    template <class U>
    Array (Array <U *> const & a)
    {
      std::copy (a.array_, a.array_ + SIZE, array_);
    }

    template <class U>
    Array & operator = (Array <U *> const & a)
    {
      std::copy (a.array_, a.array_ + SIZE, array_);
    }

    enum { SIZE = 10 };
    T array_[SIZE];
};
```
许多的智能指针，比如std::auto_ptr, boost::shared_ptr都应用了这个惯用法。

##警告
这个惯用法的典型错误是引入模板拷贝建构函数和赋值操作符函数后，就无法提供有效的非模板成员的copy建构函数及赋值操作符函数了。如果类没有声明copy建构函数和copy赋值操作符重载，编译器会自动为类定义默认的实现，这样会导致一些隐晦的错误。

##已知的应用
* std::auto_ptr
* boost::shared_ptr

##相关的惯用法
* Generic Container Idioms

##参考
* [std::auto_ptr的实现](http://www.josuttis.com/libbook/util/autoptr.hpp.html)
* [Member Templates](http://www.informit.com/guides/content.aspx?g=cplusplus&seqNum=263)
