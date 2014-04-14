Socket
===

主机字节序和网络字节序
---
主机字节序包括 Big-Endian 和 Little-Endian， 不同的CPU有不同的字节序类型，字节序是指数据在内存中保存的顺序。Big-Endian 和 Little-Endian 的定义分别如下：

- Big-Endian  
  高位字节放在内存的低地址端，低位字节放在内存的高地址端
- Little-Endian  
  低位字节放在内存的低地址端，高位字节放在内存的高地址端

如数字 0x12345678 在内存中的表示形式：  
- Big-Endian
Low Address -------> High Address
  0x12 | 0x34 | 0x56 | 0x78

- Little-Endian
Low Address -------> High Address
  0x78 | 0x56 | 0x34 | 0x12

TCP/IP协议规定，网络字节序应采用大端字节序。因此小端机器在发送/接收数据时，数据需要先进行小端到大端的转换，可以调用以下`arpa/inet.h`头文件中的库函数做网络字节序和主机字节序的转换：

- uint32\_t htonl(uint32\_t hostlong);
- uint16\_t htons(uint16\_t hostshort);
- uint32\_t ntohl(uint32\_t netlong);
- uint16\_t ntohs(uint16\_t netshort);

其中h表示host，n表示network，l表示32位长整数，s表示16短整数。

