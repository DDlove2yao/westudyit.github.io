# Union And Find Set
Nowadays, the everyday questions on the leetcode were all related to union and find set. So I write down this blog to summary the knowledge of the set.
My main development language is Java, so all of the source code will be programmed in Java.
<!-- TOC -->

- [Union And Find Set](#union-and-find-set)
    - [About the union and find set](#about-the-union-and-find-set)
    - [391 Evaluate Division](#391-evaluate-division)
        - [Solution](#solution)
    - [547. Number of Provinces](#547-number-of-provinces)
        - [Solution](#solution-1)
    - [684 Redunant Connection](#684-redunant-connection)
        - [Solution](#solution-2)

<!-- /TOC -->
## About the union and find set
Accordig to wikipedia, the union and find set is a tree-like data structure which is used to deal with disjoint sets problems.
There are two main functions in the set: union and find.
The format of the union and find is much fixed which is shown as following:
```java
class UnionAndFind{
    //parent is used to store the elements of the set or we can say to store the node of a picture. Most of the time , it's int[] kind,sometimes it may be Map.
    private int[] parent;
    //weight is used to store the weight of the line.
    private int[] weight;
    //count is used to get the length of the set
    private int count;

    public UnionAndFind(int n){
        parent = new int[n];
        weight = new int[n];
        this.count = n;
        for(int i : n-1){
            parent[i] = i;
            weight[i] = 1;
        }
    }
    
    public void union(int x, int y){
        int find_x = find(x);
        int find_y = find(y);
        if(find_x == find_y)return;
        //path compression
        if(find_x < find_y){
            parent[find_x] = find_y;
            weight[find_y] += weight[find_x];
        }else{
            parent[find_y] = find_x;
            weight[find_x] += weight[find_y];
        }
        count--;
    }
    
    public int find(int x){
        while(parent[x] != x){
            //path compression
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }

    public int count(){
        return count;
    }
}
```
Then we can use the union and find set to solve algorithm problems.
## 391 Evaluate Division
You are given an array of variable pairs equations and an array of real numbers values, where equations[i] = [Ai, Bi] and values[i] represent the equation Ai / Bi = values[i]. Each Ai or Bi is a string that represents a single variable.
You are also given some queries, where queries[j] = [Cj, Dj] represents the jth query where you must find the answer for Cj / Dj = ?.
Return the answers to all queries. If a single answer cannot be determined, return -1.0.

Note: The input is always valid. You may assume that evaluating the queries will not result in division by zero and that there is no contradiction.

Example 1:
```
Input: equations = [["a","b"],["b","c"]], values = [2.0,3.0], queries = [["a","c"],["b","a"],["a","e"],["a","a"],["x","x"]]
Output: [6.00000,0.50000,-1.00000,1.00000,-1.00000]
Explanation: 
Given: a / b = 2.0, b / c = 3.0
queries are: a / c = ?, b / a = ?, a / e = ?, a / a = ?, x / x = ?
return: [6.0, 0.5, -1.0, 1.0, -1.0 ]
```

Example 2:
```
Input: equations = [["a","b"],["b","c"],["bc","cd"]], values = [1.5,2.5,5.0], queries = [["a","c"],["c","b"],["bc","cd"],["cd","bc"]]
Output: [3.75000,0.40000,5.00000,0.20000]
```
Example 3:
```
Input: equations = [["a","b"]], values = [0.5], queries = [["a","b"],["b","a"],["a","c"],["x","y"]]
Output: [0.50000,2.00000,-1.00000,-1.00000]
```

Constraints:
```
1 <= equations.length <= 20
equations[i].length == 2
1 <= Ai.length, Bi.length <= 5
values.length == equations.length
0.0 < values[i] <= 20.0
1 <= queries.length <= 20
queries[i].length == 2
1 <= Cj.length, Dj.length <= 5
Ai, Bi, Cj, Dj consist of lower case English letters and digits.
```
Resource：LeetCode
Link：https://leetcode-cn.com/problems/evaluate-division
The copyright belongs to Lingkou Network. For commercial reprints, please contact the official authorization. For non-commercial reprints, please indicate the source.

### Solution
From the question, we can know that whether an equation can be calculate depends on whether we can deduct the equation from the given equations.
Hence, we can use set to maintain the equations that can get results. Those which has the same divisor or dividend will be divided into a same set.
We should also use Map to store the relationship of the equations.
So the code shows as following:
```java
class Solution{
    public double[] calcEquation(List<List<String>> equations, double[] values, List<List<String>> queries){
        int size = equations.size();
        UnionAndFind uaf = new UnionAndFind(size);
        //store the equations relationship
        Map<String, Integer> map = new HashMap<>();
        int id = 0;
        for(int i = 0; i < size; i++){
            List<String> equation = equations.get(i);
            String divisor = equation.get(1);
            String dividend = equation.get(0);
            if(!map.containsKey(dividend)){
                map.put(dividend, id);
                id++;
            }
            if(!map.containsKey(divisor)){
                map.put(divisor, id);
                id++;
            }
            // put the equations and values into set
            uaf.union(map.get(dividend), map.get(divisor), values[i]);
        }
        //check the set to find whether the queries can be calculated
        double[] res = new double[queries.length];
        for(int i = 0; i < queries.length; i++){
            String dividend = queries.get(i).get(0);
            String divisor = queries.get(i).get(0);
            Integer id1 = map.get(dividend);
            Integer id2 = map.get(divisor);
            if(id1 == null || id2 == null){
                res[i] = -1.0d;
            }else{
                res[i] = uaf.isConnected(id1, id2);
            }
        }
        return res;
    }
}

class UnionAndFind{
    private int[] parent;
    private double[] weight;
    public UnionAndFind(int n){
        this.parent = new int[n];
        this.weight = new int[n];
        for(int i = 0; i < n; i++){
            parent[i] = i;
            weight[i] = 1.0d;
        }
    }

    public void union(int x, int y, double value){
        int find_x = find(x);
        int find_y = find(y);
        if(find_x == find_y)return;
        parent[find_x] = find_y;
        weight[find_x] = weight[y] * value / weight[x];
    }

    public int find(int x){
        if(x != parent[x]){
            parent[x] = find[parent[x]];
            //here we can also write parent[x] = parent[parent[x]];
            weight[x] *= weight[parent[x]];
        }
        return parent[x];
    }

    public double isConnected(int x, int y){
        int find_x = find(x);
        int find_y = find(y);
        if(find_x == find_y)return weight[x]/weight[y];
        return -1.0d;
    }
}
```
## 547. Number of Provinces
There are n cities. Some of them are connected, while some are not. If city a is connected directly with city b, and city b is connected directly with city c, then city a is connected indirectly with city c.
A province is a group of directly or indirectly connected cities and no other cities outside of the group.
You are given an n x n matrix isConnected where isConnected[i][j] = 1 if the ith city and the jth city are directly connected, and isConnected[i][j] = 0 otherwise.
Return the total number of provinces.
Example 1:
```
Input: isConnected = [[1,1,0],[1,1,0],[0,0,1]]
Output: 2
```
Example 2:
```
Input: isConnected = [[1,0,0],[0,1,0],[0,0,1]]
Output: 3
```

Constraints:
```
1 <= n <= 200
n == isConnected.length
n == isConnected[i].length
isConnected[i][j] is 1 or 0.
isConnected[i][i] == 1
isConnected[i][j] == isConnected[j][i]
```
Resource：LeetCode
Link：https://leetcode-cn.com/problems/evaluate-division
The copyright belongs to Lingkou Network. For commercial reprints, please contact the official authorization. For non-commercial reprints, please indicate the source.
### Solution
Apparently, it's a union and found set problem. The result is the count of the set size.
```java
class Solution{
    public int findCircleNum(int[][] M){
        int n = M.length;
        UnionAndFind uaf = new UnionAndFind(n);
        for(int i = 0; i < n; i++){
            for(int j = 0; j < M[0].length; j++){
                if(M[i][j] == 1){
                    uaf.union(i, j);
                }
            }
        }
        return uf.count();
    }
}

class UnionAndFind{
    private int[] parent;
    private int count;
    private int[] weight;

    public UnionAndFind(int n){
        this.parent = new int[n];
        this.count = n;
        this.weight = new int[n];
        for(int i = 0; i < n; i++){
            parent[i] = i;
            weight[i] = 1;
        }
    }

    public int find(int x){
        while(parent[x] != x){
            parent[x] = parent[parent[x]];
            x = parent[x];
        }
        return x;
    }

    public void union(int x, int y){
        int find_x = find(x);
        int find_y = find(y);
        if(find_x == find_y)return;
         if (size[find_x] > size[find+y]) {
             parent[find_y] = find_x;
                    size[find_x] += size[find_y];
                } else {
                    parent[find_x] = find_y;
                    size[find_y] += size[find_x];
                }
                count--;
    }

    public int count(){
        return count;
    }
}
```
## 684 Redunant Connection
In this problem, a tree is an undirected graph that is connected and has no cycles.
The given input is a graph that started as a tree with N nodes (with distinct values 1, 2, ..., N), with one additional edge added. The added edge has two different vertices chosen from 1 to N, and was not an edge that already existed.
The resulting graph is given as a 2D-array of edges. Each element of edges is a pair [u, v] with u < v, that represents an undirected edge connecting nodes u and v.
Return an edge that can be removed so that the resulting graph is a tree of N nodes. If there are multiple answers, return the answer that occurs last in the given 2D-array. The answer edge [u, v] should be in the same format, with u < v.
Example 1:
```
Input: [[1,2], [1,3], [2,3]]
Output: [2,3]
Explanation: The given undirected graph will be like this:
  1
 / \
2 - 3
```
Example 2:
```
Input: [[1,2], [2,3], [3,4], [1,4], [1,5]]
Output: [1,4]
Explanation: The given undirected graph will be like this:
5 - 1 - 2
    |   |
    4 - 3
```
Note:
The size of the input 2D-array will be between 3 and 1000.
Every integer represented in the 2D-array will be between 1 and N, where N is the size of the input array.

Resource：LeetCode
Link：https://leetcode-cn.com/problems/evaluate-division
The copyright belongs to Lingkou Network. For commercial reprints, please contact the official authorization. For non-commercial reprints, please indicate the source.

### Solution
According to the question description, we should try not generate a circle in the graph.We should delete the last edge in the given array.

The code shows as following:
```java
public class Solution{

public int[] findRedundantConnection(int[][] edges){
        int n = edges.length;
        UnionAndFind uaf = new UnionAndFind(n);
        for(int i = 0; i < n; i++){
            int[] edge = edges[i];
            int node1 = edge[0], node2 = edge[1];
            if(uaf.find(node1) != uaf.find(node2)){
                // they wont generate a circle
                uaf.union(node1, node2);
            }else return edge;
        }
        //no circle
        return new int[0];
    }
}

class UnionAndFind{
    private int[] parent;
    public UnionAndFind(int n){
        this.parent = new int[n];
        for(int i = 0; i < n; i++){
            parent[i] = i;
        }
    }
    public void union(int x, int y){
        int find_x = find(x);
        int find_y = find(y);
        if(find_x != find_y)return;
        parent[find_x] = find_y;
    }

    public int find(int x){
        if(parent[x] != x){
            parent[x] = find(parent[x]); 
        }
        return parent[x];
    }
}
```
## 947.Most Stone Removed with Same Row or Column
On a 2D plane, we place n stones at some integer coordinate points. Each coordinate point may have at most one stone.
A stone can be removed if it shares either the same row or the same column as another stone that has not been removed.
Given an array stones of length n where stones[i] = [xi, yi] represents the location of the ith stone, return the largest possible number of stones that can be removed.

Example 1:
```
Input: stones = [[0,0],[0,1],[1,0],[1,2],[2,1],[2,2]]
Output: 5
Explanation: One way to remove 5 stones is as follows:
1. Remove stone [2,2] because it shares the same row as [2,1].
2. Remove stone [2,1] because it shares the same column as [0,1].
3. Remove stone [1,2] because it shares the same row as [1,0].
4. Remove stone [1,0] because it shares the same column as [0,0].
5. Remove stone [0,1] because it shares the same row as [0,0].
Stone [0,0] cannot be removed since it does not share a row/column with another stone still on the plane.
```
Example 2:
```
Input: stones = [[0,0],[0,2],[1,1],[2,0],[2,2]]
Output: 3
Explanation: One way to make 3 moves is as follows:
1. Remove stone [2,2] because it shares the same row as [2,0].
2. Remove stone [2,0] because it shares the same column as [0,0].
3. Remove stone [0,2] because it shares the same row as [0,0].
Stones [0,0] and [1,1] cannot be removed since they do not share a row/column with another stone still on the plane.
```
Example 3:
```
Input: stones = [[0,0]]
Output: 0
Explanation: [0,0] is the only stone on the plane, so you cannot remove it.
```

Constraints:
```
1 <= stones.length <= 1000
0 <= xi, yi <= 104
No two stones are at the same coordinate point.
```
Resource：LeetCode
Link：https://leetcode-cn.com/problems/evaluate-division
The copyright belongs to Lingkou Network. For commercial reprints, please contact the official authorization. For non-commercial reprints, please indicate the source.

### Solution
In this question, we can know that the number of the final remain stones equals the number of stones subtract the size of set.
Those which joint each other can be added into the same set.
In this problem, we need to make a change: we have set the parent array as Map to store row and column label.
```java
class Solution{

    public int removeStones(int[][] stones){
        if(stones.length < 1)return 0;
        int n = stones.length;
        UnionAndFind uaf = new UnionAndFind();
        for(int[] stone : stones){
            uf.union(stone[0]+10000, stone[1]);
        }
        return n - uaf.count();
    }
}

class UnionAndFind{
    private Map<Integer, Integer> parent;
    private int count;

    public UnionAndFind(){
        this.parent = new HashMap<>();
        this.count = 0;
    }

    public void union(int x, int y){
        int find_x = find(x);
        int find_y = find(y);
        if(find_x == find_y)return;
        parent.put(find_x, find_y);
        count--;
    }

    public int find(int x){
        if(!parent.containsKey(x)){
            parent.put(x, x);
            count++;
        }
        if(x != parent.get(x)){
            parent.put(x, find(parent.get(x)));
        }
        return parent.get(x);
    }

    public int count(){rteurn count;}
}
```
### 1319.Number of Operations to Make Network Connected
There are n computers numbered from 0 to n-1 connected by ethernet cables connections forming a network where connections[i] = [a, b] represents a connection between computers a and b. Any computer can reach any other computer directly or indirectly through the network.
Given an initial computer network connections. You can extract certain cables between two directly connected computers, and place them between any pair of disconnected computers to make them directly connected. Return the minimum number of times you need to do this in order to make all the computers connected. If it's not possible, return -1. 
Example 1:
```
Input: n = 4, connections = [[0,1],[0,2],[1,2]]
Output: 1
Explanation: Remove cable between computer 1 and 2 and place between computers 1 and 3.
```
Example 2:
```
Input: n = 6, connections = [[0,1],[0,2],[0,3],[1,2],[1,3]]
Output: 2
```
Example 3:
```
Input: n = 6, connections = [[0,1],[0,2],[0,3],[1,2]]
Output: -1
Explanation: There are not enough cables.
```
Example 4:
```
Input: n = 5, connections = [[0,1],[0,2],[3,4],[2,3]]
Output: 0
```
Constraints:
- 1 <= n <= 10^5
- 1 <= connections.length <= min(n*(n-1)/2, 10^5)
- connections[i].length == 2
- 0 <= connections[i][0], connections[i][1] < n
- connections[i][0] != connections[i][1]
- There are no repeated connections.
- No two computers are connected by more than one cable.

Resource：LeetCode
Link：https://leetcode-cn.com/problems/evaluate-division
The copyright belongs to Lingkou Network. For commercial reprints, please contact the official authorization. For non-commercial reprints, please indicate the source.

### Solution
We can know that for n computers, we need n-1 lines to connect all of the computers.
So if the number of connections + 1 < n, we return -1. It means that we cannot connect all the computers.
Here we use the union and find set to maintain the computers connected. finally we return the size of the set subtract 1.

```java
class Solution{
    public int makeConnected(int n, int[][] connected){
        int len = connected.length;
        UnionAndFind uaf = new UnionAndFind(n);
        if(len + 1 < n)return -1;
        for(int i = 0; i < len; i++){
            uaf.union(connected[i][0], connected[i][1]);
        }
        return uaf.count()-1;
    }
}

class UnionAndFind{
    private int[] parent;
    private int count;
    
    public UnionAndFind(int n){
        this.count = n;
        this.parent = new int[n];
        for(int i = 0; i < n; i++){
            parent[i] = i;
        }
    }
    
    public int find(int x){
        if(x != parent[x]){
            parent[x] = find(parent[x]);
        }
        return parent[x];
    }
    
    public void union(int x, int y){
        int find_x = find(x);
        int find_y = find(y);
        if(find_x == find_y)return;
        parent[find_x] = find_y;
        count--;
    }

    public int count(){return this.count;}
}
```