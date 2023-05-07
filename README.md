Download Link: https://assignmentchef.com/product/solved-multi-agent-pac-man-cs221
<br>
<blockquote>




 <center>

  <img decoding="async" width="359" height="197" data-src="pacman_multi_agent.png" class="lazyload" src="data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==">

  <noscript>

   <img decoding="async" src="pacman_multi_agent.png" width="359" height="197">

  </noscript>

 </center>

 <center>

  Pac-Man, now with ghosts.Minimax, Expectimax.

 </center>




</blockquote>

<h3>Introduction</h3>

For those of you not familiar with Pac-Man, it’s a game where Pac-Man (the yellow circle with a mouth in the above figure) moves around in a maze and tries to eat as many <i>food pellets</i> (the small white dots) as possible, while avoiding the ghosts (the other two agents with eyes in the above figure). If Pac-Man eats all the food in a maze, it wins. The big white dots at the top-left and bottom-right corner are <i>capsules</i>, which give Pac-Man power to eat ghosts in a limited time window (but you won’t be worrying about them for the required part of the assignment). You can get familiar with the setting by playing a few games of classic Pac-Man, which we come to just after this introduction.

In this project, you will design agents for the classic version of Pac-Man, including ghosts. Along the way, you will implement both minimax and expectimax search.

The base code for this project contains a lot of files (which are listed towards the end of this page); you, however, <b>do not</b> need to go through these files to complete the assignment. These are there only to guide the more adventurous amongst you to the heart of Pac-Man. As in previous assignments, you will be modifying only <code><a href="submission.py">submission.py</a></code>.

A basic <code><a href="grader.py">grader.py</a></code> has been included in the .zip file, but it only checks for timing issues, not functionality. You can check that Pac-Man behave as described in the problems, and run <code><a href="grader.py">grader.py</a></code> without timeout in order to test your implementation.

<h3>Warmup</h3>

First, play a game of classic Pac-Man to get a feel for the assignment:

<pre>python pacman.py</pre>

You can always add <code>--frameTime 1</code> to the command line to run in “demo mode” where the game pauses after every frame.

Now, run the provided <code>ReflexAgent</code> in <code><a href="submission.py">submission.py</a></code>:

<pre>python pacman.py -p ReflexAgent</pre>

Note that it plays quite poorly even on simple layouts:

<pre>python pacman.py -p ReflexAgent -l testClassic</pre>

You can also try out the reflex agent on the default <code>mediumClassic</code> layout with one ghost or two.

<pre>python pacman.py -p ReflexAgent -k 1</pre>

<pre>python pacman.py -p ReflexAgent -k 2</pre>

<em>Note:</em> you can never have more ghosts than the <a href="layouts/mediumClassic.lay">layout</a> permits.

<em>Options:</em> Default ghosts are random; you can also play for fun with slightly smarter directional ghosts using <code>-g DirectionalGhost</code>. You can also play multiple games in a row with <code>-n</code>. Turn off graphics with <code>-q</code> to run lots of games quickly.

So, now that you are familiar enough with the interface, inspect the <code>ReflexAgent</code> code carefully (in <code><a href="submission.py">submission.py</a></code>) and make sure you understand what it’s doing. The reflex agent code provides some helpful examples of methods that query the <code>GameState</code> (a <code>GameState</code> specifies the full game state, including the food, capsules, agent configurations and score changes: see <code><a href="submission.py">submission.py</a></code> for further information and helper methods) for information, which you will be using in the actual coding part. We are giving an exhaustive and very detailed description below, for the sake of completeness and to save you from digging deeper into the starter code. The actual coding part is very small – so please be patient if you think there is too much writing.

Note: if you wish to run Pac-Man in the terminal using a text-based interface, check out the <code>terminal</code> directory.

