# CNN-
建立卷积层

接着我们定义第一层卷积,先定义本层的Weight,本层我们的卷积核patch的大小是5x5，因为黑白图片channel是1所以输入是1，输出是32个featuremap

W_conv1=weight_variable([5,5,1,32])
接着定义bias，它的大小是32个长度，因此我们传入它的shape为[32]

b_conv1=bias_variable([32])
定义好了Weight和bias，我们就可以定义卷积神经网络的第一个卷积层h_conv1=conv2d(x_image,W_conv1)+b_conv1,同时我们对h_conv1进行非线性处理，也就是激活函数来处理喽，这里我们用的是tf.nn.relu（修正线性单元）来处理，要注意的是，因为采用了SAME的padding方式，输出图片的大小没有变化依然是28x28，只是厚度变厚了，因此现在的输出大小就变成了28x28x32

h_conv1=tf.nn.relu(conv2d(x_image,W_conv1)+b_conv1)
最后我们再进行pooling的处理就ok啦，经过pooling的处理，输出大小就变为了14x14x32

h_pool=max_pool_2x2(h_conv1)
接着呢，同样的形式我们定义第二层卷积，本层我们的输入就是上一层的输出，本层我们的卷积核patch的大小是5x5，有32个featuremap所以输入就是32，输出呢我们定为64

W_conv2=weight_variable([5,5,32,64])
b_conv2=bias_variable([64])
接着我们就可以定义卷积神经网络的第二个卷积层，这时的输出的大小就是14x14x64

h_conv2=tf.nn.relu(conv2d(h_pool1,W_conv2)+b_conv2)
最后也是一个pooling处理，输出大小为7x7x64

h_pool2=max_pool_2x2(h_conv2)
建立全连接层

好的，接下来我们定义我们的 fully connected layer,

进入全连接层时, 我们通过tf.reshape()将h_pool2的输出值从一个三维的变为一维的数据, -1表示先不考虑输入图片例子维度, 将上一个输出结果展平.

#[n_samples,7,7,64]->>[n_samples,7*7*64]
h_pool2_flat=tf.reshape(h_pool2,[-1,7*7*64]) 
此时weight_variable的shape输入就是第二个卷积层展平了的输出大小: 7x7x64， 后面的输出size我们继续扩大，定为1024

W_fc1=weight_variable([7*7*64,1024]) 
b_fc1=bias_variable([1024])
然后将展平后的h_pool2_flat与本层的W_fc1相乘（注意这个时候不是卷积了）

h_fc1=tf.nn.relu(tf.matmul(h_pool2_flat,W_fc1)+b_fc1)
如果我们考虑过拟合问题，可以加一个dropout的处理

h_fc1_drop=tf.nn.dropout(h_fc1,keep_drop)
接下来我们就可以进行最后一层的构建了，好激动啊, 输入是1024，最后的输出是10个 (因为mnist数据集就是[0-9]十个类)，prediction就是我们最后的预测值

W_fc2=weight_variable([1024,10]) b_fc2=bias_variable([10])
然后呢我们用softmax分类器（多分类，输出是各个类的概率）,对我们的输出进行分类

prediction=tf.nn.softmax(tf.matmul(h_fc1_dropt,W_fc2),b_fc2)
选优化方法

接着呢我们利用交叉熵损失函数来定义我们的cost function

cross_entropy=tf.reduce_mean(
    -tf.reduce_sum(ys*tf.log(prediction),
    reduction_indices=[1]))
我们用tf.train.AdamOptimizer()作为我们的优化器进行优化，使我们的cross_entropy最小

train_step=tf.train.AdamOptimizer(1e-4).minimize(cross_entropy)
接着呢就是和之前视频讲的一样喽 定义Session

sess=tf.Session()
初始化变量

# tf.initialize_all_variables() 这种写法马上就要被废弃
# 替换成下面的写法:
sess.run(tf.global_variables_initializer())
好啦接着就是训练数据啦，我们假定训练1000步，每50步输出一下准确率， 注意sess.run()时记得要用feed_dict给我们的众多 placeholder 喂数据哦.
