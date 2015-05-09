---
layout: post
title:  "Python Graphs and Searching with BFS & DFS"
date:   2015-05-05
---

Graphs and their associated search algorithms are important concepts to master in CS.
They are also an integral part of many technical interview problems.

In this post, we will cover how to represent graphs in Python and implement their search algorithms.

###Reprenting a Graph:

A graph can be succinctly represented in Python by using a `dict` where the keys correspond to
the nodes on the graph and the value for a given key is just the set of neighboring/adjacent nodes 
connected via an edge/arc. 

Consider the following undirected graph:

<img src="/images/post-1-example_graph1.png"/>

The above can be represented like this:

{% highlight python %}
graph = {
	'A': set(['B', 'C', 'D']),
	'B': set(['A', 'C', 'E']),
	'C': set(['A', 'B']),
	'D': set(['A', 'E', 'F']),
	'E': set(['B', 'D']),
	'F': set(['D'])
}
{% endhighlight %}
<br>
[Other][adjacency-matrix] [ways][incidence-matrix] to represent graphs exist, 
but we'll stick to using the above.

[adjacency-matrix]: http://en.wikipedia.org/wiki/Adjacency_matrix
[incidence-matrix]: http://en.wikipedia.org/wiki/Incidence_matrix

### Searching: Breadth-first Search (BFS)

Consider finding a path from a starting node to a destination node in a graph.
In searching for a destination node, BFS allows us to traverse the graph in such
a way so that the search will find the destination via a shortest length path.
If the destination node doesn't exist, the BFS will traverse all nodes and edges.

Below we can see an implementation of BFS:

{% highlight python linenos %}
from collections import deque

def bfs(graph, start_node, destination_node):
  """
  Return True if there exists a path from start to destination.
  """
  queue, visited = deque([start_node]), set()
  while queue:
    current_node = queue.popleft()
    if current_node == destination_node:
      return True
    if current_node not in visited:
      queue.extend(graph[current_node] - visited)
      visited.add(current_node)
  return False
{% endhighlight %}
<br>

When visiting a node, we add all neighboring nodes to a queue; that is, when visiting
a node that is at some path length `L` from our start, we enqueue neighboring nodes which
must be at path length `L + 1`. 

Therefore, since our starting node (L == 0) will enqueue all neighboring nodes
, this means that all nodes at path length L == 1 will be next to each other in the queue. As we process these
nodes at L == 1, we add neighboring nodes at L == 2, meaning that all L == 2 nodes will be next to each
other in the queue and so on...

Do you notice something?

We don't begin exploring ANY node at path length `L + 1` until all nodes at `L` have been visited.

Therefore, since BFS exhausts all nodes at a specific path length before exploring other nodes
at a greater path length, we can see that when BFS encounters a destination node, we also have
a shortest path to destination.

Knowing the shortest path to a destination can be very useful in many occasions (e.g. figuring out
the minimal number of steps to get from one state to another).

How can we modify our BFS code to keep track of the path to the destination_node?

This can be done by keeping track of a `prevs` dict 
that maps nodes to the previous node on their path.
Therefore, once we have reached the destination, all
nodes in the path are keys in the `prevs` dict, each
with a value corresponding to the previous node in their path.
Therefore, we use a function to list the path out in the 
correct order by following the nodes in the `prevs` dict.


This is shown below:

{% highlight python linenos %}
from collections import deque

def bfs(graph, start_node, destination_node):
  """
  Returns a path from the start to destination, if it exists.
  """
  queue = deque([start_node])
  prevs = {start_node : ''}
  while queue:
    current_node = queue.popleft()
    if current_node == destination_node:
      return _list_path(prevs, start_node, destination_node)
    for neighbor in graph[current_node] - set(prevs.keys()):
      queue.extend(neighbor)
      prevs[neighbor] = current_node

def _list_path(prevs, start, dst):
  deq = deque([dst])
  cur = dst
  while cur != start:
    deq.appendleft(prevs[cur])
    cur = prevs[cur]
  return list(deq)
{% endhighlight %}
<br>
We could also store the path for every node in the queue by
storing tuples inside the queue, where the first element
is a given node and the second element is its associated path.

This makes it easy to keep track of all paths and 
allows us to create a function that lazily generates all paths to the destination.

This is shown below:

{% highlight python linenos %}
from collections import deque

def bfs_paths(graph, start_node, destination_node):
  """
  Generate paths from start to destination.
  """
  queue = deque([(start_node, [start_node])])
  while queue:
    current_node, path = queue.popleft()
    if current_node == destination_node:
      yield path
    for node in graph[current_node] - set(path):
      queue.extend([(node, path + [node])])

print list(bfs(graph, 'A', 'F'))
# [['A', 'D', 'F'], ['A', 'B', 'E', 'D', 'F'], ['A', 'C', 'B', 'E', 'D', 'F']]
{% endhighlight %}
<br>
Let's take a look at another way to search: DFS.

### Searching: Depth-first Search (DFS)

As the name implies, when a DFS is done from a given graph
node, it explores all possible nodes along that branch before backtracking.

Like before, we can use DFS to find out if a given destination can be reached
from a start.  First, we will implement DFS using recursion:

{% highlight python linenos %}

def dfs(graph, start, destination, visited=None):
  """
  Return True if there exists a path from start to destination.
  """
  if start == destination:
    return True
  # One would think that you can do this with default params, but
  # read this python gotcha: https://goo.gl/g72Ip1
  if visited is None:
    visited = set()
  visited.add(start)
  for node in graph[start] - visited:
    if dfs(graph, node, destination, visited):
      return True
  return False

{% endhighlight %}
<br>
Next we will use a DFS that keeps track of the path
and yields the full path every time the destination has been reached.

{% highlight python linenos %}
def dfs_paths(graph, start, destination, path=None):
  """
  Generate paths from start to destination.
  """
  if start == destination:
    yield path + [destination]
  if path is None:
    path = []
  path = path + [start]
  for node in graph[start] - set(path):
    for valid_path in dfs_paths(graph, node, destination, path):
      yield valid_path

# Testing with the same graph:
# >>> list(dfs_paths(graph, 'A', 'F'))
# [['A', 'C', 'B', 'E', 'D', 'F'], ['A', 'B', 'E', 'D', 'F'], ['A', 'D', 'F']]

{% endhighlight %}
<br>
Note that since we are using recursion, we run into the problem in python with the low recursion limit.
While one can always change the default limit on the number of stack frames, you might be faced with a stack size constrained
environment; therefore, it might be good to see if we can change this to an iterative DFS approach.

So can we do it without recursion? Yes! All we have  to do is keep track of the node ordering ourselves by creating a stack, instead
of letting the program's stacking of frame functions do the work. 

Lets implement it now:

{% highlight python linenos %}
def dfs_iterative(graph, start, destination):
  """
  Generate all paths from start to destination.
  """
  stack = [(start, [start])]
  while stack:
    node, path = stack.pop()
    if node == destination:
      yield path
    stack.extend((adj, path + [adj]) for adj in graph[node] - set(path))

# Testing with the same graph:
# >>> list(dfs_iterative(graph, 'A', 'F'))
# [['A', 'D', 'F'], ['A', 'B', 'E', 'D', 'F'], ['A', 'C', 'B', 'E', 'D', 'F']]
{% endhighlight %}
<br>
That's all for now. Try using these new concepts in your problems.


