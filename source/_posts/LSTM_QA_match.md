---
title: 基于LSTM的问答匹配实践
date: 2019-02-27
categories:
- DeepLearning
- tensorflow
---
之前的博客里，都写了一些tensorflow的入门学习，当时在学习tensorflow时一个让我很头疼的问题就是，只要自己写大点深点的网络，基本都是跑不动，损失函数降了一阵就不动了，估计可能是初始化，正则防止过拟合没做好。这个LSTM网络算是我难得写出来能跑的网络，现在有空了来记录一下。


## LSTM网络
LSTM网络是由RNN变化而来，当时一直很困惑的问题就是关于RNN，横的算一层还是竖的算一层，应该是横的算一层，因此最简单的RNN即是一层隐藏层，光看RNN模型图的话，总会有些误解，实际上RNN每个单元的输入对应的状态隐藏层是可以有多个输出的，因为一个单元的输入维度也是n维，对应隐藏层的输出若是m维，则但看这个单元可理解为n到m的全连接网络。这个隐藏层维度一般就是由自己设定。

同时关于RNN和LSTM的输出，一般在理解模型时，都是以一个句子为例，只需要最后一个序列单元对应的输出，实际上通过tensorflow和keras框架得到的输出，是包括所有单元的，如RNN的输出
`outputs, last_state = tf.nn.static_rnn(cell, inputs)`
理论上可认为outputs[-1]与last_state是相等的

关于LSTM，由于其状态由两个值构成(Ct,Ht)，认为其状态是一个tuple，其输出为
`outputs, state_out = tf.contrib.rnn.static_rnn(cell, inputs, initial_state=tuple_state)`
这样得到的state_out包含两个值，若所写的网络中间有多个隐藏层，则可以有更多状态输出。

同时对于双向LSTM，其输出outputs的每个单元尺寸为两个LSTM的隐藏层输出单元的和
## LSTM网络代码
``` bash
def biLSTMCell(x, hiddenSize):
    input_x = tf.transpose(x, [1, 0, 2])# 0 1 2 变成1 0 2，最外面两个维度转置
    input_x = tf.unstack(input_x)   #多维变成低维，默认axio = 0，最外层的维度
    lstm_fw_cell = tf.contrib.rnn.BasicLSTMCell(hiddenSize, forget_bias=1.0, state_is_tuple=True)
    lstm_bw_cell = tf.contrib.rnn.BasicLSTMCell(hiddenSize, forget_bias=1.0, state_is_tuple=True)
    output, _, _ = tf.contrib.rnn.static_bidirectional_rnn(lstm_fw_cell, lstm_bw_cell, input_x, dtype=tf.float32)
    output = tf.stack(output)       #一维变成多维
    output = tf.transpose(output, [1, 0, 2])
    return output
```
需要使用unstack与stack函数变化矩阵维度结构，unstack将一个整体tf变量变成了一个数字list内部的多个tf量，作为输入进入网络输出后，再使用stack函数变回整体tf变量。
[完整代码](https://github.com/LiHuaBai/LSTM_for_QA_match)
