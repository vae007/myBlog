---
title: js做四则运算时精度丢失问题
date: 2019-10-21 16:16:48
tags: JavaScript
categories: JavaScript
---
这个问题可以说是程序员必踩的坑，因此网上针对该问题的分析有很多也很详细，解决方法也比较统一，写法也是大同小异，本以为预期效果真能如他们所说是完美的，然而效果却是差强人意。

### 一、问题

首先，先来看看两数相加的一个经典问题，网上找过不少资料的人会发现，大多数人分析精度问题都是由此展开，然而最后所谓的解决方法成也在它，败也在它。

```
0.1+0.2=0.30000000000000004
```
网上所说的完美解决方法：

```
function add(arg1,arg2){ 
  var digits1,digits2,maxDigits; 
  try{digits1=arg1.toString().split(".")[1].length}catch(e){digits1=0} 
  try{digits2=arg2.toString().split(".")[1].length}catch(e){digits2=0} 
  maxDigits=Math.pow(10,Math.max(digits1,digits2)) 
  return (arg1*maxDigits+arg2*maxDigits)/maxDigits 
} 
```
应用后的输出结果：
```
0.1+0.2=0.3
```
从结果可以看出，这方法是完美的解决了这个经典问题，那其他数做运算是不是也都能完美解决呢？在我多数测试后发现并没有，本以为有可能是开发环境或者浏览器导致的问题，但是切换了多个开发环境和浏览器也还是如此，因此可知，方法本身就有问题。

两数直接相加的输出结果：

```
2.22+2.22=4.44
4.44+2.22=6.66
6.66+2.22=8.88
8.88+2.22=11.100000000000001
11.10+2.22=13.32
13.32+2.22=15.540000000000001
```
应用网上找的方法的输出结果：
```
2.22+2.22=4.44
4.44+2.22=6.660000000000001
6.66+2.22=8.88
8.88+2.22=11.100000000000001
11.10+2.22=13.32
13.32+2.22=15.54
```
对比两次结果可以发现，有些数的精度问题是解决了，而有些数根本没有解决，甚至还导致有些本不会出现精度问题的数产生了问题，除了加法，关于减乘除网上那些方法也一概行不通，限于篇幅，这里就不再一一举例。

### 二、解决方法：
最后发现那些方法都有个共同特征，就是将原来带n位小数的浮点数乘以10的n次方再进行运算，但是没有解决的主要问题，还是数据类型运算。只要存在浮点数的运算，怎么算也都会有精度问题。

因此，还是要保证参与运算的数为整数才行。保证为整数其实这点就是他们方法中将原来带n位小数的浮点数乘以10的n次方，但还需要强制指定类型为int。但是如果直接将乘以10的n次方后的数转换为Integer对象，会导致在乘方时出现精度问题而出现精度丢失，即计算结果有误。因此还是需要利用Math库中round方法，将数四舍五入取整再进行运算。

两数相加方法：
```
  function add(arg1, arg2) {
    return (Math.round(arg1 * 100) + Math.round(arg2 * 100)) / 100;
  }
```
应用该方法进行两数相加的运算结果：
```
0.1+0.2=0.3
2.22+2.22=4.44
4.44+2.22=6.66
6.66+2.22=8.88
8.88+2.22=11.1
11.10+2.22=13.32
13.32+2.22=15.54
```
两数相减（既一正数加一负数）：
```
 function subtract(arg1, arg2) {
    return this.add(arg1, -arg2);
  }
```
两数相乘：
```
  function multiple(arg1, arg2) {
    return (Math.round(arg1 * 100) * Math.round(arg2 * 100)) / 10000;
  }
```
两数相乘：
```
  function multiple(arg1, arg2) {
    return (Math.round(arg1 * 100) * Math.round(arg2 * 100)) / 10000;
  }
```
两数相除：
```
/**
   * arg1与arg2相除，并以四舍五入的方式保留小数点后2位
   */
  function divide(arg1, arg2) {
    var d1, d2,
      n1 = Number(arg1.toString().replace(".", "")),
      n2 = Number(arg2.toString().replace(".", ""));
    try {d1 = arg1.toString().split(".")[1].length;} catch (e) {d1 = 0;}
    try {d2 = arg2.toString().split(".")[1].length;} catch (e) {d2 = 0;}
    return this.toFixed((n1 / n2) * Math.pow(10, d2 - d1), 2);
  }
 
  /**
   * arg以四舍五入的方式保留小数点后n位
   */
  function toFixed(arg, n) {
    if(n == 0) {
      return Math.round(arg)
    }
    try {
      var d, carryD, i, 
      ds = arg.toString().split('.'),
      len = ds[1].length,
      b0 = '', k = 0
      if (len > n) {
        while(k < n && ds[1].substring(0, ++k) == '0') {
          b0 += '0'
        }
        d = Number(ds[1].substring(0, n))
        carryD = Math.round(Number('0.' + ds[1].substring(n, len)))
        len = (d + carryD).toString().length
        if (len > n) {
          return Number(ds[0]) + 1
        } else if (len == n) {
          return Number(ds[0] + '.' + (d + carryD))
        }
        return Number(ds[0] + '.' + b0 + (d + carryD))
      }
    } catch (e) {}
    return arg
  }
```

<font color="red">注：js中自带的toFixed函数会把多余的小数直接截掉，因此，这里我进行重写，采用四舍五入的方式保留小数，该方法也很实用。</font>

该文章参考自CSDN，如有侵权，请联系删除。