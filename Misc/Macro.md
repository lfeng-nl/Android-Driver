# 本文介绍在linux内核中使用的各种特殊宏

### 1.EXPORT_SYMBOL/EXPORT_SYMBOL_GPL

- 作用：将标签内定义的函数或者符号对全部内核代码公开，不用修改内核代码就可以在内核模块中直接调用，

- 使用方法：

  - 在模块函数定义之后使用EXPORT_SYMBOL(符号名)；

  - 在调用该函数的模块中使用extern对之声明；

  - 首先加载定义该函数的模块，在加载调用该函数的模块；

- EXPORT_SYMBOL_GPL：作用同EXPORT_SYMBOL，但只能使符号对GPL许可的模块可用；

  > [GNU通用公共许可协议](https://baike.baidu.com/item/GNU%E9%80%9A%E7%94%A8%E5%85%AC%E5%85%B1%E8%AE%B8%E5%8F%AF%E5%8D%8F%E8%AE%AE/9610167)（英语：GNU General Public License，缩写：GNU GPL、[GPL](https://baike.baidu.com/item/GPL/2357903)），是一个广泛被使用的自由软件许可协议条款；GPL明确规定，任何源码的衍生产品，如果对外发布，都必须保持同样的许可证。


