---
layout: post
title:  "Visualizing Twitter data: a Python example"
titlecol: "#000000"
date:   2016-03-12 15:30:20 -0500
author: Gabriele Angeletti
categories: data-science
image: img/twitter-logo.png
---
Data visualization is used to synthesize data, explain concepts and communicate results and insights in a clear and efficient way. And most important, it's fun!
In this post we'll see an example of data visualization using Twitter API.

This post is realized in collaboration with a friend of mine, and university collegue
[Alessandro Stagni][ales].
We used [IPython Notebooks][ipynb] to present the work. Notebooks are a great way to show
Python work: text, code, output and even images can be mixed in one interactive document
and then converted to html. Some people are even [writing books][ipynbkalman] using ipython notebooks!

Data is collected from Twitter using the [streaming API][strAPI].
We collected 250 thousand tweets for each of the following languages: english, french, german, spanish, portuguese.

We show visualizations of length distribution for each language, and WordClouds for
the most frequent words.

WordClouds are generated using [this][wc] great package.

This is the [link to the notebook][work], hope you enjoy it!

[strAPI]: https://dev.twitter.com/streaming/overview
[wc]: https://github.com/amueller/word_cloud
[ales]: https://it-it.facebook.com/people/Alessandro-Stagni/100007421145017
[ipynb]: http://ipython.org/notebook.html
[ipynbkalman]: https://github.com/rlabbe/Kalman-and-Bayesian-Filters-in-Python
[work]: http://www.gabrieleangeletti.com/tweet_data_viz.html
