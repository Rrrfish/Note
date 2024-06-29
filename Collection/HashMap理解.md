# `HashMap`理解

## `HashMap` 简介

`HashMap` 主要用来存放键值对，它基于哈希表的 `Map` 接口实现，是常用的 `Java` 集合之一，是非线程安全的。

`HashMap` 可以存储 `null` 的 `key` 和 `value`，但 `null` 作为键只能有一个，`null` 作为值可以有多个。在计算哈键的哈希值时，`null `键哈希值为 0。

`HashMap` 默认的初始化大小为 16。之后每次扩充，容量变为原来的 2 倍。并且， `HashMap` 总是使用 2 的幂作为哈希表的大小。

### `JDK 1.8`之前

采用拉链法。`HashMap`由数组+链表组成。链表负责解决哈希冲突。

```JAVA
// JDK1.7的哈希方法
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

扰动四次，性能比1.8的差一些。可以对比看`JDK1.8`的扰动函数。

然后通过 `(n - 1) & hash` 判断当前元素存放的位置（这里的 n 指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的 hash 值以及 key 是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。





### `JDK 1.8`以后

采用拉链法。但是`HashMap`由数组+链表或红黑树组成。当链表长度大于8时，查看数组长度，若小于64则进行数组扩容；若大于64，将链表转换为红黑树，以减少搜索时间。

`HashMap` 通过 key 的 `hashCode` 经过扰动函数处理过后得到 hash 值。

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

`.hashCode()`得到32位数值，`(h >> 16)`将哈希值右移16位，再将右移后的值和原值相异或，使得哈希值高低16位混合，**增大随机性**，让数据元素更加**均衡**的散列，减少碰撞。





## 红黑树

