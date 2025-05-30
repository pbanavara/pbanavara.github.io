In the world of optimizing deep neural networks, torch compile and triton are perhaps the most important tools.I had no idea about the fundamentals so I dug in using first principles aided by ChatGPT.

#### A bit of history
In the early days of deep learning when Tensorflow came about in 2015 or so, it heralded a new age for building AI. This new age coincided with the introduction of the landmark [Imagenet](https://arxiv.org/abs/1409.0575) paper that single handedly helped advance deep learning to where it is today. Why am I reminiscing about an age old technology or framework. It's because Tensorflow was rooted in optimization. It came from the designers of battle hardened devs, who lived and breathed scale. 

In Tensorflow, you built a static computation graph, defined everything upfront, then executed a session. This means, the compiler has full control to optimize every operation and just run it. 

``` python
import tensorflow as tf

# Define graph
x = tf.placeholder(tf.float32, shape=[None, 10])
W = tf.Variable(tf.random_normal([10, 5]))
b = tf.Variable(tf.zeros([5]))
y = tf.matmul(x, W) + b

loss = tf.reduce_mean(tf.square(y))  # Some loss function
optimizer = tf.train.AdamOptimizer(0.001).minimize(loss)

# Execute in a session
with tf.Session() as sess:
    sess.run(tf.global_variables_initializer())
    for i in range(100):
        batch_x = np.random.randn(32, 10)
        sess.run(optimizer, feed_dict={x: batch_x})
```

Then came Pytorch around 2016-17. Pytorch was built by researchers and devs, not the battle hardened resource optimizers. Quite naturally, the focus was on rapid iteration, ease of use and best of all, step by step debugging. The same Tensorflow graph now written in Pytorch looks like this:

``` python
import torch
from torch.autograd import Variable
import torch.nn.functional as F
import numpy as np

# Define variables
W = Variable(torch.randn(10, 5), requires_grad=True)
b = Variable(torch.zeros(5), requires_grad=True)
lr = 0.001

# Training loop
for i in range(100):
    batch_x = np.random.randn(32, 10).astype(np.float32)
    x = Variable(torch.from_numpy(batch_x))

    y = torch.matmul(x, W) + b  # Equivalent to tf.matmul(x, W) + b
    loss = torch.mean((y ** 2))  # Equivalent to tf.reduce_mean(tf.square(y))

    # Backward and optimize manually
    loss.backward()
    W.data -= lr * W.grad.data
    b.data -= lr * b.grad.data

    # Clear gradients
    W.grad.data.zero_()
    b.grad.data.zero_()
```
The key difference is that Pytorch is in eager mode. This means, the computation graph is not built upfront. Instead, the computation is executed in a step by step manner. This is what made Pytorch so easy to use and debug.

#### Convergence
While Pytorch popularity exploded, Tensorflow was still the go to framework for production but the rise of Pytorch didn't go unnoticed. Tensorflow shot back with Eager mode in version 2.0. Pytorch introduced Torchscript to statically compile Pytorch modules. Both the frameworks started converging but Pytorch had an adoption growth advantage and there was no looking back.

#### Torchscript and how it speeds up Pytorch
``` python
# Define the model as an nn.Module
class SimpleModel(nn.Module):
    def __init__(self):
        super(SimpleModel, self).__init__()
        self.linear = nn.Linear(10, 5)

    def forward(self, x):
        return self.linear(x)

# Instantiate model
model = SimpleModel()

# Compile to TorchScript using scripting (preferred if logic exists)
scripted_model = torch.jit.script(model)
```

Now scripted_model is a compiled version of SimpleModel. It is a static graph. This means, the compiler has full control to optimize every operation and just run it, either on the CPU or GPU. The magic happens in this one line

``` python
scripted_model = torch.jit.script(model)
```
The following happens under the hood:
- Infers static types for inputs and outputs. Duck typing, list comprehensions etc is not supported.
- Supports only a subset of Python control flow (if, for, while, break, continue, return, yield, try, except, finally, with, assert, import, global, nonlocal).
  
Here's the IR representation of the compiled graph
``` python
graph(%self : __torch__.SimpleModel,
      %x.1 : Float(32:10)):
  %linear : __torch__.torch.nn.modules.linear.Linear = prim::GetAttr[name="linear"](%self)
  %weight : Float(5, 10) = prim::GetAttr[name="weight"](%linear)
  %bias : Float(5) = prim::GetAttr[name="bias"](%linear)
  %x.2 : Float(32, 5) = aten::linear(%x.1, %weight, %bias)
  return (%x.2)
  ```
  Good luck debugging this. But, you don't have to. You debug in eager mode and when you are ready to deploy you rewrite using torchscript compile and deploy. 

  #### Torch compile
  Why would you want to rewrite your model. That soon became ancient and Pytorch gave us torch compile in 2.0. 
  ``` python
  model = torch.compile(model)
  ```
  That's it. Full Python eager support, you didn't have to look for print gotchas, list comprehension breakages and all the other gotchas, that you had to deal with in torchscript. It was a gigantic step forward for Pytorch. They had to, because TensorFlow was already way ahead in this game with the launch of XLA and Tensorflow 2.0.

  SO what actually happens in torch compile?
  - First there is torch dynamo which scans your python code for pure tensor operations and skips the rest.
  - Uses Pytorch's FX to convert these tensor operations into a graph.
  - AOTAutograd (Ahead of time Autograd) is used to separate the forward and backward passes.
  - Primtorch - reduces pytorch ops like nn.linear to primitives like aten::linear.
  - Hands off this graph to the compiler backend (TorchInductor or Triton) to execute.

Everything seems straightforward except for this AOTTAutograd. What is it? To understand AOTAutograd, we need to understand how Pytorch does backprop.
  #### Autograd
  In simple terms, Each tensor operation adds a node to the dynamic autograd graph. When .backward() is called, PyTorch walks this graph backward to compute gradients. Meaning every node of the graph has to be executed as an individual python call. This is the reason why eager mode is slow.

  So let's say you have a neural net
  fc1 → relu → fc2 → loss

  When you call .backward() on the loss, Pytorch has to do this

  fc1 → relu → fc2 → loss
          ↑        ↑
         op-by-op backward

#### AOTAutograd
Now AOT Autograd converts this into 2 separate graphs
(FC1-> ReLU -> FC2 -> Loss) → compiled forward graph
(∇FC2 -> ∇ReLU -> ∇FC1)     → compiled backward graph

One might ask, what is the great advantage here. For this one needs to undertand a bit of CUDA programming. Let's aay you are adding 3 numbers. And you do it something like this.

``` python
res1 = a + b
res2 = res1 + c
```
On the GPU it's a lot faster to just do this
``` python
res = a + b + c
```
Why is the latter op faster on the GPU. Because on the GPU, the overhead is not compute, it's always memory. GPUs are memory bound. So the GPU is always waiting for data to be fetched from memory. So, if you can reduce the number of memory fetches, you can get a lot of performance boost.

Now the AOTAutograd can simply do this on the GPU

(FC1 + ReLU + FC2 + Loss) → compiled forward graph - one fused kernel
(∇FC2 + ∇ReLU + ∇FC1)     → compiled backward graph - one fused kernel

This is why AOTAutograd is so much faster. The ops are ultimately executed by the compiler backends but the transformations are done by AOTAutograd. The above fusion of multiple operations into one kernel is often called [kernel fusion](https://dl.acm.org/doi/pdf/10.1109/GreenCom-CPSCom.2010.102).

The holy grail of complier optimization is to fuse as many operations as possible and avoid memory fetches on the GPU. 

#### Triton and Torch Inductor
Now that operations are fused at the graph level (thanks to AOTAutograd), someone has to generate and execute efficient kernels for them. One way to do this is by writing custom CUDA kernels — but doing that manually for every model or fusion pattern is wasteful and unscalable.
To solve this, PyTorch introduced a compiler backend called TorchInductor.

- TorchInductor takes the fused operations from the graph,
- Applies backend-specific optimizations,

And generates performant low-level code, including CUDA, CPU, or Metal kernels.

Writing this codegen infrastructure from scratch is expensive, so the PyTorch team leveraged Triton — a GPU DSL + compiler developed independently (originally by OpenAI) — to handle pointwise ops, reductions, and simple memory-access patterns.

So while TorchInductor and Triton were developed somewhat in parallel, both teams avoided reinventing the wheel:
TorchInductor handles the graph lowering, scheduling, orchestration.
Triton generates fused CUDA kernels for the right subset of operations.
```
torch.compile()
 └──> TorchDynamo → FX graph
      └──> AOTAutograd (forward + backward separation)
           └──> TorchInductor
                └──> Generates CUDA via:
                       - Triton (for pointwise ops)
                       - C++/MLIR (for others)
```
With tools like Triton and TorchInductor, PyTorch has transformed from a research-first framework to a true high-performance ML compiler stack. The next frontier? Training and deploying billion-parameter models with zero custom CUDA. We're almost there.