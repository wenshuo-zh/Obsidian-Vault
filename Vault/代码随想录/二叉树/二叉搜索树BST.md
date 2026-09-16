左子树所有节点值 < 当前值
右子树所有节点值 > 当前值
左右子树也是BST
## 二叉搜索树的搜索

**递归**
```cpp
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        if(!root)return nullptr;
        if(root->val == val)return root;
        else if(root->val > val)return searchBST(root->left, val);
        else return searchBST(root->right, val);
    }
};
```
**迭代**
```cpp
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        while(root){
            if(root->val == val)return root;
            else if(root->val > val)root = root->left;
            else root = root->right;
        }
        return nullptr;
    }
};
```
## 验证二叉搜索树

利用特性：==BST的中序遍历是升序数组==
**递归** 
```cpp
class Solution {
public:
    vector<int> vec;
    void traversal(TreeNode* node){
        if(node == nullptr)return;
        traversal(node->left);
        vec.push_back(node->val);
        traversal(node->right);
    }
    bool isValidBST(TreeNode* root) {
        traversal(root);
        for(int i = 0; i < vec.size() - 1; i++){
            if(vec[i] >= vec[i + 1])return false;
        }
        return true;
    }
};
```
**迭代**
```cpp
class Solution {
public:
    bool isValidBST(TreeNode* root) {
        stack<TreeNode*> st;
        vector<int> vec;
        TreeNode* cur = root;
        while(cur != nullptr || !st.empty()){
            while(cur){
                st.push(cur);
                cur = cur->left;
            }
            cur = st.top();
            st.pop();
            vec.push_back(cur->val);
            cur = cur->right;
        }
        for(int i = 1; i < vec.size(); i++){
            if(vec[i - 1] >= vec[i])return false;
        }
        return true;
    }
};
```
## 二叉搜索树的最小绝对差

**递归：**
```cpp
class Solution {
public:
    int res = INT_MAX;
    TreeNode* pre = nullptr;
    void traversal(TreeNode* node){
        if(node == nullptr)return;
        traversal(node->left);
        if(pre) res = min(res, node->val - pre->val);
        pre = node;
        traversal(node->right);
    }
    int getMinimumDifference(TreeNode* root) {
        traversal(root);
        return res;
    }
};
```
## 二叉搜索树的众数
- 递归遍历树时，用全局pre和参数cur的形式可以拿到当前节点和上一个节点
- 众数可能不止有一个，所以每遇到和maxCount相等时，把元素加到结果集；如果count大于全局记录的maxCount，就把结果集清空，重新加入元素
```cpp
class Solution {
public:
    TreeNode* pre = nullptr;
    int count = 0;
    int maxCount = 0;
    vector<int> res;
    void searchBST(TreeNode* cur){
        if(!cur)return;
        searchBST(cur->left);
  
        if(!pre)count = 1;
        else if(pre->val == cur->val)count++;
        else count = 1;
        pre = cur;
        if(count == maxCount)res.push_back(cur->val);
        if(count > maxCount){
            res.clear();
            res.push_back(cur->val);
            maxCount = count;
        }
        searchBST(cur->right);
    }
    vector<int> findMode(TreeNode* root) {
        searchBST(root);
        return res;
    }
};
```
## BST的最近公共祖先

从根节点开始，值比p、q都大就去递归左树，比p、q都小去递归右树，第一次找到值在p、q中间的就是公共祖先
```cpp
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q) {
        if(root->val > p->val && root->val > q->val)
            return lowestCommonAncestor(root->left, p, q);
        else if(root->val < p->val && root->val < q->val)
            return lowestCommonAncestor(root->right ,p ,q);
        else return root;
    }
};
```
## BST的插入元素

**递归**
```cpp
class Solution {
public:
    TreeNode* insertIntoBST(TreeNode* root, int val) {
        if(!root){
            TreeNode* node = new TreeNode(val);
            return node;
        }
        if(root->val > val)root->left = insertIntoBST(root->left, val);
        if(root->val < val)root->right = insertIntoBST(root->right, val);
        return root;
    }
};
```
**迭代**
```cpp
class Solution {
public:
    TreeNode* insertIntoBST(TreeNode* root, int val) {
        TreeNode* cur = root;
        TreeNode* pre = cur;
        while(cur){
            pre = cur;
            if(cur->val > val)cur = cur->left;
            else if(cur->val < val)cur = cur->right;
        }
        TreeNode* insert = new TreeNode(val);
        if(!pre)return insert;
        if(pre->val > val)pre->left = insert;
        else pre->right = insert;
        return root;
    }
};
```
## 删除BST中的节点

