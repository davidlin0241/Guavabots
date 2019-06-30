Problem¶
Your group runs an autonomous food delivery company called Guavabot. Everything is going great - you just raised $10 million in VC funding, and you've deployed to three locations worldwide - Singapore, Tel Aviv, and Toronto. Unfortunately your intern ran rm -rf / on your production servers, losing the locations of all your bots! The bots took quite a while to develop and the prototypes are valuable, so you want to locate them and move them home. Thankfully, you have a worldwide network of students who report the locations of the bots to you via app, but these customers are not always right and may incorrectly inform you about bots' locations.

Statement¶
You have some number of bots lost in the city. Your goal is to find these bots and move them home in the shortest time. You have two operations, both that take some time: remote and scout. A remote moves all bots from a vertex to a neighbour of that vertex. A scout sends a student to a vertex and has the student report if a bot is there.

Formally, your delivery system is for a specific city, i.e. a connected weighted undirected graph 
G
=
(
V
,
E
)
such that all edge weights 
w
e
>
0
, with a designated home vertex 
h
∈
V
. The Guavabots live at a set of locations 
L
⊆
V
 with exactly one Guavabot in each starting location; but you do not know this set of starting locations. In addition, you have a set of 
k
 students that you can send to vertices on the graph to inform you if Guavabots are present there; however, the students are unreliable and are incorrect on a fixed, unknown subset of vertices.

You are given and know the graph 
G
, edge weights 
w
e
, home vertex 
h
, number of Guavabots 
|
L
|
, and the number of students 
k
. All graphs are complete, have 
|
L
|
=
5
 bots, 
n
=
100
 vertices, and 
k
∈
{
10
,
20
,
40
}
 students.

You do not know the starting locations of the Guavabots 
L
 or the opinions of the students (until you ask for them by scouting).

Actions¶
You get the bots home by performing a series of actions of your choice. Your available actions are:

scout on 
(
i
,
v
)
, where 
i
 is a student and 
v
∈
V
. In this case, student 
i
 visits the vertex and reports to you whether they see Guavabots at the vertex (they give this as a yes/no answer). However, their report may be incorrect. You have to wait for the student to move to the vertex and report you the answer, which takes 1 time.

For more information on how student reports work, see students.

remote on the directed edge 
(
u
,
v
)
, provided that 
{
u
,
v
}
∈
E
. This tells you how many Guavabots there were at 
u
 that received the command, then all the Guavabots at 
u
 move to 
v
. Regardless of the presence of Guavabots at 
u
, it takes 
w
u
v
 time to wait and see if your command has moved any Guavabots.

As the graph is undirected, you can remote along either direction on any edge.

You can only perform one action at a time, and you cannot "undo" an action.

We call an instance of this problem and the sequence of actions you take a "rescue".

Constraints¶
After you remote along an edge 
{
u
,
v
}
 in the direction 
u
→
v
, any future scouts on 
u
 or 
v
 will fail (as you already know about the existence of Guavabots at either vertex). The only exception is if no bots were remoted along the edge, in which case you can scout on 
v
 in the future.
You cannot perform a scout at the home vertex 
h
, ever.
Students¶
Each student 
i
 can give you a report on any vertex so long you haven't used it in a remote already. If a student is incorrect on a vertex, they will always report the opposite of the truth; students are incorrect a fixed, but unknown number of times. This means that for each student 
i
, there is a fixed, unknown subset 
S
i
⊆
V
 of vertices that they will be wrong at. These sets 
S
1
,
…
,
S
k
 are fixed for the input, meaning they are completely determined before your rescue even starts; in other words, the student opinions will not change depending on how your algorithm runs.

There are no guarantees on how correct the students will be beyond the fact that 
|
S
i
|
≤
|
V
|
/
2
 for all students 
i
. It is up to you to come up with an algorithm that uses the student reports as effectively as possible.

Time¶
Time is measured as follows:

When you start, the current time is zero.
There is a fixed time every scout action takes; after every scout your time is incremented by 1.
After a remote on the edge 
{
u
,
v
}
, your time is incremented by 
w
u
v
, regardless of the direction you remote on 
{
u
,
v
}
The scout cost 
1
 is much smaller than most edge weights 
w
e
.

Goal¶
You are given a score once you end the rescue. If the time taken so far is 
t
 and the number of Guavabots that are home is 
g
, this score is:

100
|
L
|
+
1
(
g
+
α
α
+
t
)
where 
α
∼
10
3
 is a constant among all inputs which is yet to be determined, but is within an order a magnitude of 
10
3
 and is about the raw time of an average solver that gets all the bots home.

You want to get as high a score as possible. The minimum score for any instance is 
0
, and the maximum score for any instance is 
100
 (neither side is necessarily attainable, your score will fall in between).

This scoring function was designed such that returning more bots is always better than spending less time. For example, say that we have 5 bots scattered on a graph:

If we return 5 bots and take 10000 time, our score is 
≈
84.8
If we return 2 bots and take 100 time, our score is 
≈
48.5
If we return 1 bot and take 10 time, our score is 
≈
33.17
Another way to see it: treat your score as a tuple 
(
g
,
t
)
. Scores will be ordered by 
g
 descending first, then by 
t
ascending.


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
