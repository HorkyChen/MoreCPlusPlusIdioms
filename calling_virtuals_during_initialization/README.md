# 初始化过程调用虚函数 (Calling Virtuals During Initialization)

##目的
在初始过程模仿调用虚函数。

##别名
* Dynamic Binding During Initialization idiom

##动机
有时需要在派生类初始化时想要调用派生类的虚函数。C++是禁止这样的操作的，毕竟在派生类初始化完前使用派生类的成员是危险的，如果虚函数不使用正在建构的对象的成员时就不会有什么问题。
```
 class Base {
 public:
   Base();
   ...
   virtual void foo(int n) const; // 通常为纯虚
   virtual double bar() const;    // 通常为纯虚
 };

 Base::Base()
 {
   ... foo(42) ... bar() ...
   // 这里无法使用动态绑定。
   // 目标: 在这些调用中使用虚函数。
 }

 class Derived : public Base {
 public:
   ...
   virtual void foo(int n) const;
   virtual double bar() const;
 };
```

##解决方案及示例
有几种方式可以达到这个效果，每种方式又都有其优缺点。它们可以分为两类，一类将初始阶段分为两个阶段，另一类则仍然只使用单一初始阶段。

双阶段初始化技术就是将初始化过程分隔出来，当然不是所有的类都可以这样做。分开的初始化过程再通过另一个函数联合在一起，这个函数不一定是成员函数。
```
class Base {
 public:
   void init();  // 是否为虚函数都可以
   ...
   virtual void foo(int n) const; // 通常是纯虚
   virtual double bar() const;    // 通常是纯虚
 };

 void Base::init()
 {
   ... foo(42) ... bar() ...
   // 大部分是从Base::Base()拷出来的
 }

 class Derived : public Base {
 public:
   Derived (const char *);
   virtual void foo(int n) const;
   virtual double bar() const;
 };
```

使用非成员函数的方法:
```
template <class Derived, class Parameter>
std::auto_ptr <Base> factory (Parameter p)
{
  std::auto_ptr <Base> ptr (new Derived (p));
  ptr->init ();
  return ptr;
}
```

还有一个非模板的实现版本，工厂方法可以移到基类中成为一个静态方法。
```
class Base {
  public:
    template <class D, class Parameter>
    static std::auto_ptr <Base> Create (Parameter p)
    {
       std::auto_ptr <Base> ptr (new D (p));
       ptr->init ();
       return ptr;
    }
};
int main ()
{
  std::auto_ptr <Base> b = Base::Create <Derived> ("para");
}
```

基类的建构需要设置为`private`，可以避免用户意外地使用。接口应当保证易于使用，而不容易犯错。然后将工厂方法指定为派生类的友元。如果移到基类内部，就把基类指定为派生类的友元。

保持单一初始化过程的方法:
前面的方法需要新增辅助类，无疑增加复杂度。将指针传给静态成员方法，也太过C式了。这里可以使用Curiously Recurring Template Pattern惯用法。
```
class Base {
};
template <class D>
class InitTimeCaller : public Base {
  protected:
    InitTimeCaller () {
       D::foo ();
       D::bar ();
    }
};
class Derived : public InitTimeCaller <Derived>
{
  public:
    Derived () : InitTimeCaller <Derived> () {
		cout << "Derived::Derived()\n";
	}
    static void foo () {
		cout << "Derived::foo()\n";
	}
    static void bar () {
		cout << "Derived::bar()\n";
	}
};
```
另外使用Base-from-member惯用法的一个复杂点的变种也可以做到。

##已知的应用

##相关的惯用法

##参考
* [Strange Inheritance](http://www.parashift.com/c++-faq/strange-inheritance.html)

