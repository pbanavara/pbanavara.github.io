### Demystifying LORA

The advent of LLMs has brought upon a paradigm shift in every task that we perform today. From search to coding complex apps, we have learned to use and harness LLMs to perform tasks that were once thought to be impossible. The growth of LLMs has been so phenomenal that even Mary Meeker who had not published a report since 2019, published a [report on AI](https://www.bondcap.com/report/pdf/Trends_Artificial_Intelligence.pdf) this year.

With growth, comes the challenges of utlization. What was once abundant becomes scarce and this is becoming true of GPUs, the workshorse processors forming the bedrock of the LLM applications. One of the growing challenges with LLMs is that they cannot be deployed as is for all use cases. Typically, a giant model is pre-trained on a very large dataset. The latest [Llama4 herd](https://ai.meta.com/blog/llama-4-multimodal-intelligence/) stands out. The model is so big that they had to call it the behemoth. 
`
**Pushing Llama to new sizes: The 2T Behemoth**

We’re excited to share a preview of Llama 4 Behemoth, a teacher model that demonstrates advanced intelligence among models in its class. Llama 4 Behemoth is also a multimodal mixture-of-experts model, with 288B active parameters, 16 experts, and nearly two trillion total parameters. Offering state-of-the-art performance for non-reasoning models on math, multilinguality, and image benchmarks, it was the perfect choice to teach the smaller Llama 4 models. We codistilled the Llama 4 Maverick model from Llama 4 Behemoth as a teacher model, resulting in substantial quality improvements across end task evaluation metrics. We developed a novel distillation loss function that dynamically weights the soft and hard targets through training. Codistillation from Llama 4 Behemoth during pre-training amortizes the computational cost of resource-intensive forward passes needed to compute the targets for distillation for the majority of the training data used in student training. For additional new data incorporated in student training, we ran forward passes on the Behemoth model to create distillation targets.
`
What makes these large models finally useful is their application in chat like interfaces. This involves a process known as fine tuning. The fine tuning Arena is filled with jargon - PEFT, LORA, QLORA and is hard to wrap your head around. To simplify this understanding I took a very first principles driven approach with OpenAI O4-mini and finally understood the concept of LORA. Here is how I went about it.

Typically you start off with a huge dataset of text which will be encoded into embeddings. For simplicity sake, let's say this is a row vector of 10 dimensions.
```
x = numpy.random.rand(10)
```
Then you have a weight matrix corresponding to the embedding matrix of 10 x 10 dimensions.
```
W = numpy.random.rand(10, 10)
```
Then the model training is nothing but a series of matrix multiplications and derivative calculations.
```
y = numpy.dot(W, x)
```
Once all the training is done the weights which were random once now have some acquired intelligence from the training dataset.
This is very hand wavy but sufficient to understand fine tuning.

Now let's say you need to fine tune this model on a smaller dataset. We have a new input vector x' which is a row vector of again 10 dimensions
```
x' = numpy.random.rand(10)
```
What actually needs to happen is that the weights of the model need to be updated to accomodate the new input vector. So typically you would do something like this
```
y' = numpy.dot(W, x')
```
Now wait a minute. Why should we use the entire matrix W when we only need to update the weights corresponding to the new input vector. This is where LORA comes in. [Low rank adaptation](https://arxiv.org/abs/2106.09685). It's a fancy word for a simple idea, use smaller weight matrices without sacrificing accuracy. Why ? Because you can save on GPU memory. That's the obvious reason, but philosophically why spend 100$ when you can get the job done for 10 ?

And that's the linear algebra magic behind LORA. What you do is fairly simple. You take a lower 'rank' matrix let's say of the order 2.
```
A = numpy.random.rand(10, 2)
```
Now use another matrix B of the same order to 'up project' the lower rank matrix A.
```
B = numpy.random.rand(2, 10)
```
Now you do the matrix math with these updated matrices.
```
# x is 1×10
x = [[x0, x1, …, x9]]       # shape (1, 10)

# Aᵀ is 10×2
# Aᵀ[j][k] corresponds to A[k][j]
# Multiply x @ Aᵀ → shape (1, 2)
y_prime = matmul(x, transpose(A))
#   (1×10) @ (10×2) = (1×2)

# Bᵀ is 2×10
# Multiply y_prime @ Bᵀ → shape (1, 10)
y_double = matmul(y_prime, transpose(B))
#   (1×2) @ (2×10) = (1×10)
```
That's it. Now you have the trained weights in A and B. All you need to do now is merge these weights with the original Weight W.
```
W_new = W + A @ B
```
This is the simplicity of LORA and why it works so well. 

#### Applications of LORA
For those of us who spent years in enterprises, multi tenancy is a first principles approach because let's face it, every internal team is a tenant and they all want diferent SLAs from the services. Mechanisms liks LORA make this even more efficient because you can now fine tune a LLM for [each tenant](https://aws.amazon.com/blogs/machine-learning/efficient-and-cost-effective-multi-tenant-lora-serving-with-amazon-sagemaker/).

#### Innovation never stops
Is LORA the most effcient way to fine tune LLMs ? Yes, until a few months ago. Newer techniques such as [PARA](https://arxiv.org/pdf/2502.01033.pdf) are being developed which trounce LORA in specific scenarios related to multi tenancy. That being said, LORA is still the most generic and efficient way to fine tune LLMs. 


