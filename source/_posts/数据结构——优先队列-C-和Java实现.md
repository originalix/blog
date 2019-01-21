---
title: 数据结构——优先队列(C++和Java实现)
date: 2017-08-18 21:10:11
tags: ["优先队列", "数据结构"]

---

十几天没有更新自己的博客了，因为目前在算法和数据结构的学习中，碰到了一些问题，例如之前就在优先队列，堆这个数据结构面前，感觉到有点吃不透概念，而使用的那本书上写的实在太抽象了，所以又查找了很多资料，最终对优先队列这个数据结构有了一定的了解。花了点时间才啃下来的知识，当然要把它记录下来了，所以今天就来回顾一下优先队列。

<!--more-->

优先队列也是一种抽象数据类型。优先队列中的每个元素都有各自的优先级。这个概念其实打几个比方会理解的比较快一点。比如我们人人都用过的windows系统，当我们打开任务管理器的时候，每个任务的优先级别是不同的，而操作系统会选择优先级别最高的任务先执行，同时我们也能在选项里标记任务的优先级。再比如夜班的急诊大夫，如果之前来了两三个感冒发烧的病人正在排队看病，这时候，匆匆忙忙抬进来一个心脏病突发的病人，那我们的大夫当然要先去治疗心脏病突发的病人了。优先队列也是一个道理，优先处理优先级别高的数据或者任务。

优先级最高的元素最先得到服务，优先级别相同的元素按照其在优先队列中的顺序得到服务。优先队列往往用堆来实现。

优先队列至少要支持这些操作：

- 插入带优先级的元素。
- 取出具有最高优先级的元素。
- 查看最高优先级的元素。（O(1)的时间复杂度）

出于性能的考虑，优先队列用堆来实现，具有O(log n)时间复杂度的插入元素性能，O(n)的初始化构造的时间复杂度。如果使用自平衡二叉查找树，插入与删除的时间复杂度为O(log n),构造二叉树的时间复杂度为O(nlogn)。

而从时间复杂度的角度，优先队列其实等价于排序算法。而接下来我们就要用C++和Java两种编程语言来实现优先队列。为什么现在要用两种语言呢，其实仅仅是我在使用了C++写完了数据结构之后，改换Java又实现了一遍，经过测试，代码是通过并满足优先队列的性质的，所以一起放出来了。

```c++

#include <iostream>
#include <algorithm>
#include <string>
#include <ctime>
#include <cmath>
#include <cassert>

using namespace std;

template <typename Item>
class MaxHeap {

private:
    Item *data;
    int count;
    int capacity;

    void shiftUp( int k ) {
        while (k > 1 && data[k/2] < data[k]) {
            swap( data[k/2], data[k] );
            k /= 2;
        }
    }

    void shiftDown( int k ) {
        while (2*k <= count) {
            int j = 2 * k; // 在此轮循环中,data[k]和data[j]交换位置
            if (j + 1 <= count && data[j] < data[j+1])
                // data[j] 是 data[2*k]和data[2*k+1]中的最大值
                j += 1;
            if (data[k] > data[j])
                break;
            swap( data[k], data[j] );
            k = j;
        }
    }

public:
    // 构造函数, 构造一个空堆, 可容纳capacity个元素
    MaxHeap(int capacity) {
        data = new Item[capacity + 1];
        count = 0;
        this->capacity = capacity;
    }

    MaxHeap(Item arr[], int n) {
        data = new Item[n+1];
        capacity = n + 1;
        for (int i = 0; i < n; i++)
            data[i+1] = arr[i];
        count = n;
        for (int i = count / 2; i >= 1; i--)
            shiftDown(i);
    }

    ~MaxHeap() {
        delete[] data;
    }

    // 返回堆中的元素个数
    int size() {
        return count;
    }

    // 返回一个布尔值, 表示堆中是否为空
    bool isEmpty() {
        return count == 0;
    }

    // 像最大堆中插入一个新的元素 item
    void insert(Item item) {
        assert(count + 1 <= capacity);
        data[ count + 1 ] = item;
        count++;
        shiftUp( count );
    }

    // 从最大堆中取出堆顶元素, 即堆中所存储的最大数据
    Item extractMax() {
        assert(count > 0);

        Item ret = data[1];

        swap( data[1], data[count] );
        count--;

        shiftDown( 1 );

        return ret;
    }

    // 获取最大堆中的堆顶元素
    Item getMax(){
        assert( count > 0 );
        return data[1];
    }
};
```

以上是C++版本的实现，接下来是Java版本的实现，测试代码写在java里面，C++的测试也是一样的用例。

```java
public class MaxHeap<Item extends Comparable> {
    protected Item[] data;
    protected int count;
    protected int capacity;

    // 构造函数，构造一个空堆，可容纳capacity个元素
    MaxHeap(int capacity) {
        data = (Item[])new Comparable[capacity + 1];
        count = 0;
        this.capacity = capacity;
    }

    // 返回堆中的元素个数
    public int size() {
        return count;
    }

    // 返回一个布尔值，表示堆中是否为空
    public boolean isEmpty() {
        return count == 0;
    }

    // 向最大堆中插入一个新元素 item
    public void insert(Item item) {
        assert (count + 1 <= capacity);
        data[count + 1] = item;
        this.count++;
        shiftUp( count );
    }

    // 从最大堆中取出堆顶元素，即堆中所存储的最大数据
    public Item extractMax() {
        assert (count > 0);
        Item ret = data[1];
        swap(1, count);
        count--;
        shiftDown( 1 );
        return ret;
    }

    // 获取最大堆中的堆顶元素
    public Item getMax() {
        assert (count > 0);
        return data[1];
    }

    // 交换堆中索引为i和j的两个元素
    private void swap(int i, int j) {
        Item item = data[i];
        data[i] = data[j];
        data[j] = item;
    }

    //********************
    //* 最大堆核心辅助函数
    //********************
    private void shiftUp(int k) {
        while (k > 1 && data[k/2].compareTo(data[k]) < 0) {
            swap(k/2, k);
            k /= 2;
        }
    }

    private void shiftDown(int k) {
        while (2 * k <= count) {
            int j = 2 * k;
            if (j + 1 <= count && data[j].compareTo(data[j+1]) < 0) {
                j += 1;
            }
            if (data[k].compareTo(data[j]) > 0) {
                break;
            }
            swap(k, j);
            k = j;
        }
    }

    //测试MaxHeap
    public static void main(String[] args) {
        MaxHeap<Integer> maxHeap = new MaxHeap<Integer>(50);
        int N = 50;
        int M = 100;
        for (int i = 0; i < N; i++) {
            maxHeap.insert( new Integer((int) (Math.random() * M)) );
        }

        Integer[] arr = new Integer[N];
        // 将maxHeap中的数据使用extractMax()取出来
        // 取出来的顺序应该是从大到小排序的
        for (int i = 0; i < N; i++) {
            arr[i] = maxHeap.extractMax();
            System.out.print(arr[i] + " ");
        }
        System.out.println();

        // 确保arr数组是从大到小排序的
        for (int i = 1; i < N; i++) {
            assert arr[i-1] >= arr[i];
        }
    }
}
```