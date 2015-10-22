# 第一次使用时建构 (Construct On First Use)
##目的
确保对象只在第一次使用到的时候才进行初始化，特别是全局的静态对象。

##别名
延迟建构或求值 (Lazy construction/evaluation)

##动机
静态对象必须在被使用前进行建构，如果考虑不周，也可能在它的初始化程中访问到全局静态对象。
```
struct Bar {
  Bar () {
    cout << "Bar::Bar()\n";
  }
  void f () {
    cout << "Bar::f()\n";
  }
};
struct Foo {
  Foo () {
    bar_.f ();
  }
  static Bar bar_;
};

Foo f;
Bar Foo::bar_;

int main () {}
```
上面代码中，`Bar::f()`会在建构前被调用，这种情况通常是典型undefined问题，应该避免。

##解决方案及示例
有两个解决方案，它们的分别取决于对象的析构函数是否有较为复杂的语义（non-trivial destruction semantics.）。将静态对象封装到一个函数中，这样函数可以确保在它使用前初始化它。

* 在第一次使用时，通过动态分配的方式建构
 ```
 struct Foo {
  Foo () {
    bar().f ();
  }
 Bar & bar () {
    static Bar *b = new Bar ();
    return *b;
 }
};
 ```
 如果对象析构的语义比较复杂，就可台通过下面的方式。

* 在第一次使用时，通过局部静态对象建构
```
struct Foo {
  Foo () {
    bar().f ();
  }
 Bar & bar () {
    static Bar b;
    return b;
 }
};
```

##已知的应用
* 单例模式经常使用这个方法。译注:另外还要考虑线程安全的解决方案Double check的方式。
* ACE(Adaptive Communication Environment)中ACE_TSS<T>模板类使用这个惯用法在TSS (Thread specific storage)中创建和使用对象。

##相关的惯用法
* Nifty/Schwarz Counter

##参考
