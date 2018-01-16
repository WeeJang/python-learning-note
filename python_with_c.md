python一直以来被作为一种快速开发的PL使用，很多工程都将其看作一种“胶水“。所谓胶水，可以认为python构建工程的骨架，其他的血肉部分使用c/c++等构建（比如做成动态库）。

一个例子是来自于著名的库，xgboost，通过源代码可以看到， 核心是通过C++进行构建的，并提供了的C的API接口，编译成动态库（ .so），最终warpped by python，做成python 的lib。在包装层，使用的就是ctypes(https://docs.python.org/2.7/library/ctypes.html)。

插一嘴，知乎上有个讨论曾经提到过ctypes的妙用，腾讯的前辈，很有意思（https://www.zhihu.com/question/29995881/answer/46397633?utm_medium=social&utm_source=wechat_session）

这里做一下简要记录：

## quick start

### env
   
    centos7 gcc4.8

### test.c 
```c++
#include <stdio.h>  

void hello(){

    printf("hello world!\n");

}
```
### compile to shared lib
```bash
gcc -shared -o libtest.so -fPIC test.c
```
### testwrapper.py
```python
import sys,os
import ctypes

CUR_DICT_PATH = os.path.split(os.path.realpath(__file__))[0]
testlib = ctypes.CDLL(os.path.join(CUR_DICT_PATH,"libtest.so"))

testlib.hello()
```

=========================================================

如果是c++ 的core,则需要使用 extern 关键字（extern "C" makes a function-name in C++ have 'C' linkage (compiler does not mangle the name) so that client C code can link to (i.e use) your function using a 'C' compatible header file that contains just the declaration of your function. Your function definition is contained in a binary format (that was compiled by your C++ compiler) that the client 'C' linker will then link to using the 'C' name.）

```c++
#include <iostream>

extern "C" void hello(){
 std::cout << "hello world!Fine" << std::endl;
}
```

=========================================================

ctypes tries to protect you from calling functions with the wrong number of arguments or the wrong calling convention. Unfortunately this only works on Windows.

也就是说，ctypes只有在win平台下才会进行传入参数的安全检查。在linux 环境下，如果参数传入不当（参数数目不够，类型错误等）直接 segment fault。

=========================================================

python原生类型的兼容性。

None, integers, longs, byte strings and unicode strings are the only native Python objects that can directly be used as parameters in these function calls. None is passed as a C NULL pointer, byte strings and unicode strings are passed as pointer to the memory block that contains their data (char *or wchar_t *). Python integers and Python longs are passed as the platforms default C int type, their value is masked to fit into the C type.

也就是说，None,integers,longs,byte strings 和unicode strings能够被直接传入这些c_api中。

 

=========================================================

memory 缓冲区

If you need mutable memory blocks, ctypes has a create_string_buffer() function which creates these in various ways. 

=========================================================

Specifying the required argument types (function prototypes)

通过制定函数参数类型，保证对调用进行type-check。这应该算是一种保障机制。通过argtypes指定。

返回类型通过restype指定。

=========================================================

Passing pointers

这一部分，我不太认同文档中所述的passing-by-reference。在C的设计中，不存在引用传值的语义，所有的均为passing-by-value的方式。可能这里描述的引用指的是指向数据区域的指针是该区域的引用吧。


这个功能通过byref()实现。

 
```python
>>> i = c_int()

>>> f = c_float()

>>> s = create_string_buffer('\000' * 32)

>>> print i.value, f.value, repr(s.value)

0 0.0 ''

>>> libc.sscanf("1 3.14 Hello", "%d %f %s",
...             byref(i), byref(f), s)

3

>>> print i.value, f.value, repr(s.value)

1 3.1400001049 'Hello'

```
