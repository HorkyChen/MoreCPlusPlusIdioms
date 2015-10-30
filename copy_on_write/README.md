# 写时拷贝 (Copy on Write)

##目的
达到延迟拷贝(lazy copy)的优化目的。和延迟初始化(lazy initialization)相似, 选择在恰当的时机更加有效。

##别名
* COW (copy-on-write)
* Lazy copy

##动机
拷贝对象有时会带来性能损失(performance penalty)。如果对象经常拷来拷去，但以很少修改，copy-on-write就能明显地提升性能。为了实现copy-on-write, 需要使用一个智能指针将真正的对象值封装起来，每次修改时都要检查一下对象的引用计数。如果对象被多次引用，就在修改前创建一个复本。

##解决方案及示例
```
#ifndef COWPTR_HPP
#define COWPTR_HPP

#include <memory>

template <class T>
class CowPtr
{
    public:
        typedef std::shared_ptr<T> RefPtr;

    private:
        RefPtr m_sp;

        void detach()
        {
            T* tmp = m_sp.get();
            if( !( tmp == 0 || m_sp.unique() ) ) {
                m_sp = RefPtr( new T( *tmp ) );
            }
        }

    public:
        CowPtr(T* t)
            :   m_sp(t)
        {}
        CowPtr(const RefPtr& refptr)
            :   m_sp(refptr)
        {}
        const T& operator*() const
        {
            return *m_sp;
        }
        T& operator*()
        {
            detach();
            return *m_sp;
        }
        const T* operator->() const
        {
            return m_sp.operator->();
        }
        T* operator->()
        {
            detach();
            return m_sp.operator->();
        }
};

#endif
```
    译注:原文代码使用boost库，都改为std的实现了。
这是一个简单的实现版本。除了必须通过智能指针解引用(dereferencing)来引用其内部对象有点不太方便外，还至少有一个缺点:类可以返回内部状态的引用:
```char & String::operator[](int)```
这样会带有一些无法预期的行为。

考虑下面的代码段:
```
CowPtr<std::string> s1 = new std::string("Hello");
char &c = s1->operator[](4); // 非常量的detach操作什么也不做
CowPtr<std::string> s2(s1); // 延迟拷贝，共享的状态
c = '!'; // 悲催啦
```
最后一行原本要修改原始的字串`s1`, 而不是它的复本`s2`，而事实上`s2`也被修改了。

一个比较好的做法是写一个自定义的copy-on-write实现，封装需要延时拷贝(lazy-copy)的类，并且保持对用户透明。为了解决上面的问题，可以标记对象为"不可共享(unshareable)"状态表示已经交出了对内存对象的引用，也就是强制进行深度拷贝。进一步优化，可以在那些不会放弃内部对象引用的non-const操作后恢复为"共享(shareable)"状态，(比如, `void string::clear()))，因为客户端代码期望这些引用都会失效。

    译注:这一部分说得不清楚。标记对象为不可共享，比如上面例子中，取出字符c后设为不可共享，再建构s2时直接进行深拷贝。另外说在non-const操作没有放弃内部对象，指的是这类操作创建了一个复本，这时候的原来的对象可以更新为shareable。


##已知的应用
* Active Template Library
* Many Qt classes ([implicit sharing](http://doc.qt.io/qt-4.8/implicit-sharing.html))

##相关的惯用法

##参考
* Herb Sutter, More Exceptional C++, Addison-Wesley 2002 - Items 13–16
* [Wikipedia: Copy-on-write](https://en.wikipedia.org/wiki/Copy-on-write)


