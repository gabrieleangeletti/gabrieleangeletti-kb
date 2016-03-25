---
layout: post
title:  "Understanding Python's underlying data structures"
titlecol: "#000000"
date:   2016-03-23 13:30:20 -0500
author: Gabriele Angeletti
categories: programming
image: img/python.png
---
Let's take a look at how Python's typical data structures (lists, tuples, dicts, sets) are implemented, and how we can take advantage of this knowledge to optimize our code. The implementation we are focusing on is CPython.

CPython is the [original implementation][cpy] of Python, written in C. Since Python is a high-level language, the code you execute has to be evaluated and interpreted somehow. CPython is the program that translates Python instructions, written following the language specification, to something the machine can actually execute. Specifically, it compiles the code into bytecode, which then get interpreted and executed.
There are other implementations of Python "the language", like [Jython][jpy] written in Java and [Iron Python][ipy] written in C#.
The high-level Python code you wrote is the same (a list of integers is `l = [1, 2, 3]` in all of them), it is the underlying implementation that changes: for example CPython creates a C array, while Jython creates a Java ArrayList.

Knowing how the language data structures are actually implemented can give you a deeper understanding of what's going on, and can allow you to optimize your code.
Now we'll see the implementation of lists, tuples, dictionaries and sets.

### Lists
Python's lists are implemented as variable-length arrays of `PyObject`s. The name is a bit confusing because it resembles Linked Lists, but they are actually arrays.

The implementation uses a contiguous array of references to other objects, and keeps a pointer to this array and the array's length in a list head structure.
This makes indexing a list an operation whose cost is independent of the size of the list or the value of the index.
When items are appended or inserted, the array of references is resized. Some cleverness is applied to improve the performance of appending items repeatedly; when the array must be grown, some extra space is allocated so the next few times don’t require an actual resize (otherwise, every append/insert would require a resize of the array resulting in very poor performances).
You can see this over-allocation at line 42 of the [CPython source code][listsc] of the list object. The over-allocation is enough to give linear time behavior over a long list of appends. The growth pattern is: 0, 4, 8, 16, 25, 46, 58, 72, 88, ...
Thus, if you perform a million consecutive appends, the resizing mechanism accounts for this and doesn't resize a lot of times.

Python lists are optimized for fast fixed-length operations and incur O(n) memory movement costs for `pop(0)` and `insert(0, v)` operations which change both the size and, more important, the position of the underlying data representation.

The following two little experiments show some optimizations we can do:

    def f(n):
        l = []
        for i in range(n):
            l.append(i)
        return l

    def g(n):
        l = []
        for i in range(n):
            l.insert(i, i)
        return l

The two functions has the same effect, they return a list of natural numbers from 0 to n. The function f is slightly faster, as you can see from this plot (the execution time was measured using the handy ipython module `%timeit`):

<img src="../../../../../img/functionsfg.png" alt="plot of python functions execution time">

The following experiment shows the (huge) O(n) cost for resizing of the `insert(0, v)` operation. If we modify the functions above to return the list of natural numbers in inverted order, we can see that it is much (very much) faster to append the elements and then `reverse()` the resulting list rather than doing `insert(0, v)` operations.

    def f(n):
        l = []
        for i in range(n):
            l.append(i)
        l.reverse()
        return l

    def g(n):
        l = []
        for i in range(n):
            l.insert(0, i)
        return l

The difference between the two is so big that instead of plotting the results I'll show you the numbers (unit is the microsecond):

    input:      [100,  1000, 1000, 50000, 100000, 200000]
    function f: [9.43, 89.1, 849, 4430, 9300, 18900]
    function g: [24.3, 463, 29300, 785000, 3240000, 13000000]

### Tuples
The main difference between lists and tuples is that tuples are immutable objects: that is once they are created, they cannot be modified anymore (of course the tuple cannot be modified, but the variable pointing to it can).
Because of this immutability property, tuples are useful for representing what other languages often call records — some related information that belongs together, like a person's information. A tuple lets us “chunk” together related information and use it as a single thing.

