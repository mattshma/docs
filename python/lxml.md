my lxml tutorial
===

Introduction
---
lxml是最富有特点并易于处理XML和HTML的一个python库，其XML工具包是C版本的libxml2和libxslt的python化实现，其结合了速度和python的简洁性，最大程度的兼容并优于ElementTree API.

lxml.etree
---
每个element对象都有这几个属性：tag, attrib, text, tail, child elements;如下：

```python
<tag attrib="attrib">text<child_tag></child_tag></tag>tail

```

获取上述这些属性值只需在元素后调用相应属性名即可，如 `element.text`。官网在这方面讲的很详细，此不浪费时间班门弄斧，略。

XPath and XSLT with lxml  
---
lxml.etree为 ElementTree 和 Element 对象中的 **find**, **findall** 和 **findtext**
等方法提供了一个简单的PATH语法。对于xapth的具体语法可参考xpath教程。

- xpath() and the XPath class  

> For ElementTree, the xpath method performs a global XPath query against the document (if absolute) or against the root node (if relative)  
> When xpath() is used on an Element, the XPath expression is evaluated against the element (if relative) or against the root tree (if absolute)

而XPath类返回的值如下 

> The return value types of XPath evaluations vary, depending on the XPath expression used:  
- True or False, when the XPath expression has a boolean result
- a float, when the XPath expression has a numeric result (integer or float)
- a 'smart' string (as described below), when the XPath expression has a string result.  
- a list of items, when the XPath expression has a list as result. The items may include Elements (also comments and processing instructions), strings and tuples. Text nodes and attributes in the result are returned as 'smart' string values. Namespace declarations are returned as tuples of strings: (prefix, URI).

XPath类将传给它的表达式编译为一个可调用的函数：

```python
>>> root = etree.XML("<root><a><b/></a><b/></root>")

>>> find = etree.XPath("//b")
>>> print(find(root)[0].tag)
b
```

lxml.html
---
lxml.html是一个专门处理HTML的包。无论是lxml.html还是lxml.etree都有如下一些函数，只不过lxml.html专门用来处理HTML文件,etree还可以用来处理XML文件，以下以lxml.html来说明

- fromstring(html)  
对 lxml.html,该方法返回`Element`对象，根据参数string是整个文档还是文档片断，选择返回`document_fromstring`或者`fragment_fromstring`。

- parse(filename or url or file-like object<sup>[1]</sup>)   
返回一个`ElementTree`对象，而不是`element`对象。可以使用`parse(args).getroot()`得到文档root节点。

[1] file-like object，如`f = StringIO('<foo><bar></bar></foo>')`

- tostring(doc)  
将文档转为string。

- Element(*args, **kw)  
创建一个新的HTML(lxml.html)或者Element(lxml.etree)元素。

eg:

```python
>>> htest = """
<!DOCTYPE html>
<html>
  <head>
    <title>This is a test</title>
  </head>
  <body>
    <h3>test</h3>
  </body>
</html>
"""
... ... ... ... ... ... ... ... ... ... ... >>> doctest = lxml.html.fromstring(htest)
>>> type(doctest)
<class 'lxml.html.HtmlElement'>
>>> print doctest
<Element html at 0x17f5d10>
>>> docstring = lxml.html.tostring(doctest)
>>> type(docstring)
<type 'str'>
>>> print docstring
<html><head><title>This is a test</title></head><body>
    <h3>test</h3>
  </body></html>
>>>
>>> gtest = lxml.html.parse("http://www.google.com")
>>> type(gtest)
<type 'lxml.etree._ElementTree'>
>>> type(gtest.getroot())
<class 'lxml.html.HtmlElement'>
>>> 
```

------
List of Reference:
- [lxml](http://lxml.de/)
- [XML Path Language (XPath)](http://www.w3.org/TR/xpath/)
- [xpath教程](http://www.w3school.com.cn/xpath/index.asp)
