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
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/intro%20example.png" height="10%" width="10%">
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

A-star search is a form of best first search whose evaluation function is this: f(n) = g(n) + h(n), where g(n) is the cost to reach the node, and h(n) is the cost to get from the node to the goal. Since g(n) gives the path cost from the start node to node n, and h(n) is the estimated cost of the cheapest path from n to the goal, we have that f(n) is the estimated cost of the cheapest solution through n. Thus, if we are trying to find the cheapest solutiion, a reasonable thing to try first is the node with the lowest value g(n) + h(n). Indeed, A-star search is a very reasonable search; under right conditions of heuristic function, A-star search is both complete and optimal.

The A-star algorithm is identical to Uniform Cost Search except that A-star search uses g + h instead of just g. Here's my implementation:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/astar%20constructor.png" height="60%" width="60%">
</p>

The constructor of AstarNode takes a state, a heuristic, a parent, a transition cost (which is updated with heuristic cost), as well as fuel cost for the multi-robot problem.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/astar%20code.png" height="80%" width="80%">
</p>

First thing to note is that I included the multi-robot coordination problem model in here as well. The fuel_cost and fuel_cost_update helper function is for multi-robot coordination to better understand the fuel_cost they used and their optimality. Alright, starting from the top. First, the function itself takes in the problem and the heuristic function used. A priority queue and a start node is created annd the start node is pushed into the prioity queue. A solution is created in order to store relavant information. A dictionary of explored set (visited_cost) is created, which will mark nodes against its cost. In the loop, a node is continuously being popped unless there's none to pop, in which case the loop will end and the solution will be returned as failure - no solution found. First, the node is checked by the goal test to see if the current node is indeed the solution node, in which case the search ends, solution information updated, and the successful solution is returned. If that's not the case, the node is added to the explored dictionary, adding to it its associated cost. Then, successors of the state is produced using the helper get_successors function. A child node is created to store the state along with new cost. Then, the child is added to the priority queue if it's not in the explored set nor in the prioity queue itself, or else if the state is already in queue, it's compared against the previous ones to see which one has a lower cost, if the new one has lower cost, it will then replace the higher cost one (old one). In this way, the priority queue will always first explore the nodes with lower cost, resulting in the optimality that was mentioned before.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/astar%20results.png" height="60%" width="60%">
</p>

This is a simple result of using the astar search, along with the heuristic of manhattan distance. The cost is reflected in the results, since manhattan distance calculates the difference in distance from current state and goal state.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/astar%20results%202.png" height="60%" width="60%">
</p>

Here icluded is a comparison between null heuristic and manhattan distance heuristic's Astar results. Notice how although they find a solution of equal length, the null heuristic one went through considerbly more nodes.

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

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/first%20test%201.png" height="10%" width="10%">
</p>

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/first%20test%202.png" height="10%" width="10%">
</p>

The first test is interesting because the robots are in some sort of corridor and in the wrong order, and they have to move around before letting A to go ahead first. The B robot basically set aside (above C), while letting A go ahead. The solution length is 16 because robot C and B waited for the A to pass, resulting in the fuel cost of only 10.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/second%20test%201.png" height="10%" width="10%">
</p>

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/second%20test%201.5.png" height="10%" width="10%">
</p>

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/second%20test%202.png" height="10%" width="10%">
</p>

The second test is interesting in the same way. C must go in first before A or B does. We'd expect something along the lines of A and B first waited before C goes, however, A and C actually went at the same time. A went a little bit more over before realizing it must let C past first, then A went in, before B does.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/third%20test%201.png" height="10%" width="10%">
</p>

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/third%20test%202.png" height="10%" width="10%">
</p>

In the third test we saw the same thing, A didn't have to move, so it stayed there most of the time. However, as soon as B closed in, A moved away a bit to let B pass before closing in together.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/test%204%201.png" height="10%" width="10%">
</p>

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/test%204.png" height="30%" width="30%">
</p>

Now test 4 is quite interesting because it has no solution, yet it still went for 7 nodes before realizing there's no way it can possibly get there.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/test%205%201.png" height="10%" width="10%">
</p>

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/test%205%202.png" height="10%" width="10%">
</p>

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/test%205%203.png" height="10%" width="10%">
</p>

And there is test 5, notice how A went out, while B also went out, to first let C in.

## Sensorless Problem Code Model

> Assume that you now only have a single robot in mazeworld, but there's a catch. The robot is blind, and doesn't know where it starts! The robot does know the map of the maze. The robot has a sensor that can tell it what direction is North (so that it can still move in intended directions). However, the robot has no other sensors.  No, it really can't tell when it hits a wall.

Now this is quite the problem. How would a robot be able to find its way out if it doesn't even know where it starts, or when it hits a wall? Luckily, there is a way. We can write a planning algorithm that gives a plan that even the blind robot can follow, knowing that the robot will **eventually** know where it is by trying out different actions. For example, if execting the action "west" from a configuration where "west" is blocked, the robot simply doesn't move.

