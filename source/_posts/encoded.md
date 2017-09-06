---
title: 彻底弄懂 Unicode 编码
date: 2017-09-02 21:29:08
categories:
  - 也许这能帮到你
tags: 
  - Unicode
---

今天，在学习 Node.js 中的 Buffer 对象时，注意到它的 *alloc* 和 *from* 方法会默认用 **UTF-8** 编码，在数组中每位对应 1 字节的十六进制数。想到了之间学习 ES6 时关于字符串的 Unicode 表示法，突然就很想知道 UTF-16 是如何进行编码的，我尝试将一些汉字转换成二进制数，然后简单的按 2 个字节一组转换成十六进制，发现对于那些码点较大的汉字，结果并不仅仅是简单的二进制转十六进制。于是，我开始在网上找资料，决心彻底弄明白 Unicode 编码。

<!-- more -->

## ASCII码

在学校学 C 语言的时候，了解到一些计算机内部的机制，知道所有的信息最终都表示为一个二进制的字符串，每一个二进制位有 0 和 1 两种状态，通过不同的排列组合，使用 0 和 1 就可以表示世界上所有的东西，感觉有点中国“太极”的感觉——“太极生两仪，两仪生四象，四象生八卦”。

在计算机种中，1 字节对应 8 位二进制数，而每位二进制数有 0、1 两种状态，因此 1 字节可以组合出 256 种状态。如果这 256 中状态每一个都对应一个符号，就能通过 1 字节的数据表示 256 个字符。美国人于是就制定了一套编码（其实就是个字典），描述英语中的字符和这 8 位二进制数的对应关系，这被称为 ASCII 码。

ASCII 码一共定义了 128 个字符，例如大写的字母 A 是 65（这是十进制数，对应二进制是0100 0001）。这 128 个字符只使用了 8 位二进制数中的后面 7 位，最前面的一位统一规定为 0。

## 历史问题

英语用 128 个字符来编码完全是足够的，但是用来表示其他语言，128 个字符是远远不够的。于是，一些欧洲的国家就决定，将 ASCII 码中闲置的最高位利用起来，这样一来就能表示 256 个字符。但是，这里又有了一个问题，那就是不同的国家的字符集可能不同，就算它们都能用 256 个字符表示全，但是同一个码
点（也就是 8 位二进制数）表示的字符可能可能不同。例如，144 在阿拉伯人的 ASCII 码中是 **گ**，而在俄罗斯的 ASCII 码中是 **ђ**。

因此，ASCII 码的问题在于尽管所有人都在 0 - 127 号字符上达成了一致，但对于 128 - 255 号字符上却有很多种不同的解释。与此同时，亚洲语言有更多的字符需要被存储，一个字节已经不够用了。于是，人们开始使用两个字节来存储字符。

各种各样的编码方式成了系统开发者的噩梦，因为他们想把软件卖到国外。于是，他们提出了一个“内码表”的概念，可以切换到相应语言的一个内码表，这样才能显示相应语言的字母。在这种情况下，如果使用多语种，那么就需要频繁的在内码表内进行切换。

## Unicode

最终，美国人意识到他们应该提出一种标准方案来展示世界上所有语言中的所有字符，出于这个目的，Unicode诞生了。

Unicode 当然是一本很厚的字典，记录着世界上所有字符对应的一个数字。具体是怎样的对应关系，又或者说是如何进行划分的，就不是我们考虑的问题了，我们只用知道 Unicode 给所有的字符指定了一个数字用来表示该字符。

对于 Unicode 有一些误解，它仅仅只是一个字符集，规定了符合对应的二进制代码，至于这个二进制代码如何存储则没有任何规定。它的想法很简单，就是为每个字符规定一个
用来表示该字符的数字，仅此而已。

### Unicode 编码方案

之前提到，Unicode 没有规定字符对应的二进制码如何存储。以汉字“汉”为例，它的 Unicode 码点是 0x6c49，对应的二进制数是 110110001001001，二进制数有 15 位，这也就说明了它至少需要 2 个字节来表示。可以想象，在 Unicode 字典中往后的字符可能就需要 3 个字节或者 4 个字节，甚至更多字节来表示了。

这就导致了一些问题，计算机怎么知道你这个 2 个字节表示的是一个字符，而不是分别表示两个字符呢？这里我们可能会想到，那就取个最大的，假如 Unicode 中最大的字符用 4 字节就可以表示了，那么我们就将所有的字符都用 4 个字节来表示，不够的就往前面补 0。这样确实可以解决编码问题，但是却造成了空间的极大浪费，如果是一个英文文档，那文件大小就大出了 3 倍，这显然是无法接受的。

于是，为了较好的解决 Unicode 的编码问题， UTF-8 和 UTF-16 两种当前比较流行的编码方式诞生了。当然还有一个 UTF-32 的编码方式，也就是上述那种定长编码，字符统一使用 4 个字节，虽然看似方便，但是却不如另外两种编码方式使用广泛。

#### UTF-8

UTF-8 是一个非常惊艳的编码方式，漂亮的实现了对 ASCII 码的向后兼容，以保证 Unicode 可以被大众接受。

UTF-8 是目前互联网上使用最广泛的一种 Unicode 编码方式，它的最大特点就是可变长。它可以使用 1 - 4 个字节表示一个字符，根据字符的不同变换长度。编码规则如下：

1. 对于单个字节的字符，第一位设为 0，后面的 7 位对应这个字符的 Unicode 码点。因此，对于英文中的 0 - 127 号字符，与 ASCII 码完全相同。这意味着 ASCII 码那个年代的文档用 UTF-8 编码打开完全没有问题。

2. 对于需要使用 N 个字节来表示的字符（N > 1），第一个字节的前 N 位都设为 1，第 N + 1 位设为0，剩余的 N - 1 个字节的前两位都设位 10，剩下的二进制位则使用这个字符的 Unicode 码点来填充。

编码规则如下：

| Unicode 十六进制码点范围 | UTF-8 二进制                         |
| :------------------- | :---------                           |
| 0000 0000 - 0000 007F | 0xxxxxxx                            |
| 0000 0080 - 0000 07FF | 110xxxxx 10xxxxxx                   |
| 0000 0800 - 0000 FFFF | 1110xxxx 10xxxxxx 10xxxxxx          |
| 0001 0000 - 0010 FFFF | 11110xxx 10xxxxxx 10xxxxxx 10xxxxxx |

根据上面编码规则对照表，进行 UTF-8 编码和解码就简单多了。下面以汉字“汉”为利，具体说明如何进行 UTF-8 编码和解码。

“汉”的 Unicode 码点是 0x6c49（110 1100 0100 1001），通过上面的对照表可以发现，`0x0000 6c49` 位于第三行的范围，那么得出其格式为 `1110xxxx 10xxxxxx 10xxxxxx`。接着，从“汉”的二进制数最后一位开始，从后向前依次填充对应格式中的 x，多出的 x 用 0 补上。这样，就得到了“汉”的 UTF-8 编码为 `11100110 10110001 10001001`，转换成十六进制就是 `0xE6 0xB7 0x89`。

解码的过程也十分简单：如果一个字节的第一位是 0 ，则说明这个字节对应一个字符；如果一个字节的第一位1，那么连续有多少个 1，就表示该字符占用多少个字节。