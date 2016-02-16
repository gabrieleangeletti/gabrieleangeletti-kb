---
layout: post
title:  "Sparse Distributed Representations in Machine Learning"
date:   2015-06-15 09:30:20 -0500
categories: "machine-learning"
---
Today we will take a look at [Sparse Distributed Representations][sparsedr]: what they are, why they differ from the standard types of representations used by computers, and why many people think they are fundamental in building true machine intelligence.

Computers use **dense** representations of information: they use a small number of bits and every combination of them represent something. For example, think about [ASCII][ascii], that uses a fixed number of bits, and any combination of those bits is something, for instance '110 0111' is 'g'.

The first thing to note here is that the individual bits has no meaning at all, they become meaningful only when they are together. In the 'g' code, the fact that the third bit is 0 does not say anything about the concept of 'g', its simply that someone has chosen that string of bits to represent a 'g'. Another thing is that this representation is not fault tolerant: if you change only one bit, you obtain a completely different meaning.

Neuroscientists have discovered that this is not how our brain works. Instead, the brain uses a Sparse Distributed Representation of information.

You can think of a binary string as a set of neurons, where each neuron can be active (1) or not active (0). These representations are **sparse** in the sense that at any given time, only a tiny fraction of the neurons are active, so you always have a very small percentage of 1s. They are also **distributed** in the sense that they are a many-to-many mapping between concepts and neurons: each concept is represented by many neurons, and each neuron participates in the representation of many concepts. Here individual neurons have semantic meaning.

For example, if you want to represent the concept of a car with a distributed representation, you can use a binary vector of length 1000, where each element represent the presence or absence of the feature it represent. In this sense neurons have semantic meaning: if the neuron in position 49 is active, you know that that car is red, regardless of all the other neurons.

In ASCII code, the fact that you have a value of 1 in index 2 is almost irrelevant. Instead, in a distributed representation, this is meaningful, in fact you can say that two sequences are semantically similar if they share some indexes of active bits, the more they share the more they are similar. So you can even sample a subset of the bits and compare only these samples to infer similarity (this is one of the advantages of a distributed representation). They have another advantage: because the representation is sparse, you can store only the indices of the active bits, and because there a few active bits at a given time, you can save a lot of space!

If you want to know more on this subject, and also on related topics, I suggest you to follow [Jeff Hawkins][jeffh], an impressive Neuroscientist in the field of Machine Intelligence. Among other things, he founded a company called [Numenta][numenta] that is doing many interest things. They developed [open source framework][nupic] written in Python that implements the algorithms they are working on. I think i will play around a bit with this framework, maybe I'll write a new post on Nupic, to show you how it works!

[sparsedr]: https://github.com/numenta/nupic/wiki/Sparse-Distributed-Representations
[ascii]: https://en.wikipedia.org/wiki/ASCII
[jeffh]: https://en.wikipedia.org/wiki/Jeff_Hawkins
[numenta]: http://numenta.org
[nupic]: https://github.com/numenta/nupic/wiki/Using-NuPIC

<div id="disqus_thread"></div>
<script>
/**
* RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
* LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
*/

var disqus_config = function () {
this.page.url = "http://www.gabrieleangeletti.com/2015/06/sparse-distributed-representations"; // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = "Sparse Distributed Representations in Machine Learning"; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};

(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');

s.src = '//gabrieleangeletti.disqus.com/embed.js';

s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
