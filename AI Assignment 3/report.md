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

Here I give a little example of all three as I run my code. First, we have minimax using Evaluation function (Plain Utility also works):

After black randomly moves a knight forward, white sees an opportunity to attack (and kill it to score better evaluation), so it moves its pawn to try to eat it. Immediately seeing an openning, black moves its queen to check the opponents' king, knowing that the king's the end goal (it's worth so much points, after all). 

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%203/Minimax%20Run%20Example%201.png" height="20%" width="20%">
</p>

Then, we see White immediately move its knight to block the queen's path to protect the king. Notice how the calls volume is a lot smaller, this is because there's only so many legal moves left to save the king. The total calls volume is only 7 greater then the max_depth calls volume in this scenario because the depth is 2, the calls at the max_depth is 5216 because there are many combinations of moves after any of those 7 initial moves.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%203/Minimax%20Run%20Example%202.png" height="20%" width="20%">
</p>

Secondly, we have alpha beta. I have alpha beta as AI players playing against each other at depth 2. It's significantly faster and they explore less moves, even in the mid-game:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%203/Alphabeta%20Run%20Example%201.png" height="20%" width="20%">
</p>

Both sides are keeping each other in check, not allowing anyside to change the value balance that much. On another game, the value's still very much similar.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%203/Alphabeta%20Run%20Example%203.png" height="20%" width="20%">
</p>

However, one side made an accident and lead the point values to be a bit different. Also, notice the intelligent gameplay still, the bishop came over and exchanged with the equal-valued knight, after doing so, the rook immediately took over the bishop.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%203/Alphabeta%20Run%20Example%204.png" height="20%" width="20%">
</p>

From the significant faster moves and fewer calls (than Minimax) at the same depth -- while keeping equal values and keeping the intelligent gameplay, it's clear that the Alphabeta algorithm works.

And lastly, we have iterative deepening alpha beta. In iterative deepening alpha beta search, I find that some moves are made actually based on an earlier depth, instead of a deeper one. This shows that while most of the time a better move may come from the deepest depth it can go (after all - more information is better), sometimes the better move comes from a shallower depth.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%203/iterative%20deepning%20search.png" height="20%" width="20%">
</p>

## Responses to Discussion Questions

For Minimax and Cutoff Test's discussion, I found that at depth 2 the algorithm's fairly fast, computing a step calls around 15,000 to 20,000 calls in early games (cutting it down to one third of that in check scenarios), and that call volume takes about 2 seconds or less. However, at depth 3, in the early games it computes around 200,000 to 300,000 calls, with each step taking about 30-40 seconds. The calls volume at max depth in both cases significantly overpowers the calls at other depths. At depth 1, the algorithm's basically flying, computing only aobut 500-1000 calls for each step. The call and time difference among the different depths makes sense, because with each new depth, the algorithm search ony layer deeper, and that layer is all the possibilities of different end-state that can be achieved by the different moves. The factor is about 10-20, indicating that at each game turn, a player has about that many possibilities for different moves. And this is only in the beginning state, the middle game will have a greater factor. (Early games and end games moves are limited due to the game construct.)

For evalution function's discussion, we have already seen in the Evaluation part of the report that the AI will cleverly attack and block moves, choosing to give the most important piece (king) a check and the defending side blocking the threat from the queen. It is not as easy to beat as RandomAI or Minimax with only utility function anymore, it's more clever and act just like a beginner/intermediate human player.

As for Alphabeta's discussion, I wasn't able to use a specific initial scenario due to lack of time. However, I checked many different examples of the AI's game play (and playing against them) to find that they all conduct the intelligent move and lead to same values of scores every time. (For example, they won't just not leave a knight untaken if they have the power of taking them -- maybe they will wait for a turn to just give the other side a check first, but eventually they will for sure take the knight.) To include a same "initial" state to show that I've checked many times, I have this initial position to show:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%203/Minimax%20same%20position%2C%20same%20value.png" height="20%" width="20%">
</p>

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%203/AlphaBeta%20same%20position%2C%20same%20value.png" height="20%" width="20%">
</p>

The discussion asks us to make sure that we show at least some evidence to show that Minimax and Alphabeta have the same value for specific positions (to show Alphabeta is doing the right thing), thus, I made sure that both are intelligent and Alphbeta will always return an action of similar value by making sure of it first, and then showing an example.

Another discussion point for alpha beta, we see that instead of taking 20-30 seconds for eachs step for depth 3 (like Minimax), Alphabeta AI searched a step at depth 3 under 4-5 seconds, with similarly intelligent moves. This makes complete sense because Alphabeta AI can reduce the time complexity from O(b^m) up to O(b^m/2), if the order of searching is optimal. This means that a call can be (at best scenario), square-rooted. 20 seconds can become 4-5 seconds for sure. The total calls also go from 200,000-300,000 to 20,000 to 40,000 in the early games.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%203/Alphabeta%20faster.png" height="20%" width="20%">
</p>

As for iterative deepening search, I tested many tests and verify that indeed, the best move changes and sometimes improved as deeper levels are searched.

## Conclusion

Adversial Search was so fun! It's quite refreshing and inspiring to build entities that compete for themselves. I look forward to the next AI adventuer! :sparkles:
