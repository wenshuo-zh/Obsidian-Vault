# dijkstra
## 朴素版
### 思想
#### 1. 这个算法是干什么的？
从一个起点出发，找到这个起点到其他所有节点的最短距离。
比如从节点 1 出发，找到去节点 2、3、4 分别要走多远。
#### 2. 核心思路
Dijkstra 每一轮都做两件事：
1. 从还没确定最短距离的节点中，挑一个目前距离最小的节点 `u`。
2. 检查经过 `u` 到其他节点的路线有没有更短的，如果更短，就更新距离。
然后继续挑下一个节点，直到没有节点可以选了。
为什么要找==距离最小的 `u`==？
因为边权没有负数，所以当前距离最小的那个未确定节点，不可能再通过其他节点绕一圈得到更短的距离，可以确定它的最短距离了。
#### 3. 两个重要数组
```cpp
vector<int> dist(n + 1, INT_MAX);
vector<bool> visited(n + 1, false);

dist[k] = 0;
```
- `dist[i]`：从起点到节点 `i`，目前找到的最短距离。找不到路线时先记为 `INT_MAX`。
- `visited[i]`：节点 `i` 的最短距离是否已经确定，`true` 表示确定了。
- `dist[k] = 0`：起点到自己距离为 0。
#### 4. 每一轮具体怎么做？
**第一步：找** `u`
遍历所有节点，跳过已经确定最短距离的节点，找到 `dist` 最小的那个，作为本轮的 `u`。
注意：每一轮都会重新找 `u`，所以 `u` 通常不一样。
**第二步：检查有没有更短的路线**
假设现在选中了节点 `u`，有一条边从 `u` 通向节点 `v`，边的权值是 `w`。
那么经过 `u` 到 `v` 的距离就是：
`dist[u] + w`
- 如果这个距离比原来的 `dist[v]` 小，就更新 `dist[v]`。
- 如果没有更小，就不更新。
```cpp
if (dist[u] + w < dist[v]) {
    dist[v] = dist[u] + w;
}
```
这个过程叫作“松弛”，简单说就是：**发现更短的路线，就把原来的距离改小。**
### 例题
#### 网络延迟时间
每一轮都会重新循环挑选距离最小的未确定节点 `u`，再检查经过这个 `u` 的路线能否缩短其他节点的距离，因此每轮的 `u` 通常不同。
```cpp
class Solution {
public:
    int networkDelayTime(vector<vector<int>>& times, int n, int k) {
        vector<vector<pair<int, int>>> graph(n+1);
        for(auto t : times){
            int u = t[0];
            int v = t[1];
            int w = t[2];
            graph[u].push_back({v, w});
        }
        //起点到下标i的最短距离
        vector<int> dist(n+1, INT_MAX);
        vector<bool> visited(n+1, false);
        dist[k] = 0;
        for(int j = 1; j <= n; j++){
            //距离最小的节点和距离
            int minDist = INT_MAX;
            int u = -1;
            for(int i = 1; i <= n; i++){
                if(!visited[i] && dist[i] < minDist){
                    u = i;
                    minDist = dist[i];
                }
            }
            if(u == -1)break;
            //找到距离最小点u，更新u的邻居，看有没有更短的路线
            visited[u] = true;
            for(auto g : graph[u]){
                int v = g.first;
                int w = g.second;
                //比较目前找到的到v的最短距离和经过u再到v的距离
                if(!visited[v] && dist[u] + w < dist[v])dist[v] = dist[u] + w;
            }
        }
        int res = 0;
        for(int i = 1; i <= n; i++){
            if(dist[i] == INT_MAX)return -1;
            res = max(res, dist[i]);
        }
        return res;
    }
};
```
## 堆优化版
利用优先队列（小顶堆）而不是普通队列，可以直接拿到最小值，省去遍历找u
```cpp
class Solution {
public:
    int networkDelayTime(vector<vector<int>>& times, int n, int k) {
        vector<vector<pair<int, int>>> graph(n+1);
        for(auto t : times){
            int u = t[0];
            int v = t[1];
            int w = t[2];
            graph[u].push_back({v, w});
        }
        vector<int> dist(n+1, INT_MAX);
        dist[k] = 0;
        //小顶堆（优先级队列）
        priority_queue<pair<int, int>, vector<pair<int, int>>, greater<pair<int, int>> >pq;
        pq.push({0, k});
        while(!pq.empty()){
            auto [d, u] = pq.top();
            pq.pop();
            if(d > dist[u])continue;
            for(auto g : graph[u]){
                int v = g.first;
                int w = g.second;
                if(dist[u] + w < dist[v]){
                    dist[v] = dist[u] + w;
                    pq.push({dist[v], v});
                }
            }
        }
        int res = 0;
        for(int i = 1; i <= n; i++){
            if(dist[i] == INT_MAX)return -1;
            res = max(dist[i], res);
        }
        return res;
    }
};
```
