# Breadth First Search
## AIM

To develop an algorithm to find the route from the source to the destination point using breadth-first search.

## THEORY
Explain the problem statement

## DESIGN STEPS

### STEP 1:
Identify a location in the google map:

### STEP 2:
Select a specific number of nodes with distance
### STEP 3:
Import the required packages and create some methods to define and understand breadthfirstsearch.

### STEP 4:
Include the nodes(locations) and values(distance) in dictionary as keys and values.

### STEP 5:
Pass the required location it will return the distance and destination.



## ROUTE MAP

![a](https://user-images.githubusercontent.com/75264748/167461272-ff4f76e5-c42e-4ed3-88e0-ea647f24d8e6.jpg)




## PROGRAM
```
Student name : Dhanasekar G
Reg.no : 212220230009
```
```python
%matplotlib inline
import matplotlib.pyplot as plt
import random
import math
import sys
from collections import defaultdict, deque, Counter
from itertools import combinations

class Problem(object):
   def __init__(self, initial=None, goal=None, **kwds): 
       self.__dict__.update(initial=initial, goal=goal, **kwds) 
       
   def actions(self, state):        
       raise NotImplementedError
   def result(self, state, action): 
       raise NotImplementedError
   def is_goal(self, state):        
       return state == self.goal
   def action_cost(self, s, a, s1): 
       return 1
   
   def __str__(self):
       return '{0}({1}, {2})'.format(
           type(self).__name__, self.initial, self.goal)
           
class Node:
   def __init__(self, state, parent=None, action=None, path_cost=0):
       self.__dict__.update(state=state, parent=parent, action=action, path_cost=path_cost)

   def __str__(self): 
       return '<{0}>'.format(self.state)
   def __len__(self): 
       return 0 if self.parent is None else (1 + len(self.parent))
   def __lt__(self, other): 
       return self.path_cost < other.path_cost
       
failure = Node('failure', path_cost=math.inf) 
cutoff  = Node('cutoff',  path_cost=math.inf)

def expand(problem, node):
   "Expand a node, generating the children nodes."
   s = node.state
   for action in problem.actions(s):
       s1 = problem.result(s, action)
       cost = node.path_cost + problem.action_cost(s, action, s1)
       yield Node(s1, node, action, cost)
       

def path_actions(node):
   "The sequence of actions to get to this node."
   if node.parent is None:
       return []  
   return path_actions(node.parent) + [node.action]


def path_states(node):
   "The sequence of states to get to this node."
   if node in (cutoff, failure, None): 
       return []
   return path_states(node.parent) + [node.state]
   
FIFOQueue = deque

def breadth_first_search(problem):
   "Search shallowest nodes in the search tree first."
   node = Node(problem.initial)
   if problem.is_goal(problem.initial):
       return node
   frontier = FIFOQueue([node])
   reached = {problem.initial}
   while frontier:
       node = frontier.pop()
       for child in expand(problem, node):
           s = child.state
           if problem.is_goal(s):
               return child
           if s not in reached:
               reached.add(s)
               frontier.appendleft(child)
   return failure
   
class RouteProblem(Problem):
   """A problem to find a route between locations on a `Map`.
   Create a problem with RouteProblem(start, goal, map=Map(...)}).
   States are the vertexes in the Map graph; actions are destination states."""
   
   def actions(self, state): 
       """The places neighboring `state`."""
       return self.map.neighbors[state]
   
   def result(self, state, action):
       """Go to the `action` place, if the map says that is possible."""
       return action if action in self.map.neighbors[state] else state
   
   def action_cost(self, s, action, s1):
       """The distance (cost) to go from s to s1."""
       return self.map.distances[s, s1]
   
   def h(self, node):
       "Straight-line distance between state and the goal."
       locs = self.map.locations
       return straight_line_distance(locs[node.state], locs[self.goal])
       
class Map:
   """A map of places in a 2D world: a graph with vertexes and links between them. 
   In `Map(links, locations)`, `links` can be either [(v1, v2)...] pairs, 
   or a {(v1, v2): distance...} dict. Optional `locations` can be {v1: (x, y)} 
   If `directed=False` then for every (v1, v2) link, we add a (v2, v1) link."""

   def __init__(self, links, locations=None, directed=False):
       if not hasattr(links, 'items'): # Distances are 1 by default
           links = {link: 1 for link in links}
       if not directed:
           for (v1, v2) in list(links):
               links[v2, v1] = links[v1, v2]
       self.distances = links
       self.neighbors = multimap(links)
       self.locations = locations or defaultdict(lambda: (0, 0))

       
def multimap(pairs) -> dict:
   "Given (key, val) pairs, make a dict of {key: [val,...]}."
   result = defaultdict(list)
   for key, val in pairs:
       result[key].append(val)
   return result
   
Thiruthani_to_SaveethaMedical = Map(
    {('tiruttani', 'arcord'):  12, ('arcord', 'Narayanapuram'): 15, ('Narayanapuram', 'Tirupachru'):31, ('Tirupachru', 'kadambathur'): 5, ('Tirupachru', 'veera ragava perumal  kovil'): 3, 
     ('veera ragava perumal  kovil', 'tiruvallur'): 3, ('tiruvallur', 'veppambaattu'):  13, ('tiruvallur', 'kadambathur'): 7, ('kadambathur', 'manaur'): 8, ('manaur', 'arakkonam'): 17, 
     ('arakkonam', 'itchiputhur'): 9, ('itchiputhur', 'tiruttani'):  6, ('arakkonam', 'Bangamainkadi'): 12, ('Bangamainkadi', 'pichipakkam'): 13,
    ('pichipakkam', 'koolangeri'): 10, ('koolangeri', 'Sengadu'):  9, ('Sengadu', 'valarpuram'): 8, ('valarpuram', 'perumalppattu'): 15, ('perumalppattu', 'tiruvallur'): 10, 
    ('valarpuram', 'Saveetha Medical College'): 8, ('Saveetha Medical College', 'poonamalle'):  10, ('poonamalle', 'Aavadi'): 10, ('Aavadi', 'veppambaattu'): 12, 
   })


r0 = RouteProblem('tiruttani', 'Saveetha Medical College', map=Thiruthani_to_SaveethaMedical)
r1 = RouteProblem('koolangeri', 'tiruvallur', map=Thiruthani_to_SaveethaMedical)
r2 = RouteProblem('tiruvallur', 'arakkonam', map=Thiruthani_to_SaveethaMedical)
r3 = RouteProblem('arcord', 'Narayanapuram', map=Thiruthani_to_SaveethaMedical)
r4 = RouteProblem('pichipakkam', 'manaur', map=Thiruthani_to_SaveethaMedical)



goal_state_path=breadth_first_search(r0)
print("GoalStateWithPath:{0}".format(goal_state_path))
print(path_states(goal_state_path) )
print("Total Distance={0} Kilometers".format(goal_state_path.path_cost))

goal_state_path=breadth_first_search(r1)
print("\n\nGoalStateWithPath:{0}".format(goal_state_path))
print(path_states(goal_state_path) )
print("Total Distance={0} Kilometers".format(goal_state_path.path_cost))

goal_state_path=breadth_first_search(r2)
print("\n\nGoalStateWithPath:{0}".format(goal_state_path))
print(path_states(goal_state_path) )
print("Total Distance={0} Kilometers".format(goal_state_path.path_cost))

goal_state_path=breadth_first_search(r3)
print("\n\nGoalStateWithPath:{0}".format(goal_state_path))
print(path_states(goal_state_path) )
print("Total Distance={0} Kilometers".format(goal_state_path.path_cost))

goal_state_path=breadth_first_search(r4)
print("\n\nGoalStateWithPath:{0}".format(goal_state_path))
print(path_states(goal_state_path) )
print("Total Distance={0} Kilometers".format(goal_state_path.path_cost))


```


## OUTPUT:

![a2](https://user-images.githubusercontent.com/75264748/167461189-aa4cd61f-3c67-4e6a-a603-58c277531a9e.jpg)



## SOLUTION JUSTIFICATION:
The Route solutions are found by Breadth First Search algorithm(following FIFO and routes travelling from left to right).
## RESULT:
Thus,an algorithm developed to find the route from the source to the destination point using breadth-first search.
