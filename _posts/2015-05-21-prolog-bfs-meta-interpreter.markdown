{% extends "base-post.html" %}
{% set post_title= post.head %}
{% set post_head= post.head %}
{% set post_path= post.link %}

{% block post_content %}
	<p class="post-content">
		Today we will build a meta-interpreter for Prolog that uses a Breadth-First search strategy. The Prolog built-in interpreter uses DFS by default to build the tree of computations. This is due to performance reasons, because other searching policies wouldn't be feasible, except for fairly easy problems. But a cool fact about Prolog is that code and data are basically the same thing, and because of that its easy to implement a Prolog program that runs other Prolog programs (i.e. a meta-interpreter).
		We can build meta-interpreters to change the behavior of our interpreter: we can for example change the search policy, do the occur check, return the depth of the computations tree, and many other things.
	</p>
	<p class="post-content">
		In this post we will change Prolog's search strategy from DFS to BFS, as an example of how we can easily change its behavior.
		The main difference between DFS and BFS is that DFS uses a Stack to mantain the list of nodes to visit, and it means that DFS always tries to expand the last node it has visited. This policy can turn into trouble if there is a
		left recursion, which is when a variable that appears in the head of a rule also appears in the first goal of the tail. Since Prolog doesn't do the occur check, which is needed exactly to avoid such situations, it loops forever, until it goes out of local stack.
		<br>
		With Breadth-First Search such situations can be avoided, because BFS will expand all the goals in a clause, instead of keep trying to expand the first node it encounter.
		This is due do the fact that BFS uses a Queue to mantain the list of nodes to visit, so it when it expands a node, it won't visit what it has just expanded, but it will continue to visit the other nodes it hasn't expanded yet.
		<br>
		So at each depth level, BFS will first visit all the nodes at that level, and then move to the next level. While this is bad for performance, because we don't want any useless path to be visited, it can always find a solution if it exists, thus BFS is sound and complete. Let's now implement the meta-interpreter.
	</p>
	<h4>Code</h4>
	<p class="post-content">
		We will use Prolog's built-in predicate clause/2.
		<br>
		clause(Goal, Body) unifies Body with the bodies that matches with Goal, and return them in the form (FirstGoal, OtherGoals), where OtherGoals can be another list of the form (FirstGoal, OtherGoals). So clause/2 will find for us all the matches with Goal. Now we need a predicate that converts goal lists of this form into a standard prolog list with square brackets. To do this we will implement the predicate conj_goals_2_list.
		Now we are ready to implement the meta-interpreter. Basically BFS is the same as DFS except that it uses a Queue instead of a Stack. The Queue works with a FIFO (first-in first-out) policy, so we always insert element at the last position of the list, and we always retrieve elements at the first position.
		In Prolog we can do this pretty easily by using the way it handles lists and append/3.
		Using the predicate append(OldList, Element, NewList), NewList will be the result of adding Element at the end of OldList.
		So the code looks like this:
		<br>
		<pre class="code-block"><code>solveBFS([Goal|Rest]) :- % list of goals to prove</code>
			<br><code>clause(Goal, Body) % we use clause/2 on the first element of the goals list, that is the first-out part of FIFO</code>
			<br><code>conj_goals_2_list(Body, List) % we convert the list format returned by clause into a standart Prolog list</code>
			<br><code>append(Rest, List, New) % we append List to the end of Rest, that is the part first-in of FIFO</code>
			<br><code>solveBFS(New) % we call recursively the solver passing it the new list</code>
			<br><code>the program will keep trying to prove the goals, until all the goals are proved, or they can't be proved</code>
		</pre>
		<br>
	</p>
	<p class="post-content">
		The full code is available at <a target="_blank" href="http://github.com/blackecho/prolog-programs">github</a>.
	</p>
{% endblock %}
