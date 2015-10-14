# 数据成员构建基类 (Base-from-Member)
##目的
通过子类的数据成员初始基类对象。

##别名

##动机
在C++中，基类会早于子类成员初始化，这样做的理由是子类的成员在初始时可能会使用基类的成员。不过有时也需要从子类的成员变量去初始一个基类对象。听起来和C++的规则有些冲突了，因为所有子类的成员传入前，基类的所有成员应当都已经被初始了，这样会导致一个无限循环初始化的问题。
下面这段代码就展示出这种情况的存在:
```
#include <streambuf>  // for std::streambuf
#include <ostream>    // for std::ostream

namespace  std {
  class streambuf;
  class ostream {
    explicit ostream(std::streambuf * buf);
    //...
  };
}
class fdoutbuf   // 一个streambuf的自定义版本
    : public std::streambuf
{
public:
    explicit fdoutbuf( int fd );
    //...
};

class fdostream
    : public std::ostream
{
protected:
    fdoutbuf buf;
public:
    explicit fdostream( int fd )
        : buf( fd ), std::ostream( &buf )
        // 这是不允许的：buf不能先于std::ostream初始化。
        // 但是std::ostream需要一个由fdoutbuf定义的std::streambuf对象。
    {}
};
```
上面的例子显示出程序员有心要自定义`std::streambuff`类，他从`std::streambuf`派生出fdoutbuf，这个类会用做`fdostream`的成员，而且`fdostream`派生自`std::ostream`。`std::ostream`在初始化时需要一个指向`std::streambuf`或者其派生类的指针。指针所指向的对象当然应该初始后的，进而再触发基类的初始化，然后就会出死循环。
译注:并不一定会出现死循环的，只是这个行为是未定义(Undefined)的。

本惯用法就是为了解决这个问题。

##解决方案及示例
这个惯用法利用基类是按照派生类声明它们的顺序进行初始化的，添加一个新的基类用于初始化派生类中可能出问题的成员。将这个新增的基类置于其它基类之前就可以了。
```
#include <streambuf>  // for std::streambuf
#include <ostream>    // for std::ostream

class fdoutbuf
    : public std::streambuf
{
public:
    explicit fdoutbuf(int fd);
    //...
};

struct fdostream_pbase // 新增加的基类
{
    fdoutbuf sbuffer; // 之前派生类里成员移到这里
    explicit fdostream_pbase(int fd)
        : sbuffer(fd)
    {}
};

class fdostream
    : protected fdostream_pbase // 这个类会在后面类之前初始化
    , public std::ostream
{
public:
    explicit fdostream(int fd)
        : fdostream_pbase(fd),   // 在std::ostream初始化前初始化这个新基类
          std::ostream(&sbuffer) // 这样再传入指针就安全了
    {}
    //...
};
int main(void)
{
  fdostream standard_out(1);
  standard_out << "Hello, World\n";

  return 0;
}
```
新增的基类`fdostream_pbase`持有`sbuffer`成员。`fdostream`也会从这个基类派生，并且保证在`std::ostream`之前进行初始化。这就可以保证`sbuffer`在传入`std::ostream`建构函数前已经被初始化，可以安全使用了。

##已知的应用
* [Boost Base from Member](http://www.boost.org/doc/libs/1_47_0/libs/utility/base_from_member.html)

##相关的惯用法

##参考
* [Boost Utility](http://www.boost.org/libs/utility/base_from_member.html)
