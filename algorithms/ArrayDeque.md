重点关注运算符操作，其它忽略



ArrayDeque 对数组的大小(即队列的容量)有特殊的要求，必须是 2^n。通过 `allocateElements`方法计算初始容量：

```java
private void allocateElements(int numElements) {
    int initialCapacity = MIN_INITIAL_CAPACITY;
    // Find the best power of two to hold elements.
    // Tests "<=" because arrays aren't kept full.
    if (numElements >= initialCapacity) {
        initialCapacity = numElements;
        initialCapacity |= (initialCapacity >>>  1);
        initialCapacity |= (initialCapacity >>>  2);
        initialCapacity |= (initialCapacity >>>  4);
        initialCapacity |= (initialCapacity >>>  8);
        initialCapacity |= (initialCapacity >>> 16);
        initialCapacity++;

        if (initialCapacity < 0)   // Too many elements, must back off
            initialCapacity >>>= 1;// Good luck allocating 2 ^ 30 elements
    }
    elements = new Object[initialCapacity];
}
```
因为numElements的最高位肯定是1，最高位后面全部补1即可

```java
0 0 0 0 1 ? ? ? ? ?     //n 
0 0 0 0 1 1 ? ? ? ?     //n |= n >>> 1;
0 0 0 0 1 1 1 1 ? ?     //n |= n >>> 2;
0 0 0 0 1 1 1 1 1 1     //n |= n >>> 4;
```

在进行5次位移操作和位或操作后就可以得到2^k-1，最后加1即可。这个实现还是很巧妙的。

```java
public void addLast(E e) {
        if (e == null)
            throw new NullPointerException();
        //tail 中保存的是即将加入末尾的元素的索引
        elements[tail] = e;
        //tail 向后移动一位
        //把数组当作环形的，越界后到0索引
        if ( (tail = (tail + 1) & (elements.length - 1)) == head)
            //tail 和 head相遇，空间用尽，需要扩容
            doubleCapacity();
}
```

这段代码中，`(tail = (tail + 1) & (elements.length - 1)) == head`这句有点难以理解。其实，在 ArrayDeque 中数组是当作**环形**来使用的，索引0看作是紧挨着索引(length-1)之后的。参考下面的图片：

![循环队列](/Users/li/Documents/note/noteImgPool/循环队列.png)

那么为什么`(tail + 1) & (elements.length - 1)`就能保证按照环形取得正确的下一个索引值呢？这就和前面说到的 ArrayDeque 对容量的特殊要求有关了。

下面对其正确性加以验证：

```shell
length = 2^n，二进制表示为: 第 n 位为1，低位 (n-1位) 全为0 
length - 1 = 2^n-1，二进制表示为：低位(n-1位)全为1

如果 tail + 1 <= length - 1，则位与后低 (n-1) 位保持不变，高位全为0
如果 tail + 1 = length，则位与后低 n 全为0，高位也全为0，结果为 0
```



可见，在容量保证为 2^n 的情况下，仅仅通过位与操作就可以完成*环形*索引的计算，而不需要进行边界的判断，在实现上更为高效。

向头部添加元素的代码如下：

```java
public void addFirst(E e) {
    if (e == null) //不支持值为null的元素
        throw new NullPointerException();
    elements[head = (head - 1) & (elements.length - 1)] = e;
    if (head == tail)
        doubleCapacity();
}
```

其它的诸如add，offer，offerFirst，offerLast等方法都是基于上面这两个方法实现的，不再赘述。