# 奇异递归模板模式 (Curiously Recurring Template Pattern)
##目的
使用一个子类作为模板参数来特化一个基类模板。
##别名
* CRTP
* Mixin-from-above


##动机
提取出基类中与类型无关的部分，以及可定制的部分，同时混合子类的实现。

##解决方案及示例
在CRTP惯用法中，一个类T继承自一个由T特化的模板。
```class T : public X<T> {...};```
只有当不知T的大小，却可以得出X<T>的大小时才有效。因为成员函数体(定义)在声明很久后才会实例化，可以在函数中通过static_cast转换就能使用子类的成员。比如:
```
template <class Derived>
  struct base
  {
      void interface()
      {
          // ...
          static_cast<Derived*>(this)->implementation();
          // ...
      }

      static void static_interface()
      {
          // ...
          Derived::static_implementation();
          // ...
      }

      // 默认实现也许会(如果存在) 或者 应当会(其它情况)被子类覆盖(overriden).
      void implementation();
      static void static_implementation();
  };

  // The Curiously Recurring Template Pattern (CRTP)
  struct derived_1 : base<derived_1>
  {
      // 这个类使用基本实现的implementation()。
      // void implementation();

      // ... 和覆盖的static_implementation()。
      static void static_implementation();
  };

  struct derived_2 : base<derived_2>
  {
      // 这个类覆盖了实现:
      void implementation();

      // ... 而使用基类的static_implementation
      // static void static_implementation();
  };
```
来自维基百科的说明:
>基类模板利用了其成员函数体（即成员函数的实现）将不被实例化直至声明很久之后（实际上只有被调用的模板类的成员函数才会被实例化）；并利用了派生类的成员，这是通过类型转化。

在上例中，Base<Derived>::interface()，虽然是在struct Derived之前就被声明了，但未被编译器实例化直至它被实际调用，这发生于Derived声明之后，此时Derived::implementation()的声明是已知的。

这种技术获得了类似于虚函数的效果，并避免了动态多态的代价。也有人把CRTP称为“模拟的动态绑定”。
`

##已知的应用
* [Barton-Nackman trick](https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Barton-Nackman_trick)
* WebKit之引用计数智能指针(RefCounted)

##相关的惯用法
* Parameterized Base Class Idiom
* Barton-Nackman trick

##参考
* [Curiously Recurring Template Pattern on Wikipedia](http://en.wikipedia.org/wiki/Curiously_Recurring_Template_Pattern)
* [The Curiously Recurring Template Pattern in C++](http://eli.thegreenplace.net/2011/05/17/the-curiously-recurring-template-pattern-in-c/)
