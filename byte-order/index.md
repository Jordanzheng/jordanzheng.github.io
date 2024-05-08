# 字节序：大端和小端


 ## 一、字节序 

**字节序**，也就是字节的顺序，指的是多字节的数据在内存中的存放顺序。

>在几乎所有的机器上，多字节对象都被存储为连续的字节序列。例如在[C语言](https://zh.wikipedia.org/wiki/C语言)中，一个类型为`int`的变量`x`地址为`0x100`，那么其对应地址表达式`&x`的值为`0x100`。且`x`的四个字节将被存储在[存储器](https://zh.wikipedia.org/wiki/存储器)的`0x100, 0x101, 0x102, 0x103`位置

## 二、大端与小端

根据x在连续的4字节内存中存储的顺序，字节序分为**大端序（Big Endian）** 与 **小端序（Little Endian）**两类，数值`0x1234`使用两个字节储存：高位字节是`0x12`，低位字节是`0x34`

- 大端序：高位字节在前，低位字节在后，人类读写数值的方法
- 小端序：低位字节在前，高位字节在后，通常x86架构以小端序存储数据

<!-- more -->

如图所示：

![img](https://img-blog.csdn.net/20150501200116979)

- Big Endian 是指低地址端 存放 高位字节。
- Little Endian 是指低地址端 存放 低位字节

###  1、为啥有大小端之分？

> 答案是，计算机电路先处理低位字节，效率比较高，因为计算都是从低位开始的。所以，计算机的内部处理都是小端字节序。在计算机内部，小端序被广泛应用于现代性 CPU 内部存储数据；而在其他场景譬如网络传输和文件存储使用大端序。
>

>使用小端序时不移动字节就能改变 number 占内存的大小而不需内存地址起始位。比如我想把四字节的 int32 类型的整型转变为八字节的 int64 整型，只需在小端序末端加零即可。
>
>```go
>44 33 22 11
>44 33 22 11 00 00 00 00
>```
>
>上述扩展或缩小整型变量操作在编译器层面非常有用，但在网络协议层非也。
>
>在网络协议层操作二进制数字时约定使用大端序，大端序是网络字节传输采用的方式。因为大端序最高有效字节排在首位（低地址端存放高位字节），能够按照字典排序，所以我们能够比较二进制编码后数字的每个字节。
>
>```go
>fmt.Println(bytes.Equal([]byte("Go"), []byte("Go")))  // true
>```

### 2、为什么要注意字节序?

当程序要与别的程序交互的时候，就涉及到字节序的处理。字节序的处理原则就是

**"只有读取的时候，才必须区分字节序，其他情况都不用考虑。"**

处理器读取外部数据的时候，必须知道数据的字节序，将其转成正确的值。然后，就正常使用这个值，完全不用再考虑字节序。

即使是向外部设备写入数据，也不用考虑字节序，正常写入一个值即可。外部设备会自己处理字节序的问题。

## 三、网络序和主机序

**网络字节序**：TCP/IP各层协议将字节序定义为 Big Endian，因此TCP/IP协议中使用的字节序是大端序。

**主机字节序**：整数在内存中存储的顺序，现在 Little Endian 比较普遍。（不同的 CPU 有不同的字节序）

> 在进行网络通信时 通常需要调用相应的函数进行主机序和网络序的转换。Berkeley socket API 定义了一组转换函数，用于16和32bit整数在网络序和本机字节序之间的转换。htonl，htons用于本机序转换到网络序；ntohl，ntohs用于网络序转换到本机序。



## 四、各语言关于字节序的处理

## 1、Java

JAVA字节序默认是**大端序（Big Endian）**。由于JVM会根据底层的操作系统和CPU自动进行字节序的转换，所以我们使用java进行网络编程，几乎感觉不到字节序的存在。

```java
import java.nio.ByteBuffer;
import java.nio.ByteOrder;
import java.util.Arrays;
 
public class JVMEndianTest {
	
	public static void main(String[] args) {
		
		int x = 0x01020304;
		
		ByteBuffer bb = ByteBuffer.wrap(new byte[4]);
		bb.asIntBuffer().put(x);
		String ss_before = Arrays.toString(bb.array());
		
		System.out.println("默认字节序 " +  bb.order().toString() +  ","  +  " 内存数据 " +  ss_before);
		
		bb.order(ByteOrder.LITTLE_ENDIAN);
		bb.asIntBuffer().put(x);
		String ss_after = Arrays.toString(bb.array());
		
		System.out.println("修改字节序 " + bb.order().toString() +  ","  +  " 内存数据 " +  ss_after);
	}
}
```



## 2、C++

 C/C++ 存储数据时的字节序依赖所在平台的CPU，所以可以通过C/C++程序判定机器的端序：

```c++
void Endianness()
{
	int a = 0x12345678;
	if( *((char*)&a) == 0x12)
		cout << "Big Endian" << endl;
	else
		cout << "Little Endian" << endl;
}
```

## 3、go 

Go中处理大小端序的代码位于 `encoding/binary` ,包中的全局变量BigEndian用于操作大端序数据，LittleEndian用于操作小端序数据，这两个变量所对应的数据类型都实行了ByteOrder接口。

```go
package main

import (
	"encoding/binary"
	"fmt"
	"unsafe"
)

const INT_SIZE = int(unsafe.Sizeof(0)) //64位操作系统，8 bytes

//判断我们系统中的字节序类型
func systemEdian() {
    
	var i = 0x01020304
	fmt.Println("&i:",&i)
	bs := (*[INT_SIZE]byte)(unsafe.Pointer(&i))

	if bs[0] == 0x04 {
		fmt.Println("system edian is little endian")
	} else {
		fmt.Println("system edian is big endian")
	}
	fmt.Printf("temp: 0x%x,%v\n",bs[0],&bs[0])
	fmt.Printf("temp: 0x%x,%v\n",bs[1],&bs[1])
	fmt.Printf("temp: 0x%x,%v\n",bs[2],&bs[2])
	fmt.Printf("temp: 0x%x,%v\n",bs[3],&bs[3])

}


func testBigEndian() {

	var testInt int32 = 0x01020304
	fmt.Printf("%d use big endian: \n", testInt)
	var testBytes []byte = make([]byte, 4)
	binary.BigEndian.PutUint32(testBytes, uint32(testInt))
	fmt.Println("int32 to bytes:", testBytes)
	fmt.Printf("int32 to bytes: %x \n", testBytes)

	convInt := binary.BigEndian.Uint32(testBytes)
	fmt.Printf("bytes to int32: %d\n\n", convInt)
}

func testLittleEndian() {

	var testInt int32 = 0x01020304
	fmt.Printf("%x use little endian: \n", testInt)
	var testBytes []byte = make([]byte, 4)
	binary.LittleEndian.PutUint32(testBytes, uint32(testInt))
	fmt.Printf("int32 to bytes: %x \n", testBytes)

	convInt := binary.LittleEndian.Uint32(testBytes)
	fmt.Printf("bytes to int32: %d\n\n", convInt)
}

func main() {
	systemEdian()
	fmt.Println("")
	testBigEndian()
	testLittleEndian()
}
```

输出：

>&i: 0xc00000a0b8
>system edian is little endian
>temp: 0x4,0xc00000a0b8
>temp: 0x3,0xc00000a0b9
>temp: 0x2,0xc00000a0ba
>temp: 0x1,0xc00000a0bb
>
>16909060 use big endian: 
>int32 to bytes: [1 2 3 4]
>int32 to bytes: 01020304 
>bytes to int32: 16909060
>
>1020304 use little endian: 
>int32 to bytes: 04030201 
>bytes to int32: 16909060



## 五、参考文章

[理解字节序](http://www.ruanyifeng.com/blog/2016/11/byte-order.html)

[golang之大端序、小端序](https://studygolang.com/articles/17887)

[字节序及 Go encoding/binary 库](https://www.golang123.com/topic/1784)

[字节顺序](https://zh.wikipedia.org/wiki/%E5%AD%97%E8%8A%82%E5%BA%8F)


