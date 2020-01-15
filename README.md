Group project with ericgao4 and rafaelProjects

Problem¶
Your group runs an autonomous food delivery company called Guavabot. Everything is going great - you just raised $10 million in VC funding, and you've deployed to three locations worldwide - Singapore, Tel Aviv, and Toronto. Unfortunately your intern ran rm -rf / on your production servers, losing the locations of all your bots! The bots took quite a while to develop and the prototypes are valuable, so you want to locate them and move them home. Thankfully, you have a worldwide network of students who report the locations of the bots to you via app, but these customers are not always right and may incorrectly inform you about bots' locations.

Statement¶
You have some number of bots lost in the city. Your goal is to find these bots and move them home in the shortest time. You have two operations, both that take some time: remote and scout. A remote moves all bots from a vertex to a neighbour of that vertex. A scout sends a student to a vertex and has the student report if a bot is there.

Full problem description: http://guavabot.cs170.org/problem/

## Algorithm 

### Algorithm 1 - Main Idea:
Calculate Shortest path for all nodes , v -> H in graph G using Dijkstra's.
Create list called remoted_nodes (to record the nodes that you know have used remote(u,v)  on (add u and v ).
Create list called majority_false which contains all the nodes that the majority of votes returned false when scout was used on that node. 
Create a list of size 2 tuples that contains all students called students, each tuple is in the form: (student_number, incorrect_votes). All students start in students with trust of 1. We can discover when a student tells us incorrect information when:
Call Scout m using all K students:
If majority of student’s claimed true but then call remote(m,u) and remote returns 0. In this case the majority of students were wrong. All students that were incorrect update s.incorrect_votes += 1. 
If majority of student’s claimed true and then call remote(m,u) and remote returns 1. In this case the majority of students were correct. Do the same update as in i.
Sort |V - 1| nodes (do not include H) from largest to smallest in terms of shortest path. Use a max-queue and call this queue max_queue.
Pop a node m off max_queue (max node by shortest path, arbitrary if paths are of equal length). Check if m is in the list called remoted_nodes.
If m is in remoted_nodes then that means there are already guava bots there, thus remote from m to its neighbor that is contained in the shortest path from m->H.
If m is not in remoted_nodes then that means you do not know if there are guava bots there. Run scout(m) using all k students if there does not exist a student that is incorrect |v|/2 times.	
If the majority of student votes say that there is a bot there then perform step a (the remoting step). Update all student’s incorrect_votes appropriately.
If there exists a student, s, in list students that has incorrect_votes == v/2. Then stop using all k students for each scout on node m. Instead use scout once with this student s because you know that this student will be 100% correct on all other nodes.
Else place m into list majority_false.
Perform 6 until max_queue is empty.
Calculate how many guava bots were routed to H from the neighbors of H. Call this value routed_h.
L represents the total number of guava bots present in the graph. L -= routed_h
If L = 0: Success.
Else: Place list majority_false into the empty max_queue. 
Pop node m off max_queue and remote(m,neighbor in shortest path)
Then add neighbor in shortest path of m->H to max_queue.
Repeat until max_queue is empty. Note if neighbor in shortest path of m is H or is already in max_queue. If so, do not add to max_queue.

### Algorithm 2 - Main Idea: 
Very Similar to Algorithm 1, except use (Kruskal or Prims) to calculate Minimum Spanning Tree. Then keep a list of size 2 tuples that contains all students called students, each tuple is in the form: (student_number, vote_value). All students start in students with trust of 1. We can discover when a student tells us incorrect information when:
1. Call Scout m using all K students:
If majority of student’s votes (sum of all students vote_value) claimed true but then call remote(m,u) and remote returns 0. In this case the majority of students were wrong. Record this information by setting the vote_value of each student that responded incorrectly to vote_value *= x, when x ranges from 0,1. This is multiplicative decrease the voting power of incorrect students. Set vote_value of each student that responded correctly to vote_value += 1 (Additive Increase). 
If majority of student’s votes (sum of all students vote_value) claimed true and then call remote(m,u) and remote returns 1. In this case the majority of students were correct. Do the same changes to vote_value done in part a for correct and incorrect students.
Note: The strategy is based off of TCP congestion control congestion avoidance phase because incorrect estimation is very expensive (some interesting parallels).
Everything else is is similar (using queue and shortest path) just replace graph G with MST.
Questions
Look at algorithm 1: sequence of actions: calculate shortest paths v -> H for all v in |V|, Place vertex in max_queue, Pop off and remote/scout as described by algorithm.
Same as 1.

### Algorithm used for the project:
Consider algorithm 1, create new dictionary called original_location that contains {vertexId, bot_value} pairs where bot_value is initialized to be 0. 
When a remote is performed from node u ->v,  add remote(u,v) [a value] to the bot_value of u and then subtract remote(u,v) from the bot_value of v. (All the vertexIds that contain a bot_value of 1 are the original starting locations of the bots.) 
Check if any node in original_location has a bot_value == -L (L being the number of total guava bots). If this is the case go directly to part d. This means that L bots were routed to this empty location and since there are only L bots in the graph, then every location not remoted from does not contain a bot and no more remoting or scouting is needed.
Perform b and c everytime a remote is called in algorithm 1.
After algorithm 1 is done running. All the nodes in original_location that have bot_value = 1 are the nodes that contained a bot initially. 
