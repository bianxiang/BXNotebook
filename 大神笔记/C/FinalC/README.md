[toc]

现在主流的计算平台（x86和ARM），int 都是4字节的。

## sizeof

sizeof的操作数是数组时，得到的是这个那个数组占用内存的大小：

```c
int array[5];
sizeof(array);
```

在32位机器上，内存地址长度都是4字节，因此不管什么类型的指针，对它使用sizeof的结果都是4。