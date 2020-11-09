# PA6 Probabilistic Reasoning Over Time

- Timothy Yang :blush:
- November 8, 2020 (20F) :fallen_leaf:
- CS76 Artificial Intelligence :robot:
- Prof. Alberto Quattrini Li :it:
- TAs: Almas Abdibayev :sparkles:, Hunter Gallant :rocket:, Maxine Perroni-Scharf :unicorn:  

![enter image description here](https://cdn.pixabay.com/photo/2018/04/09/16/57/maze-3304668_960_720.png)
> Photo Source: Pixaby.com

## Table of contents

* [Introduction](#introduction)
* [Implementation and Design Choices](#implementation-and-design-choices)
* [Evaluation](#evaluation)
* [Responses to Discussion Questions](#responses-to-discussion-questions)
* [Conclusion](#conclusion)


## Introduction

Welcome to blind robot maze part 2. This time around, however, we have a blind robot that only have one sensor pointing down, to keep track of the colors painted in the maze itself. However, the sensor only works 88% most of the time, the rest of the 12% of the time the sensor would produce an inaccurate color. Anywho, the spirit of the assignment today is to use probabilistic inference over time. Without further ado, let's get started.

## Implementation and Design Choices

In this assignment, I onlye used 2 total python files:
- Maze_Sensorless_Colors.py
- ColorSensorProblem.py

I recycled the maze files from assignment 2 and made some adjustment to create Maze_Sensorless_Colors. In the Maze_Sensorless_Colors, I built the basic maze structure that will be used for ColorSensorProblem, in which I run the actual progam.



## Evaluation

Now I quickly go through some results for evaluation of my code. Here's my results for the first

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%206/assignment_6_results.png" height="100%" width="100%">
</p>



## Responses to Discussion Questions

There happens to not have any discussion questions listed in the required test. I will still discuss a little bit of my findings, though. 

## Conclusion

Solving a sudoku with AI with internal logic is quite interesting and fun! Instead of using purely reflex mechanisms, they are using processes of reasoning that operate on internal representations of knowledge now. Can't wait to see the next assignment and get it done! :sparkles:
