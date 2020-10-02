# PA2 Mazeworld

- Timothy Yang :blush:
- September 30, 2020 :fallen_leaf:
- CS76 Artificial Intelligence :robot:
- Prof. Alberto Quattrini Li :it:
- TAs: Almas Abdibayev :sparkles:, Hunter Gallant :rocket:, Maxine Perroni-Scharf :unicorn:

![enter image description here](https://s3.amazonaws.com/cdn.pura-aventura.com/images/medium/laberinto-HR320180829-76980-5e8gkd.jpg?1535505438)
> Photo Source: Pura Adventura
## Table of contents

* [Introduction](#introduction)
* [Code Design](#code-design)
* [Building the Model](#building-the-model)
* [A-star Search](#a-star-search)
* [Multi-robot Cooperation](#multi-robot-cooperation)
* [Sensorless Problem Code Model](#sensorless-problem-model)
* [Blind Robot with Pacman Physics](#blind-robot-with-pacman-physics)
* [Conclusion](#conclusion)


## Introduction
In Mazeworld Wonderland, you find yourself in a 2D maze, along with 2 other friends. You cannot see them, but from the device you were given, you know where they are located and how close they are to you. You actually even get to know the whole maze, which makes things considerably easy. Your task is to find your friends, and together, get to the specified goal location indicated on your devices. As humans, we instinctively have many strategies to solve this problem, ranging from always following the left walls or devising a route first by looking at the map; however, as computer program AI, solving this problem becomes a little different.

Indeed, in this assignment, we're interested in solving this problem using informed search strategies through a computer program (Aritificial Intelligence). Similar to assignment 1, we'd need to provide **states** in which the AI may use to find the solution to the problem. Same as last time, the solution will be a connected sequence of states from the initial state to the goal state; each pair of states is linked by an action. What's different, however, is that this time around, the AI may take into account the goal locations, since they are informed and know something about the goal node's location using heuristics. This will be discussed more in detail by the end of the introduction.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/intro%20example.png" height="20%" width="20%">
</p>

Here's a more detailed example of the maze we are going to be working with. The periods represent open floor tiles (places that can be walked on or over), and the number signs represents walls, places in the maze where a simple robot cannot go. At least one robot will be present in a maze. It may move in any of the four directions: North (towards the top of the screen), East, South, and West. There are coordinates on the maze. (0, 0) is the bottom left corner of the maze. (6, 0) is the bottom right corner of the maze. In this particular example, perhaps we'd like to plan a path for the robot to go through the maze from the bottom right corner to the top left corner, without going through any walls. As one can already see, the program could be informed to know some information about how far away the goal location is from the starting location.

Our robot's actions in this case will be the robot going north, east, south, west, and in the case for multi-robots, not moving. To include the chance for other robots' turn, in our states, we include the marker of whose turn it is in the beginning of the state. Each robot adds two more places to the state, representing the x and y coordination of the robot. Thus, we have: (turn_marker, robot_1_x, robot_1_y, robot_2_x, robot_2_y, etc.)

Let n be the size of the square maze, and let r be the number of robots. The upper bound of all the states, regardless of legality, then is r * (n^2)^r. Consider one of the simplest problem settings. Say there are 3 robots in a 6x6 maze, then the state space is 3 * (36^3) = 139968. We get this number because there's 3 possibilities in terms of whose turn is it, 36 possibilities of the coordination for robot 1, and same for robot 2 and robot 3, so 3 * 36 * 36 * 36. In this problem specifically, robots cannot occupy the same space, so it's more like 3 * 36 * 35 * 34. However, this number isn't considering the presence of maze walls, so either way, this loose upper bound stands ground.

Similar to assignment one, every state can be think of as nodes in graph, while actions are the edges. There is an underlying graph of all possible legal states, and there are edges between them. However, unlike assignment one with uninformed searches, the algorithms that's shown in this report will explicitly search this graph. The graphs will not be generated ahead of time, but rather it will follow methods that, given a state, generate legal actions and possible successor states based on the current state, the rules of the game, and heuristics that will be used to determine which successor states to follow to get closer to the goal states.

Indeed. In assignment 2, heuristics is the star of the show, which is what makes the search strategies presented here as "informed". Using heuristics such as Manhattan distance, we are able to let the algorithm take into account information about the goal node's location while searching for solution to the goal node. In this way, the informed algorithm will find the goal much faster than uninformed strategies.

Now, let us move to discuss Code Design! :sparkles: 

## Code Design

The code design is not as straightforward, particularly due to having another problem we also investigate in this assignment - the sensorless problem, which we'll talk more in depth in * [Blind Robot with Pacman Physics](#blind-robot-with-pacman-physics) . In this assignment, we use 7 total python files, inlcuding two testing files that runs the programs, the other six each contains its own class:
- Maze.py
- MazeworldProblem.py
- SearchSolution.py
- astar_search.py
- test_mazeworld.py

- SensorlessProblem.py
- test_sensorless.py

In MazeworldProblem.py, we build the main problem model into its own class and methods. We'll talk more about this and Maze.py in the [Building the Model](#building-the-model) section. What is to note that this, along with Maze.py, is the starting point. In the main "test_mazeworld.py", the problem is stated using the class object offered by Maze.py and MazeworldProblem.py, which stores the relavant information to the problem (the start state, the goal state, etc). Under MazeWorldProblem.py we also have methods such as goal_test and get_successors to help the search algorithm functions in astar_search.py. In Maze.py, we build the basic structure of our maze and use it to create more mazes.

As in Assignment 1, we have an Search Solution.py, in which we create a solution class object for easy storing of the different solutions and their related properties such as the problem itself, the search strategy (method), the solution path, the path cost, and a counter of how many nodes were visited through that solution. When returning the solution, it has a toString method which allows it to print out all the relevant information of the solution (or lack thereof) to the problem attempted by the search method.

In astar_search.py, we write the actual strategy algorithms used and create a AstarNode class object to help with the search strategies. Every AstarNode class object stores a state, a parent, a heuristic, and the running transition cost.

In SensorlessProblem.py, we build the second problem model that deals with the sensorless robot problem. Similar to MazeworldProblem, it offers a class object and many methods that's used by the main running program test_sensorless.py in order to run the program.

## Building the Model

The model of the code starts with foxes.py. Under foxes.py, the problems are defined using the FoxProblem class offered by FoxProblem.py, and run code into motion by directly calling the search functions.

The search functions takes the problem and create a solution class to store relavant information of the class. In general, the search functions then create their start node using the problem object and run their own algorithms that utilizes functions built from the FoxProblem class. They use goal_test to check if the running state is at goal, and they use get_successors to get valid successor states that can be reached from the running state. In the end, all of the search functions will return a solution object, which as stated in Code Design, show all the information about the run through and most importantly, whether a solution is found.

Now let's talk more about the problem model and how it's built in FoxProblem.py. The FoxProblem itself takes a start state and store the start state as well as the goal state (0,0,0). In my implementation, I also include the total number of chicken and total number of foxes (taken from the start state) for easy and readable access to those in the methods listed for the class. A problem model needs to expand from its start state in order to eventually find a reachable solution state, if there is one, to do so, we need a get_successors method that returns valid successors from a state. The get_successors method will generate a set of states that's valid and nontrivial for the search algorithms, with the help of a helper method called is_state_valid, which filters out illegal and trivial states. Note, trivial states are states that are legal, but unecessary and inefficient for any solution to go through them. These include, going back to original state (this saves nodes_visited), having a state where there are far too many of one animal on one side in relation to the other, or having a next state where there are only ever 1 animal going to the second river side, as the second river side is where we want all the animals to end up at. Illegal states are all checked by the method as well, using simple and easy-to-read return false technique as opposed to a long chain of if statements. States with different combinations of chicken and fox are generated based on the original state, and checked by the is_state_valid method before adding on to the set, which was then returned for search algorithm expansion.

## A-star Search

As many all know, breadth first search is a simple strategy that first expand the root node, and then expand all of the successors of the root node, then their successors. It's a strategy that in general goes through each layer of depth in tree in order and start with expanding the shallowest nodes. This is achieved by using a first-in-first out queue for the "frontier" being searched, the new nodes gets to the back of the queue while the old ones are expanded first. To do so, I used python's dequeue data structure to store the nodes in the frontier, allowing it to optimally return the oldest node. I also use the Python set to store node states that are already explored, this way, the search will not explore the same state more than once, and the search would work properly on a graph. Using a set to do so allows optimal efficiency (O(1) time) when checking if a node has been explored (is in the data structure). On the other hand, using a linked list to keep track of which states have been visited would be a poor choice because it would always take O(n) time to look through the entire linked list.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%201/bfs.png" height="10%" width="10%">
</p>

At the start of bfs, the root node is added into the **frontier** queue unless the root node (provided by the search problem) happens to be the goal. The while loop make sure every node in queue is explored. The **explored** set stores the state that has been explored so that no nodes with repeating states will be added into the frontier queue. Then the expansion of the node begins based on the state using the get_successors method, which returns valid successor states that the node is linked to. To make sure they are linked, a child search node is created with that node and corresponding state. The check checks whether the child state is new and unexplored in both the explored set (which includes the ones that has been explored), and the frontier queue, which stores the ones that are just about to be expanded. If the child node past both tests but is not the goal, it will be added to the frontier node. On the other hand, if the node turn out to be the goal, then the **backchaining** helper method is utilized to return the correct solution path. The solution gets updated and returned to print out the test results. If at the end of the queue there were no more valid unexplored nodes to be explored, the program will return the solution at the end without a solution path. This will prompt the SearchSolution to print out "no solution found after visiting ... nodes" which indicates there's no solution. The number of visited nodes are still available due to the SearchSolution class.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%201/backchaining.png" height="60%" width="60%">
</p>

The backchaining method simply extracts the path from the graph after the search has found the goal node. Because we stored a link to a parent node in each child node, by simply using a while loop and set 
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

## Multi-robot Coordination


## Sensorless Problem Code Model


## Blind Robot with Pacman Physics


## Conclusion


