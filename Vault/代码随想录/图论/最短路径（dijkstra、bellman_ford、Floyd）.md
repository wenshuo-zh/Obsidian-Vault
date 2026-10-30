# dijkstra
## 朴素版dijkstra理论基础
Dijkstra 是一种求单源最短路径的贪心算法，要求边权非负。
每次从==未确定最短距离==的节点中，选出距==离起点最近==的节点。
将这个节点标记为已确定，再用它更新其他相邻节点的距离。
重复这个过程，直到所有可达节点的最短距离都确定。
这里最重要的是：
每次选的是当前距离起点最近的未确定节点，而不是全图权值最小的边。
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
## 堆优化版dijkstra
利用优先队列（==小顶堆==）而不是普通队列，可以直接拿到最小值，省去遍历找u
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
# bellman_ford
## bellman_ford基础理论
适合处理带有==负权边==的图，这是它和 Dijkstra 的重要区别。
==松弛n-1次==
每一轮遍历所有边，检查能不能通过当前边找到更短的路线。
如果新路线更短，就更新终点的距离。
最多重复 ==`n-1`== 轮，得到最短距离。
这里最重要的是：
每次不是选择距离最小的节点，而是遍历所有边，不断更新更短的距离。
### 卡码网94 城市间货物运输I
```cpp
#include<iostream>
#include<vector>
#include <climits>
using namespace std;
struct Edge{
    int s;//起点
    int t;//终点
    int v;//权值
};
int main(){
    int n, m;
    cin >> n >> m;
    vector<Edge> edges(m);
    for(int i = 0; i < m; i++){
        cin >> edges[i].s >> edges[i].t >> edges[i].v;
    }
    vector<int> dist(n+1, INT_MAX);//dist[i]表示起点到下标i所表示的最小距离
    dist[1] = 0;
    //循环n-1次
    for(int i = 1; i <= n-1; i++){
        //检查所有道路
        for(auto e : edges){
            if(dist[e.s] != INT_MAX && dist[e.s] + e.v < dist[e.t]){
                dist[e.t] = dist[e.s] + e.v;
            }
        }
    }
    //输出结果
    if(dist[n] == INT_MAX)cout << "unconnected";
    else cout << dist[n];
    return 0;
}
```
## 队列优化bellman_ford（SPFA）
SPFA 是 Bellman-Ford 的队列优化版，可以处理负权边。
从起点开始，将起点加入队列，每次取出一个节点，用它更新相邻节点的最短距离。
如果某个节点的==距离变小了，就把它加入队列==，等待后续更新。
重复这个过程，直到队列为空。
这里最重要的是：
不是每轮遍历所有边，而是==只处理距离发生变化的节点==，减少不必要的计算。
### 卡码网94 城市间货物运输I
```CPP
#include<iostream>
#include<vector>
#include<climits>
#include<queue>
using namespace std;
struct Edge{
    int t;//终点
    int v;//权值
};
int main(){
    int n, m;
    cin >> n >> m;
    vector<vector<Edge>> graph(n + 1);//因为要知道每个节点的出边，建立邻接表
    vector<int> dist(n+1, INT_MAX);
    int s, t, v;
    for(int i = 0; i < m; i++){
        cin >> s >> t >> v;
        graph[s].push_back({t, v});
    }
    queue<int> que;
    vector<bool> inQueue(n + 1, false);//判断节点i是否在队列里
    dist[1] = 0;
    que.push(1);
    inQueue[1] = true;
    while(!que.empty()){
        int cur = que.front();
        que.pop();
        inQueue[cur] = false;
        for(auto e : graph[cur]){
            if(dist[cur] != INT_MAX && dist[cur] + e.v < dist[e.t]){
                dist[e.t] = dist[cur] + e.v;
                if(!inQueue[e.t]){
                    inQueue[e.t] = true;
                    que.push(e.t);
                }
            }
        }
    }
    //输出结果
    if(dist[n] == INT_MAX)cout << "unconnected";
    else cout << dist[n];
    return 0;
}
```
## bellman_ford判断负权回路
如果有负权值且成环就会在环里无限循环，无法求出最小路径
解决方法：在松弛n-1轮之后再松弛一轮，如果dist变小了，就一定有负权环
### 卡码网95城市间货物运输II
```cpp
#include<iostream>
#include<vector>
#include<climits>
#include<algorithm>
using namespace std;
struct Edge{
    int s;
    int t;
    int v;
};
int main(){
    int n, m;
    cin >> n >> m;
    vector<Edge> edges(m);
    for(int i = 0; i < m; i++){
        cin >> edges[i].s >> edges[i].t >> edges[i].v;
    }
    vector<int> dist(n+1, INT_MAX);
    dist[1] = 0;
    for(int i = 1; i <= n-1; i++){
        for(auto e : edges){
            if(dist[e.s] != INT_MAX && dist[e.s] + e.v < dist[e.t]){
                dist[e.t] = dist[e.s] + e.v;
            }
        }
    }
    if(dist[n] == INT_MAX){
        cout << "unconnected";
        return 0;
    }
    bool flag = true;
    for(auto e : edges){
        if(dist[e.s] != INT_MAX && dist[e.s] + e.v < dist[e.t]){
            flag = false;
        }
    }
    if(!flag) cout << "circle";
    else cout << dist[n];
}
```
# Floyd
求==任意两个节点==间的最短路径
Floyd 就是枚举中转节点，通过 `dist[i][k] + dist[k][j]` 不断更新任意两点之间的最短距离。
## 卡码网97小明逛公园
```cpp
#include<iostream>
#include<vector>
#include<climits>
using namespace std;
int main(){
    int n, m;
    cin >> n >> m;
    vector<vector<int>> dist(n+1, vector<int>(n+1, INT_MAX));//dist[u][v]表示从节点u到节点v的距离
    //双向建图
    int u, v, w;
    for(int i = 0;i < m; i++){
        cin >> u >> v >> w;
        dist[u][v] = min(dist[u][v], w);
        dist[v][u] = min(dist[v][u], w);
    }
    for(int i = 1; i <= n; i++) dist[i][i] = 0;

    for(int k = 1; k <= n; k++){//中间节点
        for(int i = 1; i <= n; i++){//起点
            for(int j = 1; j <= n; j++){//终点
                if(dist[i][k] == INT_MAX || dist[k][j] == INT_MAX)continue;
                if(dist[i][k] + dist[k][j] < dist[i][j]){
                    dist[i][j] = dist[i][k] + dist[k][j];
                }
            }
        }
    }
    int q;
    cin >> q;
    int start, end;
    while(q--){
        cin >> start >> end;
        if(dist[start][end] == INT_MAX)cout << -1 << endl;
        else cout << dist[start][end] << endl;
    }
}
```
# A\* (A Star)算法
### 卡码网127 骑士的攻击
#### bfs解法
```cpp
#include<iostream>
#include<vector>
#include<queue>
using namespace std;
int dir[8][2] = {-2, -1, -2, 1, 2, -1, 2, 1, -1, 2, -1, -2, 1, 2, 1, -2};
void bfs(int a1, int a2, int b1, int b2, vector<vector<int>>& moves){
    queue<pair<int, int>> que;
    que.push({a1, a2});
    while(!que.empty()){
        pair<int, int> cur = que.front();
        que.pop();
        if(cur.first == b1 && cur.second == b2)break;
        for(int i = 0; i < 8; i++){
            int nx = cur.first + dir[i][0];
            int ny = cur.second + dir[i][1];
            if(nx < 1 || nx > 1000 || ny < 1 || ny > 1000)continue;
            if(!moves[nx][ny]){
                moves[nx][ny] = moves[cur.first][cur.second] + 1;
                que.push({nx, ny}); 
            }
        }        
    }
}
int main(){
    int n;
    cin >> n;
    int a1, a2, b1, b2;
    while(n--){
        cin >> a1 >> a2 >> b1 >> b2;
        vector<vector<int>> moves(1001, vector<int>(1001, 0));
        bfs(a1, a2, b1, b2, moves);
        cout << moves[b1][b2] << endl;
    }
}
```
该解法在题目数据下超时
#### A*（没写完，仅保留框架）
```cpp
#include<iostream>

#include<vector>

#include<queue>

#include <algorithm>

#include <cstdlib>

#include <climits>

using namespace std;

int dir[8][2] = {-2, -1, -2, 1, 2, -1, 2, 1, -1, 2, -1, -2, 1, 2, 1, -2};

struct node{

    int x, y;

    int g;//已经走的步数

    int f;//估计总步数

};

struct cmp{

    bool operator()(node a, node b){

        return a.f > b.f;

    }

};

//启发函数，返回从当前位置[x, y]到终点[b1, b2]，估计还需要多少步

int h(int x, int y, int b1, int b2){

    int dx = abs(x - b1);

    int dy = abs(y - b2);

    return (max(dx, dy) + 1) / 2;

}

int aStar(int a1, int a2, int b1, int b2){

    priority_queue<node, vector<node>, cmp> que;

    vector<vector<int>> dist(1001, vector<int>(1001, INT_MAX));

    dist[a1][a2] = 0;

    que.push({a1, a2, 0, h(a1, a2, b1, b2)});

    while (!que.empty()) {

        node cur = que.top();

        que.pop();

        if (cur.x == b1 && cur.y == b2) return cur.g;

        for (int i = 0; i < 8; i++) {

            int nx = cur.x + dir[i][0];

            int ny = cur.y + dir[i][1];

            if (nx < 1 || nx > 1000 || ny < 1 || ny > 1000)continue;

            int newG = cur.g + 1;

            if (newG < dist[nx][ny]) {

                // 更新 dist

                // 计算新的 f

                // 将新节点加入优先队列

            }

    }

}

int main(){

    int n;

    cin >> n;

    int a1, a2, b1, b2;

    while(n--){

        cin >> a1 >> a2 >> b1 >> b2;

        vector<vector<int>> moves(1001, vector<int>(1001, 0));

        cout << moves[b1][b2] << endl;

    }

}
```
