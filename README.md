# Distributed Segment tree

## Problem

You are given X n-dimensional data points and Y n-dimensional query points.
For each query q=(q1, q2, ..., qn) find all data points d=(d1, d2, ..., dn) such that q1 < d1, q2 < d2, ..., qn < dn. The solution should be distributed over several machines and run in parallel.

Example:

data(1,5) query(3,8) data(4,2) query(5,9) data(6,7) query(7,3) data(8,1) query(9,4)

should return

query(3,8) - 1 (only data(1,5) has both coordinates smaller)

query(5,9) - 2 (data(1,5) & data(3,8))

query(7,3) - 1 (data(4,2))

query(9,4) - 2 (data(4, 2) & data(7,3))

## 1-dimensional solution

For 1-dimensional points, the solution boils down to sorting the values and counting the number of data points before each query point (this can be done efficiently with multiple machines by counting the local results on each machine (as if there were no points other than those on one machine) and sending the message about number of data points to each machine, and then adjusting the global results based on the other machines' results); the sorting itself can also be done in parallel using the [TeraSort](https://arxiv.org/pdf/1702.04850.pdf) algorithm.

Another approach (closer to an n-dimensional solution) would be to use a segment tree (and keep in mind the following lemma- every 2 leaves of a segment tree have exactly one lowest common ancestor where their paths to the root meet):

- we sort the values (for example with TeraSort)
- we give each point a position in our sorted sequence of values (the smallest point will be given position 0, the second smallest is 1, and so on)

                                     ''

                     0                                1

            00                01             10                11

        000     001     010      011      100     101     110      111
         |       |       |        |        |       |       |        |
      data(1) query(3) data(4) query(5) data(6) query(7) data(8) query(9)

- the positions (transformed into binary numbers of appropriate length) describe the path from the root to these values (the smallest point has position 0 = b'0000...00', which means that if we want to go from the root to this point, we always have to go to the left subtree ('0' means left subtree, '1' means right subtree); the second smallest point has position 1 = b'000...01', which tells us that we always have to go to the left subtree, except for the last turn, where we have to go to the right subtree; and so on).
- having this information, we want to distribute data and query points to the top of our segment tree (at the moment only the leaf nodes contain data/query points; we want to push them forward to parents/other ancestors, preferably in such way that then helps us solve the problem)- the idea is to look at each node X and store all the data points from the left subtree and all the query points from the right subtree (this way, each data point's value inside this node is smaller than each query point's value (because all the values from the left subtree are smaller, since the values are sorted) + there will be no duplicate pairs (we will not have the same pair of data point and query point stored in any other node, because the node X is their lowest common ancestor- for all the nodes above they are in the same subtree, and for all the nodes below, it does not reach both points)) (it can also be done efficiently and in a distributed way)

In the example above, the content of the nodes would be the following:

'' - data(1) & data(4) & query(7) & query(9)

0 - data(1) & query(5)

1 - data(6) & query(9)

00 - data(1) & query(3)

01 - data(4) & query(5)

10 - data(6) & query(7)

11 - data(8) & query(9)

- after distributing the points to the nodes, we can consider each node separately (for example some nodes on one machine, other nodes on another machine and so on) and count local results (it has been shown that all the data points saved stored within a node are smaller than all the query points within the same node + there are no duplicates (+each pair appears once, since each pair of leaf nodes has its lowest common ancestor and they must be stored here if the data point is smaller than the query point)), and aggregate these local results by summing them.

Following the example:

in the '' node, there are 2 data points and query points query(7) and query(9), so we want to add these 2 data points to the final result for query(7) and query(9).

in the 0 node, there is 1 data point and 1 query node(query(5)) so we want to add 1 to the final result of query(5).

and so on...

- after adding the results, the problem is solved.

## n-dimensional solution

As mentioned before, the idea for the n-dimensional solution is similar to the 1-dimensional one with the distributed segment tree- instead of sorting the data points by one (and only one) value, we sort all the values by each dimension separately and assign n different positions to each point (depending on the dimension we are sorting by). Then, again, we want to distribute the points to the nodes in the same way- node X wants to collect all the data points from its 'left subtree' and all the query points from its 'right subtree'. It turns out that it can be computed very efficiently in parallel if we have information about all the positions for each dimension for each node. Finally, we can count the local results for each node individually and aggregate the results, just like in the 1-dimensional solution.

## Use cases

Apart from solving the problem itself, the algorithm is an important tool for computing many similar problems, such as computing the number of points in a fixed area (rectangular/cubic/described by hypercube). These problems themselves are quite general and are used in many specific areas, such as ML algorithms.