<ol class="problem">

 <li id="1a" class="writeup">[5 points] Before you code up Pac-Man as a minimax agent, notice that instead of just one adversary, Pac-Man could have multiple ghosts as adversaries. So we will extend the minimax algorithm from class (which had only one min stage for a single adversary) to the more general case of multiple adversaries. In particular, <i>your minimax tree will have multiple min layers (one for each ghost) for every max layer</i>.Specifically, consider the limited depth tree minimax search with evaluation functions taught in class. Suppose there are <span id="MathJax-Element-1-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-1" class="math"><span id="MathJax-Span-2" class="mrow"><span id="MathJax-Span-3" class="mi">n</span><span id="MathJax-Span-4" class="mo">+</span><span id="MathJax-Span-5" class="mn">1</span></span></span></span> agents on the board, <span id="MathJax-Element-2-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-6" class="math"><span id="MathJax-Span-7" class="mrow"><span id="MathJax-Span-8" class="msubsup"><span id="MathJax-Span-9" class="mi">a</span><span id="MathJax-Span-10" class="mn">0</span></span><span id="MathJax-Span-11" class="mo">,</span><span id="MathJax-Span-12" class="mo">…</span><span id="MathJax-Span-13" class="mo">,</span><span id="MathJax-Span-14" class="msubsup"><span id="MathJax-Span-15" class="mi">a</span><span id="MathJax-Span-16" class="mi">n</span></span></span></span></span>, where <span id="MathJax-Element-3-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-17" class="math"><span id="MathJax-Span-18" class="mrow"><span id="MathJax-Span-19" class="msubsup"><span id="MathJax-Span-20" class="mi">a</span><span id="MathJax-Span-21" class="mn">0</span></span></span></span></span> is Pac-Man and the rest are ghosts. Pac-Man acts as a max agent, and the ghosts act as min agents. A single <i>depth</i> consists of all <span id="MathJax-Element-4-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-22" class="math"><span id="MathJax-Span-23" class="mrow"><span id="MathJax-Span-24" class="mi">n</span><span id="MathJax-Span-25" class="mo">+</span><span id="MathJax-Span-26" class="mn">1</span></span></span></span> agents making a move, so depth 2 search will involve Pac-Man and each ghost moving two times. In other words, a depth of 2 corresponds to a height of <span id="MathJax-Element-5-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-27" class="math"><span id="MathJax-Span-28" class="mrow"><span id="MathJax-Span-29" class="mn">2</span><span id="MathJax-Span-30" class="mo">(</span><span id="MathJax-Span-31" class="mi">n</span><span id="MathJax-Span-32" class="mo">+</span><span id="MathJax-Span-33" class="mn">1</span><span id="MathJax-Span-34" class="mo">)</span></span></span></span> in the minimax game tree.Write the recurrence for <span id="MathJax-Element-6-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-35" class="math"><span id="MathJax-Span-36" class="mrow"><span id="MathJax-Span-37" class="msubsup"><span id="MathJax-Span-38" class="mi">V</span><span id="MathJax-Span-39" class="texatom"><span id="MathJax-Span-40" class="mrow"><span id="MathJax-Span-41" class="mtext">max,min</span></span></span></span><span id="MathJax-Span-42" class="mo">(</span><span id="MathJax-Span-43" class="mi">s</span><span id="MathJax-Span-44" class="mo">,</span><span id="MathJax-Span-45" class="mi">d</span><span id="MathJax-Span-46" class="mo">)</span></span></span></span> in math. You should express your answer in terms of the following functions: <span id="MathJax-Element-7-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-47" class="math"><span id="MathJax-Span-48" class="mrow"><span id="MathJax-Span-49" class="mtext">IsEnd</span><span id="MathJax-Span-50" class="mo">(</span><span id="MathJax-Span-51" class="mi">s</span><span id="MathJax-Span-52" class="mo">)</span></span></span></span>, which tells you if <span id="MathJax-Element-8-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-53" class="math"><span id="MathJax-Span-54" class="mrow"><span id="MathJax-Span-55" class="mi">s</span></span></span></span> is an end state; <span id="MathJax-Element-9-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-56" class="math"><span id="MathJax-Span-57" class="mrow"><span id="MathJax-Span-58" class="mtext">Utility</span><span id="MathJax-Span-59" class="mo">(</span><span id="MathJax-Span-60" class="mi">s</span><span id="MathJax-Span-61" class="mo">)</span></span></span></span>, the utility of a state; <span id="MathJax-Element-10-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-62" class="math"><span id="MathJax-Span-63" class="mrow"><span id="MathJax-Span-64" class="mtext">Eval</span><span id="MathJax-Span-65" class="mo">(</span><span id="MathJax-Span-66" class="mi">s</span><span id="MathJax-Span-67" class="mo">)</span></span></span></span>, an evaluation function for the state <span id="MathJax-Element-11-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-68" class="math"><span id="MathJax-Span-69" class="mrow"><span id="MathJax-Span-70" class="mi">s</span></span></span></span>; <span id="MathJax-Element-12-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-71" class="math"><span id="MathJax-Span-72" class="mrow"><span id="MathJax-Span-73" class="mtext">Player</span><span id="MathJax-Span-74" class="mo">(</span><span id="MathJax-Span-75" class="mi">s</span><span id="MathJax-Span-76" class="mo">)</span></span></span></span>, which returns the player whose turn it is; <span id="MathJax-Element-13-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-77" class="math"><span id="MathJax-Span-78" class="mrow"><span id="MathJax-Span-79" class="mtext">Actions</span><span id="MathJax-Span-80" class="mo">(</span><span id="MathJax-Span-81" class="mi">s</span><span id="MathJax-Span-82" class="mo">)</span></span></span></span>, which returns the possible actions; and <span id="MathJax-Element-14-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-83" class="math"><span id="MathJax-Span-84" class="mrow"><span id="MathJax-Span-85" class="mtext">Succ</span><span id="MathJax-Span-86" class="mo">(</span><span id="MathJax-Span-87" class="mi">s</span><span id="MathJax-Span-88" class="mo">,</span><span id="MathJax-Span-89" class="mi">a</span><span id="MathJax-Span-90" class="mo">)</span></span></span></span>, which returns the successor state resulting from taking an action at a certain state. You may use any relevant notation introduced in lecture.</li>

 <li id="1b" class="code">[10 points] Now fill out <code>MinimaxAgent</code> class in <code><a href="submission.py">submission.py</a></code> using the above recurrence. Remember that your minimax agent should work with any number of ghosts, and your minimax tree should have multiple min layers (one for each ghost) for every max layer.Your code should also expand the game tree to an arbitrary depth. Score the leaves of your minimax tree with the supplied <code>self.evaluationFunction</code>, which defaults to <code>scoreEvaluationFunction</code>. The class <code>MinimaxAgent</code> extends <code>MultiAgentSearchAgent</code>, which gives access to <code>self.depth</code> and <code>self.evaluationFunction</code>. Make sure your minimax code makes reference to these two variables where appropriate as these variables are populated from the command line options.Other functions that you might use in the code: <code>GameState.getLegalActions()</code> which returns all the possible legal moves, where each move is <code>Directions.X</code> for some X in the set {<code>NORTH, SOUTH, WEST, EAST, STOP</code>}. Go through <code>ReflexAgent</code> code as suggested before to see how the above are used and also for other important methods like <code>GameState.getPacmanState(), GameState.getGhostStates(), GameState.getScore()</code> etc. These are further documented inside the <code>MinimaxAgent</code> class.<em><strong>Implementation Hints</strong></em>

  <ul>

   <li>Pac-Man is always agent 0, and the agents move in order of increasing agent index. Use <code>self.index</code> in your minimax implementation to refer to the Pac-Man’s index. Notice that only Pac-Man will actually be running your <code>MinimaxAgent</code>.</li>

   <li>All states in minimax should be <code>GameStates</code>, either passed in to <code>getAction</code> or generated via <code>GameState.generateSuccessor</code>. In this project, you will not be abstracting to simplified states.</li>

   <li>The evaluation function in this part is already written (<code>self.evaluationFunction</code>), and you should call this function without changing it. Use <code>self.evaluationFunction</code> in your definition of <span id="MathJax-Element-15-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-91" class="math"><span id="MathJax-Span-92" class="mrow"><span id="MathJax-Span-93" class="msubsup"><span id="MathJax-Span-94" class="mi">V</span><span id="MathJax-Span-95" class="mtext">max,min</span></span></span></span></span> wherever you used <span id="MathJax-Element-16-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-96" class="math"><span id="MathJax-Span-97" class="mrow"><span id="MathJax-Span-98" class="mtext">Eval</span><span id="MathJax-Span-99" class="mo">(</span><span id="MathJax-Span-100" class="mi">s</span><span id="MathJax-Span-101" class="mo">)</span></span></span></span> in part <span id="MathJax-Element-17-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-102" class="math"><span id="MathJax-Span-103" class="mrow"><span id="MathJax-Span-104" class="mn">1</span><span id="MathJax-Span-105" class="mi">a</span></span></span></span>. Recognize that now we’re evaluating <i>states</i> rather than actions. Look-ahead agents evaluate future states whereas reflex agents evaluate actions from the current state.</li>

   <li>You can assume that you will always have at least one action returned from <code>getLegalActions</code> function.</li>

   <li>If there is a tie between multiple actions for the best move, you may break the tie however you see fit.</li>

   <li>The minimax values of the initial state in the <code>minimaxClassic</code> layout are 9, 8, 7, -492 for depths 1, 2, 3 and 4 respectively. <b>You can use these numbers to verify whether your implementation is correct.</b> Note that your minimax agent will often win (just under 50% of the time for us–be sure to test on a large number of games using the <code>-n</code> and <code>-q</code> flags) despite the dire prediction of depth 4 minimax.<pre>python pacman.py -p MinimaxAgent -l minimaxClassic -a depth=4</pre></li>

  </ul><em><strong>Further Observations</strong></em>

  <ul>

   <li>On larger boards such as <code>openClassic</code> and <code>mediumClassic</code> (the default), you’ll find Pac-Man to be good at not dying, but quite bad at winning. He’ll often thrash around without making progress. He might even thrash around right next to a dot without eating it. Don’t worry if you see this behavior. Why does Pac-Man thrash around right next to a dot? (These questions are here for you to ponder upon; no need to include in the write-up.)</li>

   <li>Consider the following run:<pre>python pacman.py -p MinimaxAgent -l trappedClassic -a depth=3</pre>Why do you think Pac-Man rushes the closest ghost in minimax search on <code>trappedClassic</code>?</li>

  </ul></li>

