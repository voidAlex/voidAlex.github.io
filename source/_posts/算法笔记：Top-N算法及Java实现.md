---
title: 算法笔记：Top-N算法及Java实现
date: 2017-08-21 23:17:43
tags: ['算法','排序','Java']
mathjax: true
---
> 这是voidAlex原创的第七篇博文。

给定一组无序的数据，需要创建一个最大的N条记录的列表，这类问题是经典的Top-N问题。

<!-- more -->

系统中常常会有这样的需求：将大量的（几百万甚至上千万）的数据排序,然后取出最Top的N条作为展示。常见的解决方案如下：

1. 使用传统的排序算法，即使用List中的Sort方法排序，然后取出前N个。最坏时间复杂度达到了$O(n^2)$;
2. 维护一个容量为N的最大堆或者排序二叉树，遍历整个List，取出前面的N个放到堆里。最坏时间复杂度为$O(nlogN)$。

使用Java集合类中的`TreeMap`可以很容易的实现一个Top-N算法：维护一个大小为N的`TreeMap`topN，遍历所有数据，并将其添加到topN中。如果`topN.size() > N`，就删除topN的第一个元素（值最小的元素）。

以下是示例代码：

```Java
private SortedMap<Double, String> topN(int n, List l){
    SortedMap<Double, String> topN = null;
    if (l != null && !l.isEmpty()){
        topN = new TreeMap<Double, String>();

        for (DataNode dataNode : l){
            topN.put(dataNode.getValue(),dataNode.getkey());
            if (topN.size() > n){
                topN.remove(topN.firstKey());
            }
        }
    }
    return topN;
}
```