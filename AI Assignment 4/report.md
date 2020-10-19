# PA4 Constraint Satisfaction Problems

- Timothy Yang :blush:
- October 18, 2020 (20F) :fallen_leaf:
- CS76 Artificial Intelligence :robot:
- Prof. Alberto Quattrini Li :it:
- TAs: Almas Abdibayev :sparkles:, Hunter Gallant :rocket:, Maxine Perroni-Scharf :unicorn:  

![enter image description here](https://images.slideplayer.com/19/5806117/slides/slide_2.jpg)
> Photo Source: Slideplayer.com
## Table of contents

* [Introduction](#introduction)
* [Implementation & Design Choices](#implementation-&-design-choices)
* [Evaluation](#evaluation)
* [Responses to Discussion Questions](#responses-to-discussion-questions)
* [Conclusion](#conclusion)

## Introduction

Map coloring is a constraint satisfaction problem, an interesting one at it. So is circuit board laying, word-search, 8-queens, etc. As with PA3, I'd love to give a more fun and exciting introduction to the CSP AI assignment I worked on, but looking at how I'm running out of time, I'd just go straight to the point and make this report short and sweet. To that end, I'm only going to talk about three key things. First, the implementation and design choices I made for CSP, Map_Coloring, Circuit_Board (and an extra credit Word_Search) CSP problems, as well as the inference and heuristics implementations. 2nd, I'm going to give a brief (but thorough) walk-through of my evaluation over my algorithm, giving results to show that everything work as planned. 3rd, I'm going to respond to some discussion questions posted in the task portion of the assignment.

## Implementation & Design Choices

In this assignment, I used 4 total python files:
- CSP.py
- Map_Coloring_CSP.py
- Circuit_Board_CSP.py

Extra credit:
- Word_Search_CSP.py

CSP.py is the backbone of the AI assignment. As discussed in a later section ([Responses to Discussion Questions](#responses-to-discussion-questions)), I created generic classes Constraint and CSP in order to let the subclasses use its own value types and not having to convert everything to integers. The abstract method satisfied will be overridden by the subclasses in order to better suit the subclass problem we are facing. The satisfied method in the Constraint class is called in the consistent method in the CSP class, to individually check whether the new assignment satisfies the constraints. In the CSP class itself, we have the majority of functions and methods. The constructor takes in variables list and domains dictionary, and create the constraint and domain dictionary. The Add_Constraint functino help the csp build its internal constraints. The consistent method uses the constraints and the satisfied method to return whether the new assignment value allows the overall assignment to be consistent. The Select_Unassigned Variable helper function takes in a heuristics_1_type (well, they all take in everything, just so that the machine don't get confused which one is for which), and determine which heuristic/way of selecting unassigned variable to use. This is where I implement my MRV and Degree heuristics. A default one is also present. Similarly, Order_Domain_Value is similar in that it takes in heuristics_2_type to determine whether to use the LCV heuristic or just the default. Here's where I implement the LCV. While the implementation of MRV and Degree is rather straightforward, my implementation of LCV is quite convoluted (well, the heuristic itself intrinsically demands so). LCV prefers the value that rules out the fewest choices for the neighboring variables in the constraint graph, thus I must find a way to count up all the choices the neighboring variables in the constraint graph have, store them into a dictionary, sort it, and return a list of domain that rules out the fewest choices for neighboring variables. I created N_conflict() and Neighbors() helper methods to help this endeavor. N_conflict returns the number of other variables that conflict with var == val. This helper method itself also have 2 helper method (conflict() and count_matching()), to further help the function. Neighbor() just return all of the parameter (variable)'s neighbor by constraints. The Inference method uses AC3 for help, and for the AC3 I pretty much takes after the textbook's implementation, with the Revise helper method and everything. I did created another helper method Neighbors_Without_Xj(self, Xi, Xj) to return the neighbors of Xi that rules out Xj for this. And then, of course, we have the Backtracking_Search() main method. My backtracking method takes in everything from assignment to Inference(bool), heuristics_1_type and heuristics_2_type for easy enabling and disabling. I keep track of the number of backtracking calls by tallying them up there in csp. The base case happens when the assignment is complete, when every variable is assigned; to that end I checked whether the length of assignment is equivalent to that of its variables. Next, we get the variable from the function Select_Unassigned_Variable. Then, from Order_Domain_Values, we get each new value to check over, assigning them into the assignment and then check consistency. If it's not consistent, we remove it and go on to the next value. Note, at this point I also have the "use_inference == True" check point, this happens when we want to check inference. The other mode is the mode that I first created which I was sure to be 100% correct. In the inference mode, we go on and check inference (which is AC3 in our case - which proved to not be that helpful), and then we call on the backtracking method recursively. If result is never None, we return the result up.

In Map_Coloring_CSP I flesh out the satisfied method by making sure that it will return true if the assignment so far still satisfies the constraint, but return false if it doesn't. (The constraint being that no two linked variable should have the same assigned domain value. I created helper function Australia_Map_Reset() and Europe_Map_Reset() to reset the Add_Constraint calls more elegantly in the main testing runs. In the main test, I created the variable list and the domain dictionary, pass that into a CSP, call Backtracking_Search to get a solution. And then print Statements to show the results.

For Circuit_Board_CSP it's a little bit more complicated. I created a GridLocation class with generic class attributes row and column for easy storing of a tile's location. Generate_grid helps generate the base grid with ".". Display_grid prints out the grid. The generate_domain function takes each component and the grid and generate all possible locations for that component, storing all of each possibility of the list of GridLocation (of each component) into another list and returned as domain. Then, we have the ComponentSearchConstraint class that overrides the original Constraint class in CSP, and updated its Satisfied method just like Map_Coloring_CSP; except this time, it checks all the locations in the assignment and see whether there's any tile in them that overlaps another. (We don't want any component to overlap another). I compare the size of a set and the list to see whether there's duplicates, using the benefit of a set which takes out repeated values. 

The extra credit Word_Search_CSP is very similar to Circuit_Board_CSP. All that's different is when I generate the grid, I generated it with random alphabets, also the generate_domain function is also different in that I generated locations where only 1 line would fit, but it can also be in diagnal positions. The words will also have a random chance (40%) to flip, because in a word search problem a flipped word can still work. A word search problem can actually make use of repeating tiles (with the same letter), however, due to lack of time, I did not implement that.

Due to time constraints, please take a look at the code itself to further get a further taste of my implementation and design choices.

## Evaluation

Here I give a little example of all three as I run my code. First, we have all the results of the Map_Coloring_CSP:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%204/Map%20Coloring%20CSP%20Results%201.png" height="100%" width="100%">
</p>

First, we have the simple test with Australia's 7 regions and 3 colors with no additional help from heuristics and inferences. Our results is a humble 8 backtracking calls and 12 color values that it checked (8,12) . With only 2 colors, it checked more values but less calls before it determines there's no solution (7, 14). Inference doesn't seem to add any efficiency, as does the MRV heuristics. However, this makes sense because in the textbook it mentioned that in map_coloring problems they don't really have an impact. For the degree heuristic test, it may seem that it actually checked more (15) values and is therefore less efficient, we'll see this is not generally the case as the problem gets large, as shown later in the Europe map example. The LCV heurisitc gets us the most efficient result with 8 calls and 11 values that were checked. Clearly the least constraining value heuristic help us order the domain values better thus we checked a few numbers (1) less of the values. 

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%204/Map%20Coloring%20CSP%20Results%202.png" height="100%" width="100%">
</p>

The next few results share the same phenomenon, with the LCV heuristic helping us reducing the values that we checked. When Degree and LCV heuristic work together, however, it worked magic. It only checked 9 values before solving the problem. The Degree heuristic helps assign the unassigned variable better by choosing the variable with most connected degrees to be assigned first, in combination with LCV it helps reduce the values to be checked greatly. Interestingly, in test 10 we see the Inference actually dragging the results down by 2 values. Next, we have the major European countries coloring map csp. The base solution gives us 48 calls and 112 values checked. The failed solution checked 16252 calls and 48757 values before it gave up.

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%204/Map%20Coloring%20CSP%20Results%203.png" height="100%" width="100%">
</p>

With everything turned on, (choosing Degree instead of MRV), we reduced the calls to 41 calls and checked only 57 values. With only MRV and LCV, we have (48, 88). For only MRV, we have (48, 112), so clearly this didn't help. With only Degree, we have (41, 89), which is a much better improvement for both calls and values. In fact, the Degree heuristic allows the calls to be checked at only 41 calls. Our winning combination of Degree and LCV yields us the 41 calls and 57 values, clearly the LCV handled the values well while the Degree drive us home both the values and calls.

Then, we have the circuit board layout constraint satisfaction problem's results:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%204/Circuit%20Board%20CSP%20Results.png" height="100%" width="100%">
</p>

In the first test, the default problem (3 x 10) is solved with only 5 calls and 23 value checks. In the failed test (adding 1 more component to it), we can see the program called 105 backtracking calls before giving up, and 1216 values were checked. For a bigger problem (5 x 12), the problem is solved in 7 calls and 97 values were checked. The next test is the longest yet, it took about 8 seconds before solving the problem with 8983 backtracking calls and 552417 values checked. This is due in large part of the h component being so late in the checking board and such a hard part to fit. 

Then, there's the very similar csp Word Search:

<p align="center">
  <img src="https://github.com/timothyyang21/Tim_AI_class/blob/master/AI%20Assignment%204/Word%20Search%20CSP%20Results.png" height="100%" width="100%">
</p>

In the first test with 9 fruits in a board of 10 x 10, the problem called 10 backtracking calls and checked 101 values before returning a board in which all the words are scrambled and fit in. In the second test with 19 countries (now to think of it I could easily squeeze in UK to make it 20), we have 20 calls and 1453 value checked before finding the result. 

I have further verified my code by breaking my code down, adding print statements, and checking the sequences individually. I will end my discussion of evaluation now and start with the responses to discussion questions. Thank you!

## Responses to Discussion Questions

The only discussion questions listed in problem assignment is for the circuit-board layout problem. The first question asks:

> In your write-up, describe the domain of a variable corresponding to a component of width w and height h, on a circuit board of width n and height m.  Make sure the component fits completely on the board.

I implement the circuit-board layout problem a bit differently than the PA4 assignment hints suggest. For the domain of a variable corresponding to a component of width w and height h on a circuit board of width n and height m, I implement it as such: a list of list of a class called GridLocation, which contains the individual tile location on the board that together contains the whole component itself. The outer list is the list of all the possible reiterations of where the components could be. The inner list is the list of GridLocations that together resembles the component itself. Together with the outer list and inner list, we may depict the domain of a variable corresponding to a component of width w and height h.

Another discussion problem asks:

> Consider components a and b above, on a 10x3 board.  In your write-up, write the constraint that enforces the fact that the two components may not overlap.  Write out legal pairs of locations explicitly.

There are a total of 48 of them, however, since my implementation actually never uses the "pairs of locations" technique suggested by the problem, I cannot offer them in short time and I fear they may be incorrect, I will still give it a try, though. My implementation enforce the constraint (checking satisfaction) by checking if there are any duplicated grid locations among all the different components' locations by using a set. The constraint that enforces the fact that the two components may not overlap is simply that no two components cannot have any of its component to mutually occupy a same spot. Let (0,0) be for component a (the starting bottom left spot), and let (3,0) be for component b, then a legal pair is (0,0)-(3,0). (By the way, it's all in this form: (x,y).) Here are all of them that I can spit out in the short time that I have for this report: (0,0)-(3,0), (0,0)-(3,1), (0,0)-(4,0), (0,0)-(4,1), (0,0)-(5,0), (0,0)-(5,1), (0,1)-(3,0), (0,1)-(3,1), (0,1)-(4,0), (0,1)-(4,1), (0,1)-(5,0), (0,1)-(5,1), (5,0)-(0,0), (5,1)-(0,0), (6,0)-(0,0), (6,1)-(0,0), (7,0)-(0,0), (7,1)-(0,0), (5,0)-(0,1), (5,1)-(0,1), (6,0)-(0,1), (6,1)-(0,1), (7,0)-(0,1), (7,1)-(0,1), (1,0)-(4,0), (1,0)-(4,1), (1,0)-(5,0), (1,0)-(5,1), (1,1)-(4,0), (1,1)-(4,1), (1,1)-(5,0), (1,1)-(5,1), (2,0)-(5,0), (2,0)-(5,1), (2,1)-(5,0), (2,1)-(5,1), (6,0)-(1,0), (6,0)-(1,1), (7,0)-(1,0), (7,0)-(1,1), (6,1)-(1,0), (6,1)-(1,1), (7,1)-(1,0), (7,1)-(1,1), (2,0)-(5,0), (2,0)-(5,1), (2,1)-(5,0), (2,1)-(5,1), (7,0)-(2,0), (7,0)-(2,1), (7,1)-(2,0), (7,1)-(2,1), and I think that's all of them.

The final discussion question asks:

> Describe how your code converts constraints, etc, to integer values for use by the generic CSP solver.

Because my implementation never converts constraints or other things to integer values for use by the generic CSP solver, I will disregard this question and describe how my implementation is designed to be generic in the first place. For the CSP class, I created a base class Constraint that takes in generic types of variables and domains. Moreover, it's got abstract method satisfied that must be implemented by other subclasses. I then created those subclasses that override this abstract method and build upon the base generic Constraint class. In this way, I may use the value type that's the most intuitive to use for those subclasses well into my CSP problem, and help out a lot with debugging, among other things. The CSP class itself takes generic inputs as well.

## Conclusion

Constraint Satisfaction Problems were actually quite fun! The general-solving algorithms is indeed quite intriguing -- there are so many different kinds of intresting problems to solve, after all! I look forward to the next AI adventuer! :sparkles:
