# More C++ Idioms
====================
![cover image](https://upload.wikimedia.org/wikibooks/en/thumb/9/9c/More_CPP_Idioms.jpg/190px-More_CPP_Idioms.jpg)
The world is invited to catalog reusable pieces of C++ knowledge (similar to the book on design patterns by GoF). The goal here is to first build an exhaustive catalog of modern C++ idioms and later evolve it into an idiom language, just like a pattern language. 

刚刚启步，欢迎一起讨论！

*这些翻译，参考了[Effective Modern C++](https://github.com/Rescape/Effective-Modern-Cpp-Zh)的制作方式，在此表示感谢。

##代码使用说明

使用gitbook作为静态编译输出，需要安装`Node.js`，然后从`npm`安装gitbook

```sh
npm install gitbook -g
```

然后git clone下来本书，然后输出静态网页，在浏览器上查看：

```sh
git clone git@github.com:XimingCheng/Effective-Modern-Cpp-Zh.git
cd Effective-Modern-Cpp-Zh
gitbook serve .
```

gitbook会默认在端口`4000`开启服务器，使用浏览器访问[http://localhost:4000/](http://localhost:4000/)就可以访问然后阅读本书的中文翻译。