**值替换法：** 左右都不为空，删除后接修改后的右子树：把最小值放到删除了的节点，再递归删除最小值
```cpp
class Solution {
public:
    TreeNode* deleteNode(TreeNode* root, int key) {
        if(!root)return root;

        if(root->val > key)root->left = deleteNode(root->left, key);
        else if(root->val < key)root->right = deleteNode(root->right, key);
        else{
            //左空右空
            if(!root->left && !root->right)return nullptr;
            //左空右不空,返回右子树
            if(!root->left && root->right)return root->right;
            //右空左不空，返回左子树
            if(root->left && !root->right)return root->left;
            //左右都不空，找右子树最小节点
            TreeNode* min = root->right;
            while(min->left)min = min->left;
            root->val = min->val;
            root->right = deleteNode(root->right, root->val);          
        }
        return root;
    }
};
```
**子树搬迁法：** 把待删节点的左子树接到待删节点的右子树的最左边
```cpp
class Solution {
public:
    TreeNode* deleteNode(TreeNode* root, int key) {
        if(!root)return root;
        
        if(root->val > key)root->left = deleteNode(root->left, key);
        else if(root->val < key)root->right = deleteNode(root->right, key);
        else{
            if(!root->left && !root->right)return nullptr;
            if(!root->left && root->right)return root->right;
            if(root->left && !root->right)return root->left;
            //找到待删节点
            TreeNode* cur = root->right;
            while(cur->left)cur = cur->left;
            cur->left = root->left;
            return root->right;
        }
        return root;
    }
};
```
## 修剪二叉搜索树

**递归**
```cpp
class Solution {
public:
    TreeNode* trimBST(TreeNode* root, int low, int high) {
        if(!root)return root;
        if(root->val < low)return trimBST(root->right, low, high);
        if(root->val > high)return trimBST(root->left, low, high);
        root->left = trimBST(root->left, low, high);
        root->right = trimBST(root->right, low, high);
        return root;
    }
};
```
**迭代：** 先让root在\[low, high]区间，再裁剪左子树，把小于low的节点剪掉，然后裁剪右子树
```cpp
class Solution {
public:
    TreeNode* trimBST(TreeNode* root, int low, int high) {
        if(!root)return nullptr;
        while (root && (root->val < low || root->val > high)) {
            if (root->val < low) root = root->right;
            else root = root->left;
        }
        TreeNode* cur = root;
        if(!root)return nullptr;
        while(cur->left){
            if(cur->left->val < low)cur->left = cur->left->right;
            else cur = cur->left;
        }
        cur = root;
        while(cur->right){
            if(cur->right->val > high)cur->right = cur->right->left;
            else cur = cur->right;
        }
        return root;
    }
};
```
## 根据有序数组构造BST

**递归：** 升序数组的中间元素一定是根，利用分割数组一步步递归
```cpp
class Solution {
public:
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        if(nums.empty()) return nullptr;
        int rootIndex = nums.size() / 2;
        TreeNode* root = new TreeNode(nums[rootIndex]);
        vector<int> leftNums(nums.begin(), nums.begin() + rootIndex);
        vector<int> rightNums(nums.begin() + rootIndex + 1, nums.end());
        root->left = sortedArrayToBST(leftNums);
        root->right = sortedArrayToBST(rightNums);
        return root;
    }
};
```
## 把BST转换为累加树
 利用反中序遍历即可
**递归法：**
```cpp
class Solution {
public:
    int pre = 0;
    void traversal(TreeNode* node){
        if(!node)return;
        traversal(node->right);
        node->val += pre;
        pre = node->val;
        traversal(node->left);
    }
    TreeNode* convertBST(TreeNode* root) {
        traversal(root);
        return root;
    }
};
```
**迭代法：**
```cpp
class Solution {
public:
    TreeNode* convertBST(TreeNode* root) {
        if(!root)return root;
        TreeNode* cur = root;
        stack<TreeNode*> st;
        int pre = 0;
        while(!st.empty() || cur){
            while(cur){
                st.push(cur);
                cur = cur->right;
            }
            cur = st.top();
            st.pop();
            cur->val += pre;
            pre = cur->val;
            cur = cur->left;
        }
        return root;
    }
};
```