---
layout: post
title: Deep Autoencoder Locality Sensitive Hashing
card-title-color: card-title-orange-700
date: 2015-09-10 9:30:20 -0500
permalink: /projects/deep-locality-sensitive-hashing
categories: project
image: ../img/tsne-viz.png
---
In this project I use a stack of denoising autoencoders to learn low-dimensional
representations of images.

These encodings are used as input to a locality sensitive
hashing algorithm to find images similar to a given query image. The results clearly
shows that this approach outperforms basic LSH by far.
[Here you can find the presentation of this project](http://www.slideshare.net/GabrieleAngeletti/project-deep-locality-sensitive-hashing).
