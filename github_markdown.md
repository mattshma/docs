markdown语法总结
---

注：本总结针对github中markdown的讲法

目录
----
- 图片
- 表格
- 代码

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

> head1 | head2 | head3
> ------|-------|------
>   t1  |  t2   |   t3
>   t4  |  t5   |   t6

 *注意表格上下均有空行*
如

   歌曲名  |  歌手
 ----------|--------
  人来人往 | 陈奕迅
   矜持    | 王菲
   最爱    | 张国荣
   有心人  | 张国荣

- 代码
在github中有3种方式表示代码
 - 4个空格或者1个tab表示代码块
 如下
            
            int main() {
                print("hello world");           
            }
            
 注意在一级列表中，代码块前面需要8个空格，在二级列表中，代码块前面需要12个空格，依此类推。

- \` \`表示代码片断
 如

           `print("hello world")`

 在github显示为 `print("hello world")`
 
 - \`\`\`\`表示加强型的代码块
 如：
 
            ````ruby
            print "hello world"
            ````
在github的显示是
````ruby
 print "hello world"
````