</ol>

<ol class="problem">

 <li id="2a" class="code">[10 points] Make a new agent that uses alpha-beta pruning to more efficiently explore the minimax tree in <code>AlphaBetaAgent</code>. Again, your algorithm will be slightly more general than the pseudo-code in the slides, so part of the challenge is to extend the alpha-beta pruning logic appropriately to multiple minimizer agents.You should see a speed-up (perhaps depth 3 alpha-beta will run as fast as depth 2 minimax). Ideally, depth 3 on <code>mediumClassic</code> should run in just a few seconds per move or faster.<pre>python pacman.py -p AlphaBetaAgent -a depth=3</pre>The <code>AlphaBetaAgent</code> minimax values should be identical to the <code>MinimaxAgent</code> minimax values, although the actions it selects can vary because of different tie-breaking behavior. Again, the minimax values of the initial state in the <code>minimaxClassic</code> layout are 9, 8, 7, and -492 for depths 1, 2, 3, and 4, respectively. Running the command given above this paragraph, the minimax values of the initial state should be 9, 18, 27, and 36 for depths 1, 2, 3, and 4, respectively.</li>

</ol>

<ol class="problem">

 <li id="3a" class="writeup">[5 points] Random ghosts are of course not optimal minimax agents, so modeling them with minimax search is not optimal. Instead, write down the recurrence for <span id="MathJax-Element-18-Frame" class="MathJax" tabindex="0"><span id="MathJax-Span-106" class="math"><span id="MathJax-Span-107" class="mrow"><span id="MathJax-Span-108" class="msubsup"><span id="MathJax-Span-109" class="mi">V</span><span id="MathJax-Span-110" class="texatom"><span id="MathJax-Span-111" class="mrow"><span id="MathJax-Span-112" class="mtext">max,opp</span></span></span></span><span id="MathJax-Span-113" class="mo">(</span><span id="MathJax-Span-114" class="mi">s</span><span id="MathJax-Span-115" class="mo">,</span><span id="MathJax-Span-116" class="mi">d</span><span id="MathJax-Span-117" class="mo">)</span></span></span></span>, which is the maximum expected utility against ghosts that each follow the random policy which chooses a legal move uniformly at random. Your recurrence should resemble that of Problem 1a (meaning you should write it in terms of the same functions that were specified in 1a).</li>

 <li id="3b" class="code">[10 points] Fill in <code>ExpectimaxAgent</code>, where your agent will no longer take the min over all ghost actions, but the expectation according to your agent’s model of how the ghosts act. Assume Pac-Man is playing against multiple <code>RandomGhost</code>, which each chooses <code>getLegalActions</code> uniformly at random.You should now observe a more cavalier approach to close quarters with ghosts. In particular, if Pac-Man perceives that he could be trapped but might escape to grab a few more pieces of food, he’ll at least try:<pre>python pacman.py -p ExpectimaxAgent -l trappedClassic -a depth=3</pre>You may have to run this scenario a few times to see Pac-Man’s gamble pay off. Pac-Man would win half the time on an average and for this particular command, the final score would be -502 if Pac-Man loses and 532 or 531 (depending on your tiebreaking method and the particular trial) if it wins (you can use these numbers to validate your implementation).Why does Pac-Man’s behavior in expectimax differ from the minimax case (i.e., why doesn’t he head directly for the ghosts)? Again, just think about it; no need to write it up.</li>

