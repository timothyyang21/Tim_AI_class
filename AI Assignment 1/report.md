# PA1 Chicken and Foxes

- Timothy Yang
- September 22, 2020
- CS76 Artificial Intelligence
- Prof. Alberto Quattrini Li
- TAs: Almas Abdibayev, Hunter Gallant, Maxine Perroni-Scharf

![enter image description here](https://hips.hearstapps.com/pop.h-cdn.co/assets/16/50/1600x800/landscape-1481817257-fox-chicken-corn.jpg?resize=980:*)
> Photo Source: Popular Mechanics
## Table of contents

* [Introduction](#introduction)
* [Code Design](#code-design)
* [Building the Model](#building-the-model)
* [Breadth First Search](#breadth-first-search)
* [Memoizing Depth First Search Discussion](#memoizing-depth-first-search-discussion)
* [Path-Checking Depth First Search](#path-checking-depth-first-search)


## Introduction
The problem of 3 chickens and 3 foxes crossing a river is a classic brain teaser. It asks, how would you get these animals across the river with a 2-animal boat without losing any chicken, if the foxes would eat the chicken up when the chicken are out-numbered?

In this assignment, we're interested in solving this problem using uninformed search strategies through a computer program (Aritificial Intelligence). To that end, we'd need to provide **states** in which the AI may use to find the solution to the problem. Indeed, the solution will be a connected sequence of states from the initial state to the goal state; each pair of states is linked by an action.

The **state** of the system is the information that would be needed to depect the current situation at some point during the task to someone agent, assuming that they already know the rules of the game.  For the chickens and foxes problem, the state could be described as how many chickens and foxes are on each side of the river, and on which side of the river the boat is.  

We do not count constants as states. The constants include the total number of chicken and foxes and the boat size (which holds 2). These can still be changed, however, these are only changed in the problem definition. 

States are either legal or not legal. The upper bound of all the states, regardless of legality, is 4 * 4 * 2 = 16 to the problem of 3 chicken and 3 foxes. We get this number because there's 4 possibilities for the chicken to be on either side of river (3-0, 2-1, 1-2, 0-3), 4 for foxes, and 2 possibilites for the boat, which may either be on one side or another (1-0, 0-1). In general, the upper bound of all states is #c * #f * #b states. 


<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%201/demo_states.png" height="60%" width="60%">
</p>

In the graph, green means the states are legal, while the red ones means they aren't. As one can see from the initial state, out of the 16 possible states, only 3 legal states may be reached by legal actions. Illegal states that may not be reached by legal actions do not have their corresponding action (edge) shown because they won't be taken either way. Legal actions are defined as actions that can be carried out from that state, and don't lead to a state where any chicken is eaten. From the 3 legal states the first state reaches, the light green states are the secondly expanded legal states, while the bright red ones are secondly expanded illegale states. 

Evevery state can be think of as nodes in graph, while actions are the edges. There is an underlying graph of all possible legal states, and there are edges between them. The algorithms that's shown in this report will implicitly search this graph. The graps will not be generated ahead of time, however, but rather it will follow methods that, given a state, generate legal actions and possible successor states, based on the current state and the rules of the game.

Now, let us get into Code Design! Special note: I use the first person plural narrative ("we" "us") in this report merely to add personalness and reflect that while it's an individual project, all of the class are working on this assignment. :sparkles: 

## Code Design

The code design is relatively simple. In this assignment, we use 4 python files, the first three each containing its own class, while the fourth one "foxes.py" is the main that runs the program:
- FoxProblem.py
- SearchSolution.py
- uninformed_search.py
- foxes.py

In FoxProblem.py, we build the main problem model into its own class and methods. We'll talk more about this in the [Building the Model](#building-the-model) section. What is to note that this is the starting point. In the main "foxes.py", the problem is stated using the class object offered by FoxProblem.py, which stores the relavant information to the problem (the start state, the goal state, etc). Under FoxProblem.py we also have methods such as goal_test and get_successors to help the search algorithm functions in uninformed_search.py. 

In Search Solution.py, we create a solution class object for easy storing of the different solutions and their related properties such as the problem itself, the search strategy (method), the solution path, and a counter of how many nodes were visited through that solution. When returning the solution, it has a toString method which allows it to print out all the relevant information of the solution (or lack thereof) to the problem attempted by the search method. This efficiency and usefulness of this class design will be shown in the dfs and ids search algorithm, as the solution is passed along to each new recursive call so that things like number of nodes visited may be recorded.

In uninformed_search.py, we write the actual strategy algorithms used and create a Searchnode class object to help with the search strategies. Every searchNode class object stores a state, a list of children, and a parent.

## Building the Model

The model of the code starts with foxes.py. Under foxes.py, the problems are defined using the FoxProblem class offered by FoxProblem.py, and run code into motion by directly calling the search functions.

The search functions takes the problem and create a solution class to store relavant information of the class. In general, the search functions then create their start node using the problem object and run their own algorithms that utilizes functions built from the FoxProblem class. They use goal_test to check if the running state is at goal, and they use get_successors to get valid successor states that can be reached from the running state. In the end, all of the search functions will return a solution object, which as stated in Code Design, show all the information about the run through and most importantly, whether a solution is found.

Now let's talk more about the problem model and how it's built in FoxProblem.py. The FoxProblem itself takes a start state and store the start state as well as the goal state (0,0,0). In my implementation, I also include the total number of chicken and total number of foxes (taken from the start state) for easy and readable access to those in the methods listed for the class. A problem model needs to expand from its start state in order to eventually find a reachable solution state, if there is one, to do so, we need a get_successors method that returns valid successors from a state. The get_successors method will generate a set of states that's valid and nontrivial for the search algorithms, with the help of a helper method called is_state_valid, which filters out illegal and trivial states. Note, trivial states are states that are legal, but unecessary and inefficient for any solution to go through them. These include, going back to original state (this saves nodes_visited), having a state where there are far too many of one animal on one side in relation to the other, or having a next state where there are only ever 1 animal going to the second river side, as the second river side is where we want all the animals to end up at. Illegal states are all checked by the method as well, using simple and easy-to-read return false technique as opposed to a long chain of if statements. States with different combinations of chicken and fox are generated based on the original state, and checked by the is_state_valid method before adding on to the set, which was then returned for search algorithm expansion.

## Breadth First Search

As many all know, breadth first search is a simple strategy that first expand the root node, and then expand all of the successors of the root node, then their successors. It's a strategy that in general goes through each layer of depth in tree in order and start with expanding the shallowest nodes. This is achieved by using a first-in-first out queue for the "frontier" being searched, the new nodes gets to the back of the queue while the old ones are expanded first. To do so, I used python's dequeue data structure to store the nodes in the frontier, allowing it to optimally return the oldest node. I also use the Python set to store node states that are already explored, this way, the search will not explore the same state more than once, and the search would work properly on a graph. Using a set to do so allows optimal efficiency (O(1) time) when checking if a node has been explored (is in the data structure). On the other hand, using a linked list to keep track of which states have been visited would be a poor choice because it would always take O(n) time to look through the entire linked list.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%201/bfs.png" height="60%" width="60%">
</p>

At the start of bfs, the root node is added into the **frontier** queue unless the root node (provided by the search problem) happens to be the goal. The while loop make sure every node in queue is explored. The **explored** set stores the state that has been explored so that no nodes with repeating states will be added into the frontier queue. Then the expansion of the node begins based on the state using the get_successors method, which returns valid successor states that the node is linked to. To make sure they are linked, a child search node is created with that node and corresponding state. The check checks whether the child state is new and unexplored in both the explored set (which includes the ones that has been explored), and the frontier queue, which stores the ones that are just about to be expanded. If the child node past both tests but is not the goal, it will be added to the frontier node. On the other hand, if the node turn out to be the goal, then the **bfs_backchaining** helper method is utilized to return the correct solution path. The solution gets updated and returned to print out the test results. If at the end of the queue there were no more valid unexplored nodes to be explored, the program will return the solution at the end without a solution path. This will prompt the SearchSolution to print out "no solution found after visiting ... nodes" which indicates there's no solution. The number of visited nodes are still available due to the SearchSolution class.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%201/bfs_backchaining.png" height="60%" width="60%">
</p>

The bfs_backchaining method simply extracts the path from the graph after the search has found the goal node. Because we stored a link to a parent node in each child node, by simply using a while loop and set 
> node = node.parent

each time, we go through all of the nodes in the solution path. This is possible because bfs enables us to know that all the shallower goal nodes must have been generated already and failed the goal test. Note, I use reverse() at the end to save some time, instead of appending new node to the front which takes O(n) times, even though this won't really make a difference in this scenario due to the solution path being relatively short.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%201/bfs%20results.png" height="100%" width="100%">
</p>

As shown in the results, the bfs results in short solution paths but have visited relatively more nodes (this comparison will become clearer when looking at the dfs results). For the problem (3,3,1), it visited 44 nodes before finding a path of length 12. For the problem (5,4,1), it visited 98 nodes before finding a path of length 16. For the problem (5,5,1), it visited 9 nodes before realizing there is no solution path for that there were no more valid unexplored states to expand into. More comparison will be made in [Path-Checking Depth First Search](#path-checking-depth-first-search). 

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%201/bfs%20analysis.png" height="20%" width="20%">
</p>

After checking with the nodes I drew in the intro and checking against print statements, this results makes sense, as the end solution turns does follow the sequence. As shown in the following print statements: the start node come in from the queue, three successor states are expanded and added into the queue, next comes (3,2,0), whose successor states are none due to my nontrivial states check, however, new successor states like (3,2,1) and (2,2,0) are added as child into the frontier (the only ones doing so as they are the only new ones).

For time complexity, bfs takes O(b^d) time, where b is the branching factor, and d is the depth of the shallowest solution. This is the case because at each depth, each branch is tracked. Memory complexity is similar. The explored set would have O(b^d-1) nodes in the explored set and O(b^d) nodes in the frontier.

## Memoizing Depth First Search (Discussion)

The discussion question asks:
> Does memoizing dfs save significant memory with respect to breadth-first search?  Why or why not?

I think not necessarily, memoizing dfs may end up using up similar amount of memory (if not same or worse), because by memoizing, or keeping track of all states that have been visited in some sort of data structure, and making sure the dfs never visits the same state twice, the space complexity may as well reach to O(b^d), particularly when the dfs may dive deep into many (or all) branches before finding a solution. So no, memoizing dfs does not save significant memory with respect to breadth-first search.

## Path-Checking Depth First Search

Unlike bfs, the dfs always expands the deepest node in the current forntier of the search tree before expanding to other branches. Previously we discuss briefly about memoizing dfs, another style of depth-first search keeps track only of states on the path currently being explored, and makes sure that they are not visited again. This ensures that at least the current path does not have any loops. For path-checking dfs, the node that the dfs explored to is always first checked against a running path that's stored by the solution. To do so, we must pass the solution through the parameter of each recursive call of dfs.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%201/dfs.png" height="60%" width="60%">
</p>

The recursive helper function of the dfs is below:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%201/recursive%20dfs.png" height="60%" width="60%">
</p>

The discussion question asks:
> Does path-checking depth-first search save significant memory with respect to breadth-first search? Draw an example of a graph where path-checking dfs takes much more run-time than breadth-first search; include in your report and discuss.

Not necessarily. Due to graph may have cycles?

## Iterative Deepening Search

* talk about iplementation and include code *

The discussion question asks:
> On a graph, would it make sense to use path-checking dfs, or would you prefer memoizing dfs in your iterative deepening search?  Consider both time and memory aspects.  (Hint.  If it's not better than bfs, just use bfs.)

I would prefer to use memoizing dfs in my iterative deepening search. The reasoning is, in an iterative deepning search, the cycle constantly starts a new for path_checking, where they must keep searching for different nodes. However, for memoizing dfs, the memory may be stored through the solution which may be reused for each ids, meaning that each cycle, the memoizing dfs may start with a deeper depth immediately and keep searching for states that are not part of its memory, which saves time. Memory wise both will be using similar amount of memory, which is O(bm) for path checking dfs and O(bm + m) for memoizing dfs. m standing for the maximum depth of the search tree.

## Lossy Chicken & Foxes (Discussion)

The Lossy Chicken & Foxes problem goes like this:
> Every fox knows the saying that you can't make an omelet without breaking a few eggs.  What if, in the service of their faith, some chickens were willing to be made into lunch?  Let us design a problem where no more than E chickens could be eaten, where E is some constant.  

So, the discussion question asks:

> What would the state for this problem be?  What changes would you have to make to your code to implement a solution?  Give an upper bound on the number of possible states for this problem.  (You need not implement anything here.)

The state for this problem then, will be (c, f, b, e). **e** signaling the number of chicken already eaten. This is the case because by keeping track of the eaten chicken, states would be notified and more (necessary) possible connections between different states would be made. The changes I'll have to make is mostly in the problem modeling is_state_valid method. This method would need to be modified so that the get_successors method would also return states where a number of chicken may be eaten, alongside with the usual states, thus the search functions may keep on going to more possible states. Speifically, we will need to modify the chicken numbers with e whenever they show up in the is_state_valid method, and rework many other boundary checking if statements in it to make sure the method will also say the cases where some determined numbers of chicken may be eaten, are safe. The goal state doesn't need to change, but we must know the E before hand, and we must use that E number to alter the get_successors and is_state_valid code.

Here's an narrow upper bound on the number of possible states for this problem: 
> if c + E > f: 2f + (c-1)(c-f+1+E)
> if c + E == f: 2c + (c-1)

Where c stands for number of chicken, f stands for number of foxes, and E stands for how many chicken may be eaten. Also, c + E must not be less than f to start off with. 
This is the case because the general case's narrow upper bound on the number of possible states for this problem is:
> if c > f: 2f + (c-1)(c-f+1)
> if c == f: 2c + (c-1)(1)

The first 2c comes from the maximum chicken number on the first side, on the other side, the foxes on the other side can have any combination. Same thing for when there's no chicken on the first side of the river -> c + c = 2c. In between the maximum number of chicken and 0, there's c-1 possibilities where the chicken number must be matched by the number of foxes on the other side. Thus, 2c + (c-1). However, if c > f, then the 2c will be 2f. Also, the c-1 becomes (c-1)(c-f+1) because now the chicken number is greater than the foxes, and the numbers in between the maximum c and 0 may vary, with each difference in c and f creating an additional possibility for chicken and number to exist on both side of the river. taking this form with E, we realize it's simply adding the E number on to the (c-f+1) and reworking the inequality condition to fit the new E in.

I'm less sure about the narrow upper bound on the number of possible states for this problem when E is concerned, so I'm also going to give a loose upper bound. From what was said about how the total number of state must be c * f * b, the very loose upper bound of possible states will be c * f * b * E. Where E is the number of chicken that may be sacrificed, this may be the case because anywhere from E to 0 chicken may be eaten. These are possible states, not necessarily counting into the legality of the states.

## Introduction

Uninformed search in AI is, while interesting and fulfilling to implement, not the most exciting nor efficient. After all, it's called uninformed search, Which means that the strategies have no additional information about states beyond that provided in the problem definition. These search strategies may be neccessary and powerful in situations where there are limited information, however, I look forward to implement informed strategies and see more efficient results!


