# NFS服务搭建

在实际工作中，用到了nfs，这里纪录下。

首先在服务端和客户端都安装showmount: `yum -y install showmount`。

## 服务端
服务端先设置要共享的目录及共享给谁，编辑`/etc/exports`

```
#共享目录      主机名称或者IP(参数1，参数2）
/data          10.2.19.*(rw,sync,no_root_squash)
```

参数如下：    
- rw     
  可读写。 
- ro     
  只读。
- no_root_squash      
  若登录到NFS主机的用户如果是root，则该用户拥有root权限。
- root_squash     
  若登录NFS主机的用户如果是root，该用户权限将被限定为匿名使用者nobody。
- all_squash    
  不管登录NFS主机的用户是用户，都被重新设定为匿名使用者nobody。 
- anonuid     
  将登录NFS主机的用户都设定成指定的user id，此ID必须存在于/etc/passwd中。 
- anongid    
  同anonuid，将登录NFS主机的用户都设置为指定的group id。
- sync     
  资料同步写入磁盘。 
- async   
  资料会先暂时存放在内存中，不会直接写入硬盘。 
- insecure     
  允许从这台机器过来的非授权访问。 

接着启用nfs服务即可`/etc/init.d/nfs start`。

## 客户端
客户端可通过`showmount -e 服务端ip`显示服务端的nfs目录。然后通过`mount -t nfs 服务端ip:/nfs目录 客户端目录`即可。

以上即完成搭建nfs服务。

若在使用nfs的过程修改了服务端的`/etc/exports`，可在服务端通过`exportfs -r`来重新加载分享的目录。



