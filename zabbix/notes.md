Zabbix
===

权限
---

若在 zabbix 中使用 netstat 等这样的命令，在 server 端使用 zabbix_get 可能获取不到数据。而在客户端运行相应脚本时，是可以运行的，这里大致可能推测出脚本中部分命令权限问题。通过查看 zabbix 日志，可能找不到该脚本的运行。这里很自然的想到是权限问题。

在 Linux 中，文件权限除了 r, w, x 外，还有 s, t, i, a 等权限位。这里介绍下。

- s   
 给文件设置 setuid 位后，普通用户可能通过以 root 用户的角色去运行该文件。设置 s 权限位文件的属主必须先有 x 权限，否则 s 权限位并不能生效。使用 `chmod +s filename` 命令设置 s 位。

- t   
 设置粘着位。当该位被设置后，只有属主和 root 有删除文件的权限。通过 `chmod +t filename` 来设置粘着位。

- i  
 不可修改权限。当该位被设置后，任何人都不能删除文件。通过 `chattr u+i filename` 来设置该位。

- a  
 只追加权限。设置该位后，文件只能追加，不能删除，而且不能使用编辑器追加。通过 `chattr +a filename` 来设置该位。
  
这里有两种方法来解决这个问题，第一种是让 zabbix 用户可以执行 netstat （在 `/etc/sudoers` 中设置），另外一种是给 netstat 设置 s 位。
