# BF

[TOC]

## 简介

BF算法，即暴力（Brute Force）算法，是普通的模式匹配算法，BF算法的思想就是将目标串S的第一个字符串与模式串T的第一个字符进行匹配，若相等，则继续比较S的第二个字符和T的第二个字符串；若不相等，则比较S的第二个字符串和T的第一个字符串，依次比较下去，直到得出最后的匹配结果。BF算法是一种蛮力算法。

## BF算法思想

首先S[0]和T[0]比较，若相等，则再比较S[1]和T[1]，一直到T[m]为止；若S[n]和T[n]不等（n ≤ m），则S向右移动一个字符串的位置，再依次进行比较。如果存在k(0 ≤ k ≤ n)，且S[k...k+m] = T[0...m]，则匹配成功；否则失败。

该算法最坏情况下要进`m*(n-m+1)`次比较，时间复杂度为`O(m*n)`。

## BF算法原理

使用普通模式匹配算法判断串A(abcac)是否为串B(ababcabcacbab)的子串。过程如下：

首先，将串A与串B的首字符对其，然后逐个判断其对应的字符是否相等。
![](images/BF (0).png)
图1中，由于串A与串B的第3个字符串匹配失败，因此需要将串B后移一个字符的位置，继续同串A匹配。
![](images/BF (1).png)
图2中，由于串A的第2个字符与串B的第1个字符串匹配失败，因此需要将串B后移一个字符的位置，继续同串A匹配。
![](images/BF (2).png)
图3中，两串的模式匹配失败，串B继续移动。一直移动到图4所示的位置才匹配成功。
![](images/BF (3).png)
至此，串A与串B共经历了6次匹配的过程才成功。通过整个模式的匹配过程，证明了串B是串A的子串（串A是串B的主串。)。

## BF算法实现

Java语言-实现方式一

```java
public int index(char[] primaryString, char[] substring) {
    int i = 0, p = 0, l = primaryString.length;
    while (p == i) {
        for (char c : substring) {
            if (p == l) return -1;
            if (primaryString[p++] != c) {
                p = ++i;
                break;
            }
        }
    }
    return i;
}
```

Java语言-实现方式二

```java
public int index(char[] primaryString, char[] substring) {
    int i = 0, j = 0, lps = primaryString.length, lss = substring.length;
    while (i < lps && j < lss) {
        if (primaryString[i] == substring[j]) {
            i++;
            j++;
        } else {
            i = i - j + 1;
            j = 0;
        }
    }
    return j == lss ? i - j : -1;
}
```