# Union And Find Set
Nowadays, the everyday questions on the leetcode were all related to union and find set. So I write down this blog to summary the knowledge of the set.
My main development language is Java, so all of the source code will be programmed in Java.
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
         if (size[rootP] > size[rootQ]) {
             parent[rootQ] = rootP;
                    size[rootP] += size[rootQ];
                } else {
                    parent[rootP] = rootQ;
                    size[rootQ] += size[rootP];
                }
                count--;
    }
}
```