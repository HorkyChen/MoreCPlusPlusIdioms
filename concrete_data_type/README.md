# Concrete Data Type
##目的
通过允许或禁止动态分配(heap或free store)的方式控制对象的作用域和生命周期。
>译注: Free-store和heap都可以理解为动态分配内存空间，其中heap更为常用，指malloc/free使用的内存区间，而free-store则是C++引入的，指的是new/delete使用的内存空间。两基本是一致的。可以参考:[What is the difference between the heap and the free store?](http://stackoverflow.com/questions/6161235/what-is-the-difference-between-the-heap-and-the-free-store)

##别名

##动机
C++将对象都视为一个变量，并且提供了两种不同的生命周期控制方式。一种是作用域内从堆上分配的对象作为作用域内变量(scoped varialble)，当离开所在的作用域时就会被释放, 比如函数中的变量。第二种方式是动态分配的变量(通常是指针)，在离开作用域时，对象并不会自动释放，比如单例的对象等。
可以通过本惯用法强制使用某个特定的管理方式。

##解决方案及示例
简单地通过存取限定就可以达到目标。下面的代码显示`MouseEventHandler`只能使用动态分配的方式。
```
class EventHandler
{
  public:
    virtual ~EventHandler () {}
};
class MouseEventHandler : public EventHandler // 继承
{
  protected:
    ~MouseEventHandler () {} // 一个保护段内的虚析构函数
  public:
    MouseEventHandler () {} // 公开的建构函数
};
int main (void)
{
  MouseEventHandler m; // 作用域内对象不允许有一个非公开的析构函数
  EventHandler *e = new MouseEventHandler (); // 动态分配没有问题
  delete e;  // 支持多态的delete, 不会有内存泄露的风险
}
```

另一种强制使用动态分配的方式可以使用单例的设计模式，不公开建构函数，而是提供一个静态函数返回动态分配的对象。这样还有一个好处是不需要使用多态的清理(虚析构函数)方式。提供一个成员函数`destry()`就成达到清理内存的目的(可能带虚表)。
```
class MouseEventHandler // 没有继随
{
  protected:
    MouseEventHandler () {} // 保护段内的建构函数
    ~MouseEventHandler () {} // 保护段内非虚的板构函数
  public:
    static MouseEventHandler * instance () { return new MouseEventHandler(); }
    void destroy () { delete this; }  // 回收内存
};
```

相对的，如果要限制使用第一种生命周期管理方式，也被称为自动变量(automatic variable), 只需要将new操作符限定为private。
```
class ScopedLock
{
  private:
    static void * operator new (size_t size); // 不允许动态分配
    static void * operator new (size_t, void * mem);  // 也不允许placement new
};
int main (void)
{
   ScopedLock s; // OK
   ScopedLock * sl = new ScopedLock (); // 标准的new和nothrow new都是不允许的
   void * buf = ::operator new (sizeof (ScopedLock));
   ScopedLock * s2 = new(buf) ScopedLock;  // 也不允许placement new
}
```
`ScopedLock`对象无法动态分配，包括nothrew，或者placement new。

##已知的应用

##相关的惯用法

##参考
* [Concrete Data Type](http://www.laputan.org/pub/sag/coplien-idioms.doc), J. Coplien
* 关于placement new，参考Effective C++.
*

