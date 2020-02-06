# CS 76 Dartmouth CS 76/176, Winter 2019
Source: https://www.cs.dartmouth.edu/~devin/cs76/05_sudoku/sudoku.html

<h1 id="sudoku-and-propositional-logic">Sudoku and propositional logic</h1>
<p>Your goal for this assignment is to write a few solvers for propositional logic satisfiability problems. As an example, we’ll use sets of sentences derived from <a href="https://en.wikipedia.org/wiki/Sudoku">sudoku</a> logic puzzles.</p>
<p>Here are some <a href="provided.zip">provided data and code</a> to get you started. The most important provided files end with the extension <code>.cnf</code>. These are representations of the problems you will solve, in conjunctive normal form.</p>
<p>There are also Python files. <code>Sudoku.py</code> is the workhorse: it converts <code>.sud</code> data files to conjunctive normal form and can be used to display text representations of boards. <code>sudoku2cnf.py</code> is a utility that lets you quickly do the conversion from the command-line. <code>display.py</code> will display a <code>*.sol</code> solution file, described later.</p>
<p><code>solve_sudoku.py</code> solves and displays a <code>.cnf</code> sudoku problem. Well, it would, if there were a “SAT.py” file containing a boolean satisfiability solver. Your main job will be to write that solver.</p>
<p>You don’t have to use any of the provided code unless you want to. The provided Sudoku problems are already in <code>.cnf</code> format, and the display and loading code is quite trivial; you could write your own in a few minutes if you prefer to.</p>
<p>This assignment is newly written for you; I hope you enjoy it as much as I have. Bug reports and clarification questions are very welcome; Piazza is the easiest way to get them to me.</p>
<h2 id="cnf-file-format">cnf file format</h2>
<p>The format of the cnf files is based loosely on that described by Ivor Spence <a href="http://www.cs.qub.ac.uk/~I.Spence/SuDoku/SuDoku.html">here</a>. Each line of a .cnf file represents a or-clause in cnf. Thus, the line</p>
<pre><code>111 112 113 114 115 116 117 118 119</code></pre>
<p>indicates that at least one of the variables 111, 112, 113, etc, must have the value <code>True</code>. Negative signs indicate negation. The line</p>
<pre><code>-111 -112</code></pre>
<p>indicates that either 111 or 112 must have the value <code>False</code>. Since this is conjunctive normal form, every clause must be satisfied to solve the problem, but there are multiple variables in most clauses, and thus multiple ways to satisfy each clause.</p>
<p>The variable names correspond to locations and values on the sudoku board. 132 indicates that row 1, column 3 has a 2 in it. For a typical 9x9 sudoku board, there are therefore 729 variables.</p>
<h2 id="the-cnf-files">The cnf files</h2>
<p>In order of increasing difficulty, the files are:</p>
<ul>
<li><code>one_cell.cnf</code>: rules that ensure that the upper left cell has exactly one value between 1 and 9.</li>
<li><code>all_cells.cnf</code>: rules that ensure that all cells have exactly one value between 1 and 9.</li>
<li><code>rows.cnf</code>: rules that ensure that all cells have a value, and every row has nine unique values.</li>
<li><code>rows_and_cols.cnf</code>: rules that ensure that all cells have a value, every row has nine unique values, and every column has nine unique values.</li>
<li><code>rules.cnf</code>: adds block constraints, so each block has nine unique values. (The complete rules for an empty sudoku board.)</li>
<li><code>puzzle1.cnf</code>: Adds a few starting values to the game to make a puzzle.</li>
<li><code>puzzle2.cnf</code>: Some different starting values.</li>
<li><code>puzzle_bonus.cnf</code>: Several starting values. Difficult; solution welcome but not required.</li>
</ul>
<h2 id="gsat">GSAT</h2>
<p>The <a href="https://en.wikipedia.org/wiki/WalkSAT">GSAT algorithm</a> is described nicely on Wikipedia. It is quite simple.</p>
<ol style="list-style-type: decimal">
<li>Choose a random assignment (a model).</li>
<li>If the assignment satisfies all the clauses, stop.</li>
<li>Pick a number between 0 and 1. If the number is greater than some threshold h, choose a variable uniformly at random and flip it; go back to step 2.</li>
<li>Otherwise, for each variable, score how many clauses would be satisfied if the variable value were flipped.</li>
<li>Uniformly at random choose one of the variables with the highest score. (There may be many tied variables.) Flip that variable. Go back to step 2.</li>
</ol>
<p>Implement GSAT (see implementation notes below before starting). You should be able to solve the first few <code>.cnf</code> problems with your implementation. The real sudoko puzzles are probably too hard to solve with GSAT in a reasonable time frame, however.</p>
<h2 id="walksat">WalkSAT</h2>
<p>The scoring step in GSAT can be slow. If there are 729 variables, and a few thousand clauses, the obvious scoring method (and I haven’t clearly thought out a better one) loops more than a million times. And that only flips a single bit in the assignment. Ouch.</p>
<p>WalkSAT chooses a smaller number of candidate variables that might be flipped. For the current assignment, some clauses are unsatisfied. <!--Take the set union of the variables in those unsatisfied clauses.--> Choose one of these unsatisfied clauses uniformly at random. The resulting set will be your candidate set. Use these candidate variables when scoring. Implement WalkSAT.</p>
<p>I found that WalkSAT needed very many iterations to solve <code>puzzle1.cnf</code> and <code>puzzle2.cnf</code>. I limited to 100,000 iterations, and chose .3 as my magic threshold value for a random move in step 3.</p>
<p>There are many variations on WalkSAT. Some variations do a GSAT step occasionally to try to escape local minima. You should implement the simple version described above as a first step, although exploration of variants is always welcome as an extension.</p>
<h2 id="implementation-notes">Implementation notes</h2>
<p>The variable names for Sudoku are things like <code>111</code>, <code>234</code>, etc. But your SAT-solvers should be generic – they should accept any CNF problem, not just Sudoku problems. So you might expect variable names like <code>VICTORIA_BLUE</code> for a map coloring problem, or <code>MINE_32</code> for minesweeper. Thus, your code should probably refer to variables by numerical indices during calculation. Perhaps 111 is variable 1, and 112 is variable 2, etc. Then your assignment can be represented using a simple list. Clauses can also be converted into lists, or perhaps sets, of integers.</p>
<h3 id="representing-variables-and-clauses-in-code">Representing variables and clauses in code</h3>
<p>In a clause, I used a negative integer to represent a negated atomic variable. So the clause <code>-1, -2</code> might represent <code>-111 or -112</code>. In my first implementation, this clever trick got me in trouble, because I decided that 111 was variable 0. Unfortunately, since -0 is the same as +0, some of the clauses involving the 0th variable got mangled – a tricky bug to find. So don’t 0-index your variables! This also has implications for assignments. If the assignment is a list, then index 0 might refer to the value of variable 1. So be careful with your indexing, too.</p>
<h3 id="output-.sol-files">Output .sol files</h3>
<p>It’s wise to output a solution file when you run the solver, since the solver might take several minutes for a hard problem. I output files with ‘.sol’ extensions that listed the name of every variable in the assignment, with either no sign (for a true value), or a negative sign (for a false value). I put each variable name on its own line in the file.</p>
<p>Notice that this is perhaps redundant, since knowing which variables are true tells you that the others are false. I did it anyway, since that way I get a complete list of the variable names, and disk space is cheap.</p>
<h2 id="extension-ideas">Extension ideas</h2>
<p>As usual, any scientifically interesting extension related to the theme of the assignment is welcome. Here are a few ideas to get you started, but you need not limit yourself to these ideas.</p>
<ol style="list-style-type: decimal">
<li><p>Resolution. Implement a resolution solver and prove some things. For example, you might set up a small sudoku problem (just the upper left block, perhaps), and initialize it with some values. Then try to prove something about some of the other values. I believe that most humans do this when trying to solve Sudoku. I know I use a combination of proofs of this form, and random walks.</p></li>
<li><p>Solve some other satisfiability problems. Map-coloring is obvious and easy. (And also not worth many points. You get what you pay for!) Can you find others?</p></li>
<li><p>Solve some interesting resolution problem(s). Perhaps minesweeper. In your report, describe the problem set-up. You might wish to constrain the problem to a small piece of the board.</p></li>
<li><p>Implement a deterministic solver for satisfiability, such as DPLL. Compare to WalkSAT.</p></li>
<li><p>Implement another random walk algorithm (perhaps one more sophisticated than walksat). Compare.</p></li>
<li><p>WalkSAT violates constraints at every step, <em>including known cell values</em>. Is the fact that the solver ignores the fact that some variables have known values a good thing? A bad thing? Make arguments for both, and do some experimental work by modifying the cnf to eliminate the known variables, and then running walksat. (That is, if 342 is known to be true, it is a constant, not a variable, and might be treated as such.)</p></li>
<li><p>Ivor Spence’s <a href="http://www.cs.qub.ac.uk/~I.Spence/SuDoku/SuDoku.html">formulation</a> adds several redundant constraints, as discussed on the linked page. Ivor argues that these constraints speed up the solver. My formulations do not include these constraints. Is your solver also faster if you add these constraints? Try it and report your results.</p></li>
</ol>
<h2 id="rubric">Rubric</h2>
<p>Up to 5 points for each of the following:</p>
<ul>
<li>WalkSAT: Correctness</li>
<li>GSAT: Correctness</li>
<li>General code quality</li>
<li>Report</li>
<li>Extensions</li>
</ul>

  <!--p &nbsp-->
  <!-- make table of contents or navlinks-->
  <!--script( src="../js/djbbook.js")-->
  <!-- pretty-print-->
  <!--script(src="http://cdnjs.cloudflare.com/ajax/libs/prettify/r298/run_prettify.js")-->
  <!-- Devin's jside library for displaying and running code-->
  <!--script( src="../brython3.2.5/brython.js") -->
  <!--script("http://brython.info/src/brython.js")-->
  <!--script( src="https://cdn.rawgit.com/brython-dev/brython/3.2.2/www/src/brython.js")-->
  <!--script( src="../processing-1.4.8.min.js")-->
  <!--script(src="../jside-0.0.6/jside.min.js")    -->
  <!--script jside();-->
  
</body>
