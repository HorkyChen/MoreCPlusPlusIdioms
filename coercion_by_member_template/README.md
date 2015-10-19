# Coercion by Member Template
##目的
允许模板类中的模板参数进行隐性的（或强制的）类型转换，以提高模板类的弹性。
##别名

##动机
It is often useful to extend a relationship between two types to class templates specialized with those types. 例如，有一个类D继承自B, C++是支持将一个指向D对象的指针赋值给一个指向B对象的指针的。但是包含它们的类型是无法共享这种关系的，比如模板类。所以Helper<D>对象不能指定给一个Helper<B>对象。
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
在模板类中定义成员模板方法用来完成参数类型所需要的隐式类型转换。在下面的例子中，模板建议函数和重载的赋值操作可以支持任何类型的U, 它支持从`T*`初始化或赋值给一个`U*`。
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

    // Supporting coercion using member template constructor.
    // This is not a copy constructor, but behaves similarly.
    template <class U>
    Ptr (Ptr <U> const & p)
      : ptr (p.ptr) // Implicit conversion from U to T required
    {
      std::cout << "Coercing member template constructor\n";
    }

    // Copy assignment operator.
    Ptr & operator = (Ptr const & p)
    {
      ptr = p.ptr;
      std::cout << "Copy assignment operator\n";
      return *this;
    }

    // Supporting coercion using member template assignment operator.
    // This is not the copy assignment operator, but works similarly.
    template <class U>
    Ptr & operator = (Ptr <U> const & p)
    {
      ptr = p.ptr; // Implicit conversion from U to T required
      std::cout << "Coercing member template assignment operator\n";
      return *this;
    }

    T *ptr;
};

int main (void)
{
   Ptr <D> d_ptr;
   Ptr <B> b_ptr (d_ptr); // Now supported
   b_ptr = d_ptr;         // Now supported
}
```
##已知的应用

##相关的惯用法

##参考
