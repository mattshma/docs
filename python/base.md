Base
===

__init__.py
---
在实际使用的过程中，不知道怎么导入`__init__.py`中定义的变量，现说明下。

- The \_\_init\_\_.py files are required to make Python treat the directories as containing packages. In the simplest case, \_\_init\_\_.py can just be an empty file, but it can also execute initialization code for the package or set the \_\_all\_\_ variable.  
- Import some files  originally at the Class level into \_\_init\_\_.py to make it available at the package level.  

```python
package/
    __init__.py
    file.py
    file2.py
    file3.py
    subpackage/
        __init__.py
        submodule1.py
        submodule2.py
```

In this example we can say that file.py has the Class File. So without anything in our \_\_init\_\_.py you would import
with this syntax: `from package.file import File`.

However we can import File into your \_\_init\_\_.py to make it available at the package level:

```python
# in your __init__.py
from file import File

# now import File from package
from package import File
```
- define `__all__` or others  
When we use `from package import *` to import modules, we import the modules which were defined in the `__all__`

Imports: Multi-Line and Absolute/Relative
---
关于相对路径和绝对路径的导入参考[Imports: Multi-Line and Absolute/Relative](http://legacy.python.org/dev/peps/pep-0328/#id7)


- Reference of List
---
- [modules](https://docs.python.org/2/tutorial/modules.html)
- [Be Pythonic: __init__.py](http://mikegrouchy.com/blog/2012/05/be-pythonic-__init__py.html)
- [how-to-import-classes-defined-in-init-py](http://stackoverflow.com/questions/582723/how-to-import-classes-defined-in-init-py)
