---
title: tensorflow学习笔记（二）——数据类型
date: 2017-08-17
categories:
- DeepLearning
- tensorflow
---

看了官网的教程，先跑了第一个例程，大概学习了tensorflow的几种数据类型以及程序流程
<!--more-->
## 数据类型
tensorflow的数据以张量的形式构成，其形状和类型的设定和实际数值的写入是分开的，主要有常量，变量和占位符三种类型：
这是基本的python数组
```
# 普通的python数组
a = [[1., 2., 3.], [4., 5., 6.]]
```
对于tensorflow常量
```
# tensorflow常量的赋值只能为定值，dtype定义了其数据为浮点型,name为其命名
b1 = tf.constant(3,dtype=tf.float32,name = 'b1')
b2 = tf.constant(a,dtype=tf.float32)
# 也可用b1 = tf.constant(3,"float")定义
# 测试后发现tf.zeros([5]),tf.ones([5,5]),tf.random_uniform([5, 5], -2.0, 2.0)创建的也为常量数据
# 其实质和constant相同，zeros等为常量的名称
# 我理解为constant常量可用于给变量或占位符进行赋值
```
对于tensorflow变量
```
c1 = tf.Variable(a, dtype = tf.float32,name = 'c1')
c2 = tf.Variable(b1, dtype = tf.float32)
c3 = tf.Variable(tf.ones([5,5]))
c4 = tf.Variable(tf.random_uniform([5, 5], -2.0, 2.0))

# tensorflow变量可直接使用python普通数组赋值，也可用tensorflow常量
# 若所赋的初始化值已定义数据类型，则可以不再定义
# 使用变量需先初始化，在计算中其值是不断变换的，一般形状大小设置固定，用作被训练的权重等
```
对于tensorflow占位符
```
# 占位符是事先设定好，用来接受外部输入的一个值，一般用作训练样本和标签的数据
# shape定义的形状大小，None代表任意大小，也可先不定义，与输入数据相同
x1 = tf.placeholder(tf.float32)
x2 = tf.placeholder(tf.float32,[None,3])
x3 = tf.placeholder(dtype = tf.float32,shape = (2,2))

# 在进行运算时，将数据从外部输入
print(sess.run([x1,x2,x3], feed_dict = {x1:[[1,2],[1,2],[3,4]],x2:[[1,2,3],[2,6,4]],x3:[[1,2],[3,4]]}))
```
## 完整代码
```
import tensorflow as tf
a = [[1., 2., 3.], [4., 5., 6.]]
b1 = tf.constant(3,dtype=tf.float32,name = 'b1')
b2 = tf.constant(a,dtype=tf.float32)
print(b1)
print(b2)
c1 = tf.Variable(a, dtype = tf.float32,name = 'c1')
c2 = tf.Variable(b1, dtype = tf.float32)
c3 = tf.Variable(tf.ones([5,5]))
c4 = tf.Variable(tf.random_uniform([5, 5], -2.0, 2.0))
x1 = tf.placeholder(tf.float32)
x2 = tf.placeholder(tf.float32,[None,3])
x3 = tf.placeholder(dtype = tf.float32,shape = (2,2))
sess = tf.Session()
# 创建图
init = tf.global_variables_initializer()
sess.run(init)
# 将所有变量初始化
print(sess.run([b1,b2]))
print(sess.run([c1,c2,c3,c4]))
print(sess.run([x1,x2,x3], feed_dict = {x1:[[1,2],[1,2],[3,4]],x2:[[1,2,3],[2,6,4]],x3:[[1,2],[3,4]]}))
```
