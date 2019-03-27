---
title: tensorflow学习笔记（三）——前馈神经网络
date: 2017-08-18
categories:
- DeepLearning
- tensorflow
---
对于机器学习而言，神经网络算是必学的内容了，也试着用tensorflow框架写了一下

<!--more-->
## 神经网络
神经网络即模仿生物学中的神经细胞进行接受信号，传播信号的过程。对于单个神经细胞，可能有n个输入以及m个输出，但对于复杂的神经网络，有多层神经网络，其中接受原始信号的为输入层，接受来自前面神经元输出的叫做隐藏层，隐藏层可以有多层，最后一层即是输出层。而前馈神经网络是一种最简单的神经网络，各神经元分层排列。每个神经元只与前一层的神经元相连。接收前一层的输出，并输出给下一层．各层间没有反馈。
## 实现原理
对于前馈神经网络的实现，利用的是教材中所给的数据input_data，为带有数字的图片。
```
# 可在tensorflow库中导入
from tensorflow.examples.tutorials.mnist import input_data
mnist = input_data.read_data_sets('MNIST_data', one_hot=True)
```
神经网络模型的构建
```
# 首先定义设置网络层的函数，输入输出间均为线性关系
def set_layer(inputs,in_size,out_size,layer_name,activate_function = None):
    # 权重和偏差量定义
    W = tf.Variable(tf.random_uniform([in_size, out_size], -1.0, 1.0), name="W" + layer_name)
    bias =  tf.Variable(tf.constant(0.1, shape=[out_size]), name="bias_"+ layer_name)
    # 线性模型
    Wx_Plus_b = tf.matmul(inputs, W) + bias
    # 防止过拟合,keep_prob为每个元素的被保留概率
    Wx_Plus_b = tf.nn.dropout(Wx_Plus_b, keep_prob)
    # 激活函数的使用
    if activate_function is None:
        outputs = Wx_Plus_b
    else:
        outputs = activate_function(Wx_Plus_b)
    return outputs
# 原始数据占位符
xs = tf.placeholder(tf.float32, [None, n_input], name="input")
ys = tf.placeholder(tf.float32, [None, n_classes], name="output")
# 保留概率
keep_prob = tf.placeholder(tf.float32)
# 神经网络的构建
h1 = set_layer(xs, n_input, hidden_units1, 'hidden_layer_1', activate_function=tf.nn.tanh)
# softmax和tf.nn.tanh为激活函数，都可用,常见的还有relu函数
h2 = set_layer(h1, hidden_units1, hidden_units2, 'hidden_layer_2', activate_function=tf.nn.tanh)
prediction = set_layer(h2, hidden_units2, n_classes, 'prediction_layer', activate_function=None)
# 预测输出在用None无激活函数效果较好，也可用tf.nn.softmax，tf.nn.tanh等激活函数
```
训练指标的设置
```
# cross_entropy = tf.reduce_mean(-tf.reduce_sum(ys * tf.log(prediction)))
# 有看到资料说和下面一句意义相同，但实际上无法使用，
# tf.nn.softmax_cross_entropy_with_logits将预测值转化为总和为一的概率值，并与实际值的计算交叉熵代价函数
# tf.reduce_mean取均值
cross_entropy = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(labels=ys, logits=prediction))
# 训练目标为使cross_entropy最小，使用梯度下降法进行训练
train_op = tf.train.GradientDescentOptimizer(learning_rate).minimize(cross_entropy)
tf.summary.scalar('loss', cross_entropy)
# 准确率
correct_prediction = tf.equal(tf.argmax(prediction, 1), tf.argmax(ys, 1))
# 得到true false数据
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
# 求均值
```
## 完整代码
```
import tensorflow as tf
from tensorflow.examples.tutorials.mnist import input_data
import time
mnist = input_data.read_data_sets('MNIST_data', one_hot=True)
def set_layer(inputs,in_size,out_size,layer_name,activate_function = None):
    W = tf.Variable(tf.random_uniform([in_size, out_size], -1.0, 1.0), name="W" + layer_name)
    bias =  tf.Variable(tf.constant(0.1, shape=[out_size]), name="bias_"+ layer_name)
    Wx_Plus_b = tf.matmul(inputs, W) + bias
    Wx_Plus_b = tf.nn.dropout(Wx_Plus_b, keep_prob)#防止过拟合,keep_prob为每个元素的被保留概率
    if activate_function is None:
        outputs = Wx_Plus_b
    else:
        outputs = activate_function(Wx_Plus_b)
    return outputs
# 参数设定
hidden_layers = 1
hidden_units1 = 200
hidden_units2 = 50
n_input = 784
n_classes = 10
learning_rate = 0.8
# 神经网络的构建
xs = tf.placeholder(tf.float32, [None, n_input], name="input")
ys = tf.placeholder(tf.float32, [None, n_classes], name="output")
keep_prob = tf.placeholder(tf.float32)
h1 = set_layer(xs, n_input, hidden_units1, 'hidden_layer_1', activate_function=tf.nn.tanh)
h2 = set_layer(h1, hidden_units1, hidden_units2, 'hidden_layer_2', activate_function=tf.nn.tanh)
prediction = set_layer(h2, hidden_units2, n_classes, 'prediction_layer', activate_function=None)
cross_entropy = tf.reduce_mean(tf.nn.softmax_cross_entropy_with_logits(labels=ys, logits=prediction))
train_op = tf.train.GradientDescentOptimizer(learning_rate).minimize(cross_entropy)
tf.summary.scalar('loss', cross_entropy)
# 训练结果准确性
correct_prediction = tf.equal(tf.argmax(prediction, 1), tf.argmax(ys, 1))
accuracy = tf.reduce_mean(tf.cast(correct_prediction, tf.float32))
# 训练
init = tf.initialize_all_variables()
n_epochs = 40
batch_size = 100
with tf.Session() as sess:
    st = time.time()
    write = tf.summary.FileWriter('logs/', sess.graph)
    sess.run(init)
    for epoch in range(n_epochs):
        n_batch = int(mnist.train.num_examples / batch_size)
        for i in range(n_batch):
            batch_xs, batch_ys = mnist.train.next_batch(batch_size)
            sess.run(train_op, feed_dict={xs: batch_xs, ys: batch_ys, keep_prob:0.75})
        print ('epoch', epoch, 'accuracy:', sess.run(accuracy, feed_dict={keep_prob:1.0, xs: mnist.test.images, ys: mnist.test.labels}))
    end = time.time()
    print ('*' * 30)
    print ('training finish. cost time:', int(end-st) , 'seconds; accuracy:', sess.run(accuracy, feed_dict={keep_prob:1.0, xs: mnist.test.images, ys: mnist.test.labels}))
```
