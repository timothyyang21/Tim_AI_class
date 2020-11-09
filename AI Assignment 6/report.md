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

In the Maze_Sensorless_Colors, I make these several adjustments: I added two command lines for "colors" and "walls" in maze construction in order to store the colors and walls for the maze. "walls" are really just an extra thing which I never really used, but "colors" is very important as the program needs to use the colors to actually do the probabilistic reasoning over time. I also updated the create_maze method in order to add the colors based on the floors. The colors are added randomly from the set of 4 colors: "red", "blue", "green", "yellow". The colors are randomly selected, even though at a later time I wish I specifically assigned them so they do not repeat next to each other, so the actual filtering program would have an easier time distilling down to one central probability.

In the ColorSensorProblem, I have created the problem class that will run the filtering program so that in the end, we can see the probability distribution for the robot's possible locations in the map. I store this in a dictinoary called current_probabilities_dictionary so that the coordinates can be easily linked to its current probability distributions. In the same fashion, I also created a dictionary for colors so that the actual colors of the coordinates can be mapped correctly. I created a detected_color_sequence list and a true_color_sequence list so that the colors that's detected by the robot's random movements may be stored. Speaking of the robot's random movements, I wrote a method random_movement in order to create a random movement for the robot that lasts about 10-20 steps. I find this number to be mostly enough for the program to figure out where the robot might be with high probability. I wish I made the random_movement method to be a little bit less random so that it will actually travel through the entire map at some point, instead of just randomly walking around in a more limited space (because it's true randomized). The get_successors function and its helper function deals with the transition state and sensor model, taking the previous probabilities and the current evidence (which is the detected_color_sequence), and changes its current probabilities back to the running class-owned dictionary. As can be shown, I used more of an ad-hoc method to implement filtering.

## Evaluation

Now I quickly go through some results for evaluation of my code. Here's my results:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%206/assignment_6_results.png" height="100%" width="100%">
</p>

As one can observe, the robot ends at tile (0,3), and its probabilities distribution over time program determined that the robot has 1/3 probability to be at (0,3). The other two coordinates that have 1/3 probability is (0,2) and (2,2). I reasoned this is the case because (0,2), (0,3), and (2,2) are all painted "blue", and the filtering program keep normalizing and give these the most probabilities because they are blue, and the robot itself didn't go far and wide enough in order for it to figure out specifically which of those three coordinates that the robot is more likely to be in. Well, at least my color filtering program didn't specify that, and I filter primarily based on the colors. Also, due to my robot random_movement program based on it being purely random, the robot really isn't able to walk out of the limited areas nearby where it is spawned at. The 7, 8, 9 numbers and onwards are the time steps. Here I included the first few time steps:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%206/assignment_6_results_2.png" height="100%" width="100%">
</p>

As one may observe, the probability for the coordinate (0,3) started off with an average probability of 0.09090909..., or 1/11 at time step 1. Based on the detected color sequence, the filtering program actually changes the (0,3)'s probability to lower before it goes up to become 1/3 in the end. At each time step, the robot believe it may be at some other locations a lot more than the others, however, overtime, eventually, at time step 6 and onwards, the probability of (0,3) and others become the most prominent probabilities, with 1/3 probability. This makes sense, as the other coordinates' probabilities grows smaller and smaller as the filtering program is more sure that the coordinates are impossible.

In the random_movement_list, we can see the robot's state of action at each time step.

## Responses to Discussion Questions

There happens to not have any discussion questions listed in the required test.

## Conclusion

Probabilistic reasoning over time is interesting and useful especially because it is more akin to the real world, when the situations themselves are not certain and may involve some kinds of uncertainty and probabilities distributions. What may be even more realistic however is how we apply AI to the real world. With that being said, I look forward to the upcoming writing assignment about AI and its implication about ethics. :sparkles:
