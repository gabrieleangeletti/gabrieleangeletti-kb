---
layout: post
title:  "Refactoring a Restricted Boltzmann Machine implementation in TensorFlow."
date:   2016-02-21 9:30:20 -0500
author: Gabriele Angeletti
image: img/refactor.png
categories: programming
highlight: true
---
I'm currently reading the excellent book [Clean Code][ccode], a book about writing code in a professional way, and I applied the lessons learned in this book to my implementation of a [Restricted Boltzmann Machine][rbm] using [TensorFlow][tf].

TensorFlow is an open source library developed at Google for distributed computation of machine learning algorithms. I'll write another post about it, for now let's focus only on refactoring the code. You don't have to be a TensorFlow expert in order to understand the refactoring, it's just about modifying names, removing comments and rearranging functions.

The old code (before the refactoring) and the new code (after the refactoring) can be found at this [github gist][gist].

Clean code is an amazing book full of lessons about writing **good** code. Indeed I learned a lot of lessons since I realized my coding style was "not so good" after I saw how clean code can be **elegant** and pleasant to the eyes. Since I haven't finished the book yet, I want about the three things I learned the most about: naming things, functions and comments.

### Naming

This quote from the book can't explain better the concept:
<blockquote>
Names should reveal intent. A name should tell you why the variable, class, etc.. exits, what it does, and how it is used. If a name requires a comment, then the name does not reveal its intent.
</blockquote>
For example, you can see in the old code that the instance variables corresponding to the number of visible/hidden units in the rbm are called respectively `nvis/nhid`.
While the meaning of these variables can be deducted from the context of the class and their usage, the name isn't much self-explaining, probably a comment would be needed to make the reader instantly understand the meaning of them. Instead, `num_visible/num_hidden` are names that can't be misunderstood. As you read them, you immediately understand what they are, why they are there, and how they are used.

### Functions

Functions should do one thing. A quote from the book:
<blockquote>
The first rule of functions is that they should be small. The second rule of functions is that they should be smaller than that.
</blockquote>
This is the single most valuable thing I learned from this book. Clean code is mainly about **readability**, **maintainability** and **elegance**. And I think that writing functions that do one thing is one of the most effective ways to achieve cleanness.

For example, look at the `_create_graph` method (`_build_model` in the new code). This function is way too long and it does a lot of different things like creating placeholders, variables, performing gibbs sampling, computing positive and negative associations, compute the update rules and the loss function. This is really a good example of a function that does way more than one thing!
In the new code, this single function is replaced by seven new functions, one for each thing the old function did. The resulting code is much cleaner.

Another thing about functions is the order in which they should be placed in the source file.
In particural, the caller should be above the callee. The author of the book explains this rule in terms of the *newspaper* metaphor: the source file should look like a newspaper article. At the top there is the title, and a short outline of what its told in the article. Then it begins explaining the broad concepts of the story, and then below are all the details of who says what, when things happened etc..
<blockquote>
Master Programmers think of systems as stories to be told rather than programs to be written. With this goal in mind, functions need to fit cleanly together into a clear and precise language to help you with that telling.
</blockquote>
In the new code, the `_build_model` function is splitted in seven new functions in a top-down fashion. To build a model, the first thing you have to do is to create placeholders (TensorFlow structure for the data to be passed to the algorithm). And indeed, `_create_placeholders` is the first function below `_build_model`.

Then, the second thing you have to do is to create variables (TensorFlow structure to store the parameters of the model), in this case the weights and biases of the Restricted Boltzmann Machine. Again, you can see that `_create_variables` is below `_create_placeholders`, and so on.

The code is much cleaner. At first you are presented with the `_build_model` function, which tells you the broad concept of constructing a RBM model using TensorFlow. As you scroll down more details are revealed to you: how do I create a placeholder? A variable? How is gibbs sampling implemented? As you dive deeper in the story, the details are revealed to you.

### Comments

By reading this book I realized that my idea about comments was completely wrong. I've always thought that comments are always a good thing, and its always better to put a comment than to put nothing. Why should comments be a bad thing? They help us expressing ourselves better!

And this is the point made by the author: **we use comments because we fail to express ourselves in code**.
Bad comments are far worse than no comments at all! Bad comments can even lie about the code. Often also good comments can lie about the code. This is because is very difficult for programmers to maintain them in a consistent state as the code evolves. Code changes very often, and if comments do not change consistently with the code, we can find a comment in another place than it was supposed to be, or a good comment that now is wrong because of a change in the code, and so on.

Comments should be kept at a minimum, and should be used only when we can't find a way to express the concept in code. When you are in a position in which you have to write a comment, think about it first and see if you can came up with a way to express yourself in code rather than with that comment.
In the old code you can find a lot of useless comments, like:

    # Randomly shuffle the input
    np.random.shuffle(trX)

You already know (or at least imagine) what the [numpy][np] shuffle function does. This comment is completely redundant. As you can tell, there are a lot of examples like this in the old code.

This was just a small example of refactoring and writing clean code. I'll write more about the topic and my experiments/lessons learned. I encourage you to take a look at the [code][gist], and please comment below if you have any suggestion/ideas on how to improve it.

[ccode]: http://www.amazon.com/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882
[rbm]: https://en.wikipedia.org/wiki/Restricted_Boltzmann_machine
[tf]: https://www.tensorflow.org/
[np]: http://www.numpy.org/
[gist]: https://gist.github.com/blackecho/db85fab069bd2d6fb3e7
