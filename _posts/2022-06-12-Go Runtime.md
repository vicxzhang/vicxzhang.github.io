---
title: Go Runtime
author: 142857
date: 2022-06-12 22:19:00 +0800
categories: [Go]
tags: [Go]
math: true
mermaid: true
---



## 1. 数组（Array）

### 初始化

```go
arr1 := [3]int{1, 2, 3}
arr2 := [...]int{1, 2, 3}
```

- 后一种声明方式在编译期会转换成前一种。

- 当数组元素 <= 4，会将数组元素放置在栈上；

- 当数组元素 > 4，会将数组元素放置在静态区并在运行时取出。

### 访问和赋值

- 访问数组的索引是非整数时，报错 `“non-integer array index %v”`。

- 访问数组的索引是负数时，报错  `“invalid array index %v (index must be non-negative)"`。

- 访问数组的索引越界时，报错  `“invalid array index %v (out of bounds for %d-element array)"`。



## 2. 切片（Slice）

![image-20220613020103911](https://s2.loli.net/2022/06/13/1IX3TzBm7ydp6cE.png){: style="max-width: 50%" .central}

```go
type SliceHeader struct {
	Data uintptr
	Len  int
	Cap  int
}
```



### 初始化

```go
arr[0:3], slice[0:3]
slice := []int{1, 2, 3}
slice := make([]int, 0, 10)
```

- 下标方式是最原始、最接近汇编的方式。
- 字面量方式主要在编译器完成，会先根据字面量创建一个数组 arr ，再使用 arr[:] 生成切片。
- make 方法主要在运行时，检查切片大小，检查是否发生逃逸，切片太大或发生逃逸，会在堆上初始化切片。



### 追加和扩容

- 追加
  - 当 `newlen <= cap`，不需要扩容，直接追加元素。
  - 当 `newlen > cap`，出发扩容。
- 扩容：
  - 若 `newlen > cap * 2`，`newlen = newcap = cap * 2`。
  - 若 `cap <= 1024`，`newcap = cap * 2，newlen = newlen`。
  - 若 `cap > 1024`，`newcap = cap * 1.25` 直至 `newcap >= cap`，`newlen = newlen`。



## 3. 哈希表（Map）



```go
type hmap struct {
	count     int			// 哈希表元素数量
	flags     uint8			
    B         uint8			// len(buckets) == 2^B
	noverflow uint16		
	hash0     uint32		// 哈希种子

	buckets    unsafe.Pointer	// bucket 数组
	oldbuckets unsafe.Pointer	// 哈希扩容时，老的 bucket 数组
	nevacuate  uintptr

	extra *mapextra
}

type mapextra struct {
	overflow    *[]*bmap
	oldoverflow *[]*bmap
	nextOverflow *bmap
}

type bmap struct {
    topbits  [8]uint8
    keys     [8]keytype
    values   [8]valuetype
    pad      uintptr
    overflow uintptr		// 拉链指针
}
```

![image-20220613024233211](https://s2.loli.net/2022/06/13/IsOQcynVzpAYMFl.png){: style="max-width: 70%" .central}



### 初始化

```go
hash := map[string]int{"1": 1, "2": 4, "3": 9}
hash := make(map[string]int, 10)
```

- 两种方法都会使用到 make 来创建哈希表。



### 访问

![image-20220613112027330](https://s2.loli.net/2022/06/13/ExtaNcJzYAVqFiv.png){: style="max-width: 70%" .central}

```go
v := hash[key]
v, ok := hash[key]
```

- `hash = hashFunction(key)`，根据 `hash % len(buckets)` 选择桶序号。
- 遍历正常桶和溢出桶中数据，先比较 `tophash[i]` 和 `hash` 高 8 位，再比较 `key[i]` 和 `key`，最后返回 `values[i]`。



### 写入

![image-20220613113407273](https://s2.loli.net/2022/06/13/Y4SnrqC16ZXIoyf.png){: style="max-width: 70%" .central}

```go
hash[key] = val
```

- `hash = hashFunction(key)`，根据 `hash % len(buckets)` 选择桶序号。
- 遍历正常桶和溢出桶中数据，先比较 `tophash[i]` 和 `hash` 高 8 位，再比较 `key[i]` 和 `key`。
- 若存在 `key[i] == key`，修改 `values[i] = val`。
- 若不存在 `key[i] == key`，添加新元素。
- 若桶未满，直接填入 `tophash[i]`、`key[i]`、`values[i]`。
- 若桶已满，创建新的溢出桶，直接填入 `tophash[i]`、`key[i]`、`values[i]`。



### 扩容







## 4. 字符串（String）





