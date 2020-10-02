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

The code design is not as straightforward, particularly due to having another problem we also investigate in this assignment - the sensorless problem, which we'll talk more in depth in * [Blind Robot with Pacman Physics](#blind-robot-with-pacman-physics) . In this assignment, we use 8 total python files, inlcuding two testing files that runs the programs, the other six each contains its own class:
- Maze.py
- MazeworldProblem.py
- SearchSolution.py
- astar_search.py
- test_mazeworld.py

- Maze_Sensorless.py
- SensorlessProblem.py
- test_sensorless.py

In MazeworldProblem.py, we build the main problem model into its own class and methods. We'll talk more about this and Maze.py in the [Building the Model](#building-the-model) section. What is to note that this, along with Maze.py, is the starting point. In the main "test_mazeworld.py", the problem is stated using the class object offered by Maze.py and MazeworldProblem.py, which stores the relavant information to the problem (the start state, the goal state, etc). Under MazeWorldProblem.py we also have methods such as goal_test and get_successors to help the search algorithm functions in astar_search.py. In Maze.py, we build the basic structure of our maze and use it to create more mazes.

As in Assignment 1, we have an Search Solution.py, in which we create a solution class object for easy storing of the different solutions and their related properties such as the problem itself, the search strategy (method), the solution path, the path cost, and a counter of how many nodes were visited through that solution. When returning the solution, it has a toString method which allows it to print out all the relevant information of the solution (or lack thereof) to the problem attempted by the search method.

In astar_search.py, we write the actual strategy algorithms used and create a AstarNode class object to help with the search strategies. Every AstarNode class object stores a state, a parent, a heuristic, and the running transition cost.

In SensorlessProblem.py, we build the second problem model that deals with the sensorless robot problem. Similar to MazeworldProblem, it offers a class object and many methods that's used by the main running program test_sensorless.py in order to run the program. Maze_Sensorless.py is created to better support the sensorless problem when animating paths, due to my particular implementation.

## Building the Model

The model of the code starts with test_mazeworld.py and test_sensorless.py. Under test_mazeworld.py and test_sensorless.py, the maze class object are first created by openning a maze file (provided by Maze.py or Maze_sensorless.py), then the problems are defined using the respective class offered by MazeworldProblem.py and SensorlessProblem.py, and run code into motion by directly calling the astar search function.In the test, not only do we print the result solution, we also call the animate_path function to animate the path, or print the maze step by step, to show how the robots moved in the solution (how the solution is solved).
 
The astar search takes the problem (which can be Mazeworld or Sensorless) and create a solution class to store relavant information of the class. The astar search also created their start node using the problem object and the problem's heuristic function, and run their own algorithms that utilizes functions built from the FoxProblem class. They use goal_test to check if the running state is at goal, and they use get_successors to get valid successor states that can be reached from the running state. In the end, the astar search will return a solution object, which as stated in Code Design, show all the information about the run through and most importantly, whether a solution is found.

Before we talk about the problem model itself, let's take a look at Maze.py. In Maze.py, a maze object class is defined as well as its many methods such as index, is_floor, has_robot, create_render_list, and toString method that all help to create a maze suitable for our program and the assignment. In particular, is_floor is useful for our get_successors function in the problem model, robot_loc (as a class variable) is very useful for animating path and creating a starting state for the problem model. We also need to design and write a create_maze function ourselves in Maze.py, in order to randomly generate our own maps to test out different scenarios for understanding the strategies better and for the discussion questions. The create_maze method itself was hard to write, but after understanding how the previous maze files work, it becomes a lot easier. What to watch out for is the indexing of file writing is different from the indexing that we usually use. Also, I use an arbituary 30% probability percentage for walls (30% walls : 70% floors).

Now let's talk in detail about the problem model and how it's built in MazeworldProblem.py. We'll talk about Sensorless problem model in a later section. The MazeworldProblem itself takes a list of goal locations and a maze. In my implementation, I also include the total number of robots (taken from the starting robot_loc) for easy and readable access to those in the methods listed for the class. There's also the start state variable which provides the starting state tuple based on the initial robot_loc, provided by the helper method.

