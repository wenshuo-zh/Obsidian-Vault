# 最小生成树
简单理解：==用尽可能小的总费用，把所有点连接起来==
比如有几个城市，城市之间修路需要花钱，现在要求所有城市都能互相连通，而且总修路费用最低。
这就是最小生成树（MST）。
需要记住的几个特点：
- 所有点都要连通。
- 不能有环，否则就有多余的连接。
- 如果有 n 个点，最小生成树一定有 n - 1 条边。
- 目标是让所有选中边的权值之和最小。
# prim
## prim的基本思想
Prim 是一种贪心算法。
==从一个点开始，每次找一条连接当前生成树和外部点的最小权值边，把新点接进来==
一直重复，直到所有点都加入生成树
这里最重要的是：
每次找的不是全图最小的边，而是==当前树连接外部点时，能选的最小边==
### 连接所有点的最小费用
#### prim
```cpp
class Solution {
public:
    int minCostConnectPoints(vector<vector<int>>& points) {
        int n = points.size();
        vector<bool> visited(n, false);
        vector<int> minDist(n, INT_MAX);
        minDist[0] = 0;//下标i连到树的最小权值
        int res = 0;
        for (int i = 0; i < n; i++) {
            int minVal = INT_MAX;
            int u = -1;
            //这个for循环找u，u是树外连到树上最小点的下标
            for (int j = 0; j < n; j++) {
                if (!visited[j] && minDist[j] < minVal) {
                    minVal = minDist[j];
                    u = j;
                }
            }
            visited[u] = true;
            res += minDist[u];
            //更新minDist
            for(int j = 0; j < n; j++){
                if (visited[j]) continue;
                int dist = abs(points[u][0] - points[j][0]) + abs(points[u][1] - points[j][1]);
                if(dist < minDist[j])minDist[j] = dist;
            }
        }
        return res;
    }
};
```
# kruskal
## kruskal的基本思想
Kruskal 也是一种贪心算法。
先把图中所有==边按权值从升序排序==，再依次遍历每条边，如果这条边连接的两个点还==不连通（用并查集判断连通）==，就选择这条边，并用并查集合并两个点所在的集合。
一直重复，直到选够 \(n-1\) 条边，构成最小生成树。
	每次考虑的都是全图当前权值最小的边，但不是所有边都会选。若这条边的两个端点已经连通，加入它就会形成环，因此要跳过
### 连接所有点的最小费用
```cpp
class Solution {
public:
    struct Edge{
        int u, v, w;
    };
    //并查集模板函数
    vector<int> father;
    void init(const int n){
        father.resize(n);
        for(int u = 0; u < n; u++){
            father[u] = u;
        }
    }
    int find(int u){
        if(u == father[u])return u;
        else father[u] = find(father[u]);
        return father[u];
    }
    bool isSame(int u, int v) {
        u = find(u);
        v = find(v);
        return u == v;
    }   
    void join(int u, int v){
        u = find(u);
        v = find(v);
        if(u == v)return;
        father[v] = u;
    }
    //sort
    static bool cmp(const Edge& a, const Edge& b){
        return a.w < b.w;
    }
    int minCostConnectPoints(vector<vector<int>>& points) {
        vector<Edge> edges;
        int n = points.size();
        int res = 0;
        //初始化结构体
        for(int i = 0; i < n; i++){
            for(int j = i+1; j < n; j++){
                int w = abs(points[i][0] - points[j][0]) + abs(points[i][1] - points[j][1]);
                edges.push_back({i, j, w}); 
            }
        }
        //按距离升序排序
        sort(edges.begin(), edges.end(), cmp);
        //利用并查集，筛选不联通的两点
        init(n);
        for(auto e : edges){
            if(isSame(e.u, e.v))continue;
            join(e.u, e.v);
            res += e.w;
        }
        return res;
    }
};
```