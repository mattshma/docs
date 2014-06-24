Decorator
===

在实际使用中，对于一些已经存在的函数，我们可能需要增加新的需求，如 日志记录，权限验证等。一种较好的方式是使用 decorator 模式来解决这个需求。

除了注意使用 `functools.wraps(func)` 来装饰func 和 对于装饰器参数和函数参数注意下外，其他可以参考如下资料。

Reference
---

- [How can I make a chain of function decorators in Python?](http://stackoverflow.com/questions/739654/how-can-i-make-a-chain-of-function-decorators-in-python/1594484#1594484）  
- [理解python中的装饰器](http://wklken.me/posts/2013/07/19/python-translate-decorator.html#)  
- [装饰器与函数式Python](http://youngsterxyf.github.io/2013/01/04/Decorators-and-Functional-Python/)  