A problem model needs to expand from its start state in order to eventually find a reachable solution state, if there is one. To do so, we need a get_successors method that returns valid successors from a state. The get_successors method will generate a set of states that's valid and nontrivial for the search algorithms, with the help of the maze's is_floor method and a helper method called check_overlap, which filters out illegal and trivial states, since robots can't be on a wall location or where another robot is at. States are generated based on possible locations for the robot in question, for the robot having the right to go, to be precise. States with different combinations of robots locations are then generated based on the original state, in paritcular, the 4 different directions as well as the original location. Each direction moves the robot by moving its x or y coordination, to that end, a tuple-list operation is used to only change the one x or y coordination where the direction dictates, each of these possibilities are added to a set of successors which is then returned. The reset_t helper function is crucial for this step, as it cleanly produces an original tuple list for each direction to then modify, before it's returned.

The goal_test function checks if the current state has robots inside all of the goal locations (if there's more than just 1). Each robot is checked against the predetermined robot goal location. If any of these location sets are mis-matched, then the method return False. Only when all robots are in the correct location does the method returns True, which will prompt the search function to stop and produce the solution.

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

In this twist, we have more than just 1 robots in the mazeworld, all trying to get to their pre-determined goal. Similar to the 1-robot version, the idea is to use Astar search with the added twist of "fuel cost". Indeed, the cost function is the total fuel expended by the robots. A robot expends one unit of fuel if it moves, and no fuel if it waits a turn. We have discussed a bit about how we would incorporate this into our code, and how we must represent the different robots in the introduction, other restrictions were also briefly touched upon, including how robots must take turns and cannot stand in the same floor as another.

Before we get into the implementation, let's get some discussion questions out of the way.

> If there are k robots, how would you represent the state of the system? Hint -- how many numbers are needed to exactly reconstruct the locations of all the robots, if we somehow forgot where all of the robots were? Further hint. Do you need to know anything else to determine exactly what actions are available from this state?

Exactly 2k + 1 numbers are needed to reconstruct the locations of all the robots, each robot must need a x and a y coordination, after all. (Assuming robots don't completely fill the maze, in that case we only need 2(k-1) + 1 numbers). Notice, the + 1 number is the "turn" indicator of which robot's turn it is. We need this in order to determine exactly what locations are available from this state.

> Give an upper bound on the number of states in the system, in terms of n and k. 

Similar to what was said before in the introduction, the loose upper bound on the number of states in the system is: k * (n^2)^k. Where the first k indicates how many possibilities of the turn indicator marker is, and n^2 indicates the x * y number of possibilities, to the kth power because each robot has 1 set of x and y coordinates, and each may combine with another robot's coordinations to result in (n^2)^k's possibilities. Technically, it's n^2 * (n-1)^2 * (n-2)^2, ... (n-k)^2, however, we assume k is much smaller than n^2 in the scenarios we test.

> Give a rough estimate on how many of these states represent collisions if the number of wall squares is w, and n is much larger than k.

As a rough estimate, 1 - k * ((n^2 - w)^k would be the states that represent collisions if the number of wall squares is w, and n is much larger than k. This is because, n^2 - w are the coordinates that are possible for any robots to be in, thus, k * (n^2-w)^k would be the over all possible legal states, wherease 1 minus that would represent the collisions with walls.

> If there are not many walls, n is large (say 100x100), and several  robots (say 10), do you expect a straightforward breadth-first search on the state space to be computationally feasible for all start and goal pairs? Why or why not?

Let's put them into our upper bound state formula. 10 * (100^2)^10 gives us 10^15 states. Assuming the worst case scenario for the breadth-first search, 10^15 states is a gigantic amount to search through. From our textbook's section 3.4's figure 3.13, we see that at depth 15 or nodes 10^15 would give us about 35 years to search using bfs, with a memory usage of about 1 exabyte. That's simply computationally unfeasible.

> Describe a useful, monotonic heuristic function for this search space. Show that your heuristic is monotonic. See the textbook for a formal definition of monotonic.

In the textbook, monotonicisty is used as a synonym for consistency, which describes a triangle inequality. In particular, a heuristic is consistent (or monotonic) if, "for every node n and every successor n' of n generated by any action a, the estimated cost of reaching the goal from n is no greater than the step cost of getting to n' plus the estimated cost of reaching the goal from n'". Simply put, a monotonic heuristic's estimate is always less than or equal to the estimated distance from any neighbouring vertex to the goal, plus the cost of reaching that neighbour. Also, a consistent heuristic is also admissable, thus it will never overestimate the cost of reaching the goal. In that sense, a useful monotonic heuristic function for this search space would be the Manhattan Distance heuristic, in a large part due to robots can't move along diagonals. It's simple to show how a manhattan distance heuristic is monotonic. For a node to reach its goal, it must travel through x horizontal space and y verticle space. The manhattan distance is simply the sum of x and y. For a node's immediate neighbor to reach its goal, it must also travel through x horizontal space and y verticle space, adding or subtracting 1 distance depending on the neighbor's direction. However, the cost to reach the neighbor is 1 itself, thus the original is always equal to or less than going through its neighbors, using the manhattan distance heuristc.

> Describe why the 8-puzzle in the book is a special case of this problem. Is the heuristic function you chose a good one for the 8-puzzle?

An 8-puzzle is a special case of this problem because each misplaced tile must reach its goal destination, similar to how the lost robots must reach their goal locations. The Manhattan heuristic function I choose is a good one for the 8-puzzle because 1) the tiles cannot move along diagonals, the distance we will count is the sum of the horizontal and verticle distances, and 2) it's admissible because all any move can do is move one tile one step closer to the goal. A manhattan distance heuristic is perfect for solving the 8-puzzle.

> The state space of the 8-puzzle is made of two disjoint sets.  Describe how you would modify your program to prove this. (You do not have to implement this.)

This is very interesting. The idea of the 8-puzzle's state pace is made of two disjoin set suggests that there are two completely separated configurations, one type may reach the "standard end goal", while the other one will not. In order to prove this, I would modify my program by only running with n=3, robots=8, and no walls. I'll generate random initial configurations and test them against the goal states. The idea is, each robot must try to find a way to get to its "standard end goal". If somehow some initial states can't reach it, that means these states would have never been reached by the other states as well, especially since moving tiles means any state that may reach the goal state may move to any configuration from that goal state. However, if there are states that can never reach the goal state, we've proved that there are indeed two disjoint sets of states in the state space of the 8-puzzle.

> Implement a model of the system and use A* search to find some paths. Test your program on mazes with between one and three robots, of sizes varying from 5x5 to 40x40. (You might want to randomly generate the 40x40 mazes.) I'll leave it up to you to devise some cool examples -- but give me at least five and describe why they are interesting. (For example, what if the robots were in some sort of corridor, in the "wrong" order, and had to do something tricky to reverse their order?)

## Sensorless Problem Code Model

## Blind Robot with Pacman Physics

> Describe what heuristic you used for the A* search. Is the heuristic optimistic? Are there other heuristics you might use? (An excellent answer might compare a few different heuristics for effectiveness and make an argument about optimality.)



## Conclusion

Informed search in AI is a lot more exciting and efficient than uninformed search. The idea of being able to use problem-specific knowledge beyond the definition of the problem itself helps a great ton in efficiently finding solutions. Having an evaluation function that takes in the cost estimate of certain actions (and act according to some heuristics) allows the AI to first look into promising steps, and cross greater leaps to find the right solutions. Still, nothing is truly exciting without having some kind of competition. I look forward to implementing Adversial Search! 🚀
