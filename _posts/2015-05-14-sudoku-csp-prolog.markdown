{% extends "base-post.html" %}
{% set post_title= post.head %}
{% set post_head= post.head %}
{% set post_path= post.link %}

{% block post_content %}
	<p class="post-content">
		Today we will model the game of <a target="_blank" href="http://en.wikipedia.org/wiki/Sudoku">Sudoku</a> as a <a target="_blank" href="http://en.wikipedia.org/wiki/Constraint_satisfaction_problem">Constraint Satisfaction Problem (CSP)</a> and then write a Prolog program that solves it.
		CSP is particularly useful when we want to model a combinatorial problem subject to <strong>constraints</strong>. In standard Sudoku we have a 9x9 grid, where each cell can be filled with an integer between 1 and 9, so the possible configurations are all the ordered combinations of 81 numbers between 1 and 9.
	</p>
	<p class="post-content">
		In the first cell, we have 9 different values that we can put in.
		<br>
		If we consider both the first and the second cells, we have \( 9^{2} \) different values that we can put in, because we have to take into account all the possible combination of
		the form { (1,1), (1,2) ... (1,9), (2,1) ... (9,9) }. So if we consider all the cells in the end we have \(9^{81}\) possible combination of values, a very huge number!!
	</p>
	<p class="post-content">
		But, two things help us here.
		<br>
		First, Sudoku is not about trying some random combination, it has some rules, and those rules greatly reduce the above number, because they tell us which combinations are right, and which
		are wrong.
		<br>
		Second, sudoku has some cells that are already filled, and these <strong>initial conditions</strong> also help in reducing the huge number of tries.
		So let's see how we can model this game in csp.
	</p>
	<p class="post-content">
		CSP has variables (what we are interested in), domains (which values the variables can assume) and constraints (rules about what values are right and what are wrong).
		The variables in this case are the 81 cells of the grid, each of which can assume a value between 1 and 9, so the domain here is the integer interval [1, 9].
		The constraints simply are the rules of the game, that are:
	</p>
		<ul class="post-content">
			<li>All the values in each row must be different each other</li>
			<li>All the values in each column must be different each other</li>
			<li>All the values in each 3x3 block must be different each other</li>
		</ul>
	<p class="post-content">
	Done! We don't need some fancy algorithm or complex trick to get the job done: we are simply describing the problem here, and we let Prolog's backtracking solve it for us. This is the true power of Prolog and logic
	Programming: you don't actually have to solve the problem, you only have to model it!
	</p>
	<h4>Code</h4>
	<p class="post-content">
		Prolog has a csp library, named <strong>clpfd</strong>. It has all the things that we need: we can define variables, domains for the variables, and costraints. More over, the constraints that the values must be different, is a type of constraint that is already implemented in clpfd. All we have to do is to use all_different(List).
	</p>
	<p class="post-content">
		The full code is available at <a target="_blank" href="http://github.com/blackecho/prolog-programs">github</a>.
	</p>
{% endblock %}
