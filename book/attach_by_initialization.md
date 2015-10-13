# 初始时挂载(Attach by Initialization)
##目的
在程序正式运行前，挂载一个自定义对象到一个框架。

    Attach a user-defined object to a framework before program execution begins.

##别名
* Static-object-with-constructor

##动机
当前应用框架，如GUI框架(Microsoft MFC等), 以及对象请求代理(Object request brokers) (一些CORBA的实现), 都会使用它们自己的消息循环(Message loops, 也被称为事件循环(Event loops))来控制整个应用，并且应用程序的main函数通常被隐藏的很深（如MFC中的AfxWinMain）。程序员可能很难更改到应用级别的main函数，这样就无法在主事件循环开始前做一些应用级别的初始化操作。

此惯用法就是提供在框架控制的事件循环开始前执行特定的代码。

##解决方案及示例
在C++里，在全局命名空间里定义的全局变量或者静态变量会在main函数执行前初始化。这些对象也被称为静态存储期对象(objects of static storage duration)。 这个特别就可以被用于挂起一个对象到一个无法更改主函数的系统里。下面是一个MFC(Microsoft Foundation Classes)的例子:
```
///// File = Hello.h
class HelloApp: public CWinApp
{
public:
    virtual BOOL InitInstance ();
};

///// File = Hello.cpp
#include <afxwin.h>
#include "Hello.h"
HelloApp myApp; // 应用级别的全局对象
BOOL HelloApp::InitInstance ()
{
  m_pMainWnd = new CFrameWnd();
  m_pMainWnd->Create(0,"Hello, World!!");
  m_pMainWnd->ShowWindow(SW_SHOW);
  return TRUE;
}
```
上面的例子中最关键的是HelloApp类的全局对象myApp，它会在应用程序的main函数执行前初始化。这里有一个副作用是使得CWinApp的建构函数也被调用，然后再会调用其它若干个类的建构函数，在这个过程中, myApp这个全局对象就被挂到(attach)到框架之中。稍后，MFC的main函数AfxWinMain就会拿到它。
代码里的HelloApp::InitInstance()成员函数只是用于做一个完全的例子，并不属于这个惯用法。它会在AfxWinMain开始执行后调用。

全局对象和静态对象可以通过多种方式初始化：缺省建构函数，带参数的建构函数，通过一个函数的返回值（如工厂方法），或者动态初始化(dynamic initialization, 如Lazy intialization)等。

#警告
在C++里静态存储期对象初始化顺序化是无法保障的。其中特别命名空间里的对象会在命名空间中的函数和变量被使用前创建，但也不一定在main函数执行前。析构的顺序同样也无标准可以约束，所以如果当一个静态对象初始化存在依赖关系时就会出现静态初始化顺序问题(static initialization order problem)。所以这个惯用也存在这个风险。

**译注**:静态对象一定要限制使用，否则也会带来启用和退出的性能问题。可以参考: [应用程序启动速度优化](http://blog.csdn.net/horkychen/article/details/37335025)。

##已知的应用
* Microsoft Foundation Classes

##相关的惯用法

##参考
* [Proposed C++ language extension to improve portability of the Attach by Initialization idiom](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/1995/N0717.htm)
