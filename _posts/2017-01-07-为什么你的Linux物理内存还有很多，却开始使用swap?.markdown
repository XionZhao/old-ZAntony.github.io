---
layout:     post
title:      "为什么你的Linux物理内存还有很多，却开始使用swap?"
subtitle:   "linux物理内存分析"
date:       2017-01-07
author:     "Antony"
header-img: "img/post-bg-js-module.jpg"
tags:
    - system
    - swap
---
>现在的机器上都是有多个`CPU`和多个`内存块`的。以前我们都是将内存块看成是一大块内存，所有`CPU`到这个`共享内存`的访问消息是一样的。

>这就是之前普遍使用的`SMP`模型。但是随着处理器的增加，共享内存可能会导致内存访问冲突越来越厉害，且如果内存访问达到瓶颈的时候，性能就不能随之增加。`NUMA（Non-Uniform Memory Access）`就是这样的环境下引入的一个模型。比如一台机器是有2个处理器，有4个内存块。我们将1个处理器和两个内存块合起来，称为一个`NUMA node`，这样这个机器就会有两个`NUMA node`。在物理分布上，`NUMA node`的处理器和内存块的物理距离更小，因此访问也更快。比如这台机器会分左右两个处理器（`cpu1`, `cpu2`），在每个处理器两边放两个内存块(memory1.1, memory1.2, memory2.1,memory2.2)，这样`NUMA node1`的cpu1访问`memory1.1`和`memory1.2`就比访问`memory2.1`和`memory2.2`更快。所以使用`NUMA`的模式如果能尽量保证本`node`内的`CPU`只访问本`node`内的内存块，那这样的效率就是最高的。

>在运行程序的时候使用`numactl -m`和`-physcpubind`就能制定将这个程序运行在哪个`cpu`和哪个`memory`中。玩转`cpu-topology`给了一个表格，当程序只使用一个`node`资源和使用多个`node`资源的比较表（差不多是38s与28s的差距）。所以限定程序在`numa node`中运行是有实际意义的。

>但是呢，话又说回来了，制定`numa`就一定好吗？`--numa`的陷阱。`SWAP`的罪与罚文章就说到了一个`numa`的陷阱的问题。现象是当你的服务器还有内存的时候，发现它已经在开始使用`swap`了，甚至已经导致机器出现停滞的现象。这个就有可能是由于`numa`的限制，如果一个进程限制它只能使用自己的`numa`节点的内存，那么当自身`numa node`内存使用光之后，就不会去使用其他`numa node`的内存了，会开始使用`swap`，甚至更糟的情况，机器没有设置`swap`的时候，可能会直接死机！所以你可以使用`numactl --interleave=all`来取消`numa node`的限制。

>综上所述得出的结论就是，根据具体业务决定`NUMA`的使用。

>如果你的程序是会占用大规模内存的，你大多应该选择关闭`numa node`的限制。因为这个时候你的程序很有几率会碰到`numa`陷阱。

>另外，如果你的程序并不占用大内存，而是要求更快的程序运行时间。你大多应该选择限制只访问本`numa node`的方法来进行处理。

转载：http://lib.csdn.net/article/linux/28546
