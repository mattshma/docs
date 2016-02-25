# sudo和su免密码登录

## su免密码
su用来切换用户，切换时需要输入目录用户的密码。若需配置用户A su无密码切换用户，需先将用户A加入wheel组，然后修改`/etc/pam.d/su`。如下：
```
# usermod -G wheel A
```
然后修改`/etc/pam.d/su`文件：
```
# Uncomment the following line to implicitly trust users in the "wheel" group.
auth        sufficient  pam_wheel.so trust use_uid
```

这样即可su无密码登录到任意帐号了。su仅执行一次命令可通过`su - B -c "shell_command"`。

## sudo免密码
sudo在切换用户时，需要输入用户本身的密码，也可配置不输入任何密码。所以必须`/etc/sudoers`内的用户才能使用sudo指令。如下:
```
# chmod u+w /etc/sudoers
```
然后修改`/etc/sudoers`：
```
## Same thing without a password
%wheel  ALL=(ALL)   NOPASSWD: ALL
```
保存后去掉`/etc/sudoers`的写权限:
```
# chmod u-w /etc/sudoers
```

sudo仅执行一次命令可通过`sudo -u B shell_command`。
