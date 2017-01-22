# Iterators & Generators & Yield

stackoverflow里有个高票回复总结的非常好。本文主要引用[该文](http://stackoverflow.com/questions/231767/what-does-the-yield-keyword-do-in-python)

## Iterable Obejct

所有能用`for ... in ...` 来读取的对象，如list, string, file, dict等，都被称为可迭代对象(Iterable object)。这种方式将的所有值都存储在内存中，如果数据量非常大，这种方式可能并不是我们想要的。

### Iterator protocol

## Generator
生成器也是可迭代的，但其 **只能读取一次 **，因为它并不是将所有数据都放在内存中，而是实例生成数据：

```
>>> mygenerator = (x*x for x in range(3))
>>> for i in mygenerator :
...    print(i)
0
1
4
>>> for i in mygenerator :
...    print(i)
# 无输出
```
这里只是将列表推导中的`[]`替换成`()`。

生成器占用最少的内存，并且不需要等到所有元素都返回才能使用它们，因此其比可迭代对象的效率高。

## yield
`yield`类似`return`，只不过其返回的是生成器。如下：

```
>>> def createGenerator() :
...    mylist = range(3)
...    for i in mylist :
...        yield i*i
...
>>> mygenerator = createGenerator() # create a generator
>>> print(mygenerator) # mygenerator is an object!
<generator object createGenerator at 0xb7555c34>
>>> for i in mygenerator:
...     print(i)
0
1
4
```
上面的写法可能有点奇怪，已经使用for构造生成器了，为什么还需要用for来执行它？这是因为调用yield返回的函数时，该函数内部代码并不能立马执行，而只是返回一个生成器对象。只有当我们使用for进行迭代时，该函数内的代码才会执行--每次返回一个值给上层for循环，该for循环运行到调用生成器位置时，生成器代码返回上一次返回位置的下一个值，直到生成器迭代完成。

## Itertools

参考[Itertools](https://docs.python.org/2/library/itertools.html)。



参考
----
- [Iterator Types](https://docs.python.org/2/library/stdtypes.html#iterator-types)
- [iterator & generator](http://anandology.com/python-practice-book/iterators.html)
- [generator](https://wiki.python.org/moin/Generators)
