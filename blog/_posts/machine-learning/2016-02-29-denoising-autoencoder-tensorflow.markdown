---
layout: post
title:  "Denoising Autoencoders: Tutorial + TensorFlow implementation"
titlecol: "#000000"
date:   2016-02-28 13:30:20 -0500
author: Gabriele Angeletti
categories: machine-learning
image: img/autoencoder.png
---
Denoising Autoencoders are a special kind of Neural Network trained to extract meaningful and **robust** features from the input data. They can be stacked to form Deep Neural Networks, and their performance is state-of-the-art in many popular benchmarks.

This tutorial is an overview of autoencoders. First, the general model is described, then we'll see what is a *denoising* autoencoder (and why it works better) and how we can stack them to obtain Deep Networks.
The implementation of the algorithm is written in Python using [TensorFlow][tf], and you can find it at this [github gist][gs].

**Note**: this tutorial is for people with at least some working knowledge of Neural Networks, and it is only intended as a brief overview of the topic. I recommend you to read the [original paper][sdaepaper] for further reference.

#### What is an Autoencoder?

Put simply, an autoencoder is a [Feed Forward Neural Network][ffnn] trained to reconstruct its input. As usual, you have an input layer, a hidden layer, and an output layer. If you have used NNs for classification tasks, you may be used to the idea that the output layer has as many nodes as there are classes in the dataset. For example, if you have worked with [MNIST][mnst] dataset (the hello world of machine learning), you saw that the output of the network was a 10 nodes (because MNIST has 10 classes) *softmax layer* where the i-th node represents the probability that the input pattern belong to the i-th class.

<img src="../../../../../img/autoencoder.png" alt="neural networks architecture">

Autoencoders are an unsupervised learning algorithm though, so labels are either not available or we don't care about them. Autoencoders are trained to reconstruct their input, and in order to do this, it is clear that the input and the output layers must have the same dimensionality.

Thus, an autoencoder learns two mappings: the first from the input to the hidden layer (we call this mapping the **encoder**), and the second from the hidden to the output layer (this one is called the **decoder**).

It's easy to see why they are called encoder/decoder: the encoder maps data from the input space to the hidden space, and the decoder maps encoded data back to the original space. You can think of the hidden representation as a *code*, from which the original data can be decoded back. Of course, the code must capture the main features in the data, otherwise it would be impossible to extract something meaningful from it.

#### Autoencoder Algorithm

**Note**: In the following notation $x$ is the input, $y$ is the encoded data, $z$ is the decoded data, $\sigma$ is a non-linear activation function (sigmoid or hyperbolic tangent usually), and $f(x ; \theta)$ means that $f$ is a function of $x$ parameterized by $\theta$.

The model can be summarized in the following way:

* ###### The input data is mapped onto the hidden layer (*encoding*). The mapping is usually an affine transformation  followed by a non-linearity. Something like:
   $$ y = f(x ; \theta) = \sigma(Wx + b) $$
* ###### The hidden layer is mapped onto the output layer (*decoding*). The mapping is an affine transformation optionally followed by a non-linearity. Something like:
$$ z = g(y ; \theta^{'}) = g(f(x ; \theta) ; \theta^{'}) = \sigma(W^{'}y + b^{'}) $$
* ###### Optionally, in order to reduce the size of the model, one can use *tied weights*, that is the decoder weights matrix is constrained to be the transpose of the encoder weights matrix: $\theta^{'} = \theta^{T}$.
* ###### The output of the autoencoder is matched against the original input using a specific criterion (cross entropy loss or mean squared error usually). Something like:
$$ e = \frac{1}{2m} \sum_{i = 1}^{m}|| x_{i} - z_{i} ||^{2} $$ for mse or
$$ e = - \frac{1}{m} \sum_{i = 1}^{m} x_{i} log(z_{i}) + (1 - x_{i})log(1 - z_{i})$$ for ce.
* ###### Using [Backpropagation][bp], small changes are made to the parameters of the model to make the reconstructions more similar to the original input. Note that with TensorFlow you don't have to explicitly compute the gradients. The Backpropagation algorithm is automatically applied to your model using one of the available [optimizers][tfopt].

#### Notes on different Architectures

The hidden layer can have either a lower or a higher dimensionality than that of the input/output layers.

In the former case, the decoder reconstructs the original input from a lower-dimensional representation of it (we call it an **under-complete** representation). For the overall thing to work, the encoder should learn to provide a low-dimensional representation that captures the essence of the data (i.e. the main factors of variations in the distribution). Intuitively, if the data was a five minute speach, the encoder would have to describe it in one minute, in other words it is forced to find a good way to summarize the data.

