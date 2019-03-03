---
layout: post
title: python二级模考1
description: >
  python二级模考第二季1错题集（https://www.python123.io/拥有版权）
author: author2
---

总分 78
错题； 选择：12/40  程序 10/60

## 主要问题；
1. [open(),输入文件读取](http://www.runoob.com/python/python-func-open.html)
2. [python库介绍]([https://blog.csdn.net/qq_26658517/article/details/8094872](https://www.cnblogs.com/maplered/p/7843232.html)2)
3. [time（）](http://www.runoob.com/python/att-time-time.html)
4. [turtle（）](https://www.cnblogs.com/chen0307/articles/9645138.html)
5. [id()](http://www.runoob.com/python/python-func-id.html)
## 订正(未在上面)：

### 选择题
树是结点的集合，它的根结点数目是 **0或1**

可以为空树
{:.faded}

---

为了提高测试的效率，应该 **集中对付那些错误群集的程序** 

背
{:.faded}

-----

下面错误 ~~**Python 最具特色的就是使用缩进来表示代码块，缩进的空格数固定为4个。**~~

缩进的空格数是可变的，但是同一个代码块的语句必须包含相同的缩进的空格是可变的，但是同一个代码块的语句必须包含相同的缩进空格数
{:.faded}

>正确的是：1，Python 是强面向对象的语言，程序中任何内容统称为对象，包括数字、字符串、函数等。 **2.Python 可以在同一行中使用多条语句，语句之间使用分号;分割。**3.Python 通常是一行写完一条语句。但如果语句很长，可以使用反斜杠\来实现多行语句。**实测函数不可分割**

---

下面错误 ~~**定义参数时，带默认值的参数不一定在无默认值参数的后面。**~~
‪‬‪‬‪‬‪‬‮‬‭‬‪‬‪‬‪‬‪‬‪‬‪‬‮‬‪‬‭‬‪def foo(p1, p2=6, p3):
	return 0
SyntaxError: non-default argument follows default argument

因为调用函数时可能会产生歧义，比如调用上面的函数foo(1, 2)，是该调用foo(1, 6, 2)呢？还是该调用foo(1, 6)呢？或者其它的什么呢？
{:.faded}

>正确的是：**1.return 可以返回多个值。** python !!! return 返回[元组](http://www.runoob.com/python/python-tuples.html)

---

机器学习方向的第三方库是 [TensorFlow](http://www.tensorfly.cn/)机器学习

---

~~~py
if -1:
    print("True.")
else:
    print("False.")
~~~
**Ture.**

if中不是0就ture
{:.faded}

---

列表不是映射类型**字典才是**

---

使用open()打开一个Windows操作系统D盘PythonTest文件夹下的文件，以下选项对路径的表示错误的是 ~~**D:\PythonTest\a.txt**~~

- 使用斜杠“/”: "c:/test.txt"… 不用反斜杠就没法产生歧义了 

- 将反斜杠符号转义: "c:\\test.txt"… 因为反斜杠是转义符，所以两个"\\"就表示一个反斜杠符号 
- 使用Python的raw string: r"c:\test.txt" … python下在字符串前面加上字母r，表示后面是一个原始字符串raw string，不过raw string主要是为正则表达式而不是windows路径设计的，所以这种做法尽量少用，可能会出问题。

- 两个//也行
{:.faded}

---

  ~~~py
  fo = open(fname, "r")
  for x in fo:
    print(x)
  fo.close()
  ~~~

**变量x表示文件中的一行字符**

---

### 程序题 
+ time()用法 略
+ jiaba() 库
>简单应用题2
请对《阿甘正传-网络版》进行中文分词，排除单个字符的分词结果，输出排序后的前10的词语。

~~~py
import jieba
txt = open("阿甘正传-网络版.txt","r",encoding ="utf-8").read()
words = jieba.lcut(txt)
counts ={}
for word in words:
    if len(word) == 1:
        continue
    else:
        counts[word] = counts.get(word,0)+1
items = list(counts.items())
items.sort(key = lambda x:x[1],reverse = True)
for i in range(10):
    word,count = items[i]
    print("{0}:{1}".format(word,count))
~~~

