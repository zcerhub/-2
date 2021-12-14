#### 字节的存放模式

##### 大端模式

数据的高位存放在地址的地位，数据的低位存放在地址的高位。PowerPC系列CPU

##### 小端模式

数据的高位存放在地址的高位，数据的低位存放在地址的低位。X86系列CPU

##### 实例

将十六进制0x12345678保存在初始低位为0x000000中。

| 内存地址 | 大端模式 | 小端模式 |
| -------- | -------- | -------- |
| 0x0000   | 0x12     | 0x78     |
| 0x0001   | 0x34     | 0x56     |
| 0x0002   | 0x56     | 0x34     |
| 0x0003   | 0x78     | 0x12     |

#### 无锁状态下的对象头

```java
public class User {
    private String name;
    private Integer age;
    private boolean sex;
}


public class UserDemo {

    public static void main(String[] args) {
        User user = new User();
        user.hashCode();
        System.out.println(ClassLayout.parseInstance(user).toPrintable());
    }

}
```

使用ClassLayout输出user对象。

##### 带有压缩指针

jvm的默认方式。

结果输出：

```
com.example.concurrent.innerlock.User object internals:
 OFFSET  SIZE                TYPE DESCRIPTION                               VALUE
      0     4                     (object header)                           01 f0 39 81 (00000001 11110000 00111001 10000001) (-2126909439)
      4     4                     (object header)                           44 00 00 00 (01000100 00000000 00000000 00000000) (68)
      8     4                     (object header)                           43 c1 00 f8 (01000011 11000001 00000000 11111000) (-134168253)
     12     1             boolean User.sex                                  false
     13     3                     (alignment/padding gap)                  
     16     4    java.lang.String User.name                                 null
     20     4   java.lang.Integer User.age                                  null
Instance size: 24 bytes
Space losses: 3 bytes internal + 0 bytes external = 3 bytes total
```

对象头占用12字节：8字节Mark World+4字节对象类型引用

boolean占用4字节：1字节boolean+3字节对齐填充

String、Integer各自占用4字节。总共24字节

##### 没有压缩指针

程序启动添加vm参数：-XX:-UseCompressedOops。

结果输出：

```
com.example.concurrent.innerlock.User object internals:
 OFFSET  SIZE                TYPE DESCRIPTION                               VALUE
      0     4                     (object header)                           01 f0 39 81 (00000001 11110000 00111001 10000001) (-2126909439)
      4     4                     (object header)                           44 00 00 00 (01000100 00000000 00000000 00000000) (68)
      8     4                     (object header)                           88 35 bf 25 (10001000 00110101 10111111 00100101) (633288072)
     12     4                     (object header)                           00 00 00 00 (00000000 00000000 00000000 00000000) (0)
     16     1             boolean User.sex                                  false
     17     7                     (alignment/padding gap)                  
     24     8    java.lang.String User.name                                 null
     32     8   java.lang.Integer User.age                                  null
Instance size: 40 bytes
Space losses: 7 bytes internal + 0 bytes external = 7 bytes total
```

对象头占用16字节：8字节Mark world+8字节类型指针

boolean占用8字节：1字节boolean+7字节对其填充

String、Integer对象各占用8字节。总共占用40 bytes

说明

- 测试jvm为64位
- 只有调用对象的hashcode()方法时才会将hashcode值写到对象头

##### 对象头分析

      01 f0 39 81 (00000001 11110000 00111001 10000001)
      44 00 00 00 (01000100 00000000 00000000 00000000)   
jol输出的对象头是小端模式。

可以看出锁状态为01，偏向标志位为0，hashcode值为0x448139f0。

##### 参考文献

- [大小端问题](https://weread.qq.com/web/reader/9b93254072456ac19b9a176kfbd32c4024cfbd7939d6371)
- [64位JVM的Java对象头详解](https://blog.csdn.net/baidu_28523317/article/details/104453927)