</ol>

<ol class="problem">

 <li id="4a" class="code">[16 points] Write a better evaluation function for Pac-Man in the provided function <code>betterEvaluationFunction</code>. The evaluation function should evaluate states (rather than actions). You may use any tools at your disposal for evaluation, including any <code><a href="util.py">util.py</a></code> code from the previous assignments. With depth 2 search, your evaluation function should clear the <code>smallClassic</code> layout with two random ghosts more than half the time for full credit and still run at a reasonable rate.<pre>python pacman.py -l smallClassic -p ExpectimaxAgent -a evalFn=better -q -n 10</pre>We will run your Pac-Man agent 20 times, and calculate the average score you obtained in the winning games. Starting from 1300, you obtain 1 point per 100 point increase in your average winning score. In <code><a href="grader.py">grader.py</a></code>, you can see the how extra credit is awarded. You get 8 extra points when you reach 2000 or more. In addition, the top 3 people in the class will get additional points.Check out the current top scorers on the <a href="http://web.stanford.edu/class/cs221/leaderboard.html">leaderboard</a>! You will be added automatically when you submit.<em><strong>Hints and Observations</strong></em>

  <ul>

   <li>Having gone through the rest of the assignment, you should play Pac-Man again yourself and think about what kinds of features you want to add to the evaluation function. How can you add multiple features to your evaluation function?</li>

   <li>You may want to use the reciprocal of important values rather than the values themselves for your features.</li>

   <li>The betterEvaluationFunction should run in the same time limit as the other problems.</li>

  </ul></li>

 <li id="4b" class="writeup">[2 points] Clearly describe your evaluation function. What is the high-level motivation? Also talk about what else you tried, what worked and what didn’t. Please write your thoughts in <code>pacman.pdf</code> (not in code comments).</li>

