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

## Introduction
The problem of 3 chickens and 3 foxes crossing a river is a classic brain teaser. It asks, how would you get these animals across the river with a 2-animal boat without losing any chicken, if the foxes would eat the chicken up when the chicken are out-numbered?

In this assignment, we're interested in solving this problem using uninformed search strategies through a computer program (Aritificial Intelligence). To that end, we'd need to provide states in which the AI may use to solve the problem. 

States are either legal or not legal. The upper bound of all the states, regardless of legality, is 4 * 4 * 2 to the problem of 3 chicken and 3 foxes. We get this number because there's 4 possibilities for the chicken to be on either side of river (3-0, 2-1, 1-2, 0-3), 4 for foxes, and 2 possibilites for the boat, which may either be on one side or another (1-0, 0-1). In general, the upper bound of all states is #c * #f * #b states. 

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%201/demonstration.png" height="60%" width="60%">
</p>

## Code Design



## Building the Model



## Breadth First Search


## Memoizing Depth First Search (Discussion)


## Path-Checking Depth First Search


## Iterative Deepening Search


## Lossy Chicken & Foxes