In the latter case, the decoder reconstruction is build from a higher-dimensional representation (we call this one an **over-complete** representation). You might think that these kind of representations are kind of useless. If the input dimension is 500, and the hidden layer dimension is 700, there is a great chance that the autoencoder will use 500 of its 700 hidden units to learn the identity mapping, and "throw away" the others.
Although this can happen, it turns out that if we constrain the hidden units so that only a fraction of them are active at any given time, it can act as a strong regularizer making the autoencoder work surprisingly well. These are the so called [Sparse Autoencoders][sae].

#### Denoising Autoencoders

So what are Denoising autoencoders? They are very similar to the model we saw previously, the difference is that in this case the input is **corrupted** before being passed to the network. The trick is that by matching the original version (not the corrupted one) with the reconstruction at error computing time, the autoencoder is trained to reconstruct the *original input* from the corrupted version. An approximate schema is the following:

* ###### the input $x$ gets corrupted by a function $q(x) = x_{corr}$.
* ###### $x_{corr}$ is used as in the previous model:
  * ###### $$ y = f(x_{corr} ; \theta) = \sigma(Wx_{corr} + b) $$
  * ###### $$ z = g(y ; \theta^{'}) = g(f(x_{corr} ; \theta) ; \theta^{'}) = \sigma(W^{'}y + b^{'}) $$
* ###### The error computation is exactly the same as the previous model (use the original x)

Intuitively, if we observe an autoencoder capable of taking in input a distorted version of something and returning a good reconstruction of it, it means that it was very clever about the representation it has learned, otherwise it wouldn't be able to reconstruct the data.
To get a sense of what corrupt the input means, below is an image taken from the MNIST dataset, you can clearly see which is the original and which is the corrupted version.

<img src="../../../../../img/8.png" width="100px !important" alt="image of an eight">
<img src="../../../../../img/8c.png" alt="image of a distorted eight">

The corruption technique used in this image is the **masking noise** technique, that is a fraction of the input elements taken at random is set to zero. In the image above, the fraction was 50% of the pixels.
The other techniques discussed in the paper are the **salt and pepper noise**, where a fraction of the input elements taken at random is set either to the maximum value or to the minimum, and the **white gaussian noise**. The authors emphasize the fact that they used these methods because they are very general and can be applied to a wide variety of data. Of course, if you know well the domain at hand (e.g. images) and use a technique to better expose the main structure in your data, the algorithm is expected to work better.

**Note**: The corruption process has to be repeated each time the same input is presented to the autoencoder. If the training set is corrupted only once at the beginning of training, the model would recognize the distortions as valid data patterns, leading to **overfitting**. By corrupting the data every time in a different way instead, the model will learn to extract the underlying structure, avoiding overfitting.

#### Stacked Denoising Autoencoders

We can stack autoencoders in order to obtain Deep Neural Networks. The stacking process is done in the following way for a classification task:

* ###### An Autoencoder (denoising or not) is "unsupervisely" trained on the input data. After training, we drop the decoder, and we encode the training set: encoded training set = $\sigma(W_{1}x + b_{1})$.
* ###### The encoded data is used as training set for a second Autoencoder. After training, we drop the decoder, and we encode the encoded training set: $\sigma(W_{2}\sigma(W_{1}x + b_{1}) + b_{2})$
* ###### We do this for as many layers we want. At the end, we attach a softmax layer on top of the last autoencoder's encoder, and we train the whole thing as if it was a simple Feed Forward Neural Network using Backpropagation (of course the initial weights of the Neural Net must be initialized at the value that the autoencoders had after training, otherwise the pretraining procedure would be useless).

The [github gist][gs] contains only an implementation of a Denoising Autoencoder. If you want to see a working implementation of a Stacked Autoencoder, as well as many other Deep Learning algorithms, I encourage you to take a look at [my repository][mydptf] of Deep Learning algorithms implemented in TensorFlow.

[tf]: https://www.tensorflow.org/
[gs]: https://gist.github.com/blackecho/3a6e4d512d3aa8aa6cf9
[sdaepaper]: http://www.jmlr.org/papers/volume11/vincent10a/vincent10a.pdf
[ffnn]: https://en.wikipedia.org/wiki/Feedforward_neural_network
[mnst]: http://yann.lecun.com/exdb/mnist/
[bp]: http://mattmazur.com/2015/03/17/a-step-by-step-backpropagation-example/
[tfopt]: https://www.tensorflow.org/versions/r0.7/api_docs/python/train.html#optimizers
[sae]: https://web.stanford.edu/class/cs294a/sparseAutoencoder.pdf
[mydptf]: https://github.com/blackecho/Deep-Learning-TensorFlow