</ol>

<em>Go Pac-Man Go!</em>

<b>Files:</b>

<table border="0" cellpadding="10">

 <tbody>

  <tr>

   <td><code><a href="submission.py">submission.py</a></code></td>

   <td>Where all of your multi-agent search agents will reside and the only file you need to concern yourself with for this assignment.</td>

  </tr>

  <tr>

   <td><code><a href="pacman.py">pacman.py</a></code></td>

   <td>The main file that runs Pac-Man games. This file also describes a Pac-Man <code>GameState</code> type, which you will use extensively in this project</td>

  </tr>

  <tr>

   <td><code><a href="game.py">game.py</a></code></td>

   <td>The logic behind how the Pac-Man world works. This file describes several supporting types like AgentState, Agent, Direction, and Grid.</td>

  </tr>

  <tr>

   <td><code><a href="util.py">util.py</a></code></td>

   <td>Useful data structures for implementing search algorithms.</td>

  </tr>

  <tr>

   <td><code><a href="graphicsDisplay.py">graphicsDisplay.py</a></code></td>

   <td>Graphics for Pac-Man</td>

  </tr>

  <tr>

   <td><code><a href="graphicsUtils.py">graphicsUtils.py</a></code></td>

   <td>Support for Pac-Man graphics</td>

  </tr>

  <tr>

   <td><code><a href="textDisplay.py">textDisplay.py</a></code></td>

   <td>ASCII graphics for Pac-Man</td>

  </tr>

  <tr>

   <td><code><a href="ghostAgents.py">ghostAgents.py</a></code></td>

   <td>Agents to control ghosts</td>

  </tr>

  <tr>

   <td><code><a href="keyboardAgents.py">keyboardAgents.py</a></code></td>

   <td>Keyboard interfaces to control Pac-Man</td>

  </tr>

  <tr>

   <td><code><a href="layout.py">layout.py</a></code></td>

   <td>Code for reading layout files and storing their contents</td>

  </tr>

 </tbody>