Let the 'state' in the search be the current set of possible locations of the robot. For example, in a 4x3 maze, the first state in the search will be the set of all 12 possible starting locations: { (0, 0), (1, 0), (2, 0), ... (3,2) }. The actions will be {north, south, east, west}. The goal test function tests if the state is a singleton, with just one element in the set. If there is just one element, then the robot knows where it is. After all, it does know the shape of the maze. Here's my implementation:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/sensorless%20constructor.png" height="60%" width="60%">
</p>

The constructor consists of a sensorless_maze object, a start state which is built by the help of the helper function, and number of robots, which is set to 1 for this blind robot problem. The state of the sensorlessProblem is a list that consists of all the possible coordinates of the robots. The first item in the state is a String, in order to mark the direction it is taking in order to cut down different states. This will be clearer after looking at the get_successor function:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/sensorless%20code.png" height="60%" width="60%">
</p>

The goal test checks whether the state is a singleton. The get_successors method returns a list of possible successors based on its current possible states. It takes the current possible state, first create a set of coordinates for easy checking if it's in original state in a later crucial manipulation. Then each direction is being considered. If it's north, then the new coordinate set (y + 1, going north), is judged by whether the new location is a floor and is included in the original set, only if those two passed will the new coordinate set be added to a new list (where only that particular direction is different), and added to a set of successors which will then be returned by the function, after being converted to a tuple. Note, the state again, is: all the possible states of possible coordinate locations. So it only makes sense that it's returning a tuple set (possible sets) of tuple lists (coordinates).

The get_successors method is used in astar_search, which allows the function to eventually dwindle down the possible states and return a path from start to end of which direction it takes to dwindle down the possible states to only 1 state, which is the goal node.

## Blind Robot with Pacman Physics

> Describe what heuristic you used for the A* search. Is the heuristic optimistic? Are there other heuristics you might use? (An excellent answer might compare a few different heuristics for effectiveness and make an argument about optimality.)

After trying to be ambitious and tried to develop the **perfect** heuristic and failing, I cameback and be humbly submit 3 heuristics that I used. First, I use the null heuristic. And the results is as followed on a 20x20 maze:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/null_heuristic.png" height="60%" width="60%">
</p>

Then, I used the heuristic that favors the one with the lower number of possible states. The result is as followed. One can see that this heuristic, although checked a lot more nodes than null heuristic, ended up finding a shorter path to check all the possible states (starting locations), at least on a bigger maze like 20x20.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/sensorless_heuristic.png" height="60%" width="60%">
</p>

Then, I used the heuristic that favors the one with the higher number of possible states. The result is as followed. One can see that this heuristic, although ended up finding a longer path to check all the possible states (starting locations), ended up checking fewer nodes. This is due to the fact that it always opt to expand the option with the most amount of possible states first, resulting the get_successors function to do more work for the search algorithm, and in the process cut down more nodes. However, the longer path is created because, again, it **always** go along and expand the one with the more possible states first, resulting that at the late-game (new term from gaming that suggests the later phase of a process), the algorithm will still always expand the one with more possible states first, resulting in longer solution lenght. Quite interesting, isn't it?

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/sensorless_heuristic_opposite.png" height="60%" width="60%">
</p>

Aside from the null heuristic, the one that favors expanding the ones with fewer number of possible states, or sensorless_heuristic, is optimistc in that it never overestimate the cost of reaching the goal. This makes sense because it always choose the ones with fewer ones first, resulting in lower and lower cost. However, the one that favors expanding the ones with higher number of possible states ended up finding a longer solution, or sensorless_heuristic_opposite, almost always overestimate the cost of reaching the goal. In terms of optimality, I'd say none of these are truly optimal, especially in bigger mazes. To be optimal, the algorithm must find the shortest solution path. In bigger mazes, the shortest solution path will have to be coming from a heuristic that favors the ones with the possible states that can be cut down the most from. This heuristic, which I tried in vain to do earlier, has to take into account of how many repeating x or y coordinate there is in average, assign the ones with the most a lower value (especially when there are many possible coordinates), and assign higher value to the onest with the most when there are only few possible coordinates left. This optimal heuristic has to be dynamic and work with the get_successors function well.

The sensorless_heuristic may have a passing "optimal" grade, however, under closer exaimination, we know that always choosing to favor expanding the states with fewer possible coordinations isn't always the best idea, sometimes, it's best to expand the states with more possible coordinations in the hope that doing so, one will be able to cut the possible coordinations down significantly (according to the process mentioned earlier).

Here included is the sensorless results of a 5x5 map as well as the animated path:

<p align="center">
  <img "https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%202/sensorless.png" height="60%" width="60%">
</p>

One can see the actions that the algorithm takes regarding how to get to those possible locations: the "Start", "South", "West", "West" corresponds to an action sequence that my algorithm might take to initially tackle this maze. The animated complete path shows us how the robot belief state changes as the path is executed. Different heuristic does this differently, of course, and the program model is highly adaptable. Animate the complete path, showing how the

## Conclusion

Informed search in AI is a lot more exciting and efficient than uninformed search. The idea of being able to use problem-specific knowledge beyond the definition of the problem itself helps a great ton in efficiently finding solutions. Having an evaluation function that takes in the cost estimate of certain actions (and act according to some heuristics) allows the AI to first look into promising steps, and cross greater leaps to find the right solutions. Still, nothing is truly exciting without having some kind of competition. I look forward to implementing Adversial Search! 🚀
