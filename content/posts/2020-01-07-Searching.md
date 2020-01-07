---
title: "Searching Methods - AI: A Modern Approach Notes"
date: 2020-01-07T13:02:01-05:00
draft: false
---

## Abstract

This article is my summary of the famous book, Artificial Intelligence: A Modern Approach Chapter 3.

## Tree Search and Graph Search

Tree search expands all child nodes and adds them into frontier even some of them have been visited.

Graph search maintains a explored list where stores all visited nodes, it only adds unexplored nodes into frontier.

## Uninformed Search

There are many types of uninformed search algorithms:

### Step-cost Problem

This kind of problems have the same step cost, we only need to find the minimal steps from initial state to goal state.

#### Breath first search: 

This method uses a FIFO (first-input-first-output) queue to implement. And is not sensitive to the choice between tree search or graph search.

##### Advantage of BFS:

* It can find the optimal solution naturally without generating overhead nodes.
* It generates the fewest nodes before finding a solution for step-cost problem.

##### Distadvantage of BFS:

* It needs to store all the nodes on last layer.
 
#### Depth first search:

DFS can be implement by two ways, one is using a LIFO (last input first output) stack, the other is calling `child()` function recursively.

Natural DFS under tree search condition is incomplete, because the depth is infinite.

##### Advantage of DFS:

* It frees node memory once it finishs digging a branch, all the nodes on the branch can be deleted. So it can run on a memory-limited machine.

##### Disadvantage of DFS:

* It generates excessive nodes compared with BFS.

#### Varaints of DFS:

##### Backtracking search

It is more memory economical. Every generated node on searching tree has a successor information on its depth, so we only need to store *m* nodes (m equals the depth of the frontier) at any time.

It is suitable when a node is super large in memory.

##### Iterative deeping depth-first search

It is simply a DFS with depth limitation. After each episode of IDS, the depth limit increases by 1.

Two types of return result: 

* cutoff: Current episode reached depth limit.
* failure: Current episode ends before reaching depth limit.


### Uniform-cost Problem

Uniform-cost problems have a postive cost for each step, however costs for different steps varys.

#### Uniform-cost search

It is origin from the two-point shortest-path algorithm of Dijkstra.

It relys on a priority queue, instead expanding randomly, it always expands the lowest path cost node on the present searching tree.

Two modification:

* The goal test is applied to a node when it is selected for expansion (fetched from the priority queue). This seems inefficient but it is essential. Once a node is generated, we cannot ensure that it will be pushed back to the top of priority queue, but the second time we met it, it must be the optimal solution, beacuse the path cost can only increase in uniformed cost problem case.

* The second modification is similar to dynamic programming, a test is added and when we enter the same state multiple times, rather than storing all of the nodes, we only store the best node that reach this state. 

### Bidirection search

Bidirection search is based on the simple math:

$$2\ast branch^{depth/2} \quad  \text{is much less than} \quad branch^{depth}$$

It needs a test to check whether the frontiers of two searches intersect.

But calculating the predecessors needs substantial ingenuity.

## Informed (Heuristic) Search

The difference between informed and uninformed search is whether we can find a heuristic function.

$$h(n)=\text{heuristic function of a node}$$

### Greedy best-first search

It chooses to expand the node that is most likely to lead to a solution. In two-point shortest route finding problem, the heuristic function is the 2-norm between the current address and goal address.

In this case:

$$h(n)=norm(n,goal)$$

### A\* search

Unlike uniform-cost search's only considering path cost from start, and greedy best-first search's only considering heuristic function value. A\* combines them both.

$$f(n)=g(n)+h(n)$$

#### Variants of A\* search

The biggest problem of A\* is that A\* stores all the nodes before finding a optimal solution.

So the variants of A\* mainly optimize the memory efficiency.

##### Iterative-deepening A\* search

Cutoff *f*-cost is the smallest *f*-cost node that exceeded the cutoff on the previous iteration.

Honestly, iterative deepening search only works efficiently for BFS, algorithms like iterative-deeping depth first search or iterative-deeping depth A\* search all suffer from the cost of repeating expanding upper layers in each iteration.

##### Recursive best-first search (RBFS)

If a node's children all cost more than a node's peer, we can just label the node with the best children cost then delete all the decendants, then move the peer node.

##### Memory-bounded A\* (MA\*) and simplified MA\* (SMA\*)

SMA\* behaves same as A\* before exhausting all the memory, after no node can be added to the memory, it removes the worst node in the tree and adds new node.

### How to choose heuristic function

* Generating admissible heuristics from relaxed problem.
* Generating admissible heuristics from subproblems.
* Learning heuristics from experience.

#### Parallel search

Not covered but useful for computer cluster.
