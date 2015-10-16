# 清理 (Clear-and-minimize)
##目的
清理容器，并将容器容量调到最低。

##别名
swap with temporary idiom。

##动机
标准库中的容器常常会分配超出实际占用的容量，这样当容量不断增加空间时就有了优化空间。换言之，容器总是会多出来一些不必要的容量，也就造成了内存浪费。本惯用法就是用于清空容器，并将容量置为最小化的0，达到节省内存的目标。

##解决方案及示例
其实很简单，就像下面这样：
```
std::vector <int> v;
//... 在v上发生调用一批push_back，然后就会出现一批remove操作。
std::vector<int>().swap (v);
```
第一行初始化实例，它会保证处于最小的0大小状态。最后一行使用一个临时的vector与v应用non-throwing swap惯用法，这个非常有效。交换后，临时对象就会被释放掉。

自从C++11后，一些容器都定义了`shrink_to_fit()`方法，它可以没有任何约束地将容器容量降到所需要的size。

##已知的应用

##相关的惯用法
* Shrink to fit
* Non-throwing swapping

##参考
* [Programming Languages -- C++](http://open-std.org/jtc1/sc22/wg21/docs/papers/2008/n2800.pdf) (Draft Standard)
