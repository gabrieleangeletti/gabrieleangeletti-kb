---
layout: post
title:  "A Breadth-First Search (BFS) Prolog meta-interpreter"
date:   2015-05-21 09:30:20 -0500
categories: "prolog"
---
Today we will build a meta-interpreter for Prolog that uses a Breadth-First search strategy. The Prolog built-in interpreter uses DFS by default to build the tree of computations. This is due to performance reasons, because other searching policies wouldn't be feasible, except for fairly easy problems. But a cool fact about Prolog is that code and data are basically the same thing, and because of that its easy to implement a Prolog program that runs other Prolog programs (i.e. a meta-interpreter).
We can build meta-interpreters to change the behavior of our interpreter: we can for example change the search policy, do the occur check, return the depth of the computations tree, and many other things.

In this post we will change Prolog's search strategy from DFS to BFS, as an example of how we can easily change its behavior.
The main difference between DFS and BFS is that DFS uses a Stack to mantain the list of nodes to visit, and it means that DFS always tries to expand the last node it has visited. This policy can turn into trouble if there is a
left recursion, which is when a variable that appears in the head of a rule also appears in the first goal of the tail. Since Prolog doesn't do the occur check, which is needed exactly to avoid such situations, it loops forever, until it goes out of local stack.

With Breadth-First Search such situations can be avoided, because BFS will expand all the goals in a clause, instead of keep trying to expand the first node it encounter.
This is due do the fact that BFS uses a Queue to mantain the list of nodes to visit, so it when it expands a node, it won't visit what it has just expanded, but it will continue to visit the other nodes it hasn't expanded yet.

So at each depth level, BFS will first visit all the nodes at that level, and then move to the next level. While this is bad for performance, because we don't want any useless path to be visited, it can always find a solution if it exists, thus BFS is sound and complete. Let's now implement the meta-interpreter.

## Code
We will use Prolog's built-in predicate clause/2.
clause(Goal, Body) unifies Body with the bodies that matches with Goal, and return them in the form (FirstGoal, OtherGoals), where OtherGoals can be another list of the form (FirstGoal, OtherGoals). So clause/2 will find for us all the matches with Goal. Now we need a predicate that converts goal lists of this form into a standard prolog list with square brackets. To do this we will implement the predicate conj_goals_2_list.
Now we are ready to implement the meta-interpreter. Basically BFS is the same as DFS except that it uses a Queue instead of a Stack. The Queue works with a FIFO (first-in first-out) policy, so we always insert element at the last position of the list, and we always retrieve elements at the first position.
In Prolog we can do this pretty easily by using the way it handles lists and append/3.
Using the predicate append(OldList, Element, NewList), NewList will be the result of adding Element at the end of OldList.
So the code looks like this:

    solveBFS([Goal|Rest]) :- % list of goals to prove

we use clause/2 on the first element of the goals list, that is the first-out part of FIFO

    clause(Goal, Body)

we convert the list format returned by clause into a standart Prolog list

    conj_goals_2_list(Body, List)

we append List to the end of Rest, that is the part first-in part of FIFO

    append(Rest, List, New)

we call recursively the solver passing it the new list. The program will keep trying to prove the goals, until all the goals are proved, or they can't be proved

    solveBFS(New)

The full code is available at [github][ghub].

[ghub]: http://github.com/blackecho/prolog-programs

<div id="disqus_thread"></div>
<script>
/**
* RECOMMENDED CONFIGURATION VARIABLES: EDIT AND UNCOMMENT THE SECTION BELOW TO INSERT DYNAMIC VALUES FROM YOUR PLATFORM OR CMS.
* LEARN WHY DEFINING THESE VARIABLES IS IMPORTANT: https://disqus.com/admin/universalcode/#configuration-variables
*/

var disqus_config = function () {
this.page.url = "www.gabrieleangeletti.com/2015/05/prolog-bfs-meta-interpreter"; // Replace PAGE_URL with your page's canonical URL variable
this.page.identifier = "A Breadth-First Search (BFS) Prolog meta-interpreter"; // Replace PAGE_IDENTIFIER with your page's unique identifier variable
};

(function() { // DON'T EDIT BELOW THIS LINE
var d = document, s = d.createElement('script');

s.src = '//gabrieleangeletti.disqus.com/embed.js';

s.setAttribute('data-timestamp', +new Date());
(d.head || d.body).appendChild(s);
})();
</script>
<noscript>Please enable JavaScript to view the <a href="https://disqus.com/?ref_noscript" rel="nofollow">comments powered by Disqus.</a></noscript>
