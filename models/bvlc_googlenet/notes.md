# What's this for
This is the study notes for reading codes for **GoogLeNet**

## deploy.prototxt
### line 6:
`input_param { shape: { dim: 10 dim: 3 dim: 224 dim: 224 } }`
Here is the definition  
`dim:10` 表示对待识别样本进行数据增广的数量，即表示1个图像会变成10个输入网络进行识别  
`dim:3` 表示图片的通道数  
`dim:224` 表示图片的长度  
`dim:224` 表示图片的宽度

### line 13 -- line 20:

		param {
    		lr_mult: 1  
    		decay_mult: 1
		}
		param {
    		lr_mult: 2
    		decay_mult: 0
		}
 
 这是在`Convolution`层中的参数，前面的param是对filters做的限制，后一个param是对biases做的限制。
 
lr\_mult:  learning rate multiplier for the filters  
decay\_mult: learning rate multiplier for the biases

In this case, we will set the weight learning rate to be the same as the learning rate given by the solver during runtime, and the bias learning rate to be twice as large as that - **this usually leads to better convergence rates.**

### line 23 
what is `pad`?

h\_o = (h\_i + 2 * pad\_h - kernel\_h) / stride\_h + 1

_o indicates output value and _i is input correspondingly.

### line 27 
	weight_filler {
		type: "xavier"
		std: 0.1
	}

In Caffe, it’s initializing the weights in your network by drawing them from a distribution with zero mean and a specific variance,

**Var(W) = 1/n**

where W is the initialization distribution for the neuron in question, and n is the number of neurons feeding into it. The distribution used is typically Gaussian or uniform.

references: [xavier](http://andyljones.tumblr.com/post/110998971763/an-explanation-of-xavier-initialization)

### line 55
	layer {
	  name: "pool1/norm1"
	  type: "LRN"
	  bottom: "pool1/3x3_s2"
	  top: "pool1/norm1"
	  lrn_param {
	    local_size: 5
	    alpha: 0.0001
	    beta: 0.75
	  }
	}
	
LRN层：LRN的几个超参数，local_size, alpha, beta等，需要在**验证集**上选择最优值  
每个输入值都将除以![lrn formula](http://img.blog.csdn.net/20141219214151464)  
n就是local_size的值

### line 357
	layer {
	  name: "inception_3a/output"
	  type: "Concat"
	  bottom: "inception_3a/1x1"
	  bottom: "inception_3a/3x3"
	  bottom: "inception_3a/5x5"
	  bottom: "inception_3a/pool_proj"
	  top: "inception_3a/output"
	}

`type: "Concat"` 结合层  

	if axis = 0: (n_1 + n_2 + ... + n_K) * c_1 * h * w, and all input c_i should be the same.
	if axis = 1: n_1 * (c_1 + c_2 + ... + c_K) * h * w, and all input n_i should be the same.
	
**Default axis=1**  
you can alter axis like below:
	
	concat_param {
		axis: 1
	}

Reference: [Concat Layer](http://caffe.berkeleyvision.org/tutorial/layers.html)

## solver.prototxt
