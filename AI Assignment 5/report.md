# PA5 Logic

- Timothy Yang :blush:
- October 27, 2020 (20F) :fallen_leaf:
- CS76 Artificial Intelligence :robot:
- Prof. Alberto Quattrini Li :it:
- TAs: Almas Abdibayev :sparkles:, Hunter Gallant :rocket:, Maxine Perroni-Scharf :unicorn:  

![enter image description here](https://images.theconversation.com/files/7533/original/3g6pkfzg-1328838628.jpg?ixlib=rb-1.1.0&q=45&auto=format&w=754&fit=clip)
> Photo Source: Conversation.com

## Table of contents

* [Introduction](#introduction)
* [Implementation and Design Choices](#implementation-and-design-choices)
* [Evaluation](#evaluation)
* [Responses to Discussion Questions](#responses-to-discussion-questions)
* [Conclusion](#conclusion)

## Introduction

The sudoku assignment is a constraint satisfication problem solved with prepositional logic. In this report, I'm going to talk about three key things. First, the implementation and design choices I made for implementing the GSAT and walkSAT algorithms. 2nd, I'm going to give a brief (but thorough) walk-through of my evaluation over my algorithm, giving results to show that everything worked as planned. 3rd, I'm going to respond to some discussion questions posted in the task portion of the assignment. Let us begin!

## Implementation and Design Choices

In this assignment, I used 5 total python files:
- SAT.py

- Sudoku.py
- sudoku2cnf.py
- display.py
- solve_sudoku.py

SAT.py is the main place where I implemented my solutions. Sudoku.py, display.py, sudoku2cnf.py, and solve_sudoku.py are all foundation files that help make the sudoku puzzle possible. In particular, Sudoku.py is the backbone of creating the sudoku problem itself. Display.py allows easy human-readable display of a sudoku. Sudoku2cnf.py helps transform a sudoku file into a cnf file, in which all variables are listed in conjunctive normal form, which helps with the logic for AI. Lastly, solve_sudoku.py is the testing python file to which a user may test the solvers with different cnf files. Each cnf files are designed to be in increasingly difficult order of cnfs to satisfy. There's the one_cell.cnf, where the satisfaction constraints only are placed on one cell (that there can only be one value between 1-9. There's the all_cells.cnf, to which constraints are placed on all cells, and each cell can only have one number between 1-9. And so on. In any case, the SAT.py file is the main file we should focus on.

In my implementation, I created quite a lot of helper functions for both the gsat and walksat algorithm. These helper functions are mostly quite straightforward, even though some of them have to deal with reading the cnf files, which makes things harder. Let's start with my constructor and instance variables of SAT; Gsat and walksat algorithm are both contained in the SAT object in the SAT class. The class takes a file name (a cnf file), and store it for itself. It then immediately uses that file to create these following three variables for the convenience of the algorithms: total_number_of_variables, corresponding_index_list, and total_clauses. These three are created here so that we don't need to create it over and over, having it once in the very beginning saves time. These three variables are gotten by its corresponding helper initialization functions, which all take the file name as their parameters. While the explanation for creating the other two variables it's self-explantory, for corresponding_index_list I created it because it facilitates getting the corresponding variables to index, which will be very useful in checking satisfiability in the algorithms. I also have h and p values, which are the threshold value for gsat and walksat's algorithms respectively, in order to determine how often should the algorithm randomly flips values or flipping them strategically. I set this number at 0.3. Max_flips determined how many flips are allowed before the algorithms should stop trying. Working_model is the model that constantly gets updated while solution_model is the one that will be used in the write solution function. One thing to note is that my implementations allow the input to be in any form, and only change them into indices in lists so all I have to work with are integers.

Let's talk about gsat right away. For my gsat, I did a variant of it in the beginning just so that the program won't run forever, by setting a forloop of limit at max_flips, just like how walksat does it. First thing in gsat, I checked whether the model already satisfies the cnf constraints, if it does, we set the solution model and return true for the algorithm, which will indicate to solve_sudoku that there's a solution, which will prompt it to display the solution. The satisfies method helps check the satisfiability. How I check it is simply to make sure that every line is true, whenver there's a line that's false, I break the check and return false, becuase having one clause in cnf form that's false will make the whole cnf false. Also, I check how a line is true by checking whether if at least one of the line's variables are true. Because in cnf form, each clause is composed of variables joined by unions, which means as long as one is true, then all is true. I utilized some neat if checking statements and boolean maneuvers for this. Moving on, next, I use the h value to see whether this iteration we go into the random step, which essentially just choose a random value to flip and flip them. In the other situation, I score how many clauses would be satisfied if the variable value were flipped, using the score helper function, whose implementation is quite straightforward. One thing that I spent a lot of time decoding on is that I didn't realize I was just passing the reference when checking "if the variable value were flipped", so essentially for a long while I had been flipping it and flipping it back to no avail. But thankfully that's figured out by making sure to make a copy of the list instead of just passing references. For each score, I check it against the highest score, if it's equally high, it gets into the list, if it's higher, then the old list is cleared and it gets appended to the list. A variable then is randomly chosen from the list, and used to flip the value in the model list.

Walksat is very similar, with 1 major twist. In the first situation, instead of chooing just any random variable, we select a variable from a clause that we know for sure are unsatisfied to flip. This way, we increase our chances of flipping the right ones and get closer to the solution faster. The helper function used here are also quite straightforward. By the way, a flip is simply a adjustment of timing a negative 1, to make it the opposite value of what it was before. The rest of walksat is the same implementation. Also, for the random generation of a model in the beginning. I used a ration of 8/9 to 1/9 as to which variables should be arbitrarily selected as false or true, this is because in the real sudoku, each cell can only have 1 value out of the 9 possible value, so this ration makes the most sense and should be the most convenient for the algorithms.

Finally, write_solution writes the actual number for the corresponding index of the sudoku, so it ignores all negative (are false) variables and only place the true ones.

Due to time constraints, please take a look at the code itself to further get a further taste of my implementation and design choices.

## Evaluation

Now I quickly go through some results for evaluation of my code. First, we have the results of the gsat with all_cells.cnf, with h set to 0.9 which means there's only 0.1 chance of randomization.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%205/gsat_all_cells_0.9.png" height="100%" width="100%">
</p>

As one can see, it took 930 seconds to run through the gsat algorithm. At each iteration, it picks from the max_list which contains variables that have the most connections to unsatisfied clauses, and changes that to reduce the number of unsatisfied clauses left, which shows that the algorithm is working:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%205/algorithm_working.png" height="100%" width="100%">
</p>

Notice how when situation 1 shows up, the number of clauses unsatisfied actually didn't get reduced (it only get reduced by situation 2 - strategic cleaning); this is because situation 1 is all about random picking of variable to flip in gsat, and more likely than not it will pick a wrong number to flip when the algorithm is at its very end (with an easy cnf file).

Here's the results of the walksat algorithm with all_cells.cnf, with p set to 0.9 which means 0.1 chance of randomization.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%205/walksat_all_cells_0.9.png" height="100%" width="100%">
</p>

In walksat with the same condition, it takes a much less time to get to a solution than gsat, with only 839 seconds. This is due to the clever randomization part of walksat, instead of choosing a pure random variable, it chooses a variable randomly from a pool of unsatisfied clauses with that specific variable. Only 0.1 of the time does it go into the randomization situation in this case though, so it still took a lot of situation 2 to solve the problem, which is why it's still considered slow. However, let's see what happens when I adjust the p value from 0.9 to 0.3 for the all_cells.cnf file:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%205/walksat_all_cells_0.3.png" height="100%" width="100%">
</p>

The time spent decreased to 240 seconds, or 4 minutes. This is simply because for the all_cells.cnf satisfaction logic problem, all the conjunctive constraints are very straightforward and has little overlaps. In fact, when I set the p = 0, the result is a little over 1 second:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%205/walksat_all_cells_0.png" height="100%" width="100%">
</p>

This is due to the fact that the algorithm doesn't need to go through situation 2 at all, all it has to do is to go through situation 1 and all variables chosen would work. But that's not really interesting, let's see what happens when we increase our difficulty level for walksat 1 step further to rows.cnf, with p set to 0.3.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%205/walksat_rows_0.3.png" height="100%" width="100%">
</p>

With p = 0.3, the walksat algorithm took 1312 seconds to run rows.cnf. This makes sense because now the algorithm has to jump around and jump out of local maximas in order to get to a complete solution. Towards the end, when the unsatisfied clauses were only 3 or so, it can be observed clearly that it stayed there for a while, keep jumping around trying new numbers, before finding a solution.

Lastly, I also tested my walksat algorithm for rows_and_cols.cnf, with p = 0.1 because I thought that'd allow the algorithm to run faster. 

Due to time constraints, I'm unale to wait out for the solution before turning in the report, here's a working-in-progress report, though:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%205/walksat_rows_and_cols_0.3.png" height="100%" width="100%">
</p>

Just like last time, it revolves around a few unsatisfied variables and must jump between them to try to jump out of the local maxima. I suspect this will take 1.5 times than last time. I shall continue to wait out the result to see how everything works out!

Anyways, I have further verified my code by breaking my code down, adding print statements, and checking the sequences individually. Thank you so much!

## Responses to Discussion Questions

There happens to not have any discussion questions listed in the required test. I will still discuss a little bit of my findings, though. First, I found that gsat takes a lot more time to solve then same difficulty-level cnfs than walksat. This is surely due to the fact that walksat chooses to randomize only on the ones with unsatisfied clauses, rather than just choosing random ones -- ones in which they could have already been satisfied -- to work with. Also, partly due to this, for the easier cnfs, for gsat, at least, having the arbitrary number up actually helps with finding the solution faster. This is presumebly due to the fact that having more chances of randomly choosing variables to flip will lead to more chances of the solution being unstable, and thus needing to go through stabilizing steps even more. I have quite a high confidence that for all_cells.cnf, a h value of 0.99 (meaning there's only a 0.01 chance of randomizing), will allow the gsat algorithm to go the fastest it can be. Or maybe not, because the randomization value is very important to allow us to jump out of the local minima; I can totally see the use of this more for later, more difficult cnf. When I set h = 0.9, even gsat gets to the solution in about 15 minutes. Each iteration took about 20 seconds. This number makes sense as the algorithm will have to go through the cnf file again and again to check the validity of all clauses, and each iteration only flips one variable, which may only have influence over 1 to several unsatisfied clauses.

The randomization steps essentially helps the algorithms jump away from local minima, while the strategic steps make sure that the algorithm is always getting to some kind of high score (higher the number of satisfied clause means getting closer to getting done). The push and pull of these two allows the algorithm to eventually find a solution.

## Conclusion

Solving a sudoku with AI with internal logic is quite interesting and fun! Instead of using purely reflex mechanisms, they are using processes of reasoning that operate on internal representations of knowledge now. Can't wait to see the next assignment and get it done! :sparkles:
