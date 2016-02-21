---
layout: post
title:  "Sparse Distributed Representations in Machine Learning"
date:   2015-06-15 09:30:20 -0500
author: Gabriele Angeletti
categories: machine-learning
image: img/sparse-representation.png
---
Today we will take a look at [Sparse Distributed Representations][sparsedr]: what they are, why they differ from the standard types of representations used by computers, and why many people think they are fundamental in building true machine intelligence.

Computers manipulate information through so called **dense** representations: basically, they use a small number of bits and every combination of them represent something. For example, think about [ASCII][ascii] code. It uses a fixed number of bits, and any combination of those bits represents something, for instance '110 0111' is 'g'.

There are two things that make us thing this kind of representations are not suitable for machine learning applications.
First, the individual bits has no meaning at all. They become meaningful only when they are put together. For example, pick the code corresponding to 'g' in ASCII. Someone has chosen that combination of bits to represent a 'g', but the fact that the third bit is 0 says anything about the concept of 'g'.
Second, this kind of representations is not fault tolerant: if you change only one bit, you obtain a completely different meaning.

Neuroscientists have discovered that this is not how the human brain works. Instead, the brain uses a Sparse Distributed Representation of information.

You can think of a binary string as a set of neurons, where each neuron can be active (1) or not active (0). These representations are **sparse** in the sense that at any given time, only a tiny fraction of the neurons are active, so you always have a very small percentage of 1s. They are also **distributed** in the sense that they are a many-to-many mapping between concepts and neurons: each concept is represented by many neurons, and each neuron participates in the representation of many concepts. Here individual neurons have semantic meaning.

For example, if you want to represent the concept of a car with a distributed representation, you can use a binary vector of length 1000, where each element represent the presence or absence of the feature it represent. In this sense neurons have semantic meaning: if the neuron in position 49 is active, you know that that car is red, regardless of all the other neurons.

In ASCII code, the fact that the second bit is 1 is almost irrelevant. Instead, in a distributed representation, this is meaningful, in fact you can say that two sequences are semantically similar if they share some indexes of active bits, the more they share the more they are similar. So you can even sample a subset of the bits and compare only those samples to infer similarity (this is one of the advantages of a distributed representation). They have another advantage: because the representation is sparse, you can store only the indices of the active bits, and because there a few active bits at a given time, you can save a lot of space!

Sparse distributed representations are used by many algorithms: they are at the core of a new
brach of Artificial Intelligence algorithms, [Deep Learning][deeplearning], that is having huge success in the field, beating previous state-of-the-art algorithms in almost all branches, from image to audio to text.
There are many kind of Deep Learning Neural Networks, but all of them are based on the concept of
unsupervised pretraining: you train one layer at a time in an unsupervised fashion, and the output of one layer is used as training set for the next layer in the hierarchy. All these layers learn to detect **features** of the training data, the higher the layer in the hierarchy, the more abstract and meaningful the features are expected to be. For instance, if we are recognizing faces in images, in the first layer one would expect to see edges, strokes, blobs, while in the second layer one would expect to see something like an eye, a mouth, and so no.
All these layers are indeed learning Sparse Distributed Representations of the training set, and this is way this is useful to know and study this topic in order to grasp the concepts behind Deep Learning.

[sparsedr]: https://github.com/numenta/nupic/wiki/Sparse-Distributed-Representations
[ascii]: https://en.wikipedia.org/wiki/ASCII
[deeplearning]: https://en.wikipedia.org/wiki/Deep_learning
