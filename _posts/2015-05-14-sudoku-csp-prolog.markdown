---
layout: post
title:  "Solving Sudoku as a Constraint Satisfaction Problem (CSP) using Prolog"
date:   2015-05-14 14:30:20 -0500
categories: "prolog"
---
Today we will model the game of [Sudoku][sudoku] as a [Constraint Satisfaction Problem (CSP)][csp] and then write a Prolog program that solves it.
CSP is particularly useful when we want to model a combinatorial problem subject to **constraints**. In standard Sudoku we have a 9x9 grid, where each cell can be filled with an integer between 1 and 9, so the possible configurations are all the ordered combinations of 81 numbers between 1 and 9.

<img src="../../img/sudoku.png" width="50%" style="display: block;margin: 0 auto;clear right;">

In the first cell, we have 9 different values that we can put in.
If we consider both the first and the second cells, we have $ 9^{2} $ different values that we can put in, because we have to take into account all the possible combination of
the form $ (1,1), (1,2) ... (1,9), (2,1) ... (9,9) $. So if we consider all the cells in the end we have $ 9^{81} $ possible combination of values, a very huge number!!

But, two things help us here:

1. Sudoku is not about trying some random combination, it has some rules, and those rules greatly reduce the above number, because they tell us which combinations are right, and which are wrong.
2. Sudoku has some cells that are already filled, and these **initial conditions** also help in reducing the huge number of tries.

So let's see how we can model this game in csp.

CSP has variables (what we are interested in), domains (which values the variables can assume) and constraints (rules about what values are right and what are wrong).
The variables in this case are the 81 cells of the grid, each of which can assume a value between 1 and 9, so the domain here is the integer interval [1, 9].
The constraints simply are the rules of the game, that are:

* All the values in each row must be different each other
* All the values in each column must be different each other
* All the values in each 3x3 block must be different each other

Done! We don't need some fancy algorithm or complex trick to get the job done: we are simply describing the problem here, and we let Prolog's backtracking solve it for us. This is the true power of Prolog and logic
Programming: you don't actually have to solve the problem, you only have to model it!

## Code
Prolog has a csp library, named <strong>clpfd</strong>. It has all the things that we need: we can define variables, domains for the variables, and costraints. More over, the constraints that the values must be different, is a type of constraint that is already implemented in clpfd. All we have to do is to use all_different(List).

The full code is available at [github][ghub].

[sudoku]: http://en.wikipedia.org/wiki/Sudoku
[csp]: http://en.wikipedia.org/wiki/Constraint_satisfaction_problem
[ghub]: http://github.com/blackecho/prolog-programs

<div id="disqus_thread"></div>
<script>
/**
* RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
* LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
*/

var disqus_config = function () {
this.page.url = "www.gabrieleangeletti.com/2015/05/sudoku-csp-prolog"; // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = "Solving Sudoku as a Constraint Satisfaction Problem (CSP) using Prolog"; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};

(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');

s.src = '//gabrieleangeletti.disqus.com/embed.js';

s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
