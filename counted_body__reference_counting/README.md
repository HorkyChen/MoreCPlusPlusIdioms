# 引用计数 (Counted Body / Reference Counting)

##目的
资源或对象的逻辑共享的管理，避免代价太大的拷贝，并且对于动态分配的资源可以完整的释放。

##别名
* Reference Counting (intrusive)
* Counted Body

##动机
运用Handle/Body惯用法时，实体(Body)的拷贝的代价太大，因为拷贝引入了内存分配和拷贝建构。这种拷贝可以通过使用指针或引用来避免，但它们又产生谁来清理对象的问题。有一些句柄(Handle)必须负责释放为实体(Body)分配的内存，通常也是最后一个。如果没有自动回收内存机制，就会与内置类型之间有明显的差异。
```
并没有说明白。待补充......
```

##解决方案及示例
解决方案是在实体类中增加引用计数来帮助内存管理，因为得名"Counted Body"。内存管理添加到Handle类，特别是它的初始化、赋值、拷贝和析构。
```
#include <cstring>
#include <algorithm>
#include <iostream>

class StringRep {
  friend class String;

  friend std::ostream &operator<<(std::ostream &out, StringRep const &str) {
    out << "[" << str.data_ << ", " << str.count_ << "]";
    return out;
  }

 public:
  StringRep(const char *s) : count_(1) {
    strcpy(data_ = new char[strlen(s) + 1], s);
  }

  ~StringRep() { delete[] data_; }

 private:
  int count_;
  char *data_;
};

class String {
 public:
  String() : rep(new StringRep("")) {
    std::cout << "empty ctor: " << *rep << "\n";
  }
  String(const String &s) : rep(s.rep) {
    rep->count_++;
    std::cout << "String ctor: " << *rep << "\n";
  }
  String(const char *s) : rep(new StringRep(s)) {
    std::cout << "char ctor:" << *rep << "\n";
  }
  String &operator=(const String &s) {
    std::cout << "before assign: " << *s.rep << " to " << *rep << "\n";
    String(s).swap(*this);  // copy-and-swap idiom
    std::cout << "after assign: " << *s.rep << " , " << *rep << "\n";
    return *this;
  }
  ~String() {  // StringRep deleted only when the last handle goes out of scope.
    if (rep && --rep->count_ <= 0) {
      std::cout << "dtor: " << *rep << "\n";
      delete rep;
    }
  }

 private:
  void swap(String &s) throw() { std::swap(this->rep, s.rep); }

  StringRep *rep;
};
int main() {

  std::cout << "*** init String a with empty\n";
  String a;
  std::cout << "\n*** assign a to \"A\"\n";
  a = "A";

  std::cout << "\n*** init String b with \"B\"\n";
  String b = "B";

  std::cout << "\n*** b->a\n";
  a = b;

  std::cout << "\n*** init c with a\n";
  String c(a);

  std::cout << "\n*** init d with \"D\"\n";
  String d("D");

  return 0;
}
```
省掉不必要的拷贝就会带来更有效的实现。这个惯用法假设程序员可以修改实体类的代码。不然的话，还可以使用Detached Counted Body。把引用计数放到实体类内部称为侵入式(intrusive)引用计数。如果存在实体类外面就称为非侵入式引用计数。这个实现是浅拷贝的变种，兼有深拷贝的语义和Smalltalk 键值(name-value)对的有效性。

输出结果如下:
```
*** init String a with empty
empty ctor: [, 1]

*** assign a to "A"
char ctor:[A, 1]
before assign: [A, 1] to [, 1]
String ctor: [A, 2]
dtor: [, 0]
after assign: [A, 2] , [A, 2]

*** init String b with "B"
char ctor:[B, 1]

*** b->a
before assign: [B, 1] to [A, 1]
String ctor: [B, 2]
dtor: [A, 0]
after assign: [B, 2] , [B, 2]

*** init c with a
String ctor: [B, 3]

*** init d with "D"
char ctor:[D, 1]
dtor: [D, 0]
dtor: [B, 0]
```
##影响
创建多个引用计数会导致相同实体的多次删除，这个未定义的。必须格外留意对相同实体的创建多个引用计数。侵入式的引用计数基本没有这个问题。如果是使用非侵入式的引用计数，就要要求程序员避免出现重复的引用计数。

##已知的应用
* boost::shared_ptr (非侵入式的引用计数)
* boost::intrusive_ptr (侵入式的引用计数)
* std::shared_ptr
* Qt toolkit, 如QString

##相关的惯用法
* Handle Body
* Detached Counted Body (非侵入式引用计数)
* Smart Pointer
* Copy-and-swap

##参考
* [Advanced C++中文版](http://product.china-pub.com/16697) 9.5垃圾收集


