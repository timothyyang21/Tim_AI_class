# PA3 Chess

- Timothy Yang :blush:
- October 10, 2020 :fallen_leaf:
- CS76 Artificial Intelligence :robot:
- Prof. Alberto Quattrini Li :it:
- TAs: Almas Abdibayev :sparkles:, Hunter Gallant :rocket:, Maxine Perroni-Scharf :unicorn:

![enter image description here](https://cdn.pixabay.com/photo/2019/03/02/09/46/sketch-4029522_1280.jpg)
> Photo Source: Pixabay
## Table of contents

* [Introduction](#introduction)
* [Implementation & Design Choices](#implementation-&-design-choices)
* [Evaluation](#evaluation)
* [Responses to Discussion Questions](#responses-to-discussion-questions)
* [Conclusion](#conclusion)

## Introduction

Chess! What a brilliant game that has inspire and entertain people throughout history. I'd love to give a more fun introduction to the Adversial-Search AI assignment I worked on, but looking at I'm running out of time, I'd just go straight to the point and make this report short and sweet. To that end, I'm only going to talk about three key things. 1st, the implementation and design choices I made for Minimax & Cutoff, Evaluation Function, Alpha-Beta Pruning, and Iterative Deepening. 2nd, I'm going to give a brief (but thorough) walk through of my evaluation over my algorithm, giving results to show that everything work as planned. 3rd, I'm going to respond to some discussion questions posted in the task portion of the assignment.

## Implementation & Design Choices

In this assignment, I used 7 total python files, inlcuding one testing file test_chess.py that runs the programs, the other six each contains its own class:
- ChessGame.py
- HumanPlayer.py
- RandomAI.py
- MinimaxAI.py
- AlphaBetaAI.py
- IterativeAlphaBetaAI.py
- test.chess.py

ChessGame.py give us the backbones of the AI assignment. HumanPlayer.py and RandomAI.py has already largely been designed and coded for us. The only made I change in them is to update the choose_move function to include "player_color" to keep track of which side the AI is on. ChessGame.py is modified similarly, with additionally a list of moves that's been made for any certain game. This comes in handy in my AI implementation in order to reconstruct the board state for which the AI used to make prediction moves on.

MinimaxAI.py, AlphaBetaAI.py, and IterativeAlphaBetaAI.py are the main focus of the assignment. IterativeAlphaBetaAI is simply AlphaBetaAI but using a iterative deepening search version, searching all the way up to the designed max depth and choose the best move to return. I choose AlphaBeta AI because this AI gives the same results with MinimaxAI but with faster performance. I decided to create a new class for it because it's the easiest and most intuitive to do so. My AlphaBeta AI is simply an upgraded/improved version of my MinimaxAI.py, in that it uses the AlphaBeta pruning technique to get rid of leaves that are comparitively useless to make the algorithm faster. I added the alpha beta value manipulation as suggested from the book, and the one thing that's very different from Minimax AI is the Actions() function. Actions(state) is Alpha-beta pruning is the key to alpha-beta's success. In this funciton, instead of just returning a list of moves, we order the best moves to the front so it's faster (by checking them and comparing them against the ongoing alpha beta values). The try order is as follows: captures first, then threats, then forward moves, backward moves. To accomplish this, I created two lists, the first list captures the "captures first, then threats", and the second list "forward moves and backward moves", and I added the list together to return for the function. I created an Is_Forward() helper function for this function. Other than this and the alpha beta updates, everything else is the same as Minimax.

For minimax, I decided to passed in the move_list and player_color for faster and more intuitive coding. After all, the speed of creating a current board state is similar to popping all the way back to the beginning. Also, player_color allows the AI algorithms to know which player they are handling max and min for. Max will try to return the maximized value out of the choices min gives it (with depth contraints). The Minimax_Decision function utilizes this to return a move best for the current player, by calling the recurrsive max or min value functions. To avoid repition, I used a value of 20% of the chances of which the program replaces the current return action with another action with the same value. This avoids time used to sort and pick random actions. Speaking of values, there's the utility function and the evaluation function. Utility function gives 0.5 for stalemate scenarios, 1 for the max side winning, -1 for min side winning, and 0 for depth-limit reached. (We are using depth limit here too because we don't want the program to run forever, we want it to run like real games.) Evaluation function returns a value for the board that's passed on. This value determines which side may have an edge over another. Taken from the textbook, it's 1 for pawns, 3 for bishops and knights, 5 for rooks, and 8 for queens. Because I used pseudo_legal_moves (and corresponding constraints that can be found all over my code), I aslo include 100 for kings in case any moves kills a king. The program will end once there's no more kings, though (in test_chess.py). I have negative numbers for all the points for the min player, as the min player is trying to minimize the points in order to win:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%203/design%20choices.png" height="60%" width="60%">
</p>

Due to time constraints, please take a look at the code itself to further get a taste of my implementation and design choices.

## Evaluation

Here I give a little example of all three as I run my code

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%203/Minimax%20Run%20Example%201.png" height="60%" width="60%">
</p>

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%203/Minimax%20Run%20Example%202.png" height="20%" width="20%">
</p>


## Responses to Discussion Questions


## Conclusion

Adversial Search was so fun! It's quite refreshing and inspiring to build entities that compete for themselves. I look forward to the next AI adventuer! :sparkles:
