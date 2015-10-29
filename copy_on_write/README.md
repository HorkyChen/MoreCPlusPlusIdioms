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

#include <boost/shared_ptr.hpp>

template <class T>
class CowPtr
{
    public:
        typedef boost::shared_ptr<T> RefPtr;

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
这是一个简单的实现版本。除了必须通过智能指针解引用(dereferencing)来引用其内部对象有点不太方便外，还至少有一个缺点:类可以返回内部状态的引用:
```char & String::operator[](int)```
这样会带有一些无法预期的行为。

考虑下面的代码段:
```
CowPtr<String> s1 = "Hello";
char &c = s1->operator[](4); // 非常量的detach操作什么也不做
CowPtr<String> s2(s1); // 延迟拷贝，共享的状态
c = '!'; // 悲催啦
```
最后一行原本要修改原始的字串s1, 而不是它的复本，而事实上s2也被修改了。

一个比较好的做法是写一个自定义的copy-on-write实现，封装需要lazy-copy的类，并且保持对用户透明。

##已知的应用

##相关的惯用法

##参考
