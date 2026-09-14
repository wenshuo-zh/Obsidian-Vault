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