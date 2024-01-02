### 前言

KMP算法的核心，是一个被称为部分匹配表(Partial Match Table)的数组。我觉得理解KMP的最大障碍就是很多人在看了很多关于KMP的文章之后，仍然搞不懂PMT中的值代表了什么意思。这里我们抛开所有的枝枝蔓蔓，先来解释一下这个数据到底是什么。

对于字符串“[abababca](https://www.zhihu.com/search?q=abababca&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A281346746%7D)”，它的PMT如下表所示：

![img](https://pic1.zhimg.com/80/v2-e905ece7e7d8be90afc62fe9595a9b0f_1440w.webp?source=2c26e567)

### 理解一些概念

前缀：如果字符串A和B，存在A=BS，其中S是任意的非空字符串，那就称B为A的前缀。例如，”Harry”的前缀包括{”H”, ”Ha”, ”Har”, ”Harr”}，我们把所有前缀组成的集合，称为字符串的前缀集合。

后缀：同样可以定义后缀A=SB， 其中S是任意的非空字符串，那就称B为A的后缀，例如，”Potter”的后缀包括{”otter”, ”tter”, ”ter”, ”er”, ”r”}，然后把所有后缀组成的集合，称为字符串的后缀集合。要注意的是，字符串本身并不是自己的后缀。

PMT：**PMT中的值是字符串的前缀集合与后缀集合的交集中最长元素的长度** 。例如，对于”aba”，它的前缀集合为{”a”, ”ab”}，后缀 集合为{”ba”, ”a”}。两个集合的交集为{”a”}，那么长度最长的元素就是字符串”a”了，长 度为1，所以对于”aba”而言，它在PMT表中对应的值就是1。再比如，对于字符串”ababa”，它的前缀集合为{”a”, ”ab”, ”aba”, ”abab”}，它的后缀集合为{”baba”, ”aba”, ”ba”, ”a”}， 两个集合的交集为{”a”, ”aba”}，其中最长的元素为”aba”，长度为3。

### 为什么要使用PMT表

可以加速字符串的查找。

要在主字符串"[ababababca](https://www.zhihu.com/search?q=ababababca&search_source=Entity&hybrid_search_source=Entity&hybrid_search_extra=%7B%22sourceType%22%3A%22answer%22%2C%22sourceId%22%3A281346746%7D)"中查找模式字符串"abababca"。如果在 j 处字符不匹配，那么由于前边所说的模式字符串 PMT 的性质，主字符串中 i 指针之前的 PMT[j −1] 位就一定与模式字符串的第 0 位至第 PMT[j−1] 位是相同的。

而我们上面也解释了，模式字符串从 0 到 j−1 ，在这个例子中就是”ababab”，其前缀集合与后缀集合的交集的最长元素为”abab”， 长度为4。

所以就可以断言，主字符串中i指针之前的 4 位一定与模式字符串的第0位至第 4 位是相同的，即长度为 4 的后缀与前缀相同。这样一来，我们就可以将这些字符段的比较省略掉。具体的做法是，保持i指针不动，然后将j指针指向模式字符串的PMT[j −1]位即可。

简言之，以图中的例子来说，在 i 处失配，那么主字符串和模式字符串的前边6位就是相同的。又因为模式字符串的前6位，它的前4位前缀和后4位后缀是相同的，所以我们推知主字符串i之前的4位和模式字符串开头的4位是相同的。就是图中的灰色部分。**那这部分就不用再比较了。因此指针会移到第五位。**

![img](https://pic1.zhimg.com/80/v2-03a0d005badd0b8e7116d8d07947681c_1440w.webp?source=2c26e567)

### next数组的产生

我们看到如果是在 j 位 失配，那么影响 j 指针回溯的位置的其实是第 j −1 位的 PMT 值。

所以为了编程的方便， 我们将PMT数组向后偏移一位，得到next数组。

**即：next(index)就是当模版字符串在index = j失配的时候，下一个j应该指向的下标**

![](https://pica.zhimg.com/80/v2-40b4885aace7b31499da9b90b7c46ed3_1440w.webp?source=2c26e567)

具体的程序如下所示：

```cpp
int KMP(char * t, char * p) 
{
	int i = 0; 
	int j = 0;

	while (i < (int)strlen(t) && j < (int)strlen(p))
	{
		if (j == -1 || t[i] == p[j]) 
		{
			i++;
           		j++;
		}
	 	else 
           		j = next[j];
    	}

    if (j == strlen(p))
       return i - j;
    else 
       return -1;
}
```

现在，我们再看一下如何编程快速求得next数组。

其实，求next数组的过程完全可以看成字符串匹配的过程。

**即p串错开一位匹配**

**以1起始的模式字符串为主字符串，瞄准后缀。**

**以0起始的模式字符串为目标字符串，瞄准前缀。**

这里为什么是从next[2]开始匹配呢？

因为已知next数组是由pmt数组往后推一位产生的。

pmt[0] = 0，因此next[0] = -1，next[1] = 0

即next[i+1] = pmt[i]，因此我们从next[2]开始匹配就是从pmt[1]开始匹配

![](https://pica.zhimg.com/80/v2-645f3ec49836d3c680869403e74f7934_1440w.webp?source=2c26e567)

![](https://picx.zhimg.com/80/v2-06477b79eadce2d7d22b4410b0d49aba_1440w.webp?source=2c26e567)

![](https://pic1.zhimg.com/80/v2-8a1a205df5cad7ab2f07498484a54a89_1440w.webp?source=2c26e567)

![](https://picx.zhimg.com/80/v2-f2b50c15e7744a7b358154610204cc62_1440w.webp?source=2c26e567)

![](https://pica.zhimg.com/80/v2-bd42e34a9266717b63706087a81092ac_1440w.webp?source=2c26e567)

最终代码如下（代码是leecode中的题目，用了kmp的算法）

```

/**
 * @param {string} haystack
 * @param {string} needle
 * @return {number}
 */
var strStr = function(haystack, needle) {
    //构建next数组O(m)
    const getNext = (needle)=>{
        let i = 1;
        let j = 0;
        const next = [-1,0];
        while(i < needle.length-1){
            if(needle[i] === needle[j]){
                next[i+1] = j+1;
                i++
                j++
            }else{
                if(j === 0){
                    next[i+1] = 0;
                    i++;
                }else{
                    j = next[j]
                }
            }
        }
        return next
    }
    //KMP匹配O(n)
    let i = 0;
    let j = 0;
    const next = getNext(needle)
    console.log(next)
    //只要有一个到头了遍历就结束了
    while(j !== needle.length && i !== haystack.length){
        if(haystack[i] === needle[j]){
            if(j === needle.length-1){
                return i - j
            }
            i++;
            j++;
        }else{
            if(j === 0){
                i++;
            }else{
                j = next[j]
            }
        };
    };
    return -1;
};

```

### 参考文章

KMP算法部分：https://www.zhihu.com/question/21923021/answer/281346746

NEXT数组构建部分：https://leetcode.cn/problems/find-the-index-of-the-first-occurrence-in-a-string/solutions/575568/shua-chuan-lc-shuang-bai-po-su-jie-fa-km-tb86
