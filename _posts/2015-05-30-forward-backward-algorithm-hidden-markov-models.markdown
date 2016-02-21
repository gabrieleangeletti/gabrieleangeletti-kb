---
layout: post
title:  "The Forward-Backward Algorithm for Hidden Markov Models"
date:   2015-05-30 10:30:20 -0500
author: Gabriele Angeletti
categories: machine-learning
image: img/markov-teaser.png
---
Today we will see how we can do inference in a [Hidden Markov Model (HMM)][hidden-markov-model], using the [Forward-Backward (FB) algorithm][forward-backward]. First of all I will briefly recap what a HMM is, then I'll show you the algorithm in detail.

A HMM is a statistical model that can be used to do a lot of interesting things, it has many applications in speech-recognition, handwriting recognition, and many others.
The basic idea is that if we have some observations about something, they can be explained in terms of some states (which are hidden) to which these observations depend. If we take for example handwriting recognition, the hidden states would be the letters of the alphabet, and the observed variables would be the letters that one actually writes.

<img class="responsive-img" src="../../../../img/hmm.png" alt="hidden markov model graphic representation">

The arrows in the image indicate a conditional probability, so there are conditional probabilities between states and between
states and observations.
These two quantities themselves, together with the initial distributions for the states, entirely define an HMM.
So you have two matrices, the transition probability matrix $T$ (or transition model) and the observation probability matrix $O$
(or observation model).

$T$ contains the probabilities of changing state: if the model has N states, then the matrix will have dimensions NxN, in which the $T_{(i,j)}$ cell contains the probability that the next state will be j, given that the current state is i.

$O$ contains the probabilities of observing a particular value, given the current state: for example the $O_{(i,j)}$ cell
contains the probability of observing value i given that the current state is j.

When you have defined a HMM, you can compute the posterior probability distribution of the states given the observations, thus estimating the
correlations between the observed variables and the hidden states. In the handwriting example this means that we would be able to recognize the character
that one has written given the actual messy character that she/he wrote.

The FB algorithm allows you to compute these posterior probabilities for internal hidden states.

The basic idea is that this probability is proportional to the joint distribution $ P(Z_k,X_{1:t}) $ and, using [Bayes theorem][bayes] and the independence of $ X_{1:k} $ and $ X_{k+1:t} $ given $ Z_k $, we obtain:

$$ P(Z_k | X_{1:t}) \propto P(Z_k , X_{1:t}) = P(Z_k | X_{1:k}) P(X_{k+1:t} | Z_k) $$

This algorithm rely on the idea of [Dynamic Programming][dp], that is a fancy name for saying that we reuse things we have already computed to make things faster.

The forward part computes the posterior probability of state $Z_k$ given the first k observations $ X_{1:k}$, and does
this $\forall k = 1 ... n$.

The backward part computes the probability of the remaining observations $X_{k+1:n}$ given state $Z_k$, again for each k.

The last part of the algorithm is to merge the forward and the backward part to give the full posterior probabilities.

This algorithm gives you probabilities for all hidden states, but it doesn't give probabilities for sequences of states (see the [Viterbi Algorithm][viterbi])

## Forward Part

We want to use dynamic programming, so we want to express $P(Z_k | X_{1:k})$ as a function of k-1, so that we can setup a recursion. We achieve that by using [marginalization][marg].
So using marginalization we can bring $ Z_{k-1} $ to the equation:

$$ P(Z_k,X_{1:k}) = \sum_{Z_{k-1}} P(Z_k, Z_{k-1}, X_{1:k-1}, X_k) $$

Where the sum is over all the values which $ Z_{k-1} $ can assume. Now we can use the [chain rule for probability][chainrule]</a> and see what happens:

$$ \sum_{Z_{k-1}} P(Z_k, Z_{k-1}, X_{1:k-1}, X_k) = \sum_{Z_{k-1}} P(X_k | Z_k, Z_{k-1}, X_{1:k-1}) P(Z_k|Z_{k-1}, X_{1:k-1}) P(Z_{k-1} | X_{1:k-1}) P(X_{1:k-1}) $$

the last two terms can be unified in: $P(Z_{k-1}, X_{1:k-1})$. So now we use [D-separation][dsep] to simplify the above equation. In the first term we can remove $ Z_{k-1}$ and $ X_{1:k-1} $, because $ X_k $ is independent of them, given $ Z_k $. We can see it graphically using D-separation: the idea is that any path in the graph from $ X_k $ to $ Z_{k-1}$ or $ X_{1:k-1} $, must pass through $ Z_k $, so we say that they are independent given $ Z_k $ and we remove them.
The same goes for the second term, in which we remove $ X_{1:k-1} $. Let's now write the simplified equation:

$$ P(Z_k,X_{1:k}) = \sum_{Z_{k-1}} P(X_k | Z_k) P(Z_k|Z_{k-1}) P(Z_{k-1}, X_{1:k-1}) $$

Let's examine this equation. The first term is the emission probability, which we have assumed we know. The second term is the transition probability, which also we have assumed we know. The third term is the recursion term over k-1, so we compute this value $\forall k = 2 ... n$ and we are done.
For k = 1, the value is simple to compute:

$$ P(Z_1,X_1) = P(X_1 | Z_1) P(Z_1) $$

## Backward Part

For the backward part, we compute the values in a similar way, by using marginalization and the chain rule.

$$ P(X_{k+1, t}, Z_k) = \sum_{Z_{k+1}} P(X_{k+1:t}, Z_{k+1} | Z_k) = \sum_{Z_{k+1}} P(X_{k+1}, X_{k+2:t}, Z_{k+1} | Z_k) $$

Now we again use the chain rule and see what happens:

$$ \sum_{Z_{k+1}} = P(X_{k+2:t} | Z_{k+1}, X_{k+1}, Z_k) P(X_{k+1} | Z_{k+1}, Z_k) P(Z_{k+1}, Z_k) $$

Like in the forward part, we can use D-separation to simplify the equation. In the first term, we can remove $ X_{k+1}$ and $Z_k$, and in the second
term we can remove $Z_k$. So now the equation is:

$$ \sum_{Z_{k+1}} = P(X_{k+2:t} | Z_{k+1}) P(X_{k+1} | Z_{k+1}) P(Z_{k+1}, Z_k) $$

And again: we have an emission probability (second term), a transition probability (third term), and a recursion term over k+1 (first term).
We again compute this value $\forall k = 1 ... n-1$ and we are done. For k=n, the value is 1 (you can compute it yourself is pretty easy).

[hidden-markov-model]: https://en.wikipedia.org/wiki/Hidden_Markov_Model
[forward-backward]: https://en.wikipedia.org/wiki/Forward%E2%80%93backward_algorithm
[bayes]: http://en.wikipedia.org/wiki/Bayes'_theorem
[dp]: http://en.wikipedia.org/wiki/Dynamic_programming
[viterbi]: http://en.wikipedia.org/wiki/Viterbi_algorithm
[marg]: http://en.wikipedia.org/wiki/Marginal_distribution
[chainrule]: http://en.wikipedia.org/wiki/Chain_rule_(probability)
[dsep]: http://www.andrew.cmu.edu/user/scheines/tutor/d-sep.html
