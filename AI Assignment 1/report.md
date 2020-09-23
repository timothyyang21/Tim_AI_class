# PA1 Chicken and Foxes

- Timothy Yang
- September 22, 2020
- CS76 Artificial Intelligence
- Prof. Alberto Quattrini Li

![enter image description here](https://hips.hearstapps.com/pop.h-cdn.co/assets/16/50/1600x800/landscape-1481817257-fox-chicken-corn.jpg?resize=980:*)
> Photo Source: Popular Mechanics
## Table of contents

* [Introduction](#introduction)
* [Code Design](#code-design)
* [Building the Model](#building-the-model)

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

In Search Solution.py, we create a solution class object for easy storing of the different solutions and their related properties such as the problem itself, the search strategy (method), the solution path, and a counter of how many nodes were visited through that solution. When returning the solution, it has a toString method which allows it to print out all the relevant information of the solution (or lack thereof) to the problem attempted by the search method. 

In uninformed_search.py, we write the actual strategy algorithms used and create a Searchnode class object to help with the search strategies. Every searchNode class object stores a state, a list of children, and a parent.

## Building the Model

The model of the code starts with foxes.py. Under foxes.py, the problems are defined using the FoxProblem class offered by FoxProblem.py, and run code into motion by directly calling the search functions.

The search functions takes the problem and create a solution class to store relavant information of the class. 

## Breadth First Search


## Memoizing Depth First Search (Discussion)


## Path-Checking Depth First Search


## Iterative Deepening Search


## Lossy Chicken & Foxes (Discussion)