Python's tuple implementation is basically the same as list but with optimizations that exploit the particular features of tuples, like fixed size. A very clever (in my opinion) optimization is one that is employed with allocation of small tuples.
Taken from [here][stackovtuple]:
<blockquote>
To avoid allocating a bunch of small objects, CPython recycles memory for many small tuples. There is a fixed constant (PyTuple_MAXSAVESIZE) such that all tuples less than this length are eligible to have their space reclaimed. Whenever an object of length less than this constant is deallocated, there is a chance that the memory associated with it will not be freed and instead will be stored in a "free list" (more on that in the next paragraph) based on its size. That way, if you ever need to allocate a tuple of size n and one has previously been allocated and is no longer in use, CPython can just recycle the old array.
</blockquote>

### Dictionaries
Python dictionaries are implemented using hash tables. A hash table is an array whose indexes are obtained using a hash function on the keys. The key can be a number, a string or an object, the important thing is that it must be an immutable object (i.e. we can define a hash function on it). The hash function is a function `h : K -> N` which takes in input a key `k` and return an integer `h(k)` representing the array index of the value corresponding to the key.
The goal of a hash function is to distribute the keys evenly in the array. A good hash function minimizes the number of collisions e.g. different keys having the same hash.

When there is a collision, it must be resolved. A naive approach would be to have linked lists of values instead of values in the values array, but then the lookup operation would not be O(1) anymore. Instead, Python uses a quadratic probing sequence to find a free slot in the case of collisions.

So every time you want to verify if a dictionary contains a key, what the interpreter does is to hash the value of the key to obtain the index and then verify if array[index] is empty. We would want to minimize the number of such operations.

For example, a common use case of dictionaries is counting object, like word frequencies. In these cases, items are initialized once and modified many times. To implement this function we could write something like this:

    def f(words):
        wdict = {}
        for word in words:
            if word not in wdict:
                wdict[word] = 0
            wdict[word] += 1
        return wdict

The problem with the above code is that except for the first time a new word is seen, the if test will fail every time, wasting a lot of hash function computations. We can avoid this with the following code:

    def g(words):
        wdict = {}
        for word in words:
            try:
                wdict[word] += 1
            except KeyError:
                wdict[word] = 1

Note: it is important to use KeyError in the except clause. You want the dictionary to be modified only when this exception occurs.

If the list is small the two functions are absolutely interchangeable. When the list get reasonably large, the second approach gets usually faster. For example, with the list I tried I got `3.08s` execution time with the first approach and `2.66s` with the second one.

### Sets
The implementation of Python sets derive from the dictionary implementation. Indeed, there is some [evidence][markmail] that originally the implementation was mostly a copy-paste from the dict implementation.

Python sets are implemented as dictionaries with dummy values (i.e. the keys are the members of the set) and there are optimizations for cache locality which take advantage of this lack of values. Thus, sets membership is done in O(1) as in dictionaries. Note that in the worst case (i.e. all hash collisions) the membership-checking is O(n).

Because of this amazingly fast membership checking, sets are the obvious choice when you have a list of values and you don't care about the order of the items, but just whether an item belongs to the list or not. Of course this is true only if the set is already there. If you include the set creation time, then a list is usually faster, as the following experiment shows:

    def f(e):
        l = range(1000000)
        return e in l

    def g(e):
        s = set(range(1000000))
        return e in l

`f(645000)` took `30.7ms` while `g(645000)` took `142ms`. But if we measure only the membership-checking time, then things are different: the list took `13.9ms` while the set only `65.4ns`! So keep in mind that in order to take advantage of the set data structure you have to be in a situation when you create the set once and read it many times.

That’s it for now. I hope you enjoyed the article. Please write a comment if you have any feedback.

PS: for further reference, there's no better place than the CPython source code: [listobject source code][listsc], [dictobject][dictsc], [tupleobject][tuplesc], [setobject][setsc].

[cpy]: http://stackoverflow.com/questions/17130975/python-vs-cpython
[jpy]: http://www.jython.org/
[ipy]: http://ironpython.net/
[listsc]: https://hg.python.org/cpython/file/tip/Objects/listobject.c
[tuplesc]: https://hg.python.org/cpython/file/tip/Objects/tupleobject.c
[dictsc]: https://hg.python.org/cpython/file/tip/Objects/dictobject.c
[setsc]: https://hg.python.org/cpython/file/tip/Objects/setobject.c
[markmail]: http://markmail.org/message/ktzomp4uwrmnzao6
[stackovtuple]: http://stackoverflow.com/questions/14135542/how-is-tuple-implemented-in-cpython
