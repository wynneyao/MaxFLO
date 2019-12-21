## Summary
Our project focused on parallelizing and evaluating the performance of two commonly used algorithms for solving the maximum flow problem on graphs, Dinic’s and Push-relabel. We created sequential and shared-memory (OpenMP) parallel versions of the algorithms. To enhance the performance of the parallel algorithms, we also integrated lock-free updates to certain variables and concurrent data structures (using the thread building block library in C++). Our evaluation of the algorithms involved running the algorithms on realistically-sized graphs and comparing the speedup of the parallel implementation to the sequential.  
You can find our final paper [here](https://docs.google.com/document/d/1_dfUpSdmUni91AaYEJRZ0LUOZwGeUtZY3ZUYwjEe1kA/edit?usp=sharing).

## Background
Applications of max flow include the project selection problem, image segmentation, airline scheduling, circulation-demand problem, and more. Because of the variety of problems that can be solved by finding a max flow through a network, we found this problem interesting. Furthermore, the benefits of optimizing max flow would lead to faster algorithms for other real world problems.

## Challenge
The most common challenge that researchers faced when implementing parallel graph algorithms is a lot of existing max flow algorithms are sequential by nature, making parallelism a challenge. Also, graph problems are often memory bandwidth bound because of the rather random accesses of memory. Furthermore, because of this bound on bandwidth, there is a low computation to communication ratio, making it a difficult problem on which to optimize performance. One challenge would be to figure out some way to access memory such that there is greater locality (i.e., avoiding having a large amount of page faults or cache misses by using sharding).

## Resources
We will need a 16-core hyper-threaded CPU, probably by using an AWS EC2 instance, so we would need credit for this. We will not use any existing codebases, but will use the pseudocode for the algorithms from GeeksforGeeks articles as well as from these following papers: Munakata & Hashier [1] for the genetic algorithm and Baumstark Blelloch and Shun [2] for Push Re-label. We plan to use the evaluation workload used in Baumstark et. al.’s paper to evaluate our parallel and sequential implementations on realistic input graphs.

## Platform Choice
The machines that we are planning to use are an 8-core CPU, a 16-core CPU and possibly a GPU. The workload we plan to use for evaluating our parallel algorithms will be the same as the ones used in the Baumstark et. al.’s paper which was evaluated on 2-32 threads. Since, many times graph algorithms are memory bandwidth bound we are also thinking of implementing some our parallel algorithms in CUDA to run them on higher memory bandwidth GPUs. We plan to implement the sequential and parallel algorithms in C++ since we will have access to parallelization libraries like OpenMP.

## Goals 
What we plan to achieve is to parallelize the 2 algorithms and get a significant speedup over the sequential versions. We also want to generate graphs comparing them to each other and to existing implementations. We plan to test our implementations on a large benchmark of DIMAC-formatted graphs, used in Blelloch’s paper, to evaluate our sequential and parallel implementations on real-sized graphs.
If we finish our initial plan with enough time to spare, then a further goal that we hope to achieve would be to implement the parallel algorithms in a different programming model (i.e. MPI), use different optimized data structures in our algorithms (i.e. dynamic trees for Dinic’s), explore fine-grained locking, and/or try lock free implementations and compare their speedups as well. 

## Schedule 
![NewSchedule](NewSchedule.png)

# Algorithm Descriptions 

## Dinic’s
Dinic’s algorithm is made up of 2 main steps:
(1) BFS: Labels each vertex of the residual graph with its shortest distance from the source node (i.e. the level of the node). 
(2) sendFlow: Pushes as much flow as possible from the source node to a neighboring 1st level node, then from the 1st level node to a neighboring 2nd level node … until the flow reaches the sink node of the residual graph. It repeats this process pushing as much flow from the source as possible.
These two steps are repeated until the BFS returns false because the sink node is unreachable from the source node in the residual graph at which point maximum flow has been pushed from the source to the sink.

## Push-Relabel
The original serial version of push-relabel involves first sending out a preflow, which initially sends out flow equal to the capacities of the edges from the source to the source’s neighbors. Vertices in push-relabel have heights. These heights reflect how far away from the sink the vertex is. Each vertex can also have an excess amount of flow. Vertices that have nonzero excess flow are called active vertices. From the active set, a vertex is chosen. It can only push onto its neighbors that have a height 1 less than its own height. We call the edge between the active vertex and such a neighbor an admissible edge. If the active vertex doesn’t have any neighbors on which to push, the vertex’s height will be relabeled to be the minimum of its neighbors heights plus 1 (so that the next round, it can push somewhere). The active vertex set updates according to the updated flows, and this process continues until there are no more active vertices. The output is the amount of excess flow the sink has.  

We based our parallel implementation off of the paper by Baumstark, et. al. The parallel high level algorithm involves first doing the preflow, same as before. For one iteration, the active vertices are processed in parallel and possible pushes are performed. Each of the active vertices has most if not all of its excess flow pushed out in one iteration. However, the modified excess flows are saved separately and not applied until the end. New labels are computed in parallel but not applied until the end of the iteration. At the end of processing all the active vertices, the new labels and the new excess changes are applied and a new active set is created before the next iteration. After a few iterations, a global relabel is called, which uses a reverse BFS to assign the vertices heights that mirror their distances from the sink again. If the working set is empty, the algorithm stops and outputs the sink’s excess flow as the max flow. 

# Checkpoint

## Work Completed So Far
We have implemented the 2 sequential algorithms and have a detailed understanding of how to proceed with parallelizing push-relabel. Additionally, we have a sequential implementation of the Genetic Algorithm for solving the max flow problem, that we have ready to use for one of our reach goals. We have also tested and collected timings on our two sequential algorithms on large Delaunay graph test cases (up to 4096 vertices and 12264 edges). 

## Preliminary Results
We tested our sequential implementations on a few medium sized, delaunay graphs with the number of edges ranging from 3056 to 12264. We used delaunay graphs since these were the graphs used in the Blelloch paper and ultimately we want to compare our results to this paper’s. It appears as though Dinics performs pretty well on medium sized graphs, which makes sense since Dinic’s is an optimized sequential maxflow algorithm. Push-relabel on the other hand seems to struggle with larger sized test cases, with a time of almost 5 minutes for the graph with 12k edges. We predict the reason for this is because push-relabel is inherently parallelizable so we will see benefits of the implementation once we add parallelization to it. 

![PreResults](PreResults.png)




