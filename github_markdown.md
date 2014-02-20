markdown语法总结
====

注：本总结针对github中markdown的语法

目录
----
- 图片
- 表格
- 代码
- 缩进
- 空格

----------

- 图片  
插入图片的语法为

        ![title](url "description")

    可以先在git仓库中新建文件夹，然后将图片放到该文件夹中。
    如  
    
        ![test](https://raw2.github.com/hqjma/docs/master/img/identicon.png "desc")

    显示结果是

    ![test](https://raw2.github.com/hqjma/docs/master/img/identicon.png "desc")

- 表格  
插入表格的语法为

        head1 | head2 | head3
        ------|-------|------
        t1  |  t2   |   t3
        t4  |  t5   |   t6

 *注意表格上下均有空行*  
 如
 
~~~ 
    歌曲名  |  歌手
 ----------|--------
  人来人往 | 陈奕迅
   矜持    | 王菲
   最爱    | 张国荣
   有心人  | 张国荣
~~~

在github中的显示为  

   歌曲名  |  歌手
 ----------|--------
  人来人往 | 陈奕迅
   矜持    | 王菲
   最爱    | 张国荣
   有心人  | 张国荣

- 代码   
在github中有3种方式表示代码  
 - 4个空格或者1个tab表示代码块  
 如
            
            int main() {
                print("hello world");           
            }
            
    注意在一级列表中，代码块前面需要8个空格，在二级列表中，代码块前面需要12个空格，依此类推。

 - \` \`表示代码片断  
    如

           `print("hello world")`

    在github显示为 `print("hello world")`
 
 - \`\`\`表示加强型的代码块  
    如
 
            ```ruby
            print "hello world"
            ```
 
    在github的显示是  

        ```ruby
        pint "hello world"
        ````

- 缩进   
在markdown中，如果有层次关系，对于 **列表**， n级层次在n-1级的基础上前面至少多 **n-1** 个空格，则有层次关系。

        - list
         - list
           - list
          
    在github中表示如下：

    > - list
    >  - list
    >    - list

    而如果列表下有文本，希望文本和列表对齐，则文本每次以 **4** 个空格的方式缩进。

- 空格
这部分其实是HTML的内容，但有时候可能需要展示文件的组织结构。
- `&nbsp;`，不断行的空白，一个字符宽度
- `&ensp;`，半个空白，一个字符宽度
- `&emsp;`，一个空白，两个字符宽度
- `&thinsp;`，窄空白，小于一个字符宽度

如下：  
        >.  
         |---- Capfile  
         |---- config  
         |&emsp;&emsp;&emsp;|---- deploy  
         |&emsp;&emsp;&emsp;|&emsp;&emsp;&emsp;|---- production.rb  
         |&emsp;&emsp;&emsp;|&emsp;&emsp;&emsp; \`---- staging.rb  
         |&emsp;&emsp;&emsp;\`---- deploy.rb  
         \`---- lib  
         &emsp;&emsp;&emsp;\`---- capistrano  
         &emsp;&emsp;&emsp;&emsp;&emsp;&emsp;\`---- tasks 

/注：空白处为`&emsp;`/


-----
Reference:  
- [GitHub 风格的 Markdown 语法](https://github.com/cssmagic/blog/issues/13)
- [Markdown语法说明](http://uliweb.clkg.org/tutorial/view_chapter/32)
