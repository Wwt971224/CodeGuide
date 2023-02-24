---
title: 筛选素数 SieveOfEratosthenes
lock: need
---

# 《程序员数学：筛选素数》—— 如何计算100内的素数？

作者：小傅哥
<br/>博客：[https://bugstack.cn](https://bugstack.cn)
<br/>源码：[https://github.com/fuzhengwei/java-algorithms](https://github.com/fuzhengwei/java-algorithms)

> 沉淀、分享、成长，让自己和他人都能有所收获！😄

## 一、前言

素数在小傅哥前面的文章关于 [RSA 加密算法](https://bugstack.cn/md/algorithm/logic/math/2022-11-20-primality.html)中已经讲解过它的使用场景。对于一个素数的判断，通常可以使用折半求模计算方式来判断是否为素数。那么如果是给定范围的1...N个数字，找出这里所有的素数要怎么计算呢？

```java
public boolean isPrime(long number) {
    boolean isPrime = number > 0;
    // 计算number的平方根为k，可以减少一半的计算量
    int k = (int) Math.sqrt(number);
    for (int i = 2; i <= k; i++) {
        if (number % i == 0) {
            isPrime = false;
            break;
        }
    }
    return isPrime;
}
```

这样的方式就不太适合于把每个数字都轮训一遍效率也会比较低。那么本章中小傅哥就来分享另外一种筛选素数的计算方式[**埃拉托色尼筛法**](https://en.wikipedia.org/wiki/Sieve_of_Eratosthenes)

## 二、什么是埃拉托色尼筛法

在数学中，**Eratosthenes 筛法**是一种古老的算法，它可以用于查找不超过给定极限的所有素数。

<div align="center">
    <img src="https://bugstack.cn/images/article/algorithm/logic/sieve-of-eratosthenes-01.png?raw=true" width="550px">
</div>

它通过从第一个素数2开始，将每个素数的倍数迭代标记为合数。也就是2的下一个合数是4，之后依次是6、8、10、12 ... 100。当计算到100以后，再找另外一个素数3，从3开始找下一个合数6、9...直至结束后继续循环。当所有的合数都被染色后，剩余的数字就是指定范围内的所有素数了。

举个例子，找到小于30以内的素数：`2  3  4  5  6  7  8  9  10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30`

- 第一组计算2：2  3  ~~4~~  5  ~~6~~  7 ~~ 8~~ 9  ~~10~~ 11 ~~12~~ 13 ~~14~~ 15 ~~16~~ 17 ~~18~~ 19 ~~20~~ 21 ~~22~~ 23 ~~24~~ 25 ~~26~~ 27 ~~28~~ 29 ~~30~~
- 第二组计算3：2  3  ~~4~~  5  ~~6~~  7 ~~ 8~~ ~~9~~  ~~10~~ 11 ~~12~~ 13 ~~14~~ ~~15~~ ~~16~~ 17 ~~18~~ 19 ~~20~~ ~~21~~ ~~22~~ 23 ~~24~~ 25 ~~26~~ ~~27~~ ~~28~~ 29 ~~30~~
- 第三组计算5：2  3  ~~4~~  5  ~~6~~  7 ~~ 8~~ ~~9~~  ~~10~~ 11 ~~12~~ 13 ~~14~~ ~~15~~ ~~16~~ 17 ~~18~~ 19 ~~20~~ ~~21~~ ~~22~~ 23 ~~24~~ 25 ~~26~~ ~~27~~ ~~28~~ 29 ~~30~~

最后剩余数字就都是素数了，包括： 2  3     5     7           11    13          17    19          23                29

## 三、Eratosthenes 算法实现

**筛选算法**：—— 这里小傅哥额外加了一些辅助代码*primesMap*，用于打印结果，方便大家学习。

```java
public class SieveOfEratosthenes {

    public List<Integer> sieveOfEratosthenes(int maxNumber) {
        // 方便打印结果，可删除此代码
        Map<Integer, List<Integer>> primesMap = new HashMap<>();

        List<Integer> isPrime = new ArrayList<>(maxNumber + 1);
        isPrime.add(0);
        isPrime.add(1);

        List<Integer> primes = new ArrayList<>();
        for (int number = 2; number < maxNumber; number++) {
            if (!isPrime.contains(number)) {
                primes.add(number);
                // 方便打印结果，可删除此代码
                primesMap.put(number, new ArrayList<>());

                int nextNumber = number * number;
                while (nextNumber <= maxNumber) {
                    isPrime.add(nextNumber);
                    nextNumber += number;

                    // 方便打印结果，可删除此代码
                    primesMap.get(number).add(nextNumber);
                }
            }
        }

        System.out.println("筛选素数过程");
        for (Integer key : primesMap.keySet()) {
            System.out.println(key + "：" + JSON.toJSONString(primesMap.get(key)));
        }

        return primes;

    }

}
```

- 以2开始循环计算，从“p*p”开始标记“p”的倍数，而不是从“2*p”开始。这之所以有效，是因为在这一点上，较小的倍数“p”的将已标记为“false”。—— 这是一个优化处理。
- 处理非常大的数字时，可能会导致溢出在这种情况下，可以将其更改为：让`nextNumber=2*number;`—— 你可以尝试压榨一下。

## 三、Eratosthenes 算法测试

**单元测试**：计算1-100内的素数

```java
@Test
public void test_SieveOfEratosthenes() {
    SieveOfEratosthenes sieveOfEratosthenes = new SieveOfEratosthenes();
    List<Integer> primes = sieveOfEratosthenes.sieveOfEratosthenes(100);
    System.out.println("素数：" + JSON.toJSONString(primes));
}
```

**测试结果**

```java
筛选素数过程
2：[6,8,10,12,14,16,18,20,22,24,26,28,30,32,34,36,38,40,42,44,46,48,50,52,54,56,58,60,62,64,66,68,70,72,74,76,78,80,82,84,86,88,90,92,94,96,98,100,102]
3：[12,15,18,21,24,27,30,33,36,39,42,45,48,51,54,57,60,63,66,69,72,75,78,81,84,87,90,93,96,99,102]
67：[]
5：[30,35,40,45,50,55,60,65,70,75,80,85,90,95,100,105]
7：[56,63,70,77,84,91,98,105]
71：[]
73：[]
11：[]
13：[]
79：[]
17：[]
19：[]
83：[]
23：[]
89：[]
29：[]
31：[]
97：[]
37：[]
41：[]
43：[]
47：[]
53：[]
59：[]
61：[]
素数：[2,3,5,7,11,13,17,19,23,29,31,37,41,43,47,53,59,61,67,71,73,79,83,89,97]
```

- 在 HashMap 中保留了每一个素数在100内对应的合数，以此类推不断地查找。最终筛选后剩余的数字就是素数。
- 整个计算过程的时间复杂度是：`O(n log(log n))`

## 五、常见面试题

- 如何判断一个数字是否为素数
- 如何计算1-n中有多少个素数