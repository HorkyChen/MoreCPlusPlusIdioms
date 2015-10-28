# Checked delete
##目的
提高delete表达式的安全性。

##别名

##动机
C++允许通过delete删除一个指向未完整定义的类(Incomplete class)的指针。如果类有一个较重的析构函数，或者有一个delete运算符重载，这个行为就是未定义的。有些编译器会对这些情况报警，但程序往往会忽略或者屏蔽这些警告。
**译注**：所谓Incomplete class, 指未能完整定义，无法确定其最终size的类，比如:
```
class A{
    private:
        A a;  // 没有提前声明Class A;
}
```

下面这个例子里, main.cpp知道Object的定义，`main()`调用delete.cpp里定义的delete_object()并不能看到Obect的完整定义，而只是一个前置声明。所以就出现了delete一个部分定义类型的对象，其行为就是未定义的。
```
////////////////////
// File: deleter.hpp
////////////////////
// 只是声明Object, 但不定义。
struct Object;
void delete_object(Object* p);

////////////////////
// File: deleter.cpp
////////////////////
#include "deleter.hpp"

// 删除一个类型不明确的对象。
void delete_object(Object* p) { delete p; }

////////////////////
// File: object.hpp
////////////////////
struct Object
{
  // 对于仅部分定义的对象，在它析构时，这个自定义的析构函数不会被调用。
  ~Object() {
     // ...
  }
};

////////////////////
// File: main.cpp
////////////////////
#include "deleter.hpp"
#include "object.hpp"

int main() {
  Object* p = new Object;
  delete_object(p);
}
```

##解决方案及示例
此惯用法为解决这个问题，不再直接调用`delete`,而是调用一个函数模板去清理内存。

下面是Boost工具库中boost::checked_delete的实现。 它借助`sizeof()`触发编译错误，这也是编译期进行断言的一类做法。如果`T`声明了但没有定义，`sizeof(T)`就会出现编译错误或者返回0，具体行为依赖于编译器。如果`sizeof(T)`返回0， checked_delete就会因为声明一个-1大小的数组而触发编译错误。数组名·`type_must_be_complete`会出现在错误消息中以帮助说明问题。
```
template<class T>
inline void checked_delete(T * x)
{
    typedef char type_must_be_complete[ sizeof(T)? 1: -1 ];
    (void) sizeof(type_must_be_complete);
    delete x;
}
template<class T>
struct checked_deleter : std::unary_function <T *, void>
{
    void operator()(T * x) const
    {
        boost::checked_delete(x);
    }
};
```
**NOTE**: 这个技术可以应用于数组的删除。

**WARNING**: std::auto_ptr没有相似的机制。std::auto_ptr声明时，模板参数类型不需要有明确定义，所以当使用非完整类型实例化std::auto_ptr时同样会出现未定义的析构行为。


##已知的应用

##相关的惯用法

##参考
http://www.boost.org/libs/utility/checked_delete.html

