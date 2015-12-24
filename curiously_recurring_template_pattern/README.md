# Curiously Recurring Template Pattern
##目的
使用一个子类作为模板参数来特化一个基类。
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

##已知的应用
[Barton-Nackman trick](https://en.wikibooks.org/wiki/More_C%2B%2B_Idioms/Barton-Nackman_trick)

##相关的惯用法
* Parameterized Base Class Idiom
* Barton-Nackman trick

##参考
[Curiously Recurring Template Pattern on Wikipedia](http://en.wikipedia.org/wiki/Curiously_Recurring_Template_Pattern)
