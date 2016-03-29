# `ls -al`返回项说明

`ls -al`除了返回rwx外，还会返回其他的一些值。引用[linux forum](http://www.unix.com/tips-and-tutorials/19060-unix-file-permissions.html)中的一张图。

``` 
 |------file mode------|
 |                     |
 |
 |       |----full-----|
         |
 |-type| |   |--basic--|
 |     | |   |         |
 oo0 000 000 000 000 000
 ... ... ... ... ... ...
    |     |   |   |   |
    |     |   |   |   |---- rwx for other
    |     |   |   |
    |     |   |   |-------- rwx for group
    |     |   |
    |     |   |------------ rwx for user
    |     |
    |     |---------------- set uid, set gid, sticky bit
    |
    |---------------------- file type: regular (-)
                                       directory (d)
                                       character special (c)
                                       block special (b)
                                       fifo (p)
                                       symbolic link (l)
                                       socket (s)

```

## Setuid bit
当文件设置SUID后，执行该文件的用户以该命令拥有者的权限去执行。即无论谁来执行这个文件，都会以该文件所有者的身份执行。suid只对文件有效，设置suid的命令为`chmod u+s filename`。

## Setgid bit
只对目录有效。目录被设置该位后, 任何用户在此目录下创建的文件，都具有和该目录所属的组相同的组。即若目录A所属组为`www-data`，则无论在A下使哪种用用户或组，创建的新文件所属组仍为`www-data`。设置guid的命令为`chmod g+s folder`。

## Sticky bit
粘滞位。主要用来防止删除该文件，设置该位，即使用户对该文件有写权限，也无法删除该文件。粘滞位对文件有效，设置粘滞的命令为`chmod o+t filename`。

另外，和rwx一样，从上图可知还可通过八进制形式设置这3个位。对应的位如下：

```
 abc
 |||
 || `------ sticky bit。若设置，则该位为1。
 ||
 | `------- setgid bit。若设置，则该位为2。 
 | 
 `--------- setuid bit。若设置，则该位为4。
```

即可设置文件权限为4777，2664等。通过ls -al查看标志位，如：

- rwsrw-r-- 表示有setuid标志
- rwxrwsrw- 表示有setgid标志
- rwxrw-rwt 表示有sticky标志

可看若设置了标志位，对应x位会被代替。如果本来在该位上有x, 则这些特殊标志显示为小写字母 (s, s, t)。否则, 显示为大写字母 (S, S, T)

## acl
有时候文件显示为`-rw-r--r--+`，后面的`+`符号代表该文件或目录设置了acl。通过`getfacl filename`可查看。

## selinux acl
若文件显示为`-rw-r--r--.`，说明开启了SELinux ACL。



