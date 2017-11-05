---
layout: post
title: Tensorflow初探索
---

Tensorflow是目前很火的一个深度学习框架，借助TensorFlow可以极大地简化我们的工作量，同时降低我们运用机器学习的门槛。通过TensorFlow，我们可以不必关心过多的数学细节，而可以将精力集中在模型的构造和优化上。

### 为什么使用Tensorflow
- 使用Python语言，上手难度低
- 支持使用GPU运算，可以大幅提高计算效率
- 支持C++、Java、Go等多种主流语言，可移植性强
- Google出品，社区活跃

### TF快速安装
TensorFlow[安装](https://www.tensorflow.org/install/)非常简单，仅需要Python和pip即可，官方有详细的文档及命令行描述，推荐使用虚拟环境安装，以便隔离Python环境。以Ubuntu16.04，Python2.7为例：
``` bash
# 安装虚拟环境
sudo apt-get install python-pip python-dev python-virtualenv

# 在用户目录下创建tensorflow的虚拟环境
virtualenv --system-site-packages ~/Env/tensorflow

# 切换到虚拟环境
source ~/Env/tensorflow/bin/activate

# 需要确保pip版本大于8.1，如果版本不满足条件则需要升级
pip -V
pip install -U pip

# 接下来直接安装TensorFlow
pip install --upgrade tensorflow

# 如果需要安装GPU版本，执行（需要NVIDIA显卡的支持）
pip install --upgrade tensorflow-gpu

```

### 第一个例子
``` python
import tensorflow as tf
a = tf.constant(3.0)
b = tf.constant(4.0)
c = tf.add(a, b)

with tf.Session() as sess:
    res = sess.run(c)

print(a)
print(b)
print(c)
print(res)
```

运行结果
```
Tensor("Const:0", shape=(), dtype=float32)
Tensor("Const_1:0", shape=(), dtype=float32)
Tensor("Add:0", shape=(), dtype=float32)
7.0
```

### TensorFlow中的一些基本概念
TensorFlow是一个流式的计算系统，其使用有向图来表示一个计算任务，其中每一个节点表示一个操作（operate），操作之间由数据，也就是张量（tensor）相连接
- Tensor（张量）：用于表示计算中使用到的各种数据，经常会使用到的类型分为以下三种
1. Constant（常量）：就是我们在第一个例子中看到的，是一个固定的值
2. Variable（变量）：在计算和迭代过程中是可变的，非常需要注意的一点是，变量不能够直接使用，在使用前一定要通过函数`sess.run(tf.global_variables_initializer())`进行初始化
3. PlaceHolder（占位符）：通常用于模型训练的输入，用于告诉模型输入的形状

- Graph（计算图）：由操作和张量组成的集合，每个TensorFlow程序至少会拥有一个图，程序会根据图完成相应的任务，注意Graph仅仅是定义了计算执行的过程以及需要的数据类型，实际计算还需要通过Session进行

- Session（会话）：Session是实际的执行单元，每个Session会包含一个图，如果没有为Session指定图，则Session会加载默认的图，只有通过会话，实际的计算操作才会得到执行。

### 神经网络的一些基础知识
- ##### 神经网络中的最基本结构，神经元
现实中神经元的样子：
![现实神经元](http://img.m.xinwanapp.com/t017ca150019ae39548.jpg)
模型中神经元的样子：
![人工神经网络神经元](http://img.m.xinwanapp.com/t012998e32444be9117.jpg)
更详细的分解开来是这样子：
![人工神经网络神经元](http://img.m.xinwanapp.com/t01aa8081bf06232984.jpg)

人工神经元的数学表达式为：
```
z = w1 * x1 + w2 * x2 + ... + b
y = f(z)

# 表示成矩阵形式
y = f( W * X + b )
```
$$E=mc^2$$
`关于人工神经元的一些个人理解：`
> 可以把一个神经元近似看做一个投票系统。
> 
> 以有三个输入的神经元为例。有人问我一道菜是不是好吃，有ABC三个人给我提供了意见，A（好吃），BC（不好吃），他们提供的意见可以对应到3个输入，分别为[1，-1，-1]。但是这三个人的口味跟我并不是太一致，AB跟我的口味比较接近（口味对应到权值w），但跟C的口味相反，再考虑到我看过这道菜，看起来好像还是比较好吃的（对应到偏置），综合这几个条件，我给这道菜打了一个分。1 * 1 + （-1）* 1 + （-1）* （-1） + 0.5 = 1.5 分，于是我得出一个结论，这道菜是好吃的


- ##### 多层神经网络
多层神经网络就是由多个人工神经元按照一定的规则组合起来构成的网络，例如最常见的全连接网络，由多层结构，每一层的神经元相互之间无连接，而层之间是全连接，即某一层的一个神经元与其前一层的每一个神经元都有连接，与其后一层的每个神经元也都有连接
![多层神经网络](http://img.m.xinwanapp.com/t012a1817bf41817065.png)

`关于多层神经网络的一些个人理解：`
> 可以把多层神经网络近似看成一个分层选举系统 
>
> 跟召开全国人民代表大会一样，对于某个议题，首先是村里的人大代表听取全体村民的意见，汇总之后得到自己的结论。之后村级人大代表将自己的意见传达给县里的人大代表，县里的人大代表也在听取了各村人大代表的意见后继续向上汇报，最终国家领导人听取了各全国人大代表的意见，并最终做出自己的判断。（顺带一提，与这个例子更接近的可能是卷积网络而非全连接的网络，因为每个村里的人大代表只听取本村村民的意见）

- ##### 卷积神经网络
![卷积神经网络示意图](http://img.m.xinwanapp.com/t01ba60d6e69f315042.jpg)

- ##### 关于梯度下降法
训练神经网络的主要工作就是要找到一组参数W，使得输入参数在通过神经网络后得到预期的结果。为了得到这些参数，常常会用到梯度下降法进行求解。而梯度下降法会根据函数当前计算值与目标值之间的误差，以及函数的梯度求解出最佳值。这部分是可以说是神经网络训练中最麻烦的一部分，而TensorFlow等深度学习框架则是替我们完成了这些复杂导数的计算，大大降低了我们使用神经网络的门槛
![梯度下降法示意图](http://img.m.xinwanapp.com/t01af74ee28384e778d.jpg)
`以求解函数 y = x^2 / 2 的最小值为例`
> 首先，该函数的导数 y' = x。最初随机选定一个初始值 x = 5，y = 12.5，y' = 5，求解最小值需要将 x 向梯度的负方向移动，假设步长为2，则下一次迭代的时候 x = 3, y = 4.5, y' = 3，继续，x = 1, y = 0.5, y' = 1，梯度依旧大于0，x继续往负方向移动，x = -1, y = 0.5, y' = -1，此时梯度的符号发生了改变，x向正方向移动，x = 1, y = 0.5, y' = 1。此时，函数的最小值已经不再继续变小，就可以认为函数已经收敛了，我们已经得到了问题的解也就是 函数的最小值为 0.5

### TensorFlow主要功能的使用与演示
- 训练数据的保存与加载
- 模型的保存与加载
- TensorfBoard的功能与使用

`训练数据保存与加载示例`
``` python
import os
import numpy as np
import tensorflow as tf
from PIL import Image

sourceDir = "source"
outPath = "data.record"
tfWriter = tf.python_io.TFRecordWriter(outPath + ".record")

# save data
for imgName in os.listdir(sourceDir):
    imgPath = sourceDir + "/" + imgName
    img = Image.open(imgPath)
    img = img.resize((400, 400), Image.BICUBIC)

    imgRaw = img.tobytes()
    example = tf.train.Example(features=tf.train.Features(feature={
        'img': tf.train.Feature(bytes_list=tf.train.BytesList(value=[imgRaw]))
    }))
    tfWriter.write(example.SerializeToString())
tfWriter.close()

# read data
queue = tf.train.string_input_producer([outPath + ".record"])

reader = tf.TFRecordReader()
_, serialize = reader.read(queue)

features = tf.parse_single_example(serialize, features = {
    'img' : tf.FixedLenFeature([], tf.string),
})

img = tf.decode_raw(features['img'], tf.uint8)
img = tf.reshape(img, [400, 400, 3])

coord = tf.train.Coordinator()

with tf.Session() as sess:
    threads = tf.train.start_queue_runners(sess = sess, coord = coord)
    imgData = sess.run(img)
    
restoreImg = Image.fromarray(imgData, "RGB")
restoreImg.show()
```

`mnist示例`
``` python
# Copyright 2015 The TensorFlow Authors. All Rights Reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
# ==============================================================================

"""A deep MNIST classifier using convolutional layers.

See extensive documentation at
https://www.tensorflow.org/get_started/mnist/pros
"""
# Disable linter warnings to maintain consistency with tutorial.
# pylint: disable=invalid-name
# pylint: disable=g-bad-import-order

from __future__ import absolute_import
from __future__ import division
from __future__ import print_function

import argparse
import sys

from tensorflow.examples.tutorials.mnist import input_data

import tensorflow as tf

FLAGS = None


def deepnn(x):
    """deepnn builds the graph for a deep net for classifying digits.

    Args:
        x: an input tensor with the dimensions (N_examples, 784), where 784 is the
        number of pixels in a standard MNIST image.

    Returns:
        A tuple (y, keep_prob). y is a tensor of shape (N_examples, 10), with values
        equal to the logits of classifying the digit into one of 10 classes (the
        digits 0-9). keep_prob is a scalar placeholder for the probability of
        dropout.
    """
    # Reshape to use within a convolutional neural net.
    # Last dimension is for "features" - there is only one here, since images are
    # grayscale -- it would be 3 for an RGB image, 4 for RGBA, etc.
    with tf.name_scope('reshape'):
        x_image = tf.reshape(x, [-1, 28, 28, 1])

    # First convolutional layer - maps one grayscale image to 32 feature maps.
    with tf.name_scope('conv1'):
        W_conv1 = weight_variable([5, 5, 1, 32])
        b_conv1 = bias_variable([32])
        h_conv1 = tf.nn.relu(conv2d(x_image, W_conv1) + b_conv1)

    # Pooling layer - downsamples by 2X.
    with tf.name_scope('pool1'):
        h_pool1 = max_pool_2x2(h_conv1)

    # Second convolutional layer -- maps 32 feature maps to 64.
    with tf.name_scope('conv2'):
        W_conv2 = weight_variable([5, 5, 32, 64])
        b_conv2 = bias_variable([64])
        h_conv2 = tf.nn.relu(conv2d(h_pool1, W_conv2) + b_conv2)

    # Second pooling layer.
    with tf.name_scope('pool2'):
        h_pool2 = max_pool_2x2(h_conv2)

    # Fully connected layer 1 -- after 2 round of downsampling, our 28x28 image
    # is down to 7x7x64 feature maps -- maps this to 1024 features.
    with tf.name_scope('fc1'):
        W_fc1 = weight_variable([7 * 7 * 64, 1024])
        b_fc1 = bias_variable([1024])

        h_pool2_flat = tf.reshape(h_pool2, [-1, 7*7*64])
        h_fc1 = tf.nn.relu(tf.matmul(h_pool2_flat, W_fc1) + b_fc1)

    # Dropout - controls the complexity of the model, prevents co-adaptation of
    # features.
    with tf.name_scope('dropout'):
        keep_prob = tf.placeholder(tf.float32)
        h_fc1_drop = tf.nn.dropout(h_fc1, keep_prob)

    # Map the 1024 features to 10 classes, one for each digit
    with tf.name_scope('fc2'):
        W_fc2 = weight_variable([1024, 10])
        b_fc2 = bias_variable([10])
        y_conv = tf.matmul(h_fc1_drop, W_fc2) + b_fc2
    return x_image, y_conv, keep_prob

def conv2d(x, W):
    """conv2d returns a 2d convolution layer with full stride."""
    return tf.nn.conv2d(x, W, strides=[1, 1, 1, 1], padding='SAME')

def max_pool_2x2(x):
    """max_pool_2x2 downsamples a feature map by 2X."""
    return tf.nn.max_pool(x, ksize=[1, 2, 2, 1], strides=[1, 2, 2, 1], padding='SAME')

def weight_variable(shape):
    """weight_variable generates a weight variable of a given shape."""
    initial = tf.truncated_normal(shape, stddev=0.1)
    return tf.Variable(initial)

def bias_variable(shape):
    """bias_variable generates a bias variable of a given shape."""
    initial = tf.constant(0.1, shape=shape)
    return tf.Variable(initial)

def main(_):
    with tf.Session() as sess:
        # Import data
        mnist = input_data.read_data_sets(FLAGS.data_dir, one_hot=True)

        # Build the graph for the deep net
        if FLAGS.load_model == False:
            # Create the model
            x = tf.placeholder(tf.float32, [None, 784])
            y_ = tf.placeholder(tf.float32, [None, 10])

            x_image, y_conv, keep_prob = deepnn(x)

            with tf.name_scope('loss'):
                loss = tf.nn.softmax_cross_entropy_with_logits(labels=y_, logits=y_conv)
            cross_entropy = tf.reduce_mean(loss)

            with tf.name_scope('adam_optimizer'):
                train_step = tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)

            with tf.name_scope('accuracy'):
                correct_prediction = tf.equal(tf.argmax(y_conv, 1), tf.argmax(y_, 1))
                correct_prediction = tf.cast(correct_prediction, tf.float32)
                accuracy = tf.reduce_mean(correct_prediction)

            tf.add_to_collection('x', x)
            tf.add_to_collection('y_', y_)
            tf.add_to_collection('x_image', x_image)
            tf.add_to_collection('y_conv', y_conv)
            tf.add_to_collection('keep_prob', keep_prob)
            tf.add_to_collection('loss', loss)
            tf.add_to_collection('cross_entropy', cross_entropy)
            tf.add_to_collection('train_step', train_step)
            tf.add_to_collection('accuracy', accuracy)

            # save model
            saver = tf.train.Saver()

            # init variables，恢复模型的时候不能调用，否则会把模型中已经保存的参数重新初始化
            sess.run(tf.global_variables_initializer())
        else:
            print("Recover model from %s" % FLAGS.model_dir)
            saver = tf.train.import_meta_graph(FLAGS.model_dir + "/model.meta")
            # saver = tf.train.Saver(filename=FLAGS.model_dir + "/model.meta")
            saver.restore(sess, FLAGS.model_dir + "/model")

            x = tf.get_collection('x')[0]
            y_ = tf.get_collection('y_')[0]
            x_image = tf.get_collection('x_image')[0]
            keep_prob = tf.get_collection('keep_prob')[0]
            y_conv = tf.get_collection('y_conv')[0]
            loss = tf.get_collection('loss')[0]
            cross_entropy = tf.get_collection('cross_entropy')[0]
            train_step = tf.get_collection('train_step')[0]
            accuracy = tf.get_collection('accuracy')[0]
            print("Restore graph success")
            print(accuracy)

            # graph = tf.get_default_graph()
            # tf.contrib.graph_editor.get_tensors(graph)
            # accuracy = graph.get_tensor_by_name('accuracy/Mean:0')

        # Add some summary
        tf.summary.image('x_image', x_image, 10)
        tf.summary.scalar('mean_loss', cross_entropy)
        tf.summary.scalar('stddev', tf.sqrt(tf.reduce_mean(tf.square(loss - cross_entropy))))
        tf.summary.scalar('accuracy', accuracy)
        merged = tf.summary.merge_all()
        train_writer = tf.summary.FileWriter(FLAGS.log_dir, tf.get_default_graph())
        # train_writer.add_graph(tf.get_default_graph())

        for i in range(20000):
            batch = mnist.train.next_batch(50)
            if i % 100 == 0:
                train_accuracy, summary = sess.run([accuracy, merged], feed_dict={x: batch[0], y_: batch[1], keep_prob: 1.0})
                print('step %d, training accuracy %g' % (i, train_accuracy))
                train_writer.add_summary(summary, i)
                saver.save(sess, FLAGS.model_dir + "/model")
            sess.run(train_step, feed_dict={x: batch[0], y_: batch[1], keep_prob: 0.5})

        # print('test accuracy %g' % accuracy.eval(feed_dict={x: mnist.test.images, y_: mnist.test.labels, keep_prob: 1.0}))

if __name__ == '__main__':
    parser = argparse.ArgumentParser()
    parser.add_argument('--data_dir', type=str, default='./data_dir', help='Directory for storing input data')
    parser.add_argument('--log_dir', type=str, default='./log_dir', help='Directory for storing log data')
    parser.add_argument('--model_dir', type=str, default='./model_dir', help='Directory for storing model data')
    parser.add_argument('--load_model', default=False, help='Whether load old model or build new model')
    FLAGS, unparsed = parser.parse_known_args()
    tf.app.run(main=main, argv=[sys.argv[0]] + unparsed)
```