</table>




5/5 - (1 vote)

This (and every) assignment has a written part and a programming part.

The full assignment with our supporting code and scripts can be downloaded as <a href="../pacman.zip">pacman.zip</a>.

<ol class="problem">

 <li class="writeup template">This icon means a written answer is expected in <code>pacman.pdf</code>.</li>

 <li class="code template">This icon means you should write code in <code><a href="submission.py">submission.py</a></code>.</li>

</ol>

You should modify the code in <code><a href="submission.py">submission.py</a></code> between

<pre># BEGIN_YOUR_CODE</pre>

and

<pre># END_YOUR_CODE</pre>

but you can add other helper functions outside this block if you want. Do not make changes to files other than <code><a href="submission.py">submission.py</a></code>.Your code will be evaluated on two types of test cases, <b>basic</b> and <b>hidden</b>, which you can see in <code><a href="grader.py">grader.py</a></code>. Basic tests, which are fully provided to you, do not stress your code with large inputs or tricky corner cases. Hidden tests are more complex and do stress your code. The inputs of hidden tests are provided in <code><a href="grader.py">grader.py</a></code>, but the correct outputs are not. To run the tests, you will need to have <code><a href="graderUtil.py">graderUtil.py</a></code> in the same directory as your code and <code><a href="grader.py">grader.py</a></code>. Then, you can run all the tests by typing

<pre>python grader.py</pre>

This will tell you only whether you passed the basic tests. On the hidden tests, the script will alert you if your code takes too long or crashes, but does not say whether you got the correct output. You can also run a single test (e.g., <code>3a-0-basic</code>) by typing

<pre>python grader.py 3a-0-basic</pre>

We strongly encourage you to read and understand the test cases, create your own test cases, and not just blindly run <code><a href="grader.py">grader.py</a></code>.

<hr>

General Instructions

Problem 1: Minimax

Problem 2: Alpha-beta pruning

Problem 3: Expectimax