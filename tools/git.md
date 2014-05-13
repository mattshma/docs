Git
===

更新fork的仓库
---
当fork的仓库更新后，自己需要同步这些新提交的内容，操作如下：

```
$ git remote add upstream git@github.com:beitian/docs.git #upstream 表示上游代码库名称
$ git remote update upstream
$ git rebase upstream/master
$ git push  
```
可以参考[git rebase](http://blog.csdn.net/mliubing2532/article/details/7577843)

