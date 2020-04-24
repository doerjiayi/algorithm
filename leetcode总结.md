#贪心
##[盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water)
矮边是乘积的成员，而宽度是一直减少的，贪心移动矮边

class Solution {
public:
    int maxArea(vector<int>& height) {
        int left = 0;int right = height.size()-1;
        int nMax = 0;
        while(left<right){
            nMax = max(nMax, min(height[left], height[right])*(right-left));
            if(height[left]<height[right])left++;
            else right--;
        }
        return nMax;
    }
};
##[摆动序列](https://leetcode-cn.com/problems/wiggle-subsequence)
方向不同的才计数

class Solution {
public:
    int wiggleMaxLength(vector<int>& nums)
    {
        if (nums.size() == 0)return 0;
        int op = 0, g = 1;
        for (int i = 1; i < nums.size(); i++) {
            if (nums[i] > nums[i - 1])
            {
                if (op <= 0)g++;
                op = 1;                 
            }
            else if (nums[i] < nums[i - 1])
            {
                if (op >= 0)g++;
                op = -1;
            }
        }
        return g;
    }
};
##[最接近的三数之和](https://leetcode-cn.com/problems/3sum-closest)    
class Solution {
public:
    int threeSumClosest(vector<int>& nums, int target)
    {
        if (nums.size() < 3)return 0;
        sort(nums.begin(),nums.end());
        int res(0),diff(INT_MAX);
        for(int i = 0;i < nums.size()-2;++i)
        {
            int left = i + 1,right = nums.size()-1;
            while(left < right)//贪心遍历
            {
                int s = nums[i] + nums[left] + nums[right];
                if (s == target)return s;
                int d = abs(target - s);
                if (d < diff)
                {
                    res = s;
                    diff = d;
                }
                if (s < target)++left;
                else --right;
            }
        }
        return res;
    }
};
##[搜索二维矩阵 II](https://leetcode-cn.com/problems/search-a-2d-matrix-ii)
 
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        if (matrix.size() == 0 || matrix[0].size() == 0)return false;
        int m = matrix.size();
        int n = matrix[0].size();
        int x = m - 1;
        int y = 0;
        for(;x >= 0 && y < n;)
        {
            if (matrix[x][y] < target)//贪心
            {
                ++y;
            }
            else if (matrix[x][y] > target)
            {
                --x;
            }
            else return true;
        }
        return false;
    }
};
##[最长连续序列](https://leetcode-cn.com/problems/longest-consecutive-sequence)
class Solution {
public:
    int longestConsecutive(vector<int>& nums)
    {
        if (nums.size() <1)return nums.size();
        unordered_set<int> st(nums.begin(),nums.end());//哈希表
        int res(0),left,right;
        for(auto n:nums)
        {
            if (st.count(n - 1) == 0)//只计算左边界开始的
            {
                right = left = n;
                while(st.count(right+1))//贪心算法
                {
                    ++right;
                }
                res = max(res,right - left + 1);
            }
        }
        return res;
    }
};
##[最大子序和](https://leetcode-cn.com/problems/maximum-subarray)    
class Solution {
public:
    int maxSubArray(vector<int>& nums)
    {
        if (nums.size() == 0) return 0;
        int res = nums[0],sum(0);
        for(auto& n:nums)
        {
            sum += n;
            res = max(res,sum);//贪心
            if (sum < 0)sum = 0;//断开不需要的
        }
        return res;
    }
};

#分治
##[单词拆分 II](https://leetcode-cn.com/problems/word-break-ii)    

class Solution {
public:
    unordered_map<string,vector<string>> m;//复杂度 K * n
    vector<string> wordBreak(string s, vector<string>& wordDict)
    {
        if (s.size() == 0)return {""};//空的为收敛条件
        if (m.count(s))return m[s];
        vector<string> res;
        for(auto& w:wordDict)
        {
            if (s.substr(0,w.size()) == w)
            {
                vector<string> subv =
wordBreak(s.substr(w.size()),wordDict);//分治
                for(auto& t:subv)
                {
                     res.push_back(t.size() ? w + string(" ") + t : w + t);
                }
            }
        }
        return m[s] = res;//从后往前收敛，数组已是本子串的所有情况
    }
};
##[扁平化嵌套列表迭代器](https://leetcode-cn.com/problems/flatten-nested-list-iterator)   
class NestedIterator {
public:
    vector<int> nums;
    int cnt = 0;
    NestedIterator(vector<NestedInteger> &nestedList) {
        flat(nestedList);
    }
    void flat(vector<NestedInteger> &nestedList)//递归追加到列表
    {
        for(auto & i:nestedList)
        {
            if (i.isInteger())
            {
                nums.push_back(i.getInteger());
            }
            else
            {
                flat(i.getList());
            }
        }
    }
    int next() {
        return nums[cnt++];
    }
    bool hasNext() {
        return cnt < nums.size();
    }
};
##[为运算表达式设计优先级](https://leetcode-cn.com/problems/different-ways-to-add-parentheses) 
class Solution {
public:
    //1.按照运算符做分割，然后用分治算法解。
    //2.边界条件为：如果剩下的的字符串中没有运算符，即剩下的字符串中有且仅有数字。
    vector<int> diffWaysToCompute(string input) {
        vector<int> res;
        for(int i=0;i<input.size();i++)
        {
            char c = input[i];
            if(c=='+'||c=='-'||c=='*')
            {
                auto res1 = diffWaysToCompute(input.substr(0,i));
                auto res2 = diffWaysToCompute(input.substr(i+1));
                for(int r1:res1)//所有的结果的组合
                {
                    for(int r2:res2)
                    {
                        if(c=='+') res.push_back(r1+r2);
                        else if(c=='-') res.push_back(r1-r2);  
                        else res.push_back(r1*r2);                        
                    }
                }
            }
        }
        if(res.empty()) res.push_back(stoi(input));
        return res;
    }
};
#dfs
（排列）结束的条件为遍历的长度
##[全排列](https://leetcode-cn.com/problems/permutations)  
选择一个，递归，恢复原来，不需要检查去重
class Solution {
public:
    vector<vector<int>> res;
    vector<vector<int>> permute(vector<int>& nums)
    {
        dfs(nums,0);
        return res;
    }
    //不含重复数组
    void dfs(vector<int>& nums,int pos)
    {
        if (pos == nums.size())
        {
            res.push_back(nums);
            return;
        }
        for(int i = pos;i < nums.size();++i)
        {
            swap(nums[pos],nums[i]);
            dfs(nums,pos+1);
            swap(nums[pos],nums[i]);
        }
    }
};

##[全排列 II](https://leetcode-cn.com/problems/permutations-ii)
选择一个，利用有序性去重，继续选下一个，为了保持顺序而不恢复，但递归需要拷贝
class Solution {
    vector<vector<int>> res;
public:
    vector<vector<int>> permuteUnique(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        dfs(nums, 0);
        return res;
    }
    //含重复数组，结果序列需要不重复
    void dfs(vector<int> nums, int pos) {//拷贝数组，为了不影响上层的数组
        if (pos == nums.size())
            res.push_back(nums);
        else {
            for (int i = pos; i < nums.size(); i++) {
                if (i != pos && nums[pos] == nums[i])
continue;   //不与相同的交换
                swap(nums[pos], nums[i]);
                dfs(nums, pos + 1);
            }
        }
    }
};
##[组合](https://leetcode-cn.com/problems/combinations)
结束的条件为达到目标，比如数量
路径为

class Solution {
public:
    vector<vector<int>> vv;
    vector<int> v;
    vector<vector<int>> combine(int n, int k) {
        vv.clear();
        v.clear();
        dfs(1,n,k);
        return vv;
    }
    void dfs(int pos,int n, int k)
    {
        for(int i = pos;i <= n;++i)
        {
            v.push_back(i);//选择
            if (v.size() == k)vv.push_back(v);
            else dfs(i + 1,n,k);
            v.pop_back();//不选择，需要放后面
        }
    }
};
##[子集](https://leetcode-cn.com/problems/subsets)    
dfs，选择或者不选择，没去重
class Solution {
public:
    vector<vector<int>> res;
    vector<int> path;
    vector<vector<int>> subsets(vector<int>& nums) {
        if (nums.size() == 0)return{};
        res.push_back(path);
        dfs(0,nums);
        return res;
    }
    void dfs(int pos ,vector<int>& nums)
    {
        for(int i = pos;i < nums.size();++i)
        {
            path.push_back(nums[i]);
            res.push_back(path);
            dfs(i+1,nums);
            path.pop_back();
        }
    }
};
##[子集 II](https://leetcode-cn.com/problems/subsets-ii)
dfs，利用有序性去重，去重在后面执行，因为需要先遍历
class Solution {
public:
    vector<int> path;
    vector<vector<int>> result;
    vector<vector<int>> subsetsWithDup(vector<int>& nums) {
        path.clear();
        result.clear();
        result.push_back(path);
        sort(nums.begin(),nums.end());
        dfs(nums,0,path,result);
        return result;
    }
    void dfs(vector<int> &nums,int pos,vector<int>&
path,vector<vector<int>> &result)
    {
        if(pos==nums.size())
            return ;
        for(int i=pos;i<nums.size();i++)
        {
            path.push_back(nums[i]);
            result.push_back(path);
            dfs(nums,i+1,path,result);
            path.pop_back();
            while(nums[i]==nums[i+1]) i++;//开始位置不用重复的数
        }
    }
};
##[组合总和](https://leetcode-cn.com/problems/combination-sum)
需要从小到大处理，因为使用了条件过滤（target >= candidates[i]）
class Solution {
public:
    vector<vector<int>> results;
    vector<int> result;
    vector<vector<int>> combinationSum(vector<int>& candidates, int
target) {
        sort(candidates.begin(), candidates.end());
        dfs(candidates,target,0);
        return results;
    }
    void dfs(vector<int>& candidates,int target, int index){
        if(target == 0){
            results.push_back(result);
            return;
        }
        for(int i = index; i < candidates.size() && target >= candidates[i];
++i)
        {
            result.push_back(candidates[i]);
            dfs(candidates,target - candidates[i], i);
            result.pop_back();
        }
    }
};
或者
class Solution {
public:
    vector<vector<int>> results;
    vector<int> result;
    vector<vector<int>> combinationSum(vector<int>& candidates, int
target) {
        //sort(candidates.begin(), candidates.end());
        dfs(candidates,target,0);
        return results;
    }
    void dfs(vector<int>& candidates,int target, int index){
        if(target == 0){
            results.push_back(result);
            return;
        }
        if (target > 0)
        {
            for(int i = index; i < candidates.size(); ++i)
            {
                result.push_back(candidates[i]);
                dfs(candidates,target - candidates[i], i);
                result.pop_back();
            }
        }
    }
};
##[组合总和 II](https://leetcode-cn.com/problems/combination-sum-ii)    
dfs,需要排序，利用有序性去重
class Solution {
public:
    vector<vector<int>> results;
    vector<int> result;
    vector<vector<int>> combinationSum2(vector<int>& candidates, int
target) {
        sort(candidates.begin(), candidates.end());
        dfs(candidates,target,0);
        return results;
    }
    void dfs(vector<int>& candidates,int target, int index)
    {
        if(target == 0)
        {
            results.push_back(result);
            return;
        }
        for(int i = index; i < candidates.size() && target >=
candidates[i];)//需要从小到大才能这么判断
        {
            result.push_back(candidates[i]);//选择
            dfs(candidates,target - candidates[i], i+1);
            result.pop_back();//不选择
            ++i;
            while (i < candidates.size() && candidates[i] ==
candidates[i-1])++i;//本层去重
        }
    }
};
##
[组合总和 III](https://leetcode-cn.com/problems/combination-sum-iii)
--------------------------------------------------------------------
class Solution {
public:
    vector<int> path;
    vector<vector<int>> result;
    vector<int> candidates = {1,2,3,4,5,6,7,8,9};
    vector<vector<int>> combinationSum3(int k, int n) {
        dfs(k,n,0,0);
        return result;
    }
    void dfs(int k,int sum,int pos,int s)
    {
        if (path.size() > k) return;
        if(s > sum) return;
        if(s==sum && path.size() == k)
        {
            result.push_back(path);
            return;
        }
        for(int i=pos;i<candidates.size();i++)
        {
            path.push_back(candidates[i]);
            dfs(k,sum,i+1,s+candidates[i]);
            path.pop_back();
        }
    }
};
（其他）

##[复原IP地址](https://leetcode-cn.com/problems/restore-ip-addresses)    

class Solution {
public:
    vector<string> restoreIpAddresses(string s) 
    {
        if(s.size() == 0)return {};
        vector<string> res;
        string ip;
        dfs(s,0,0,ip,res);
        return res;
    }
private:
    void dfs(string &str,int step,int index,const string& ip,vector<string> &res){
        if(step == 4 || index >= str.size())
        {
            if(step == 4 && index == str.size())
            {
                res.push_back(ip);
            }
            return;
        }
        string part;
        for(int i = 1;i <= 3 && index + i <= str.size();++i)
        {
            if(i != 1 && str[index] == '0')break;// 不能以0开始(单个0可以)
            part = str.substr(index,i);
            if(stoi(part) <= 255)
            {
                string tmp = step ? ip + "." + part:part;
                dfs(str,step+1,index+i,tmp,res);
            }
        }
    }
};
##[串联所有单词的子串](https://leetcode-cn.com/problems/substring-with-concatenation-of-all-words)    

class Solution {
public:
    vector<int> findSubstring(string s, vector<string>& words) {
        if (s.size() == 0 || words.size() == 0)return {};
        vector<int> res;
        unordered_map<string,int> wordsMap;
        int wl = words[0].size();
        for(auto & str:words)wordsMap[str]++;
        int wordNum = words.size();
        unordered_map<string,int> tempWordsMap;
        for(int i = 0;i < s.size() - wl * wordNum + 1;++i)
        {
            int k = i;
            string str = s.substr(k,wl);
            int counter(0);
            tempWordsMap.clear();
            while (wordsMap[str] > 0)
            {
                if (++tempWordsMap[str] > wordsMap[str])break;
                k += wl;   
                str = s.substr(k,wl);
                ++counter;
            }
            if (counter == wordNum)
            {
                res.push_back(i);
            }
        }
        return res;
    }
};
##[N皇后](https://leetcode-cn.com/problems/n-queens)

dfs遍历，一行一行的去遍历，每行只能一皇后，需要检查列和斜线。
到了最后一行就是收敛条件。然后把该结果放入结果集
class Solution {
public:
    vector<vector<string>> res;
    vector<vector<string>> solveNQueens(int n)
    {
        //每行皇后列位置，只需要记录每行的下标
        vector<int> cols(n,0);
        dfs(cols, n, 0);
        return res;
    }
private:
    void dfs(vector<int>& cols, int n, int x)
    {
        if(n == x)
        {
            vector<string> v(n, string(n, '.'));
            for(int i = 0; i < n; ++ i)
            {
                v[i][cols[i]] = 'Q';
            }
            res.push_back(v);
            return;
        }
        //一行中需要遍历所有的列
        for(cols[x] = 0; cols[x] < n; ++cols[x])
        {
            if(safe(cols, x))
            {
                dfs(cols, n, x + 1);//下一行
            }
        }
    }
    bool safe(vector<int> &cols, int x)
    {
        //检查之前的行的皇后
        for(int i = 0; i < x; ++i)
        {//列值是否相同   ,  是否在同一斜线
            if(cols[i] == cols[x] || abs(cols[x] - cols[i]) == abs(x - i))
            {
                return false;
            }
        }
        return true;
    }
};
##[括号生成](https://leetcode-cn.com/problems/generate-parentheses)    

class Solution {
public:
    vector<string> generateParenthesis(int n) {
        if (n == 0) return {};
        vector<string > ret;
        dfs(ret, "", n, n);
        return ret;
    }
    //利用二叉树递归思想
    void dfs(vector<string> &ret, string tmp, int left, int right)
    {
        if (0 == left && 0 == right)
        {
            ret.push_back(tmp);
            return;
        }
        if (left > 0)
            dfs(ret, tmp + '(', left - 1, right);
        if (left < right)
            dfs(ret, tmp + ')', left, right - 1);
    }
};
##[正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching)    

class Solution {
public:
    bool isMatch(string s, string p) 
    {
        return dp(s,p,0,0);
    }
    bool dp(string& s,string& p,int i,int j)
    {
        if(j >= p.size()) return i == s.size();
        bool j_match = i < s.size() && (s[i]==p[j] || p[j]=='.');
        if(j + 1 < p.size() && p[j + 1]=='*'){
            return dp(s,p,i,j+2)||(j_match &&dp(s,p,i+1,j));
        }
        return j_match && dp(s,p,i+1,j+1);
    }
};
#二分
##[有效的完全平方数](https://leetcode-cn.com/problems/valid-perfect-square) 

二分尝试
class Solution {
public:
    bool isPerfectSquare(int num) {
        int left = 1,right = num;
        long mid,tmp;
        while(left <= right)
        {
            mid = left + (right - left)/2;;
            tmp = mid * mid;
            if (tmp == num)return true;
            else if (tmp < num)left = mid + 1;
            else right = mid - 1;
        }            
        return false;
    }
};
##[第一个错误的版本](https://leetcode-cn.com/problems/first-bad-version)    

// Forward declaration of isBadVersion API.
bool isBadVersion(int version);
class Solution {
public:
    int firstBadVersion(int n) {
        if(n < 1)return -1;
        int low = 1,high = n;
        int mid;
        while(low + 1< high)// +1
条件是为了保证low比high至少小1，来确认第一个bad version
        {
            mid = low + (high - low) / 2;
            if(isBadVersion(mid))
            {
                high = mid;//需要包含该bad version
            }
            else
            {
                low = mid;//需要包含该good version
            }
        }
        if(isBadVersion(low))//需要先判断low
        {
            return low;
        }
        else if(isBadVersion(high))
        {
            return high;
        }
        return -1;
    }
};
##[搜索插入位置](https://leetcode-cn.com/problems/search-insert-position)    

class Solution {
public:
    int searchInsert(vector<int>& nums, int target) {
        int low = 0, high = nums.size()-1;
        while(low <= high) {
            int mid = (low+high)>>1;
            if(nums[mid] == target) 
                return mid;
            else if(nums[mid] > target) 
                high = mid-1;
            else
                low = mid+1;
        }
        return low;  
    }
};
##[矩形区域不超过 K 的最大数值和](https://leetcode-cn.com/problems/max-sum-of-rectangle-no-larger-than-k)    

class Solution {
public:
    int maxSumSubmatrix(vector<vector<int>>& matrix, int k) {
        if(matrix.size() ==0) return 0;  
        int row = matrix.size(), col = matrix[0].size();  
        int res =INT_MIN;  
        for(int i = 0; i < col; i++)  
        {  
            vector<int> rowSum(row, 0); 
            for(int j = i; j < col; j++)  
            {  
                set<int> st{0};  
                int curSum = 0, curMax = INT_MIN;
                for(int x = 0; x < row; x++)  // col[i,j] row[0,row]
                {  
                    rowSum[x] += matrix[x][j];  
                    curSum += rowSum[x];  
                    auto it = st.lower_bound(curSum - k);  //之前某行到当前行的矩阵值最接近k
                    if(it != st.end()) 
                    {
                        curMax = max(curSum - *it, curMax);  
                    }
                    st.insert(curSum);  
                }  
                res = max(curMax, res);  
            }  
        }  
        return res; 
    }
    //复杂度 n * n * m * log(m)
};
    /*
    思路: 一种naive的算法就是枚举每个矩形块, 时间复杂度为O((mn)^2), 可以做少许优化时间复杂度可以降低到O(mnnlogm), 其中m为行数, n为列数. 
先求出任意两列之间的所有数的和, 然后再枚举任意两行之间的和, 而我们优化的地方就在后者. 我们用s[x]来表示第x行从a列到b列的和. 遍历一遍从第0行到最后一行的求和数组, 并依次将其放到二叉搜索树中, 这样当我们知道了从第0行到当前行的和的值之后, 我们就可以用lower_bound在O(log n)的时间复杂度内找到能够使得从之前某行到当前行的矩阵值最接近k. 也就是说求在之前的求和数组中找到第一个位置使得大于(curSum - k), 这种做法的原理是在curSum之下规定了一个bottom-line, 在这上面的第一个和就是(curSum-val)差值与k最接近的数. 还需要注意的是预先为二叉搜索树加一个0值, 这种做法的原理是如果当前curSum小于k, 那么至少本身是一个潜在的解. 
    */
##[Pow(x, n)](https://leetcode-cn.com/problems/powx-n)    

class Solution {
public:
    double myPow(double x, int n) {
        double res(1.0);
        for(int k = n;k != 0;k/=2)
        {
            if (k % 2 == 1)
            {
                res *= x;    
            }
            x *= x;
        }
        return n < 0? 1/res:res;
    }
};

#连续序列

##[乘积最大子序列](https://leetcode-cn.com/problems/maximum-product-subarray)    

class Solution {
public:
    int maxProduct(vector<int>& nums) {
        int local_max = nums[0],local_min = nums[0],global_max = nums[0],tmp;
        for(int i = 1;i < nums.size();++i)
        {
            tmp = max(max(local_max * nums[i],local_min * nums[i]),nums[i]);
            local_min = min(min(local_max * nums[i],local_min * nums[i]),nums[i]);
            local_max = tmp;
            global_max = max(local_max,global_max);
        }
        return global_max;
    }
};
##二维数组

##[旋转图像](https://leetcode-cn.com/problems/rotate-image)    

class Solution {
public:
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
         for(int i = 0; i < n - 1; i++){//i只能取到 n - 2, 因为n - 1是对称轴
             for(int j = 0; j < n - 1 - i; j++){//j只能取到n - 1 - i, 在对称轴的左边
                 swap(matrix[i][j], matrix[n - 1 - j][n - 1 - i]);
             }
         }
         for(int i = 0; i < n / 2; i++){//i只能取到横向中间轴的上面
             for(int j = 0; j < n; j++){//j可以取到所有值
                 swap(matrix[i][j], matrix[n - 1 - i][j]);//按横向轴翻转，j不变；i变为n-1-i
             }
         }
    }
};
/*
首先以从对角线为轴翻转，然后再以x轴中线上下翻转即可得到结果，如下图所示(其中蓝色数字表示翻转轴)：
1  2  3　　　 　　 9  6  3　　　　　  7  4  1
4  5  6　　-->　　8  5  2　　 -->     8  5  2　　
7  8  9 　　　 　　7  4  1　　　　　  9  6  3
*/
##[Z 字形变换](https://leetcode-cn.com/problems/zigzag-conversion)    

class Solution {
public:
    /*
    分析一下每一层下标递增的规律，找到每个内层for循环 j更新的方法即可
第一层 和 最后一层 每次更新 (numRows - 1) * 2
中间层按照奇偶要么更新 （numRows - 1 - i）* 2 要么更新 i*2
L     D     R
E   O E   I I
E C   I H   N                     
T     S     G      
LEETCODEISHIRING
LDREOEIIECIHNTSG
*/
    string convert(string s, int numRows) {
        if (numRows == 1) return s;
        ostringstream oss;
        auto size = s.size();
        for (int i = 0; i < numRows; i++)
        {
            int cnt = 0;
            for (int j = i; j < size; )
            {
                //0 6 12 1 5 7 11 13 2 4 8 10 14 3 9 15 
                //L D R E O E I I E C I H N T S G 
                //cout << j << " "; 
                oss << s[j];
                if (i == 0 || i == numRows - 1)
                {
                    j = j + (numRows - 1) * 2;// (4 - 1 - 0) * 2 = 6
                }
                else
                {
                    if (cnt % 2 == 0)
                    {
                        j = j + (numRows - 1 - i) * 2;// (4 - 1 - 1) * 2 = 4
                    }
                    else
                    {
                        j = j + i * 2;// (4) 1 * 2 = 2
                    }
                }
                cnt++;
            }
        }
        return oss.str();
    }
};

#哈希表
##[两数之和](https://leetcode-cn.com/problems/two-sum)    
class Solution {
public:
    vector<int> twoSum(vector<int>& nums, int target) {
        vector<int> res;  
        unordered_map<int, int> mp;  
        for(int i = 0;i < nums.size();++i)
        {
            if (mp.find(target-nums[i]) != mp.end())
            {
                res.push_back(i);
                res.push_back(mp[target-nums[i]]);
                return res;
            }
            mp[nums[i]] = i;
        }
        return res;
    }
};

##[三数之和](https://leetcode-cn.com/problems/3sum)    

class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        if (nums.size() == 0) return {};
        sort(nums.begin(), nums.end());
        unordered_map<int,int> map;
        for (int i = 0; i < nums.size(); i++) {
            map[nums[i]] = i;
        }
        vector<vector<int> > ret;
        for (int i = 0; i < nums.size() - 2; i++) {
            if (nums[i] > 0) return ret;
            if (i > 0 && nums[i] == nums[i-1]) continue;
            for (int j = i+1; j < nums.size() - 1; j++) {
                if (j >i +1 && nums[j]==nums[j-1]) continue;
                int two = nums[i] + nums[j];
                auto it = map.find(-two);
                if (it!= map.end() && it->second > j) {
                    vector<int> r = {nums[i],nums[j],-two};
                    ret.push_back(r);
                }
            }
        }
        return ret;
    }
};

##[四数之和](https://leetcode-cn.com/problems/4sum)  

需要利用哈希表访问快，以及数组的有序性来去重
class Solution {
public:
    vector<vector<int>> fourSum(vector<int>& nums, int target) {
        vector< vector<int> > res;
        if (nums.size() < 4) return res;
        sort(nums.begin(), nums.end());
        unordered_map<int, vector<pair<int, int> > >
hmap;           //两数和之组合
        for (int i = 0; i < nums.size(); i ++)
            for (int j = i + 1; j < nums.size(); j ++)
                hmap[nums[i] + nums[j]].push_back(pair<int, int>(i, j));
        for (int i = 0; i < nums.size(); i ++)//a
        {
            if (i && nums[i] == nums[i-1])continue;
            for (int j = i + 1; j < nums.size(); j ++)//b
            {
                if (j > i + 1 && nums[j] == nums[j-1])continue;
                int gap = target - nums[i] - nums[j];
                if (hmap.find(gap) != hmap.end())
                {
                    auto &vec = hmap[gap];
                    for (int k = 0; k < vec.size(); k++)//c\\d
                    {
                        if (vec[k].first <= j)continue;
                        vector<int> v({nums[i], nums[j] ,nums[ vec[k].first ],
nums[ vec[k].second ] });//a≤b≤c≤d
                        if (res.size() && res.back() == v)continue;
                        res.push_back(v);             
                    }
                }
            }
        }
        return res;
    }
};

##[单词模式](https://leetcode-cn.com/problems/word-pattern)

class Solution {
public:
    bool wordPattern(string pattern, string str) {
        unordered_map<char, int> mP;
        unordered_map<string, int> mS;
        stringstream in(str);
        int i = 0;
        for (string word; in >> word; ++i) 
        {
            if (mP.find(pattern[i]) != mP.end() || mS.find(word) != mS.end()) 
            {
                if (mP[pattern[i]] != mS[word]) return false;
            }
            else 
            {
                mP[pattern[i]] = mS[word] = i + 1;
            }
        }
        return i == pattern.size();
    }
};

#动规

需要找到前面k-1个成员（或者第k-1个）与第k个成员的关系

##[单词拆分](https://leetcode-cn.com/problems/word-break)    

哈希表是为了快速访问，保存动规状态才能递推
class Solution {
public:
    bool wordBreak(string s, vector<string>& wordDict)
    {
        unordered_set<string> st(wordDict.begin(),wordDict.end());
        vector<bool> dp(s.size()+1,false);       
        dp[0] = true;
        for(int i = 1;i <=s.size();++i)
        {
            for(int j = 0;j < i;++j)
            {
                if (dp[j] && st.count(s.substr(j,i-j)))
                {
                    dp[i] = true;
                    break;
                }
            }
        }
        return dp[s.size()];
    }
};

##[不同的子序列](https://leetcode-cn.com/problems/distinct-subsequences)

class Solution {
public:
    /*
跟LCS类似，用双序列动态规划解决。
1. 设计：
dp[i][j]表示从第一个字符串前i个组成的子串转换为第二个字符串前j个组成的子串共有多少种方案。
2. 递推：
s[i - 1] == t[j - 1]， 则dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];
s[i - 1] != t[j - 1]，则dp[i][j] = dp[i - 1][j];
3. 边界条件：
dp[i][0] = 1（t没有字符只有一种方案）
4. 返回值：
dp[sz1][sz2]
    */
    int numDistinct(string s, string t) {// S 的子序列中 T 出现的个数
         int sz1 = s.size(), sz2 = t.size();
         vector<vector<long>> dp(sz1 + 1,vector<long>(sz2 + 1,0));
         for (int i = 0; i <= sz1; ++i) {
             dp[i][0] = 1;//边界条件,（t没有字符只有一种方案）
         }
         for (int i = 1; i <= sz1; ++i) {
             for (int j = 1; j <= sz2; ++j) {
                 if (s[i - 1] == t[j - 1]) {
                     dp[i][j] = dp[i - 1][j - 1] + dp[i - 1][j];//如(aatt,aat) 2= (aat,aa) 1 + (aat,aat) 1
                 }
                 else {
                     dp[i][j] = dp[i - 1][j];//如,(aate,aat) 1 = (aat,aat) 1
                 }
             }
         }
         return dp[sz1][sz2];
    }
};
 
##[三角形最小路径和](https://leetcode-cn.com/problems/triangle)    
 
class Solution {
public:
    /*
    可以用一个一维数组来存储自底向上的路径向量，路径向量初始化为三角形数阵底部向量，此后每计算一次，更新一次，空间复杂度为O(n),且不影响输入三角形数阵的原始数据
    */
    int minimumTotal(vector<vector<int>>& triangle) {
        int length=triangle.size();  
        if(length==0)return 0;//特殊情况处理，容错机制  
        if(length==1)return triangle[0][0];    
        vector<int> sum=triangle[triangle.size()-1];//初始化路径和存储向量，自底向上  
        for(int i=triangle.size()-2;i>=0;i--)//自底向上迭代  
        {  
            for(int j=0;j<triangle[i].size();j++)  
            {  
                sum[j]=min(triangle[i][j]+sum[j],triangle[i][j]+sum[j+1]);  
            }  
        }  
        return sum[0];  
    }
};

##[最大正方形](https://leetcode-cn.com/problems/maximal-square)    

class Solution {
public:
    int maximalSquare(vector<vector<char>>& matrix) {
        int ret=0;
        if(matrix.empty()||matrix[0].empty()){
            return ret;
        }
        int m=matrix.size();
        int n=matrix[0].size();
        vector<vector<int> > v(m,vector<int>(n,0));
        for(int i=0;i<m;i++){
            if(matrix[i][0]=='1'){
                v[i][0]=1;
                ret=1;
            }
        }
        for(int j=0;j<n;j++){
            if(matrix[0][j]=='1'){
                v[0][j]=1;
                ret=1;
            }
        }
        for(int i=1;i<m;i++){
            for(int j=1;j<n;j++){
                if(matrix[i][j]=='1'){
                    v[i][j]=min(v[i-1][j],min(v[i-1][j-1],v[i][j-1]))+1;
                    ret=max(ret,v[i][j]);
                }
            }
        }
        return ret*ret;
    }
    /*
    动态规划，从左上角开始，如果当前位置为1，那么到当前位置包含的最大正方形边长为左/左上/上的值中的最小值加一，因为边长是由短板控制的。注意返回的是面积
    */
};

##[爬楼梯](https://leetcode-cn.com/problems/climbing-stairs)    

class Solution {
public:
    int climbStairs(int n) {
        if (n <= 2)return n;
        int vec[3];
        vec[0] = 1;vec[1] = 2;//初始化值（n 为1和2 ）
        for(int i=3; i<=n; i++)//计算 n -2 次
        {//地推公式为： vec[n] = vec[n-1] + vec[n-2]
            vec[2] = vec[0]+vec[1];
            vec[0] = vec[1];//移动变量
            vec[1] = vec[2];
        }
        return vec[2];
    }
};

##[分割回文串](https://leetcode-cn.com/problems/palindrome-partitioning)    

class Solution {
public:
    //回溯
    vector<vector<string>> res;
    vector<string> tmp;
    vector<vector<string>> partition(string s) {
        if (s.size() == 0)return {};
        dfs(s,0);
        return res;
    }
    void dfs(string &s,int pos)
    {
        if (pos == s.size())
        {
            res.push_back(tmp);
            return;
        }
        for(int i = pos;i < s.size();++i)
        {
            if (IsPlalindrome(s,pos,i))
            {
                tmp.push_back(s.substr(pos,i-pos+1));
                dfs(s,i+1);
                tmp.pop_back();
            }
        }
    }
    bool IsPlalindrome(string &s,int b,int e)
    {
        for(;b <= e;++b,--e)
        {
            if (s[b] != s[e])return false;
        }
        return true;
    }

};

##[分割回文串 II](https://leetcode-cn.com/problems/palindrome-partitioning-ii)    

class Solution {
public:
    int minCut(string s) {
        int len = s.size();
        vector<vector<bool>> P(len,vector<bool>(len,false));
        int dp[len + 1];
        for (int i = 0; i <= len; ++i) {
            dp[i] = len - i - 1;
        }
        for (int i = len - 1; i >= 0; --i) {
            for (int j = i; j < len; ++j) {
                if (s[i] == s[j] && (j - i <= 1 || P[i + 1][j - 1])) {
                    P[i][j] = true;
                    dp[i] = min(dp[i], dp[j + 1] + 1);
                }
            }
        }
        return dp[0];
    }
};

##[格雷编码](https://leetcode-cn.com/problems/gray-code)

class Solution {
public:
    vector<int> grayCode(int n) 
    {
        vector<int> res;
        res.push_back(0);
        for(int i = 0;i < n;++i)
        {
            for(int j = res.size()-1;j >= 0;--j)
            {
                res.push_back((1 << i) + res[j]);
            }
        }
        return res;
    }
};

##[解码方法](https://leetcode-cn.com/problems/decode-ways)    

class Solution {
public:
    int numDecodings(string s) {
        if (s.length() == 0) return 0;  
        vector<int> nums(s.length()+1,0);
        nums[0] = 1;  
        for(int i=1; i<=s.length(); i++) {  
            if (s[i-1] != '0') nums[i] += nums[i-1];  
            if (i>1 && s[i-2] == '1') nums[i] += nums[i-2];  
            else if (i>1 && s[i-2] == '2' && s[i-1] >= '0' && s[i-1] <= '6') nums[i] += nums[i-2];  
        }  
        return nums[s.length()];  
    }
};

##[买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock)

class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int res(0),loc(0);
        int mi(INT_MAX);
        for(int i = 0;i < prices.size();++i)
        {
            mi = min(prices[i],mi);
            loc = max(loc,prices[i]-mi);
            res = max(res,loc);
        }
        return res;
    }
};

##[买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii)    

class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int res(0);
        for(int i = 1;i < prices.size();++i)
        {
            res += max(prices[i] - prices[i-1],0);
        }
        return res;
    }
};

##[买卖股票的最佳时机 III](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-iii)

class Solution {
public:
    int maxProfit(vector<int>& prices) {
        if (prices.size() == 0) return 0;
        int g[3] = {0};//g[j]为最多可进行j次交易的最大利润，此为全局最优
        int l[3] =
{0};//l[j]为最多可进行j次交易并且最后一次交易在最后一天卖出的最大利润，此为局部最优
        for (int i = 1; i < prices.size(); ++i) {
            int diff = prices[i] - prices[i-1];
            for (int j = 2; j >= 1; --j)
            {
                l[j] = max(g[j - 1] + max(diff, 0), l[j] +
diff);//需要从后往前计算，否则影响后面的计算
                g[j] = max(l[j], g[j]);
            }
        }
        return g[2];
    }
    /*
local[j] = max(global[j - 1] + max(diff, 0), local[j] + diff)
//局部最优值是比较前一天并少交易一次的全局最优加上大于0的差值，和前一天的局部最优加上差值中取较大值
global[j] = max(local[j], global[j]) //全局最优比较局部最优和前一天的全局最优
    */
};
 
##[编辑距离](https://leetcode-cn.com/problems/edit-distance)    
 
class Solution {
public:
    int minDistance(string word1, string word2) {
        if (word1.size() == 0 && word2.size() == 0)return 0;
        int size1 = word1.size(), size2 = word2.size();
        vector<vector<int>> dp(size1 + 1, vector<int>(size2 + 1, 0));
        for (int i = 0; i <= size1; i++) dp[i][0] = i;//边际条件
        for (int j = 0; j <= size2; j++) dp[0][j] = j;
        for (int i = 1; i <= size1; i++) {
            for (int j = 1; j <= size2; j++) {
                int replace = word1[i - 1] == word2[j - 1] ? dp[i - 1][j - 1] :
dp[i - 1][j - 1] + 1;//替换或相同
                int ins_del = min(dp[i][j - 1] + 1, dp[i - 1][j] +
1);//插入或者删除
                dp[i][j] = min(replace, ins_del);
            }
        }
        return dp[size1][size2];
    }
};
##[不同路径](https://leetcode-cn.com/problems/unique-paths)    

class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<vector<int>> array(m,vector<int>(n,1));  
        for(int i=1; i<m;i++)//直接从第二行、第二列开始计算
        {  
            for(int j=1;j<n;j++)
            {  
                array[i][j]=array[i-1][j]+array[i][j-1];  //dp,每个格子来源于左边和上边的格子之和
            }  
        }  
        return array[m-1][n-1];  
    }
};

##[不同路径 II](https://leetcode-cn.com/problems/unique-paths-ii)    

class Solution {
public:
    int uniquePathsWithObstacles(vector<vector<int>>& obstacleGrid) {
        if (!obstacleGrid.size() || !obstacleGrid[0].size())return 0;
        int m = obstacleGrid.size(), n = obstacleGrid[0].size();
        vector<vector<long long>> array(m,vector<long long>(n,0));  
        for(int i = 0;i < m && !obstacleGrid[i][0];++i)array[i][0] = 1;
        for(int i = 0;i < n && !obstacleGrid[0][i];++i)array[0][i] = 1;
        for(int i=1; i<m;i++){  
            for(int j=1;j<n;j++){  
                if (!obstacleGrid[i][j])
                array[i][j]=array[i-1][j]+array[i][j-1];  
            }  
        }  
        return array[m-1][n-1];
    }
};

##[最小路径和](https://leetcode-cn.com/problems/minimum-path-sum)    

class Solution {
public:
    int minPathSum(vector<vector<int>>& grid) {
        int m = grid.size();  
        int n = grid[0].size();  
        int paths[m][n];  
        paths[0][0] = grid[0][0];  
        for(int i = 1; i < m; ++i) paths[i][0] = paths[i - 1][0] +
grid[i][0];  
        for(int j = 1; j < n; ++j)  paths[0][j] = paths[0][j - 1] +
grid[0][j];  
        for(int i = 1; i < m; ++i)  
        for(int j = 1; j < n; ++j)  
        paths[i][j] = std::min(paths[i - 1][j], paths[i][j - 1]) + grid[i][j];  
        return paths[m - 1][n - 1];  
    }
};
 
##[跳跃游戏](https://leetcode-cn.com/problems/jump-game)    
 
class Solution {
public:
    bool canJump(vector<int>& nums) {
          int rightMost = 0;
          for (int i = 0; i < nums.size(); i++) {
            if (rightMost < i) break;
            rightMost = max(rightMost, i + nums[i]);
          }
          return rightMost >= (nums.size() - 1);
    }
};

##[跳跃游戏 II](https://leetcode-cn.com/problems/jump-game-ii)    

class Solution {
public:
    int jump(vector<int>& nums) {
        if (nums.size() <= 1) return 0;
        int curPos = 0,lastPos = 0,cnt = 0;
        for(int i = 0;i <= nums.size()-1;++i)
        {
            curPos = max(nums[i]+i,curPos);//最大距离
            if (i == lastPos)//超过位置才能计数
            {
                lastPos = curPos;
                cnt++;
                if (curPos>=nums.size()-1)break;//已到
            }
        }
        return cnt;
    }
};

##[最大整除子集](https://leetcode-cn.com/problems/largest-divisible-subset) 
class Solution {
public:
    /*
    这道题给了我们一个数组，让我们求这样一个子集合，集合中的任意两个数相互取余均为0，而且提示中说明了要使用DP来解。那么我们考虑，较小数对较大数取余一定不为0，那么问题就变成了看较大数能不能整除这个较小数。那么如果数组是无序的，处理起来就比较麻烦，所以我们首先可以先给数组排序，这样我们每次就只要看后面的数字能否整除前面的数字。定义一个动态数组dp，其中dp[i]表示到数字nums[i]位置最大可整除的子集合的长度，还需要一个一维数组parent，来保存上一个能整除的数字的位置，两个整型变量mx和mx_idx分别表示最大子集合的长度和起始数字的位置，我们可以从后往前遍历数组，对于某个数字再遍历到末尾，在这个过程中，如果nums[j]能整除nums[i], 且dp[i] < dp[j] + 1的话，更新dp[i]和parent[i]，如果dp[i]大于mx了，还要更新mx和mx_idx，最后循环结束后，我们来填res数字，根据parent数组来找到每一个数字
    */
    vector<int> largestDivisibleSubset(vector<int>& nums) {
        sort(nums.begin(), nums.end());
        vector<int> dp(nums.size(), 0), parent(nums.size(), 0), res;
        int mx = 0, mx_idx = 0;
        for (int i = nums.size() - 1; i >= 0; --i) {//从后往前计算，这样才能获得起始位置
            for (int j = i; j < nums.size(); ++j) {
                if (nums[j] % nums[i] == 0 && dp[i] < dp[j] + 1) {
                    dp[i] = dp[j] + 1;//dp[i]表示到数字nums[i]位置最大可整除的子集合的长度
                    parent[i] = j;//保存上一个能整除的数字的位置
                    if (mx < dp[i]) {
                        mx = dp[i];//最大子集合的长度
                        mx_idx = i;//起始数字的位置
                    }
                }
            }
        }
        for (int i = 0; i < mx; ++i) {
            res.push_back(nums[mx_idx]);
            mx_idx = parent[mx_idx];
        }
        return res;
    }
};

 
##[H指数](https://leetcode-cn.com/problems/h-index)    
 
坐标和数值的关系，利用了有序性
class Solution {
public:
    int hIndex(vector<int>& citations)
    {
        if (citations.size() == 0)return 0;
        sort(citations.begin(),citations.end());
        int h(0);
        for(int i = citations.size() -1;i >= 0;i--)
        {
            if (citations.size() - i <= citations[i])
            {
                ++h;
            }
            else break;
        }
        return h;
    }
};
 
##[H指数 II](https://leetcode-cn.com/problems/h-index-ii)
 
坐标和数值的关系，利用了有序性
class Solution {
public:
    int hIndex(vector<int>& citations)
    {
        if (citations.size() == 0)return 0;
        int h(0);
        for(int i = citations.size() -1;i >= 0;i--)
        {
            if (citations.size() - i <= citations[i])
            {
                h = max((int)citations.size() - i,h);
            }
        }
        return h;
    }
};
 
##[零钱兑换](https://leetcode-cn.com/problems/coin-change)  
   
第k个与前面某个的关系
class Solution {
public:
    int coinChange(vector<int>& coins, int amount)
    {
        if (!amount)return 0;
        if (coins.size() == 0)return -1;
        long dp[amount+1] = {0};
        for(long i = 1;i <= amount;++i)
        {
            for(auto& c:coins)
            {
                int j = i - c;
                if (j>= 0 && (j == 0||dp[j]))
                {
                    dp[i] = dp[i] ? min(dp[j] + 1,dp[i]):dp[j] + 1;
                }
            }
        }
        return dp[amount] > 0? dp[amount]:-1;
    }
};
 
##[打家劫舍](https://leetcode-cn.com/problems/house-robber)    
 
class Solution {
public:
    int rob(vector<int>& nums) {
        if (nums.size() == 0)return 0;
        int v1(0),v2 = nums[0],l,g;
        g = v2;
        for(int i = 1;i < nums.size();++i)
        {
            l = max(v1 + nums[i],v2);//局部最优
            v1 = v2;v2 = l;//局部推移
            g = max(g,l);//全局最优
        }
        return g;
    }
};
 
##[打家劫舍 II](https://leetcode-cn.com/problems/house-robber-ii)
 
第一个开始，倒数2结束；或者第二个开始，倒数一结束
class Solution {
public:
    int rob(vector<int>& nums) {
        if (nums.size() == 0)return 0;
        if (nums.size() == 1)return nums[0];
        return max(rob(nums,0,nums.size()-1),rob(nums,1,nums.size()));
    }
    int rob(vector<int>& nums,int b,int e)//[b,e)
    {
        int size = e - b;  
        if(size == 0) return 0;
        int v1 = 0,v2 = nums[b],l,g = nums[b];
        for(int i = b + 1;i < e;++i)
        {
            l = max(nums[i] + v1,v2);//局部最优
            v1 = v2;
            v2 = l;//局部推移
            g = max(l,g);//全局最优
        }
        return g;
    }
};
 
##[打家劫舍 III](https://leetcode-cn.com/problems/house-robber-iii)
 
dp。每个子树含取root，不取root两个状态的值，也是局部最优解；父树依赖子树状态来得到自身两状态，最终是全局最优解。
class Solution {
public:
    vector<int> getMoney(TreeNode* node) {  
        vector<int> ret(2, 0);  
        if(!node) return ret;  
        vector<int> lRet = getMoney(node->left);  
        vector<int> rRet = getMoney(node->right);  
        ret[0] = lRet[1] + rRet[1] + node->val;  //取root
        ret[1] = max(lRet[0], lRet[1]) + max(rRet[0], rRet[1]);//不取root  
        return ret;  
    }  
    int rob(TreeNode* root) {
        vector<int> ret = getMoney(root);  
        return max(ret[0], ret[1]);  
    }
};
 
##[最长上升子序列](https://leetcode-cn.com/problems/longest-increasing-subsequence)
 
class Solution {
public:
    int lengthOfLIS(vector<int>& nums) 
    {
        if(nums.size()==0)return 0; 
        int len=0;
        int dp[nums.size()];
        dp[0]=nums[0];
        for(int i=1;i<nums.size();i++)
        {
            if(nums[i]>dp[len])
            {
                dp[++len]=nums[i];
            }
            else
            {
                int j=lower_bound(dp,dp+len,nums[i])-dp;
                dp[j]=nums[i];
            }
        }
        return len+1;
    }
};
 
##[鸡蛋掉落](https://leetcode-cn.com/problems/super-egg-drop)
class Solution {
public:
    int superEggDrop(int K, int N) {
        vector<int> dp(K + 1, 0);
        int m(0);
        for (;dp[K] < N;m++)
        {
            for (int k = K; k > 0; --k)//从上往下计算
                dp[k] += dp[k - 1] + 1;//分成三段 上 、下 、本层
        }
        return m;
    }
};
 
#集合
##[天际线问题](https://leetcode-cn.com/problems/the-skyline-problem)    
 
 
##[常数时间插入、删除和获取随机元素](https://leetcode-cn.com/problems/insert-delete-getrandom-o1)    
class RandomizedSet {
public:
    /** Initialize your data structure here. */
    RandomizedSet() {
    }
    /** Inserts a value to the set. Returns true if the set did not already contain the specified element. */
    bool insert(int val) {
        auto p = s.insert(val);
        return p.second;
    }
    /** Removes a value from the set. Returns true if the set contained the specified element. */
    bool remove(int val) {
        auto i = s.erase(val);
        return i > 0;
    }
    /** Get a random element from the set. */
    int getRandom() {
        int n = rand() % s.size();
        auto iter = s.begin();
        for(int i = 0;i < n;++i)iter++;
        return *iter;
    }
    unordered_set<int> s;
};

#几何

##[矩形面积](https://leetcode-cn.com/problems/rectangle-area)    
 
class Solution {
public:
    int computeArea(int A, int B, int C, int D, int E, int F, int G, int H) {
        long area = (long) (C - A) * (D - B) + (long)(G - E) * (H - F);
        long width = max((long)min(C, G) - (long)max(A, E), (long)0);
        long height = max((long)min(D, H) - (long)max(B, F),(long) 0);
        return (int)(area - width * height);
    }
};
 
#[直线上最多的点数](https://leetcode-cn.com/problems/max-points-on-a-line)    
 
 
class Solution {
public:
    int maxPoints(vector<Point>& points) 
    {
        if (points.size() == 0)return 0;
        int res(0);
        for(int i = 0;i < points.size();++i)
        {
            map<pair<int,int>,int> counterM;
            int dup(0),local(0);
            for(int j = 0;j <points.size();++j)//计算过的就不用再计算
            {
                if (points[j].x == points[i].x && points[j].y == points[i].y)
                {
                    dup++;
                    continue;
                }
                pair<int,int> p = GetMin(points[i],points[j]);
                counterM[p]++;
                //printf("1)%d %d \n",p.first,p.second);
            }
            local = dup;
            for(auto c:counterM)
            {
                //printf("2)%d %d %d \n",local,c.second , dup);
                local = max(local,c.second+dup);
            }
            //printf("2)%d %d %d\n",res,dup,local);
            res = max(res,local);
        }
        return res;
    }
    pair<int,int> GetMin(Point &p1,Point &p2)
    {
        int dx = p1.x - p2.x;
        int dy = p1.y - p2.y;
        int dmin = GetMinCommon(dx,dy);
        return make_pair(dx/dmin,dy/dmin);
    }
    int GetMinCommon(int x,int y)
    {
        if (y == 0)return x;
        return GetMinCommon(y,x%y);
    }
};

#拓扑结构


 
##[课程表](https://leetcode-cn.com/problems/course-schedule)
 
利用了哈希表记录入度表，利用哈希表记录入度的出度，利用set来遍历
class Solution {
public:
    bool canFinish(int numCourses, vector<pair<int, int>>& prerequisites) {
        set<int> st;
        unordered_map<int,int> m;// 入度的数量
        unordered_map<int,vector<int>> dep;//出度的所有入度（依赖关系）
        for(auto& iter:prerequisites)
        {
            m[iter.first]++;//入度计数
            dep[iter.second].push_back(iter.first);
            st.insert(iter.first);//只是处理有依赖的课程
            st.insert(iter.second);
        }
        while(st.size())
        {
            int left = st.size();
            for(auto iter = st.begin();iter!=st.end();)
            {
                int learn = *iter;
                if (m[learn] == 0)//没有入度的
                {
                    //printf("1)%d\\n",learn);
                    for(auto i:dep[learn])//减少所有出度的入度（减少依赖）
                    {
                        if (m[i])m[i]--;
                    }
                    st.erase(iter++);
                }
                else
                {
                    iter++;                    
                }
            }
            if (left == st.size())return
false;//没有减少课程的则说明不能继续       
        }
        return true;
    }
};

 
#[课程表 II](https://leetcode-cn.com/problems/course-schedule-ii)
 
因为只需要返回一种，可以直接遍历

class Solution {
public:
    vector<int> findOrder(int numCourses, vector<pair<int, int>>& prerequisites) {
        unordered_map<int,int> inM;//入度列表
        set<int> courseSt;//课程
        unordered_map<int,vector<int>> depM;//出度列表
        for(auto &iter:prerequisites)
        {
            inM[iter.first]++;
            depM[iter.second].push_back(iter.first);
            courseSt.insert(iter.first);
            courseSt.insert(iter.second);
        }
        vector<int> res;
        for(int i = 0;i < numCourses;++i)
        {
            //printf("1)%d\n",i);
            if (!courseSt.count(i))res.push_back(i);//没有依赖的则先学习
        }
        while(courseSt.size())
        {
            int left = courseSt.size();
            for(auto iter = courseSt.begin();iter != courseSt.end();)
            {
                if (inM[*iter] == 0)
                {
                    //printf("2)%d\n",*iter);
                    res.push_back(*iter);
                    for(auto i:depM[*iter])
                    {
                        if (inM[i]) inM[i]--; 
                    }
                    courseSt.erase(iter++);
                }
                else 
                {
                    iter++;
                }
            }
            if (courseSt.size() == left)return {};//不能完成
        }
        return res;
    }
};


 
#[单词接龙 II](https://leetcode-cn.com/problems/word-ladder-ii)
 
拓扑图的方式一层层遍历获取临接节点，临接节点为修改一个字符就出现在字典内的；每遍历一层后再从字典删除该前一层的节点。每一个新的节点、新的路径，需要拷贝，因为需要发现所有的可能的路径（所有的有向无环图）。

class Solution {
public:
    vector<vector<string>> findLadders(string beginWord, string endWord,
vector<string>& wordList)
    {
        vector<vector<string>> res;
        unordered_set<string> dict(wordList.begin(), wordList.end());
        vector<string> path{beginWord};
        queue<vector<string>> paths;
        paths.push(path);
        int level = 1, minLevel = INT_MAX;
        unordered_set<string> words;
        while (paths.size()) {
            auto path = paths.front(); paths.pop();//每个path都处理
            if (path.size() > level) {//到了新的一层
                for (string w : words)
dict.erase(w);//处理掉前一层的词，否则会回环
                words.clear();
                level = path.size();
                if (level > minLevel)
break;//长度已超过最短长度，则表示已到最后一层
            }
            string last = path.back();
            for (int i = 0; i < last.size(); ++i) {
                string newLast = last;
                for (char ch = 'a'; ch <= 'z'; ++ch) {
                    newLast[i] = ch;
                    if (dict.count(newLast))
                    {
                        words.insert(newLast);//本层的单词
                        vector<string> nextPath = path;//新的路径的尝试
                        nextPath.push_back(newLast);
                        if (newLast == endWord) {
                            res.push_back(nextPath);
                            minLevel = level;
                        }
                        else paths.push(nextPath);
                    }
                }
            }
        }
        return res;
    }
};

 
##[单词接龙](https://leetcode-cn.com/problems/word-ladder) 
 
拓扑图的方式一层层遍历获取临接节点，临接节点为修改一个字符就出现在字典内的；每遍历一个临接节点就从字典删除该节点
class Solution {
public:
    int ladderLength(string beginWord, string endWord, vector<string>&
wordList)
    {
        if(beginWord.size() == 0|| endWord.size() == 0||beginWord.size() !=
endWord.size() || wordList.size() == 0)return 0;
        //拓扑图的思想
        unordered_set<string> ws(wordList.begin(),wordList.end());
        unordered_map<string,int> wl;//临接网
        list<string> l;//遍历的队列
        l.push_back(beginWord);
        wl[beginWord] = 1;
        while(l.size())
        {
            string w = l.front();
            l.pop_front();
            int level = wl[w];
            for(int i = 0;i < w.size();++i)
            {
                char c = w[i];
                for(int j = 'a';j <= 'z';++j)
                {
                    if (w[i] == (char)j)continue;
                    w[i] = (char)j;
                    if (ws.count(w) > 0)//测试所有的可能的临接节点
                    {
                        //printf("%s %d\\n",w.c_str(),level);
                        if (w == endWord) return level + 1;
                        ws.erase(w);
                        l.push_back(w);
                        wl[w] = level + 1;
                    }
                }
                w[i] = c;
            }
        }
        return 0;
    }
};

 
#前缀树
##[实现 Trie (前缀树)](https://leetcode-cn.com/problems/implement-trie-prefix-tree) 
 
本质上是多叉树，利用了当前的访问的状态能够依赖之前访问的状态
class Trie {
public:
    struct node
    {
        node():isWord(false){memset(children,0,sizeof(children));}
        ~node(){for(int i = 0;i < 26;++i){if (children[i]) {delete
children[i];children[i] = NULL; } }}
        node* children[26];
        bool isWord;
    };
    node *root;
    /** Initialize your data structure here. */
    Trie() {
        root = new node();
    }
    ~Trie(){delete root;}
    /** Inserts a word into the trie. */
    void insert(string word) {
        if (word.size() == 0)return;
        node *p = root;
        for(int i = 0;i < word.size();++i)
        {
            if (!p->children[word[i] - 'a']) p->children[word[i] - 'a'] = new
node();
            p = p->children[word[i] - 'a'];
        }
        p->isWord = true;
    }
    /** Returns if the word is in the trie. */
    bool search(string word)
    {
        if (word.size() == 0)return true;
        node *p = root;
        for(int i = 0;i < word.size();++i)
        {
            if (!p->children[word[i] - 'a']) return false;
            p = p->children[word[i] - 'a'];
        }
        return p->isWord;
    }
    /** Returns if there is any word in the trie that starts with the given
prefix. */
    bool startsWith(string prefix) {
        if (prefix.size() == 0)return true;
        node *p = root;
        for(int i = 0;i < prefix.size();++i)
        {
            if (!p->children[prefix[i] - 'a']) return false;
            p = p->children[prefix[i] - 'a'];
        }
        return true;
    }
};


 
##[添加与搜索单词 - 数据结构设计](https://leetcode-cn.com/problems/add-and-search-word-data-structure-design)    
 
class WordDictionary {
public:
   struct TrieNode {
    public:
        TrieNode *child[26];
        bool isWord;
        TrieNode() : isWord(false) 
        {
            memset(child,0,sizeof(child));
        }
    };
    shared_ptr<TrieNode> root = make_shared<TrieNode>();
    // Adds a word into the data structure.
    void addWord(const string &word) {
        TrieNode *p = root.get();
        for (auto &i : word) {
            if (!p->child[i - 'a']) p->child[i - 'a'] = new TrieNode();
            p = p->child[i - 'a'];
        }
        p->isWord = true;
    }
    // Returns if the word is in the data structure. A word could
    // contain the dot character '.' to represent any one letter.
     bool search(const string &word) {
        if (word.size() == 0)return false;
        return dfs(word, root.get(), 0);
    }
    bool dfs(const string &word, TrieNode *p, int pos) {
        if (pos == word.size()) return p->isWord;
        for(int i = pos;i < word.size();++i)
        {
            if (word[i] == '.')
            {
                for (auto &child : p->child) {
                    if (child && dfs(word, child, i + 1)) return true;
                }
                return false;
            }
            if (!p->child[word[i] - 'a'])return false;
            p = p->child[word[i] - 'a'];
        }
        return p->isWord;
    }
}; 

/*
这道题如果做过之前的那道 Implement Trie (Prefix Tree)
实现字典树(前缀树)的话就没有太大的难度了，还是要用到字典树的结构，唯一不同的地方就是search的函数需要重新写一下，因为这道题里面'.'可以代替任意字符，所以一旦有了'.'，就需要查找所有的子树，只要有一个返回true，整个search函数就返回true，典型的DFS的问题，其他部分跟上一道实现字典树没有太大区别
*/


#回溯
 


##[被围绕的区域](https://leetcode-cn.com/problems/surrounded-regions)    


##[单词搜索](https://leetcode-cn.com/problems/word-search)
 
回溯就是使用多个方向的dfs来搜索，过程中为了避免回环需要设置路径成员为特殊字符。

class Solution {
public:
    int height;int width;
    bool exist(vector<vector<char>>& board, string word) {
        height=board.size();
        if(height==0)  return false;
        width=board[0].size();
        for(int i=0;i<height;i++)
            for(int j=0;j<width;j++)
               if(dfs(board,word,i,j,0)) return true;
        return false;
    }
    bool dfs(vector<vector<char>>& board,string& word,int i,int j,int
pos)//从节点board[i][j]开始查找word
    {
        if(i<0||j<0||i>=height||j>=width||board[i][j]=='\\0'||board[i][j]!=word[pos])return
false;
        if(pos==word.size()-1)return true;
        char t=board[i][j];
        board[i][j]='\\0';
        if(dfs(board,word,i,j+1,pos+1)||
           dfs(board,word,i+1,j,pos+1)||
           dfs(board,word,i-1,j,pos+1)||
           dfs(board,word,i,j-1,pos+1))  return true;
        board[i][j]=t;
        return false;
    }
};


 
##[单词搜索 II](https://leetcode-cn.com/problems/word-search-ii)    
 
class Solution {
public:
    int height;int width;
    vector<string> findWords(vector<vector<char>>& board,
vector<string>& words) {
        vector<string> res;
        unordered_set<string> setRes;
        height=board.size();
        if(height==0)  return res;
        width=board[0].size();
        if (words.size() == 0)return res;
        for(const auto &w :words)
        {
            if (exist(board,w))setRes.insert(w);
        }
        for(auto &s:setRes)res.emplace_back(s);
        return res;
    }
    //find
    bool exist(vector<vector<char>>& board,const string &word) {
        for(int i=0;i<height;i++)
            for(int j=0;j<width;j++)
               if(backtrack(board,word,i,j,0)) return true;
        return false;
    }
    bool backtrack(vector<vector<char>>& board,const string& word,int i,int
j,int pos)//从节点board[i][j]开始查找word
    {
        if(i<0||j<0||i>=height||j>=width||board[i][j]==0||board[i][j]!=word[pos])return
false;
        if(pos==word.size()-1)return true;
        char t=board[i][j];
        board[i][j]=0;
        if(backtrack(board,word,i,j+1,pos+1)||
           backtrack(board,word,i+1,j,pos+1)||
           backtrack(board,word,i-1,j,pos+1)||
           backtrack(board,word,i,j-1,pos+1))  
        {
            board[i][j]=t;
            return true;
        }
        board[i][j]=t;
        return false;
    }
}; 

 
##[矩阵置零](https://leetcode-cn.com/problems/set-matrix-zeroes)    
 

 
##[矩阵中的最长递增路径](https://leetcode-cn.com/problems/longest-increasing-path-in-a-matrix)    
 

class Solution {
public:
    int m,n;
    int longestIncreasingPath(vector<vector<int>>& matrix) 
    {
        if (matrix.size() == 0||matrix[0].size() == 0)return 0;
        m = matrix.size();
        n = matrix[0].size();
        vector<vector<int>> dp(m,vector<int>(n,0));
        int maxL = 0;
        for(int i = 0;i < m;++i)
        {
            for(int j = 0;j < n;++j)
            {
                maxL = max(maxL,extend(i,j,matrix,dp));
            }
        }
        return maxL;
    }
    int extend(int i,int j,vector<vector<int>>& matrix,vector<vector<int>> &dp)
    {
        if (!dp[i][j])
        {
            dp[i][j] = 1;
            if (j + 1< n && matrix[i][j] < matrix[i][j+1])dp[i][j] = max(dp[i][j],1 + extend(i,j+1,matrix,dp));
            if (j - 1>= 0 && matrix[i][j] < matrix[i][j-1])dp[i][j] = max(dp[i][j],1 + extend(i,j-1,matrix,dp));
            if (i + 1< m && matrix[i][j] < matrix[i+1][j])dp[i][j] = max(dp[i][j],1 + extend(i+1,j,matrix,dp));
            if (i - 1>= 0 && matrix[i][j] < matrix[i-1][j])dp[i][j] = max(dp[i][j],1 + extend(i-1,j,matrix,dp));
        }
        return dp[i][j];
    }
};


 
##[岛屿的最大面积](https://leetcode-cn.com/problems/max-area-of-island)    
 
class Solution {
public:
    int maxAreaOfIsland(vector<vector<int>>& grid) {
        if (grid.size() == 0 ||grid[0].size() == 0)return 0;
        int maxArea(0);
        int x = grid[0].size();
        int y = grid.size();
        for(int i = 0;i < y;++i)
        {
            for(int j= 0;j < x;++j)
            {
                if (grid[i][j] == 1) maxArea = max(maxArea,findArea(i,j,x,y,grid));
            }
        }
        return maxArea;
    }
    int findArea(int m,int n,int x,int y,vector<vector<int>>& grid)
    {
        if (grid[m][n] != 1)return 0;
        int tmp(1);
        grid[m][n]=2;
        if (m + 1< y && grid[m + 1][n] == 1) {tmp += findArea(m+1,n,x,y,grid);}
        if (n + 1< x && grid[m][n+1] == 1) {tmp += findArea(m,n+1,x,y,grid);}
        if (m > 0 && grid[m-1][n] == 1) {tmp += findArea(m-1,n,x,y,grid);}
        if (n > 0 && grid[m][n-1] == 1) {tmp += findArea(m,n-1,x,y,grid);}
        return tmp;
    }
};


 
##[岛屿的个数](https://leetcode-cn.com/problems/number-of-islands)    
 
回溯就是使用多个方向的dfs来搜索，过程中为了避免回环需要设置路径成员为特殊字符。找到一个岛屿计数一次
class Solution {
public:
    int m,n,count = 0;
    int numIslands(vector<vector<char>>& grid) {
        if(grid.size() == 0 || grid[0].size() == 0)return 0;
        m = grid.size();
        n = grid[0].size();
        for(int i = 0;i < m;++i)
        {
            for(int j = 0;j < n;++j)
            {
                if (grid[i][j] == '1')
                {
                    ++count;
                    extend(grid,i,j);
                }
            }
        }
        return count;
    }
    void extend(vector<vector<char>>& grid,int i,int j)
    {
        if (i >= 0 && i < m && j >= 0 && j < n)
        {
            if (grid[i][j] == '1')
            {
                grid[i][j] = '2';
                extend(grid,i,j+1);
                extend(grid,i,j-1);
                extend(grid,i+1,j);
                extend(grid,i-1,j);
            }
        }
    }
};

#位操作
 

 
##[2的幂](https://leetcode-cn.com/problems/power-of-two)
 

class Solution {
public:
//n & (n-1) 就是为了 去掉最右边的位数 1.剩下的为0则为2 的幂
bool isPowerOfTwo(int n) {
	return (n>0) && !(n & (n-1));
}
}; 

 
##[4的幂](https://leetcode-cn.com/problems/power-of-four)
 
因为整形是4字节的，再利用1位的特性。
class Solution {
public:
    bool isPowerOfFour(int num) {
        if (num <= 0)return false;
        if(num & (num - 1)) return false; // 多个位
        return  (0x55555555 & num) > 0;
    }
};

 
##[位1的个数](https://leetcode-cn.com/problems/number-of-1-bits) 
每次去掉最后一位
class Solution {
public:
    int hammingWeight(uint32_t n) {
        int c(0);
        while(n)
        {
            n = n & (n-1);
            ++c;
        }
        return c;
    }
};

 
##[只出现一次的数字](https://leetcode-cn.com/problems/single-number)
异或两相同数会抵消，剩下的就是一次的
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int res(0);
        for(auto &n:nums)
        {
            res ^= n;
        }
        return res;
    }
};
 
##[只出现一次的数字 II](https://leetcode-cn.com/problems/single-number-ii)
 

遍历，记录出现一次的位，出现2次的位为之前出现一次且本数，算了2次后才能算3次，3次的为出现1次且2次
class Solution {
public:
    int singleNumber(vector<int>& nums) {
        int ones = 0;//记录只出现过1次的bits
        int twos = 0;//记录只出现过2次的bits
        int threes;
        for(int i = 0; i < nums.size(); i++){
            int t = nums[i];
            twos |= ones&t;//要在更新ones前面更新twos
            ones ^= t;
            threes = ones&twos;//ones和twos中都为1即出现了3次
            ones &= ~threes;//抹去出现了3次的bits
            twos &= ~threes;
        }
        return ones;
    }
};
 
##[只出现一次的数字 III](https://leetcode-cn.com/problems/single-number-iii)

异或操作让出现两次的会抵消了，剩下的两个数发异或，与其自身补码与操作，则是获取某数中的一位，
该位只存在于该数中；再重新遍历判断和异或所有数
class Solution {
public:
    vector<int> singleNumber(vector<int>& nums) {
        int AXORB = 0;
        for (int num : nums) {
            AXORB ^= num;
        }
        int bitFlag = AXORB & (~AXORB +
1);//与自身的补码是为了获取其中一位,并且该位只是存在于其中一个数中
        vector<int> res = vector<int>(2,0);
        for (int num : nums) {
            if ((num & bitFlag) == 0) {
                res[0] ^= num;
            } else {
                res[1] ^= num;
            }
        }
        return res;
    }
};

 

#常用结构
 
##链表
 （缓存）
##[LRU缓存机制](https://leetcode-cn.com/problems/lru-cache)
 
利用哈希表的快速访问，记录的是链表迭代器，因为迭代器需要被移动；链表的访问顺序判断访问热度
class LRUCache {
public:
    LRUCache(int capacity)
    {
        s = capacity;
    }
    int get(int key)
    {  
        auto it = m.find(key);
        if (it != m.end())
        {
            l.splice(l.begin(),l,it->second);
            return it->second->second;
        }
        return -1;
    }  
    void put(int key, int value)
    {  
        auto it = m.find(key);
        if (it != m.end())
        {
            l.splice(l.begin(),l,it->second);
            it->second->second = value;
        }
        else
        {
            if (l.size() + 1> s)
            {
                m.erase(l.back().first);
                l.pop_back();
            }
            l.push_front(make_pair(key,value));
            m[key] = l.begin();
        }
    }  
private:  
    unordered_map<int,list<pair<int,int>>::iterator> m;//key value (iter)
    list<pair<int,int>> l;//key value
    int s;
};
 
##[LFU缓存](https://leetcode-cn.com/problems/lfu-cache)
 
lfu（最不经常使用的淘汰掉算法）可以处理缓存污染问题（是指系统将不常用的数据从内存移到缓存，造成常用数据的挤出，降低了缓存效率的现象）
需要记录最少使用次数，需要用哈希表记录次数对应的键列表，再用哈希表记录键、值和次数，还有哈希表记录键在次数列表中的迭代器。
每次获取键则需要更新次数对应的键列表，和其键对应的迭代器（因为所在的列表变了）。
每次插入则先获取，有就更新值，没则可能需要移除旧的次数最小的和最少使用的键，然后插入新值，新值次数都为1.
class LFUCache {
public:
    LFUCache(int capacity) {
        cap = capacity;
        minfre = 0;
    }
    int get(int key)
    {
        if (keyVFre.count(key) == 0) return -1;   
        //处理键的使用频次
        int& kfre =
keyVFre[key].second;//键的使用次数，修改次数，记录键的使用次数迭代器
        freKeys[kfre].erase(keyFreIter[key]);//删除次数列表中的键的迭代器（为了效率才记录）
        freKeys[++kfre].push_back(key);
        keyFreIter[key] =
--freKeys[kfre].end();//不能没有记录键的次数的列表的跌代器，为了管理键的频次
        if (freKeys[minfre].size() == 0)
++minfre;//判断最小次数没有了就上升，因为是访问。贪心算法方式
        return keyVFre[key].first;
    }
    void put(int key, int value) {
        if (cap <= 0) return;
        if (get(key) != -1) {
            keyVFre[key].first = value;
            return;
        }
        if (keyVFre.size() >= cap)
        {//erase  fre         
            auto& l = freKeys[minfre];
            keyVFre.erase(l.front());
            keyFreIter.erase(l.front());
            l.pop_front();
            //printf("minfre %d\\n",minfre);
        }
        {//new one
            keyVFre[key] = {value, 1};
            freKeys[1].push_back(key);
            keyFreIter[key] = --freKeys[1].end();
            minfre = 1;
        }
    }
    int cap, minfre;
    unordered_map<int, pair<int, int>> keyVFre;// key : val fre
    unordered_map<int, list<int>> freKeys;// fre : keys
    unordered_map<int, list<int>::iterator> keyFreIter;// key : freKeys iter
};
 

（其他）

 
##[复制带随机指针的链表](https://leetcode-cn.com/problems/copy-list-with-random-pointer)    
 
class Solution {
public:
    Node* copyRandomList(Node* head) {
        if (!head)return NULL;
        shared_ptr<Node> dummy = make_shared<Node>();
        unordered_map<Node*,Node*> mp;//old => new
        for(Node* tmp = head,* tmpDummy = dummy.get();tmp;tmp = tmp->next)
        {
            tmpDummy->next = mp[tmp] = new Node(tmp->val,NULL,NULL);//先处理next指针，哈希表中节点映射的是新链表的节点
            tmpDummy = tmpDummy->next;
        }
        for(Node* tmp = head;tmp;tmp = tmp->next)
        {
            mp[tmp]->random = mp[tmp->random];//再处理random指针，哈希表中的random映射的是新链表的节点
        }
        return dummy->next;
    }
};
 

 
##[删除链表中的节点](https://leetcode-cn.com/problems/delete-node-in-a-linked-list)    
 
class Solution {
public:
    void deleteNode(ListNode* node) {
        ListNode* tmp = node->next;
        node->val = tmp->val;
        node->next = tmp->next;
        delete tmp;
    }
};

 
##[相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists)    
 
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if (!headA || !headB)return NULL;
        unordered_map<ListNode *,int> m;
        ListNode *tmp = headB;
        while(tmp)
        {
            m[tmp] = 1;
            tmp = tmp->next;
        }
        tmp = headA;
        while(tmp)
        {
            if (m[tmp] > 0)return tmp;
            tmp = tmp->next;
        }
        return NULL;
    }
};
 

 
##[对链表进行插入排序](https://leetcode-cn.com/problems/insertion-sort-list)    
 
##[排序链表](https://leetcode-cn.com/problems/sort-list)    
 
class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if (!head)return head;
        ListNode* cur = head;
        for(;cur;cur = cur->next)      
        {
            ListNode* mark = cur,* tmp = cur;
            while(tmp)
            {
                if (mark->val > tmp->val)
                {
                    mark = tmp;
                }
                tmp = tmp->next;
            }
            swap(cur->val,mark->val);
        }
        return head;
    }
};
 

 
##[对链表进行插入排序](https://leetcode-cn.com/problems/insertion-sort-list)    
 
class Solution {
public:
    ListNode* insertionSortList(ListNode* head) {
        if (!head)return NULL;
        int len(0);
        ListNode* temp = head;
        while (temp) 
        {
            ++len;
            temp = temp->next;
        }
        while (--len)
        {
            ListNode*pre = head;
            ListNode*node = pre->next;
            int tmplen = len;
            while (tmplen--)
            {
                if (pre->val > node->val)swap(pre->val,node->val);
                pre = pre->next;
                node = node->next;
            }
        }
        return head;
    }
};
 
##[环形链表](https://leetcode-cn.com/problems/linked-list-cycle)
 
使用快慢指针的方式遍历判断
class Solution {
public:
    bool hasCycle(ListNode *head)
    {
        ListNode *slow = head,*fast = head;
        while(fast && fast->next)
        {
            if (slow == fast->next || slow == fast->next->next)return true;
            slow = slow->next,fast = fast->next->next;
        }
        return false;
    }
};
 
##[环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii)
 
使用快慢指针找到重叠，重置慢指针，再找到重叠就是入环点
class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if( head == NULL || head->next == NULL )return NULL;
        ListNode* fp = head,* sp = head;  
        while( fp  && fp->next )
        {  
            sp = sp->next;  
            fp = fp->next->next;  
            if( fp == sp )
            {   
                break;  
            }  
        }  
        if( !fp || !fp->next )return NULL;  
        sp = head;  
        while( fp != sp )
        {  
            sp = sp->next;  
            fp = fp->next;  
        }  
        return sp;  
    }
};
 
##[重排链表](https://leetcode-cn.com/problems/reorder-list)
 
class Solution {
public:
    //先使用快慢指针将链表从中间分割成两段，然后后半段就地逆置．之后合并插入到前半段链表即可，时间复杂度O(n)。
    void reorderList(ListNode* head) {
        if(!head || !head->next) return;  
        ListNode *slow = head, *fast = head;  
        while(fast->next && fast->next->next)  
            slow = slow->next, fast = fast->next->next;  
        fast = slow->next, slow->next = NULL;  
        ListNode *p = fast;
        ListNode *q = fast->next;
        fast->next = NULL;  
        while(q)//翻转后半段  
        {  
            auto tem = q->next;  
            q->next = p;  
            p = q, q = tem;  
        }  
        q = head;  
        while(q && p)  //两个串联
        {  
            auto tem1 = q->next, tem2 = p->next;  
            p->next = q->next;  
            q->next = p;  
            q = tem1, p = tem2;  
        }  
    }
};
 
##[删除排序链表中的重复元素](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list)    
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        if(head == NULL)  
            return head;  
        ListNode* pre = head;  
        ListNode* cur = head->next;
        ListNode* del(NULL);
        while(cur!=NULL)  
        {  
            if(cur->val == pre->val)  
            {
                del = cur;
                pre->next = cur->next;
                cur = cur->next;
                delete del;
            }
            else
            {
                pre = cur;
                cur = cur->next; 
            }
        }  
        return head;  
    }
};
 
##[删除排序链表中的重复元素 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-list-ii)    
class Solution {
public:
    ListNode* deleteDuplicates(ListNode* head) {
        if (!head || !head->next)return head;
        ListNode* dummy = new ListNode(-1);
        ListNode* pre = dummy;
        ListNode* cur = head;
        pre->next = cur;
        bool boDel(false);
        while(cur && cur->next)
        {
            if (cur->val == cur->next->val)
            {
                ListNode* del = cur->next;
                cur->next = cur->next->next;
                delete del;
                boDel = true;
            }
            else
            {
                if (boDel)//之前删除了的,这里cur是最后一个
                {
                    ListNode* del = cur;
                    cur = cur->next;
                    delete del;
                    pre->next = cur;
                    boDel = false;
                }
                else
                {
                    pre = cur;
                    cur = cur->next;
                }
            }
        }
        if (boDel)//之前删除了的,这里cur是最后一个
        {
            ListNode* del = cur;
            cur = cur->next;
            delete del;
            pre->next = cur;
        }
        head = dummy->next;
        delete dummy;
        return head;
    }
};

##[分隔链表](https://leetcode-cn.com/problems/partition-list)    
 
class Solution {
public:
    ListNode* partition(ListNode* head, int x) {
        shared_ptr<ListNode> big = make_shared<ListNode>(0);
        shared_ptr<ListNode> small = make_shared<ListNode>(0);
        ListNode* bigtmp = big.get();
        ListNode* smalltmp = small.get();
        ListNode* temp = head;
        while (temp)
        {
            if (temp->val >= x)
            {
                bigtmp->next = temp;
                bigtmp = temp;
            }
            else 
            {
                smalltmp->next = temp;
                smalltmp = temp;
            }
            temp = temp->next;
        }
        bigtmp->next = NULL;
        smalltmp->next = big.get()->next;
        return small.get()->next;
    }
};
 
##[旋转链表](https://leetcode-cn.com/problems/rotate-list)    
 
连成一圈再断开，注意计算断开位置
class Solution {
public:
    ListNode* rotateRight(ListNode* head, int k) {
        if (!head || !head->next) return head;
		// 获取长度和末尾节点tail
		int len = 1;
		ListNode* tail = head;
		while (tail->next && len++) tail = tail->next;
		if (k % len == 0) return head;
		// 找到倒数第k%len(也是断开节点的)的前一个节点
		int pos = len - k % len;
		ListNode *pre = head;
		while (--pos) pre = pre->next;
		// 连成一个圈
		tail->next = head;
        //从pre位置处断开
		head = pre->next;
		pre->next = NULL;
		return head;
    }
};
 
##[移除链表元素](https://leetcode-cn.com/problems/remove-linked-list-elements)    
 
class Solution {
public:
    ListNode* removeElements(ListNode* head, int val) {
        if (head == NULL) return NULL;
        ListNode* d(NULL);
        ListNode* p = head;  
        while (p->next != NULL) {  
            if (p->next->val == val) 
            {
                d = p->next;
                p->next = p->next->next; 
                delete d;
            }
            else p = p->next;
        }  
        if (head->val == val){d = head;head = head->next;delete d;}
        return head;
    }
};
 
##[反转链表](https://leetcode-cn.com/problems/reverse-linked-list)    
 
class Solution {
public:
    ListNode* reverseList(ListNode* head) 
    {
        if (!head || !head->next)return head;
        ListNode* pre = NULL;
        ListNode* cur = head;
        while(cur)
        {
            ListNode* ne = cur->next;
            cur->next = pre;
            pre = cur;
            cur = ne;
        }
        return pre;
    }
};
 
##[反转链表 II](https://leetcode-cn.com/problems/reverse-linked-list-ii)    
 
class Solution {
public:
    ListNode* reverseBetween(ListNode* head, int m, int n) {
        if (head == NULL)  
            return NULL;  
        ListNode *beforeReverse = NULL;  
        ListNode *p = head;  
        for(int i = 0; i < m - 1; i++)  
        {  
            beforeReverse = p;  
            p = p->next;  
        }  
        ListNode *reverseH = p;  // q 1 p 2 
        ListNode *pPre = p; 
        p = p->next;  
        for(int i = m ; i < n; i++)  //2
        {  
            ListNode *pNext = p->next;        
            p->next = pPre;  
            pPre = p;  
            p = pNext;  
        }  
        reverseH->next = p;  // 2 => 5 
        if (beforeReverse)  
            beforeReverse->next = pPre;  
        else  
            head = pPre;  
        return head;  
    }
};

 
##[合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists)    
 
class Solution {
public:
    ListNode* mergeTwoLists(ListNode* l1, ListNode* l2) 
    {
        if (!l1 || !l2)return l1 ? l1 :l2;
        shared_ptr<ListNode> dummy(new ListNode(-1));
        ListNode* pre = dummy.get();
        while(l1 && l2)
        {
            if (l1->val <= l2->val)
            {
                pre->next = l1;
                l1 = l1->next;
            }
            else
            {
                pre->next = l2;
                l2 = l2->next;
            }
            pre = pre->next;
        }
        if (l2)pre->next = l2;
        else pre->next = l1;
        return dummy->next;
    }
};
 
##[合并K个排序链表](https://leetcode-cn.com/problems/merge-k-sorted-lists)    
 
class Solution {
public:
    ListNode * dummy;
    Solution(){dummy = new ListNode(-1);}
    ~Solution(){delete dummy;}
    ListNode* mergeKLists(vector<ListNode*>& lists) {
        if(lists.empty()){
            return nullptr;
        }
        list<ListNode*> l;
        for(auto h:lists)l.emplace_back(h);
        while(l.size() > 1){
            ListNode* l1 = l.front();
            l.pop_front();
            ListNode* l2 = l.front();
            l.pop_front();
            l.emplace_back(mergeTwoLists(l1,l2));
        }
        return l.front();
    }
    ListNode *mergeTwoLists(ListNode *l1, ListNode *l2) {
        if(!l1 || !l2) return l1? l1:l2;
        ListNode * tmp = dummy;
        while (l1 && l2)
        {
            if(l1->val <= l2->val)
            {
                tmp ->next = l1;
                l1 = l1->next;
            }
            else
            {
                tmp->next = l2;
                l2 = l2->next;
            }
            tmp = tmp->next;
        }
        if (l1) tmp->next = l1;
        else tmp->next = l2;
        return dummy->next;
    }
    /*
    考虑分治的思想来解这个题（类似归并排序的思路）。把这些链表分成两半，如果每一半都合并好了，那么我就最后把这两个合并了就行了。这就是分治法的核心思想。
但是这道题由于存的都是指针，就具有了更大的操作灵活性，可以不用递归来实现分治。就是先两两合并后在两两合并。。。一直下去直到最后成了一个。（相当于分治算法的那棵二叉树从底向上走了）。
第一次两两合并是进行了k/2次，每次处理2n个值,即2n * k/2 = kn 次比较。
第二次两两合并是进行了k/4次，每次处理4n个值,即4n * k/4 = kn 次比较。
。。。
最后一次两两合并是进行了k/(2^logk)次（=1次），每次处理2^logK  * N个值（kn个），即1*kn= kn 次比较。
所以时间复杂度：
O(KN* logK)
空间复杂度是O(1)。
    */
};
 
##[两两交换链表中的节点](https://leetcode-cn.com/problems/swap-nodes-in-pairs)    
class Solution {
public:
    ListNode* swapPairs(ListNode* head) {
        ListNode* dummy = new ListNode(-1);
        dummy->next = head;
        ListNode* pre = dummy;
        ListNode* cur = head;
        while (cur && cur->next)
        {
            ListNode* next = cur->next;
            cur->next = next->next;//先连接后面
            pre->next = next;//连接前面
            next->next = cur;//连接中间
            pre = cur;//继续
            cur = cur->next;
        }
        head = dummy->next;
        delete dummy;
        return head;
    }
};

##[k个一组翻转链表](https://leetcode-cn.com/problems/reverse-nodes-in-k-group)    
class Solution {
public:
    ListNode* dummy = nullptr;
    Solution(){}
    ~Solution(){if (dummy) {delete dummy;}}
    ListNode* reverseKGroup(ListNode* head, int k) {
        dummy = new ListNode(-1);
        dummy->next = head;
        ListNode* pre = dummy;
        ListNode* cur = head;
        list<ListNode*> vec;
        while (cur)
        {
            ListNode* tmp = cur;
            for(;tmp && vec.size() < k; tmp = tmp->next)
            {
                vec.emplace_back(tmp);
            }
            if(vec.size() == k)
            {
                while(vec.size())
                {
                    pre->next = vec.back();
                    pre = pre->next;
                    vec.pop_back();
                }
                pre->next = tmp;
            }
            cur = tmp;
        }
        head = dummy->next;
        return head;
    }
};

##[删除链表的倒数第N个节点](https://leetcode-cn.com/problems/remove-nth-node-from-end-of-list)    
class Solution {
public:
    ListNode* removeNthFromEnd(ListNode* head, int n) {
        if (!head)return head;
        int len(0);
        ListNode* tmp = head;
        while(tmp)
        {
            ++len;
            tmp = tmp->next;
        }
        tmp = head;
        if (n == len)
        {
            ListNode* del = head;
            head = del->next;
            delete del;
            return head;
        }
        for(int k = len - n -1; k > 0;--k)
        {
            tmp = tmp ->next;
        }
        ListNode* del = tmp ->next;
        if (del)
        {
            tmp->next = del->next;
            delete del;
        }
        return head;
    }
};

##[回文链表](https://leetcode-cn.com/problems/palindrome-linked-list)    
/**
 * Definition for singly-linked list.
 * struct ListNode {
 *     int val;
 *     ListNode *next;
 *     ListNode(int x) : val(x), next(NULL) {}
 * };
 */
class Solution {
public:
    bool isPalindrome(ListNode* head) {
        if (!head || !head->next)return true;
        int len(0);
        ListNode* tmp = head;
        while(tmp)
        {
            ++len;
            tmp = tmp->next;
        }
        tmp = head;
        ListNode* pre = head;
        int skip = len % 2 == 0 ? len /2 :len/2 +1;
        for(int i = 0;i < skip;++i)
        {
            pre = tmp;
            tmp = tmp->next;
        }
        ListNode* head2 = tmp;
        pre = head;
        ListNode* cur = pre->next;
        for(int i =  len/2 - 1;i > 0;--i)
        {
            ListNode* ne = cur->next;
            cur->next = pre;
            pre = cur;
            cur = ne;
        }
        head->next = NULL;
        while(pre)
        {
            if (pre->val != head2->val) return false;
            pre = pre->next;
            head2 = head2->next;
        }
        return true;
    }
};

##[奇偶链表](https://leetcode-cn.com/problems/odd-even-linked-list)    
class Solution {
public:
    ListNode* dummy1;
    ListNode* dummy2;
    Solution(){dummy1 = new ListNode(-1);dummy2 = new ListNode(-1);}
    ~Solution(){delete dummy1;delete dummy2;}
    ListNode* oddEvenList(ListNode* head) 
    {
        if (!head || !head->next)return head;
        ListNode* tmp1 = dummy1;
        ListNode* tmp2 = dummy2;
        int i(0);
        while(head)
        {
            if (i % 2 == 0)
            {
                tmp1->next = head;
                tmp1 = tmp1->next;
            }
            else
            {
                tmp2->next = head;
                tmp2 = tmp2->next;
            }
            ++i;
            head = head->next;
        }
        tmp2->next = NULL;
        tmp1->next = dummy2->next;
        return dummy1->next;
    }
};

##[两数相加](https://leetcode-cn.com/problems/add-two-numbers)    
 
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) 
    {
        if (!l1 && !l2)return NULL;
        ListNode* head = new ListNode(0);
        ListNode* tmp = head;
        int val(0);
        while (l1 || l2 || val)//由于链表本身也是从低到高位的，可以直接从头计算
        {
            val = (l1 ? l1->val:0 ) + (l2 ? l2->val:0) + val;
            head->next = new ListNode(val % 10);
            val /= 10;//进位
            if (l1)l1 = l1->next;
            if (l2)l2 = l2->next;
            head = head->next;
        }
        head = tmp->next;
        delete tmp;
        return head;
    }
};

 
##[两数相加 II](https://leetcode-cn.com/problems/add-two-numbers-ii)    
class Solution {
public:
    ListNode* addTwoNumbers(ListNode* l1, ListNode* l2) 
    {
        if (!l1)return l2;
        if (!l2)return l1;
        int len1(0),len2(0);
        ListNode* p1 = l1, *p2 = l2;
        while(p1)
        {
            ++len1;
            p1 = p1->next;
        }
        while(p2)
        {
            ++len2;
            p2 = p2->next;
        }
        p1 = l1, p2 = l2;
        vector<ListNode*> v;
        ListNode* pre =  new ListNode(-1);
        ListNode* dummy =  pre;
        if (len1 > len2)
        {
            p1 = l1;
            for(int i = 0;i < len1 - len2;++i)
            {
                pre->next = new ListNode(p1->val);
                pre = pre->next;
                p1 = p1->next;
                v.push_back(pre);
                //printf("1 %d\n",pre->val);
            }
        }
        else if (len1 < len2)
        {
            p2 = l2;
            for(int i = 0;i < len2 - len1;++i)
            {
                pre->next = new ListNode(p2->val);
                pre = pre->next;
                p2 = p2->next;
                v.push_back(pre);
                //printf("2 %d\n",pre->val);
            }
        }
        while(p1 && p2)
        {
            pre->next = new ListNode(p1->val+p2->val);
            pre = pre->next;
            p1 = p1->next;
            p2 = p2->next;
            v.push_back(pre);
            //printf("3 %d\n",pre->val);
        }
        int c(0);
        for(int i = v.size()-1;i >= 0;--i)
        {
            int n = v[i]->val + c;
            v[i]->val = n % 10;
            c = n /10;
        }
        if (c) 
        {
            ListNode* ne =  new ListNode(c);
            ne->next = dummy->next;
            dummy->next = ne;
        }
        pre = dummy->next;
        delete dummy;
        return pre;
    }
};

#栈
 
##[用队列实现栈](https://leetcode-cn.com/problems/implement-stack-using-queues)    
class MyStack {
    int cur;
    queue<int> q[2];
public:
    /** Initialize your data structure here. */
    MyStack():cur(0) {
    }
    /** Push element x onto stack. */
    void push(int x) {
         q[cur].push(x);
    }
    /** Removes the element on top of the stack and returns that element. */
    int pop() {
        while (q[cur].size() > 1) 
        {
             q[1 - cur].push(q[cur].front());
             q[cur].pop();
          }
          int t = q[cur].front();
          q[cur].pop();
          cur = 1 - cur;
          return t;
    }
    /** Get the top element. */
    int top() {
        while (q[cur].size() > 1) 
        {
             q[1 - cur].push(q[cur].front());
             q[cur].pop();
          }
          int t = q[cur].front();
          q[1 - cur].push(q[cur].front());
          q[cur].pop();
          cur = 1 - cur;
          return t;
    }
    /** Returns whether the stack is empty. */
    bool empty() {
        return q[cur].empty();
    }
};

##[用栈实现队列](https://leetcode-cn.com/problems/implement-queue-using-stacks)    
class MyQueue {
public:
    /** Initialize your data structure here. */
    MyQueue() {
    }
    /** Push element x to the back of queue. */
    void push(int x) {
        s.push(x);
    }
    /** Removes the element from in front of queue and returns that element. */
    int pop() {
        while(s.size() > 1)
        {
            tmp.push(s.top());
            s.pop();
        }
        int res = s.top();
        s.pop();
        while(tmp.size() > 0)
        {
            s.push(tmp.top());
            tmp.pop();
        }
        return res;
    }
    /** Get the front element. */
    int peek() {
        while(s.size() > 1)
        {
            tmp.push(s.top());
            s.pop();
        }
        int res = s.top();
        while(tmp.size() > 0)
        {
            s.push(tmp.top());
            tmp.pop();
        }
        return res;
    }
    /** Returns whether the queue is empty. */
    bool empty() {
        return s.size() == 0;
    }
    stack<int> s;
    stack<int> tmp;
};

##[最小栈](https://leetcode-cn.com/problems/min-stack)    
 
class MinStack {
public:
    /** initialize your data structure here. */
    MinStack() {
    }
    void push(int x) {
        s.push(x);
        if (sMin.size() == 0 || sMin.top() >= x)sMin.push(x);
    }
    void pop() {
        int x = s.top();
        s.pop();
        if (x == sMin.top()) sMin.pop();
    }
    int top() {
        return s.top();
    }
    int getMin() {
        return sMin.top();
    }
    stack<int> s;
    stack<int> sMin;
};

 
##[有效的括号](https://leetcode-cn.com/problems/valid-parentheses)
 
class Solution {
public:
    bool isValid(string s)
    {
        int top=-1,index=0,length=s.size();  
        string stack;
        stack.resize(length,0);
        while(index < length)
        {  
            if(s[index]==')')//弹栈操作
            {  
                if(top>=0 && stack[top]=='(')top--;  
                else return false;  
            }
            else if(s[index]=='}')
            {  
                if(top>=0 && stack[top]=='{')top--;  
                else return false;  
            }
            else if(s[index]==']')
            {  
                if(top>=0 && stack[top]=='[')top--;  
                else return false;  
            }
            else stack[++top]=s[index];//压栈操作  
            index++;  
        }  
        return top==-1;  
    }
};

 
##[下一个更大元素 I](https://leetcode-cn.com/problems/next-greater-element-i)
 
class Solution {
public:
    vector<int> nextGreaterElement(vector<int>& findNums, vector<int>&
nums) {
        vector<int> res;
        stack<int> st;
        unordered_map<int, int> m;
        for (int num : nums) {
            //和其右边第一个较大数之间的映射，因为没有重复元素就可以这样处理
            while (st.size() && st.top() < num) {
                m[st.top()] = num; st.pop();
            }
            st.push(num);//每个都要压
        }
        for (int num : findNums) {
            res.push_back(m.count(num) ? m[num] : -1);
        }        
        return res;
    }
};

 
##[下一个更大元素 II](https://leetcode-cn.com/problems/next-greater-element-ii)    
 
class Solution {
public:
    vector<int> nextGreaterElements(vector<int>& nums) {
        int n = nums.size();
        vector<int> result(n, -1);
        stack<int> s;
        for (int i = 0; i < n; i++) {
            while (s.size() && nums[s.top()] < nums[i])
{//第一遍处理数在数组右边的第一个较大
                result[s.top()] = nums[i];
                s.pop();
            }
            s.push(i);//压下标，需要找回位置
        }
        for (int i = 0; i < n; i++) {
            while (s.size()&& nums[s.top()] < nums[i])
{//第二遍处理剩下的在循环数组右边的第一个较大
                result[s.top()] = nums[i];
                s.pop();
            }
        }
        return result;
    }
};

 
#堆


例如，常用数据处理方式
最小堆用于处理前k大
最大堆用于处理前k小
priority_queue<int,vector<int>,less<int>>
pq;//最大堆，默认为priority_queue<int>
priority_queue<int,vector<int>,greater<int>> pq;//最小堆
struct cmp 
{
        bool operator() (const pair<int, int> &a,const pair<int, int>
&b)const {return a.second > b.second;} //b.second优先级高，也就是在堆上面
};
priority_queue<pair<int,int>,vector<pair<int,int>>,cmp> pq;//最小堆
struct cmp{  
        bool operator() (const pair<string, int> &a,const pair<string, int>
&b)
        {  
            if (a.second != b.second) return a.second >
b.second;//b.second优先级高，也就是在堆上面
            return a.first < b.first;  //a.first优先级高，也就是在堆上面
        }  
    };
priority_queue<pair<string, int>, vector<pair<string, int>>,cmp>
pq;  //次数的最小堆（字母的最大堆）
set 也可以当做唯一性的最小堆（因为红黑树的有序性，map同理）


 
##[数据流的中位数](https://leetcode-cn.com/problems/find-median-from-data-stream)    
 
class MedianFinder {
public://priority_queue，  https://www.cnblogs.com/huashanqingzhu/p/11040390.html
    priority_queue<int> big;//大顶堆  ，等同于 priority_queue<int, vector<int>, less<int> >
    priority_queue<int,vector<int>,greater<int>> small;//小顶堆
    /** initialize your data structure here. */
    MedianFinder() {}
    void addNum(int num) {
        if (big.size() == small.size())//to big
        {
            if (small.size() && small.top() < num)
            {
                small.push(num);
                num = small.top();
                small.pop();
            }
            big.push(num);
        }
        else//to small
        {
            if (num < big.top())
            {
                big.push(num);
                num = big.top();
                big.pop();
            }
            small.push(num);
        }
    }
    double findMedian() {
        if (big.size() == 0)return 0.0;
        return big.size() > small.size() ?  big.top() :double(big.top() + small.top())/2;
    }
};
 
##[前K个高频元素](https://leetcode-cn.com/problems/top-k-frequent-elements)    
 
class Solution {
public:
    struct cmp//规则：pair默认的比较，先比较第一个元素，第一个相等比较第二个。修改为cmp
    {
        bool operator() (const pair<int, int> &a,const pair<int, int>
&b)const
        {
            return a.second > b.second;
        }
    };
    vector<int> topKFrequent(vector<int>& nums, int k) {
        if(nums.size() == 0)return {};
        vector<int> res;
        unordered_map<int,int> m;
        for(auto n:nums)m[n]++;
        priority_queue<pair<int,int>,vector<pair<int,int>>,cmp>
pq;//小顶堆，装最大的k个数
        for(auto i:m)
        {
            pq.push(i);//使用pair成员比较容易兼容map的遍历成员
            if (pq.size() > k) pq.pop();
        }
        while(pq.size())
        {
            res.push_back(pq.top().first);
            pq.pop();
        }
        return res;
    }
};

 
##[前K个高频单词](https://leetcode-cn.com/problems/top-k-frequent-words)    
 
class Solution {
public:
    struct cmp{  
        bool operator() (const pair<string, int> &a,const pair<string, int>
&b)
        {  //b.second优先级高(次数，小顶堆，需要保存次数比较大的)
            if (a.second != b.second) return a.second > b.second;
            return a.first < b.first;  //a.first优先级高(字符串,大顶堆，需要保存字符串值比较小的)
        }  
    };
    vector<string> topKFrequent(vector<string>& words, int k) {
        unordered_map<string, int> map;  
        priority_queue<pair<string, int>, vector<pair<string, int>>,cmp>
pq;  //小顶堆
        vector<string> res;  
        for (string& s : words)  
        {  
            map[s]++;  
        }  
        for (auto& p : map)  
        {  
            pq.push(p);  
            if (pq.size() > k)pq.pop();
        }  
        while(pq.size() > 0)  
        {  
            res.push_back(pq.top().first);  
            pq.pop();  
        }  
        return res;  
    }
};

 
##[最长有效括号](https://leetcode-cn.com/problems/longest-valid-parentheses)    
给定一个只包含 '(' 和 ')' 的字符串，找出最长的包含有效括号的子串的长度。
class Solution {
public:
    int longestValidParentheses(string s) {
         int res = 0, l = 0;
         stack<int> si;
         for(int i = 0; i < s.size(); ++ i)
         {
             if(s[i] == '(') si.push(i);
             else
             {
                 if(si.empty())l = i + 1;//不是有效括号，记录一下，跳过
                 else
                 {
                     si.pop();
                     if(si.empty())res = max(res, i - l + 1);
                     else res = max(res, i - si.top());
                 }
             }
         }
         return res;
    }
};

#数字--找规律
##[整数拆分](https://leetcode-cn.com/problems/integer-break)    
class Solution {
public:
    int integerBreak(int n) 
    {
        if (n <= 3)return n -1;
        int res(1);
        while(n >= 5)
        {
            res *= 3;
            n -= 3;
        }
        res *= n;
        return res;
    }
};

##[Nim游戏](https://leetcode-cn.com/problems/nim-game)    
class Solution {
public:
    bool canWinNim(int n) {
        return n % 4 != 0;
    }
};

##[加一](https://leetcode-cn.com/problems/plus-one)    
class Solution {
public:
    vector<int> plusOne(vector<int>& digits) {
        // 从后往前，碰到第一个不是9的就对其加1，然后返回即可
         for(int i = digits.size() - 1; i >= 0; -- i)
         {
             if(digits[i] == 9)
                 digits[i] = 0;
             else
             {
                 ++ digits[i];
                 return digits;
             }
         }
         // 最高位改成1，最后再添加个0
         digits[0] = 1;
         digits.push_back(0);
         return digits;
    }
};

##[完全平方数](https://leetcode-cn.com/problems/perfect-squares)    
class Solution {
public:
    int numSquares(int n) 
    {
        if (n <= 1)return n;
        int dp[n+1] = {0};
        for (int i = 1;i <= n;++i)//从前往后动规
        {
            dp[i] = i;//最多的次数
            for(int k = 1;k * k <= i;++k)//所有可能的递推情况，选取次数少的
            {
                dp[i] = min(dp[i - k * k] + 1,dp[i]);
            }
        }
        return dp[n];
    }
};
 
##[超级丑数](https://leetcode-cn.com/problems/super-ugly-number)    
 

##[x 的平方根](https://leetcode-cn.com/problems/sqrtx)    
 
class Solution {
public:
    int mySqrt(int x) {
        double begin = 0; double end = x;  
        double result = 1;  //需要从1开始，避免小数被取整时被清除
        double mid = 1;  
        while(abs(x - result) > 0.000001){  
            mid = (begin + end) / 2;  
            result = mid  * mid;
            if(result > x)  end = mid;  
            else begin = mid;  
        }  
        return (int)mid;  
    }
};
 
##[快乐数](https://leetcode-cn.com/problems/happy-number) 
 
就是检查是否会循环
class Solution {
public:
    bool isHappy(int n)
    {
        unordered_set<int> st;
        while(n != 1)
        {
            st.insert(n);
            int i(0);
            while (n)
            {
                i += (n % 10) * (n % 10);
                n /=10;
            }
            if (st.count(i) > 0) return false;
            n = i;
        }
        return true;
    }
};
 
##[计数质数](https://leetcode-cn.com/problems/count-primes)    
 
贪心方式填数
class Solution {
public:
    set<int> st;
    int countPrimes(int n) {
        if (n <=1)return 0;
        vector<bool> primes(n,true);
        for(int i = 2;i * i < n;++i)
        {
            for(int j = i * i;j < n;j += i)//i*i起点，小的已被填
            {
                primes[j] = false;
            }
        }
        int c(0);
        for(int i = 2;i < n;++i)
        {
            if (primes[i]) ++c;
        }
        return c;
    }
};
 
##[整数替换](https://leetcode-cn.com/problems/integer-replacement)    
class Solution {
public:
    int integerReplacement(int n) {
        if (n <=3) return n-1;
        int cnt(0);
        long k = n;
        while(k > 3)
        {
            if (k % 2 == 1)
            {
                if ((k + 1) % 4 == 0) 
                {
                    ++k;
                }
                else --k;
                ++cnt;
            }
            else 
            {
                k /= 2;
                ++cnt;
            }
        }
        cnt += k-1;
        return cnt;
    }
};

##[求众数 II](https://leetcode-cn.com/problems/majority-element-ii)    
class Solution {
public:
    vector<int> majorityElement(vector<int>& nums) {
        vector<int> res;
        int m = 0, n = 0, cm = 0, cn = 0;//最多2个
        for (auto a : nums) {
            if (a == m) ++cm;
            else if (a ==n) ++cn;
            else if (cm == 0) m = a, cm = 1;
            else if (cn == 0) n = a, cn = 1;
            else --cm, --cn;
        }
        //验证
        cm = cn = 0;
        for (auto a : nums) {
            if (a == m) ++cm;
            else if (a == n) ++cn;
        }
        if (cm > nums.size() / 3) res.push_back(m);
        if (cn > nums.size() / 3) res.push_back(n);
        return res;
    }
};
 
##[数字1的个数](https://leetcode-cn.com/problems/number-of-digit-one)    
 
class Solution {
public:
    int countDigitOne(int n) {
        int res = 0, a = 1, b = 1;
        while (n > 0) {
            res += (n + 8) / 10 * a + (n % 10 == 1) * b;
            b += n % 10 * a;
            a *= 10;
            n /= 10;
        }
        return res;
    }
    /*
    这道题让我们比给定数小的所有数中1出现的个数，之前有道类似的题Number of 1 Bits 位1的个数，那道题是求转为二进数后1的个数，我开始以为这道题也是要用那题的方法，其实不是的，这题实际上相当于一道找规律的题。那么为了找出规律，我们就先来列举下所有含1的数字，并每10个统计下个数，如下所示：

1的个数          含1的数字                                                                        数字范围
1                   1                                                                                     [1, 9]
11                 10  11  12  13  14  15  16  17  18  19                              [10, 19]
1                   21                                                                                   [20, 29]
1                   31                                                                                   [30, 39]
1                   41                                                                                   [40, 49]
1                   51                                                                                   [50, 59]
1                   61                                                                                   [60, 69]
1                   71                                                                                   [70, 79]
1                   81                                                                                   [80, 89]
1                   91                                                                                   [90, 99]
11                 100  101  102  103  104  105  106  107  108  109          [100, 109]

21                 110  111  112  113  114  115  116  117  118  119             [110, 119]

11                 120  121  122  123  124  125  126  127  128  129          [120, 129]
通过上面的列举我们可以发现，100以内的数字，除了10-19之间有11个‘1’之外，其余都只有1个。如果我们不考虑[10, 19]区间上那多出来的10个‘1’的话，那么我们在对任意一个两位数，十位数上的数字(加1)就代表1出现的个数，这时候我们再把多出的10个加上即可。比如56就有(5+1)+10=16个。如何知道是否要加上多出的10个呢，我们就要看十位上的数字是否大于等于2，是的话就要加上多余的10个'1'。那么我们就可以用(x+8)/10来判断一个数是否大于等于2。对于三位数区间 [100, 199] 内的数也是一样，除了[110, 119]之间多出的10个数之外，共21个‘1’，其余的每10个数的区间都只有11个‘1’，所以 [100, 199] 内共有21 + 11 * 9 = 120个‘1’。那么现在想想[0, 999]区间内‘1’的个数怎么求？根据前面的结果，[0, 99] 内共有20个，[100, 199] 内共有120个，而其他每100个数内‘1’的个数也应该符合之前的规律，即也是20个，那么总共就有 120 + 20 * 9 = 300 个‘1’。那么还是可以用相同的方法来判断并累加1的个数
    */
};


#其他
 
##[Excel表列序号](https://leetcode-cn.com/problems/excel-sheet-column-number)    
 
##[数字范围按位与](https://leetcode-cn.com/problems/bitwise-and-of-numbers-range)    
class Solution {
public:
    int rangeBitwiseAnd(int m, int n) {
        while(n > m)
        {
            n = n & (n-1);
        }
        return n;
    }
};
 
##[颠倒二进制位](https://leetcode-cn.com/problems/reverse-bits)    
class Solution {
public:
    uint32_t reverseBits(uint32_t n) {
        uint32_t k(0);
        for(int i = 0;i <32;++i)
        {
            k |= n & (1<<i) ? 1<<(31 - i) :0; 
        }
        return k;
    }
};

##[分数到小数](https://leetcode-cn.com/problems/fraction-to-recurring-decimal)    
class Solution {
public:
    string fractionToDecimal(int numerator, int denominator) {
        int s1 = numerator > 0 ? 1 : -1;
        int s2 = denominator > 0 ? 1 : -1;
        s1 = s1 * s2;
        long num = abs( (long)numerator );
        long den = abs( (long)denominator );
        long d = num / den;
        long rem = num % den;
        string res = to_string(d);
        if (rem == 0) 
        {
            if (d > 0 && s1 == -1)return res = "-" + res;
            return res;
        }
        if (s1 == -1)res = "-" + res;
        unordered_map<long long, int> m;
        res += ".";
        string s = "";
        int pos = 0;
        while (rem != 0) 
        {
            if (m.find(rem) != m.end())//余数重复则循环开始重复 
            {
                s.insert(m[rem], "("); 
                s += ")";
                return res + s;
            }
            //printf("pos:%d rem:%d\n",pos,rem);
            m[rem] = pos;//余数前面的位置
            s += to_string((rem * 10) / den);
            rem = (rem * 10) % den;
            ++pos;
        }
        return res + s;
    }
};

##[阶乘后的零](https://leetcode-cn.com/problems/factorial-trailing-zeroes)    
class Solution {
public:
    int trailingZeroes(int n) {
        int count(0);
        while (n > 0)
        {
            count += n /5;
            n /=5;
        }
        return count;
    }
};

##[最大数](https://leetcode-cn.com/problems/largest-number)    
class Solution {
public:
    struct Node
    {
        string str;
        Node(const string &s):str(s){}
        bool operator > (const Node& n)const
        {
            return compare(str,n.str);
        }
        bool compare(string str1,string str2)const
        {
            if (str1.size() == str2.size())
            {
                return str1 > str2;
            }
            else if (str1.size() < str2.size())
            {
                string part1(str2.c_str(),str1.size()),part2(str2.c_str()+str1.size());//str2
                if (str1 != part1)
                {
                    return str1 > part1;
                }
                else //较短的跟较长的剩余部分比较
                {
                    return compare(str1,part2);
                }
            }
            else //if (str1.size() > str2.size())
            {
                string part1(str1.c_str(),str2.size()),part2(str1.c_str()+str2.size());//str1
                if (part1 != str2)
                {
                    return part1 > str2;
                }
                else //较短的跟较长的剩余部分比较
                {
                    return compare(part2,str2);
                }
            }
        }
    };
    string largestNumber(vector<int>& nums) 
    {
        if (nums.size() == 0)return "";
        vector<Node> l;
        for(auto& num:nums)l.push_back(Node(to_string(num)));
        sort(l.begin(),l.end(),greater<Node>());//从大到小,重载 operator >
        string res;
        for(int i = 0;i < l.size();++i)
        {
            if (res.size() == 0 && l[i].str == "0")continue;//开头0的要去掉
            res += l[i].str;
        }
        return res.size() ? res :"0";
    }
};


##[移除元素](https://leetcode-cn.com/problems/remove-element)    
class Solution {
public:
    int removeElement(vector<int>& nums, int val) {
        int num=0;  
        for(int i=0;i<nums.size();++i)  
            if(nums[i]!=val)nums[num++]=nums[i];  
        return num;
    }
};

##[两数相除](https://leetcode-cn.com/problems/divide-two-integers)    
class Solution {
public:
    int divide(int dividend, int divisor) {
        if (dividend == 0 || divisor == 0) return 0;  
        if (dividend == INT_MIN && divisor == -1) return INT_MAX; 
        long a = abs((long)dividend);  
        long b = abs((long)divisor);  
        long ret = 0;  
        while (a >= b) {  
            long c = b;  
            for (int i = 0; a >= c; ++i, c <<= 1) {  
                a -= c;  
                ret += 1 << i;  
            }  
        }  
        return ((dividend^divisor)>>31) ? (int)(-ret) : (int)(ret);  
    }
};

##[二进制求和](https://leetcode-cn.com/problems/add-binary)    
class Solution {
public:
    int myAtoi(string str) 
    {
        if (str.size() == 0)return 0;
        int i = 0;
        int op = 1;
        int num = 0;
        int sign = 0;
        for(;i < str.size();++i)
        {
            //printf("%c\n",str[i]);
            if (str[i] == ' ')
            {
                if (sign > 0) return num;
                continue;
            }
            else if (str[i] == '+')
            {
                if (sign++ > 0) return num;
                continue;
            }
            else if (str[i] == '-')
            {
                if (sign++ > 0) return num;
                op = -1;
                continue;
            }
            else if (isdigit(str[i])) 
            {
                break;
            }
            return num;
        }
        //printf("%d\n",i);
        for(;i < str.size();++i)
        {
            //printf("%c\n",str[i]);
            if (isdigit(str[i]))
            {
                if (abs((long)num * 10 + (str[i] - '0')) > (long)INT_MAX) 
                {
                    return op == 1? INT_MAX: INT_MIN;
                }
                num *= 10;
                num += (str[i] - '0');
            }
            else break;
        }
        return op * num;
    }
};

##[整数转罗马数字](https://leetcode-cn.com/problems/integer-to-roman)    

##[罗马数字转整数](https://leetcode-cn.com/problems/roman-to-integer)    


##[电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number)    
 
##[整数反转](https://leetcode-cn.com/problems/reverse-integer)    
 
##[字符串转换整数 (atoi)](https://leetcode-cn.com/problems/string-to-integer-atoi)    
 
##[回文数](https://leetcode-cn.com/problems/palindrome-number)    
class Solution {
public:
    bool isPalindrome(int x) 
    {
         if(x < 0)   return false;
         int len = 1;
         while(x / len >= 10) len *= 10;
         int left,right;
         while(x > 0)    
         {
             left = x / len;
             right = x % 10;
             if(left != right) return false;
             x = (x % len) / 10;
             len /= 100;
         }
         return true;
    }
};

##[3的幂](https://leetcode-cn.com/problems/power-of-three)    
long i=1;
class Solution {
public:
    Solution(){while(i * 3 < (long)INT_MAX) i *=3;}
    bool isPowerOfThree(int n) {
        if (n < 1)return false;
        return (i % n) == 0;
    }
};
class Solution {
public:
    bool isPowerOfThree(int n) {
        // 1162261467 is 3^19,  3^20 is bigger than int  
        return ( n>0 &&  1162261467%n==0);
    }
};


##[比特位计数](https://leetcode-cn.com/problems/counting-bits)    
class Solution {
public:
    vector<int> countBits(int num) {
        vector<int> ret(1, 0);  
        int cnt = 0;  
        while(cnt < num) {  
            int sz = ret.size();  
            for(int i=0; i<sz&&cnt<num; ++i,++cnt) {  
                ret.push_back(ret[i]+1);  
            }  
        }  
        return ret;  
    }
};

##[各位相加](https://leetcode-cn.com/problems/add-digits)    
class Solution {
public:
    /*
        通过输入距离来发现规律
123456789 10 11 12 13 14 15
123456789 1 2 3 4 5 6
    */
    int addDigits(int num) {
        if (num == 0)return 0;
        return 1 + (num-1)%9;  
    }
};

##[丑数](https://leetcode-cn.com/problems/ugly-number)    
class Solution {
public:
    bool isUgly(int num) {
         if(num == 0)return false;
         int num1 = 0,num2 = 0,num3 = 0;
         while(0 == num1|| 0 == num2 || 0 == num3)
         {
             num1 = num %2;
             num2 = num %3;
             num3 = num %5;
             if (0 == num1)
             {
                 num /=2;
             }
             if(0 == num2)
             {
                 num /=3;
             }
             if(0 == num3)
             {
                 num /=5;
             }
         }
         return 1 == num; 
    }
};

##[丑数 II](https://leetcode-cn.com/problems/ugly-number-ii)    
 
##[缺失数字](https://leetcode-cn.com/problems/missing-number)    
class Solution {
public:
    int missingNumber(vector<int>& nums) {
        int size = nums.size();
        if(size < 1)
            return 0;
        int total = size*(size + 1)/2;
        int i,temp = 0;
        for(i = 0;i < size;++i)
            temp += nums[i];
        return total - temp;
    }
};

##[两整数之和](https://leetcode-cn.com/problems/sum-of-two-integers)    
 
##[查找和最小的K对数字](https://leetcode-cn.com/problems/find-k-pairs-with-smallest-sums)    
 
##[猜数字大小](https://leetcode-cn.com/problems/guess-number-higher-or-lower)    
 
##[猜数字大小 II](https://leetcode-cn.com/problems/guess-number-higher-or-lower-ii)    
 
##[寻找两个有序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays)    
 
##[汉明距离](https://leetcode-cn.com/problems/hamming-distance)    
 
##[第三大的数](https://leetcode-cn.com/problems/third-maximum-number)    

利用了set的唯一性和有序性
class Solution {
public:
    int thirdMax(vector<int>& nums) {
        set<int> s;
        for (int i : nums) {
            s.insert(i);
            if (s.size() > 3)
s.erase(s.begin());//删除小的，类似最小堆，但是是唯一的，因为排序了
        }
        return s.size() == 3 ? *s.begin() : *s.rbegin();
    }
};

 
##[数字的补数](https://leetcode-cn.com/problems/number-complement)    

class Solution {
public:
    int findComplement(int num) {
        if (!num)return 1;
        int complement(0);
        int n(0);
        while (num)
        {
            if ((num & 1) == 0) complement |= 1 << n;
            num = num >> 1;
            ++n;
        }
        return complement;
    }
};

##[七进制数](https://leetcode-cn.com/problems/base-7)    

class Solution {
public:
    string convertToBase7(int num) {
        if (!num)return "0";
        string res;
        bool flag(false);
        if (num > 0) flag = true;
        num = abs(num);
        while (num)
        {
            res.push_back('0' + (num % 7));
            num /=7;
        }
        std::reverse(res.begin(),res.end());
        return flag ? res:'-' + res;
    }
};

##[学生出勤记录 I](https://leetcode-cn.com/problems/student-attendance-record-i)    

class Solution {
public:
    bool checkRecord(string s) {
        int cntA = 0, cntL = 0;
        for (char c : s) {
            if (c == 'A') {
                if (++cntA > 1) return false;
                cntL = 0;
            } else if (c == 'L') {
                if (++cntL > 2) return false;
            } else {
                cntL = 0;
            }
        }
        return true;
    }
};

##[自除数](https://leetcode-cn.com/problems/self-dividing-numbers)    

class Solution {
public:
    vector<int> selfDividingNumbers(int left, int right) {
        vector<int> v;
        for(int i = left ;i <= right;++i)
        {
            int n = i;
            while (n > 0)
            {
                int d = n % 10;
                if (d == 0)break;
                else if (i % d != 0)break;
                n /= 10;
                if (n == 0)v.push_back(i);
            }
        }
        return v;
    }
};

##[宝石与石头](https://leetcode-cn.com/problems/jewels-and-stones)    


 
##[机器人能否返回原点](https://leetcode-cn.com/problems/robot-return-to-origin)    

class Solution {
public:
    bool judgeCircle(string moves) {
        int x = 0, y = 0;  
        for (int i = 0; i < moves.length(); i++)  
        {  
            if (moves[i] == 'L') x--;  
            else if (moves[i] == 'R') x++;  
            else if (moves[i] == 'U') y++;  
            else y--;  
        }  
        return x == 0 && y == 0;  
    }
};


#数组
 

##排列组合
 
##[第k个排列](https://leetcode-cn.com/problems/permutation-sequence)    
 
class Solution {
public:
    string getPermutation(int n, int k) {
        vector<int> permutation(n, 1);
        for (int i = 1; i < n; ++i) {
            permutation[i] = permutation[i - 1] * i;
        }
        vector<char> digits = { '1', '2', '3', '4', '5', '6', '7', '8', '9' };
        int num = n;
        string res;
        while (--num) {
            int t = (k - 1) / (permutation[num]);//找首位
            res.push_back(digits[t]);
            digits.erase(digits.begin() + t);
            k = k - t * permutation[num];//剩下的
        }
        res.push_back(digits[k - 1]);
        return res;
    }
};
 
##[目标和](https://leetcode-cn.com/problems/target-sum)    
 
class Solution {
public:
    int res = 0;
    int findTargetSumWays(vector<int>& nums, int S) 
    {
        int sum = 0;
        for(int i = 0; i < nums.size(); ++i) 
        {
            sum += nums[i];  
        }
        if(sum == S) res++;  
        dfs(nums, sum, 0, S);  
        return res;  
    }
    void dfs(vector<int>& nums,int sum, int pos, int S)
    {
        for(int i = pos; i < nums.size(); ++i)
        {  
            int tmp = sum - 2 * nums[i];                           
            if(tmp == S) res++;   
            if(tmp >= S) dfs(nums, tmp, i+1, S);
        }                                    
    } 
};
 
##[四数相加 II](https://leetcode-cn.com/problems/4sum-ii)    
 
class Solution {
public:
    int fourSumCount(vector<int>& A, vector<int>& B, vector<int>& C, vector<int>& D) 
    {
        int res(0);
        unordered_map<int,int> m1,m2;
        for(int i = 0;i < A.size();++i)
        {
            for(int j = 0;j < B.size();++j)
            {
                m1[A[i] + B[j]]++;
            }
        }
        for(int i = 0;i < C.size();++i)
        {
            for(int j = 0;j < D.size();++j)
            {
                m2[C[i] + D[j]]++;
            }
        }
        for(auto& iter:m1)
        {
            res += m2[-iter.first] * iter.second;//能够组合成的元组的数
        }
        return res;
    }
};
 
##[下一个排列](https://leetcode-cn.com/problems/next-permutation)    
 
class Solution {
public:
    void nextPermutation(vector<int>& nums) {
        int j=-1,k=0;
        for( int i=nums.size()-2;i>=0;i-- )//j=max{i|pi<pi+1}
        {
            if( nums[i]<nums[i+1] )
            {
                j=i;break;
            }
        }
        if( j==-1 )
        {//3 2 1
            reverse(nums.begin(),nums.end());
        }
        else
        {
            for( int i=nums.size()-1;i>=0;i-- )//k=max{i|pi>pj}
            {
               if( nums[i]>nums[j] )
               {
                  k=i;break;
               }
            }
            swap(nums[j],nums[k]);//swap pj pk
            reverse(nums.begin()+j+1,nums.end());//reverse pj+1 pn
        }
    }
    /*
    字典序：
1234的全排列从小到大的顺序也就是字典序的顺序，依次如下：
1234,1243,1324,1342,1423,1432,
2134,2143,2314,2341,2413,2431,
3124,3142,3214,3241,3412,3421,
4123,4132,4213,4231,4312,4321
    */
};


#区间和


 
[和为K的子数组](https://leetcode-cn.com/problems/subarray-sum-equals-k)    
 

##其他数组
 
##[在排序数组中查找元素的第一个和最后一个位置](https://leetcode-cn.com/problems/find-first-and-last-position-of-element-in-sorted-array)    
class Solution {
public:
    vector<int> searchRange(vector<int>& nums, int target) {
        vector<int> res;  
        int l=0,len=nums.size(),r=len-1,mid;  
        while(l<=r){  
            mid=(l+r)>>1;  
            if(nums[mid]==target)break;  
            else if(nums[mid]>target)r=mid-1;  
            else l=mid+1;  
        }  
        if(l<=r){  
            l=mid-1;  
            while(l>=0 && nums[l]==nums[mid])l--;  
            r=mid+1;  
            while(r<len && nums[r]==nums[mid])r++;  
            res.push_back(l+1);  
            res.push_back(r-1);  
        }else{  
            res.push_back(-1);  
            res.push_back(-1);  
        }  
        return res;  
    }
};


##[寻找重复数](https://leetcode-cn.com/problems/find-the-duplicate-number)    
class Solution {
public:
    int findDuplicate(vector<int>& nums)
    {
        //因为两个值相同，若作为下标则会有环
        if (nums.size() < 1)return -1;
        int sp(0),fp(0);
        do
        {
            sp = nums[sp];
            fp = nums[nums[fp]];
        }while(sp != fp);
        fp = 0;
        do
        {
            sp = nums[sp];
            fp = nums[fp];
        }while(sp != fp);//最终的是相同的数
        return sp;
    }
};
 
##[除自身以外数组的乘积](https://leetcode-cn.com/problems/product-of-array-except-self)    
 
class Solution {
public:
    vector<int> productExceptSelf(vector<int>& nums) {
        vector<int> res(nums.size(),1);
        int left = 1,right = 1;
        for(int i = 0;i < nums.size();++i)
        {
            res[i] *= left;
            left *= nums[i];
        }
        for(int i = nums.size()-1;i >= 0;--i)
        {
            res[i] *= right;
            right *= nums[i];
        }
        return res;
    }
};
 
##[区域和检索 - 数组不可变](https://leetcode-cn.com/problems/range-sum-query-immutable)    
 
class NumArray {
public:
    NumArray(vector<int> nums) {
        if (nums.size() > 0)
        {
            sums.emplace_back(nums[0]);
            for(int i = 1;i < nums.size();++i)
            {
                sums.emplace_back(sums[i - 1] + nums[i]);
            }
        }
    }
    int sumRange(int i, int j) {
        if (i < 0|| j < i|| j >= sums.size())return 0;
        if (i == 0)return sums[j];
        return sums[j] - sums[i-1];
    }
    vector<int> sums;
};
 
##[汇总区间](https://leetcode-cn.com/problems/summary-ranges)    
 
##[最长公共前缀](https://leetcode-cn.com/problems/longest-common-prefix)    
 
跟第一个比较
class Solution {
public:
    string longestCommonPrefix(vector<string>& strs) {
        if(strs.size() == 0)return "";
        int n = strs.size();
        const string& firstStr = strs[0];
        int len = firstStr.size();//最长前缀的长度
        for(int i = 1; i < n; i++)
        {
            int k = 0;
            for(; k < len && k < strs[i].size(); k++)
                if(firstStr[k] != strs[i][k])break;
            if(k < len)len = k;
        }
        return firstStr.substr(0, len);
    }
};
 
##[逆波兰表达式求值](https://leetcode-cn.com/problems/evaluate-reverse-polish-notation)    
 
class Solution {
public:
    int evalRPN(vector<string>& tokens) {
        stack<long> sk;
        for(int i = 0;i < tokens.size();++i)
        {
            if (tokens[i].size() == 1 && tokens[i][0] == '+')//表达式的规则是后面压的数是被操作的对象
            {
                long x1 = sk.top();
                sk.pop();
                long x2 = sk.top();
                sk.pop();
                sk.push(x2 + x1);
            }
            else if (tokens[i].size() == 1 && tokens[i][0] == '-')
            {
                long x1 = sk.top();
                sk.pop();
                long x2 = sk.top();
                sk.pop();
                sk.push(x2 - x1);
            }
            else if (tokens[i].size() == 1 && tokens[i][0] == '*')
            {
                long x1 = sk.top();
                sk.pop();
                long x2 = sk.top();
                sk.pop();
                sk.push(x2 * x1);
            }
            else if (tokens[i].size() == 1 && tokens[i][0] == '/')
            {
                long x1 = sk.top();
                sk.pop();
                long x2 = sk.top();
                sk.pop();
                sk.push(x2 / x1);
            }
            else
            {
                if (tokens[i][0] == '-')
                {
                    long x = strtoll(tokens[i].c_str()+1,NULL,10);
                    sk.push(-x);
                }
                else
                {
                    long x = strtoll(tokens[i].c_str(),NULL,10);
                    sk.push(x);
                }
            }
        }
        return sk.top();
    }
};

 
##[寻找峰值](https://leetcode-cn.com/problems/find-peak-element)    
 
class Solution {
public://
    int findPeakElement(vector<int>& nums) {
        if (nums.size() <= 1)return 0;
        if (nums.size() == 2)return (nums[0] > nums[1] ? 0:1);
        int left = 0,right = nums.size()-1,mid(0);
        while(left <= right)//二分法查找上升或者下降的坡
        {
            mid = left + (right - left)/2;
            if (mid == left) 
            {
                return nums[left] > nums[right] ? left: right; 
            }
            if (nums[mid] < nums[mid+1])//寻找峰点，峰点在右边
            {
                left = mid;
            }
            else //(nums[mid] >= nums[mid+1]) 寻找峰点，峰点在左边
            {
                right = mid;
            }
        }
        return 0;
    }
};

 
##[同构字符串](https://leetcode-cn.com/problems/isomorphic-strings)    

class Solution {
public:
    bool isIsomorphic(string s, string t) {
         if(s.length() != t.length())return false;
         unordered_map<char,char> m;
         for(int i = 0; i< s.length();++i)
         {
             if(m.find(s[i]) == m.end())m[s[i]] = t[i];
             else if (m[s[i]] != t[i])return false;
         }
         m.clear();
         for(int i = 0; i< t.length();++i)
         {
             if(m.find(t[i]) == m.end())m[t[i]] = s[i];
             else if (m[t[i]] != s[i])return false;
         }
         return true;

    }
};

 
##[颜色分类](https://leetcode-cn.com/problems/sort-colors)    
 
class Solution {
public:
    void sortColors(vector<int>& nums) {
        // 荷兰国旗问题        
        int begin = 0, cur = 0, end = nums.size() - 1;
        while (cur <= end){
            if (nums[cur] == 0){
                swap(nums[cur++], nums[begin++]);
            }
            else if (nums[cur] == 2){
                swap(nums[cur], nums[end--]);
            }
            else {
                cur++;
            }
        }
    }
};

 
##[两数之和 II - 输入有序数组](https://leetcode-cn.com/problems/two-sum-ii-input-array-is-sorted)    
 
class Solution {
public:
    vector<int> twoSum(vector<int>& numbers, int target) {
        int l = 0, r = numbers.size() - 1;
        while (l < r) {
            int sum = numbers[l] + numbers[r];
            if (sum == target) return {l + 1, r + 1};
            else if (sum < target) ++l;
            else --r;
        }
        return {};
    }
};

 
##[加油站](https://leetcode-cn.com/problems/gas-station)    
 
class Solution {
public:
    int canCompleteCircuit(vector<int>& gas, vector<int>& cost) {
        int n = gas.size();
        if (n == 0)return -1;
        for(int i = 0;i < n;)
        {
            int g(0);
            int j = i;
            for(;j < i + n;++j)//循环遍历
            {
                g += gas[j%n] - cost[j%n];
                if (g < 0)break;
            }
            if (g >= 0)return i;
            if (j < n && j > i + 1) i = j;//跳过已经验证过的
            else ++i;
        }
        return -1;
    }
};

 
##[旋转数组](https://leetcode-cn.com/problems/rotate-array)
 

翻转会调换位置，再翻转回来就行
class Solution {
public:
    void rotate(vector<int>& nums, int k) {
        if (nums.size() <= 1)return;
        k = k % nums.size();
        reverse(nums.begin(),nums.end());
        reverse(nums.begin(),nums.begin()+k);
        reverse(nums.begin()+k,nums.end());
    }
};
 
 
##[删除排序数组中的重复项 II](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array-ii)    
 
class Solution {
public:
    int removeDuplicates(vector<int>& nums) 
    {
        int len = nums.size();
        if (len <= 2)return len;
        int count = 2;
        for(int i = 2;i < len;++i)
        {
            if (nums[i] != nums[count-2]) 
            nums[count++] = nums[i];
        }
        return count;
    }
};
 
##[求众数](https://leetcode-cn.com/problems/majority-element)    
 
class Solution {
public:
    int majorityElement(vector<int>& nums) {
        int count(0);
        int n(0);
        for(auto& num:nums)
        {
            if (!count)
            {
                count = 1;
                n = num;
            }
            else if (n == num)
            {
                ++count;
            }
            else --count;
        }
        return n;
    }
};
 
##[杨辉三角](https://leetcode-cn.com/problems/pascals-triangle)    
 
class Solution {
public:
    vector<vector<int>> generate(int numRows) {
        vector<vector<int> > vecs;
        if(numRows <= 0)return vecs;
        if (1 == numRows) return {{1}} ;
        vector<int> tempvec[2];
        int cur = 0;
        tempvec[cur].push_back(1);
        for(int i = 2;i<=numRows;i++)
        {
            tempvec[1-cur].push_back(1);
            for(int j = 1;j< tempvec[cur].size();j++)
            {
                 tempvec[1-cur].push_back(tempvec[cur][j-1] + tempvec[cur][j]);  
            }
            tempvec[1-cur].push_back(1);
            vecs.push_back(tempvec[cur]);
            tempvec[cur].clear();
            cur = 1 - cur;
        }
        vecs.push_back(tempvec[cur]);
        return vecs;
    }
};
 
##[杨辉三角 II](https://leetcode-cn.com/problems/pascals-triangle-ii)    

class Solution {
public:
    vector<int> getRow(int rowIndex) {
        if (rowIndex < 0) return {};
        if (rowIndex == 0) return {1};
        vector<int> rows[2];
        int cur= 0;
        rows[cur].push_back(1);
        for(int i = 1;i <= rowIndex;++i)
        {
            rows[1 - cur].push_back(1);
            for(int j = 1;j < rows[cur].size();++j)
            {
                rows[1 - cur].push_back(rows[cur][j - 1] + rows[cur][j]);
            }
            rows[1 - cur].push_back(1);
            rows[cur].clear();
            cur = 1 - cur;
        }
        return rows[cur];
    }
};
 
##[合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array)    
 
class Solution {
public:
    void merge(vector<int>& nums1, int m, vector<int>& nums2, int n) 
    {
        int k = m + n -1;
        int i = m - 1,j = n - 1;
        while(i >= 0 && j >= 0)
        {
            if (nums1[i] > nums2[j])
            {
                nums1[k--] = nums1[i--];
            }
            else 
            {
                nums1[k--] = nums2[j--];
            }
        }
        while(i >= 0)nums1[k--] = nums1[i--];
        while(j >= 0)nums1[k--] = nums2[j--];
    }
};
 
##[搜索二维矩阵](https://leetcode-cn.com/problems/search-a-2d-matrix)
 把二维矩阵当做一维数组来计算，就是访问下标时需要转换下标
class Solution {
public:
    bool searchMatrix(vector<vector<int>>& matrix, int target) {
        if (!matrix.size() || !matrix[0].size())return false;
        int rows=matrix.size();
        int cols=matrix[0].size();
        int left=0,right=(rows*cols-1);
        int mid,m,n,val;
        while(left <= right){
            mid= (right+left)>>1;
            m= mid / cols;
            n= mid % cols;
            if(matrix[m][n]==target) return true;
            else if (matrix[m][n] < target) left = mid+1;
            else right= mid - 1;
        }
        return false;
    }
};
 
##[螺旋矩阵](https://leetcode-cn.com/problems/spiral-matrix)    
 
注意边界条件
class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
       if (matrix.size() == 0)return {};
        vector<int> ret;
        int row = matrix.size() - 1;
        int col = matrix[0].size() - 1;
        for (int x = 0, y = 0; x <= row && y <= col; x++, y++,row--,col--)
        {
            for(int j=y ; j<=col ; ++j)//首行
            {
                ret.push_back(matrix[x][j]);
            }
            for (int i = x + 1; i <= row; ++i)//最右列(上到下，需要跳过第一个)
            {
                ret.push_back(matrix[i][col]);
            }
            for (int j = col - 1; j >= y && x != row;
--j)//最底行(右到左，需要跳过第一个，判断重复行)
            {
                ret.push_back(matrix[row][j]);
            }
            for (int i = row - 1; i > x && y != col;
--i)//最左列(下到上，需要跳过第一个，判断重复列)
            {
                ret.push_back(matrix[i][y]);
            }
        }
        return ret;
    }
};
 
##[螺旋矩阵 II](https://leetcode-cn.com/problems/spiral-matrix-ii)    
 
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) 
    {
        if (n <= 0)return {};
        vector<vector<int>> res(n,vector<int>(n,0));
        int row(n-1),col(n-1),x(0),y(0),cnt(0);
        for(;x <= row && y <= col;++x,++y,--row,--col)
        {
            for(int i = y;i <= col;++i)
            {
                res[x][i] = ++cnt;//printf("1)%d %d %d\n",x,i,cnt);
            }
            for(int i = x+1;i <= row;++i)
            {
                res[i][col] = ++cnt;//printf("2)%d %d %d\n",i,col,cnt);
            }
            for(int i = col-1;i >= y && x != row;--i)
            {
                res[row][i] = ++cnt;//printf("3)%d %d %d\n",row,i,cnt);
            }
            for(int i = row-1;i >= x + 1&& y != col;--i)
            {
                res[i][y] = ++cnt;//printf("4)%d %d %d\n",i,y,cnt);
            }
        }
        return res;
    }
};
 
##[搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array)    
 
##[搜索旋转排序数组 II](https://leetcode-cn.com/problems/search-in-rotated-sorted-array-ii)    
 
##[数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array) 
 
class Solution {
public:
    int findKthLargest(vector<int>& nums, int k) {
        priority_queue<int,vector<int>,greater<int>> pq;//最小堆
        for(auto & num:nums)
        {
            pq.push(num);
            if (pq.size() > k) pq.pop();
        }
        return pq.top();
    }
};
 
##[存在重复元素](https://leetcode-cn.com/problems/contains-duplicate)    

哈希表
class Solution {
public:
    bool containsDuplicate(vector<int>& nums) {
        unordered_map<int,int> m;
        for(auto& num:nums)
        {
            if (m[num]++ > 0)return true;
        }
        return false;
    }
};
 
##[存在重复元素 II](https://leetcode-cn.com/problems/contains-duplicate-ii)    
 
遍历时检查下标差
class Solution {
public:
    bool containsNearbyDuplicate(vector<int>& nums, int k) {
        unordered_map<int,int> intmap;
        for(int i = 0;i < nums.size();++i)
        {
            if(intmap.end() != intmap.find(nums[i]) &&(i - intmap[nums[i]]) <=
k)
            {
                return true;
            }
            else
            {
                intmap[nums[i]] = i;
            }
        }
        return false;
    }
};
 
##[存在重复元素 III](https://leetcode-cn.com/problems/contains-duplicate-iii)
 
利用set的有序性计算键的差值，消除非法下标，就可以获取合法条件值
class Solution {
public:
    bool containsNearbyAlmostDuplicate(vector<int>& nums, int k, int t) {
        set<long> st;
        int j = 0;
        for (int i = 0; i < nums.size(); ++i) {
            if (i - j > k) st.erase(nums[j++]);//消除下标非法位置的
            auto iter = st.lower_bound((long)nums[i] -
t);//获取和小于等于t的下一个数（n >= nums[i] - t)  ,nums[i] - n <= t）
，再去判断其绝对值
            if (iter != st.end() && abs(*iter - nums[i]) <= t) return true;
            st.insert(nums[i]);
        }
        return false;
    }
};
 
##[长度最小的子数组](https://leetcode-cn.com/problems/minimum-size-subarray-sum)    
 
class Solution {
public:
    int minSubArrayLen(int s, vector<int>& nums) {
           int res = INT_MAX, left = 0, sum = 0;
            for (int i = 0; i < nums.size(); ++i) {
                sum += nums[i];
                while (left <= i && sum >= s) {
                    res = min(res, i - left + 1);
                    sum -= nums[left++];
                }
            }
            return res == INT_MAX ? 0 : res;
    }
};
 
##[合并区间](https://leetcode-cn.com/problems/merge-intervals)    
 
先排序，再遍历检查和合并
class Solution {
public:
    vector<Interval> merge(vector<Interval>& intervals) {
        if (intervals.size()<=1)  return  intervals;
        vector<Interval> res;
        sort(intervals.begin(),intervals.end(), cmp);  
        Interval pre = intervals[0] ;  
        for (int index=1; index<intervals.size(); index++)
        {  
            Interval tmp = intervals[index] ;  
            if (tmp.start > pre.end)
            {  
                res.push_back(pre);  
                pre = tmp;  
            }
            else
            {  
                pre.end = max(tmp.end, pre.end);  
            }  
        }  
        res.push_back(pre);  
        return res;  
    }
    static bool cmp(const Interval &val1, const Interval &val2){  
        return val1.start < val2.start;  
    }  
};
 
##[插入区间](https://leetcode-cn.com/problems/insert-interval)    
 
##[删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array)    
 
##[递增的三元子序列](https://leetcode-cn.com/problems/increasing-triplet-subsequence)    
 
##[滑动窗口最大值](https://leetcode-cn.com/problems/sliding-window-maximum)
 

使用链表保存窗口中的从大到小的列表，表头的就是需要的值
class Solution {
public:
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        if (nums.size() == 0 || !k) return {};
        vector<int> res;
        list<int>
l;//链表表头始终保存的是最大的元素，递减较小的成员会排在后面作为替补成员
        for(int i = 0;i < nums.size();++i)
        {
            if (l.size() && l.front() == i - k)l.pop_front();
            while(l.size() && nums[l.back()] < nums[i])
            {
                l.pop_back();
            }
            l.push_back(i);
            if (i >= k - 1)res.push_back(nums[l.front()]);
        }
        return res;
    }
};

 
##[移动零](https://leetcode-cn.com/problems/move-zeroes)
 
从前往后边遍历交换，保持稳定性
class Solution {
public:
    void moveZeroes(vector<int>& nums) {
        int k = 0;
        for(int i = 0;i < nums.size();++i)
        {
            if (nums[i])            
            {
                swap(nums[k++],nums[i]);
            }
        }
    }
};

 
##[计算右侧小于当前元素的个数](https://leetcode-cn.com/problems/count-of-smaller-numbers-after-self)    
class Solution {
public:
    vector<int> countSmaller(vector<int>& nums) {
        if (nums.size() == 0)return {};
        vector<int> res(nums.size());//从后面往前查找
        vector<int> sorted;
        for(int i = nums.size()-1;i >=0;--i)
        {
            //在sorted中寻找插入的坐标（递增排序）
            int pos = lower_bound(sorted.begin(),sorted.end(),nums[i]) - sorted.begin();
            sorted.insert(sorted.begin()+pos,nums[i]);
            res[i] = pos;
        }
        return res;
    }
};

##[打乱数组](https://leetcode-cn.com/problems/shuffle-an-array)    
class Solution {
public:
    Solution(vector<int> nums) {
        m_numsOri = m_nums = nums;
    }
    /** Resets the array to its original configuration and return it. */
    vector<int> reset() {
        return m_numsOri;   
    }
    /** Returns a random shuffling of the array. */
    vector<int> shuffle() {
        for(int i = 0;i < m_nums.size();++i)
        {
            swap(m_nums[i],m_nums[rand() % m_nums.size()]);
        }
        return m_nums;
    }
    vector<int> m_nums;
    vector<int> m_numsOri;
};

##[有序矩阵中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-sorted-matrix)    
 
##[摆动排序 II](https://leetcode-cn.com/problems/wiggle-sort-ii)    
class Solution {
public:
    void wiggleSort(vector<int>& nums) 
    {
        vector<int> tmp = nums;
        sort(tmp.begin(),tmp.end());
        int mid = (tmp.size() + 1)/2 -1;
        int n = tmp.size() -1;
        for(int i = 0;i < nums.size();++i)
        {
            if (i % 2 == 0)nums[i] = tmp[mid--];
            else nums[i] = tmp[n--];
        }
    }
};

##[组合总和 Ⅳ](https://leetcode-cn.com/problems/combination-sum-iv)    
 
##[找到所有数组中消失的数字](https://leetcode-cn.com/problems/find-all-numbers-disappeared-in-an-array)    
 
##[两个数组的交集](https://leetcode-cn.com/problems/intersection-of-two-arrays)
 

使用哈希表的快速访问
class Solution {
public:
    /*
    复杂度为m + n
    */
    vector<int> intersection(vector<int>& nums1, vector<int>& nums2) {
        vector<int> res;
        unordered_set<int> s1(nums1.begin(),nums1.end());
        unordered_set<int> s2(nums2.begin(),nums2.end());
        for(auto iter:s2)
        {
            if (s1.find(iter) != s1.end())res.push_back(iter);
        }
        return res;
    }
};

 
##[两个数组的交集 II](https://leetcode-cn.com/problems/intersection-of-two-arrays-ii)    
 
##[最大连续1的个数](https://leetcode-cn.com/problems/max-consecutive-ones)
 
遍历数组和判断
class Solution {
public:
    int findMaxConsecutiveOnes(vector<int>& nums) {
        int maxLen(0);
        int left(-1);
        for(int i=0;i < nums.size();++i)
        {
            if (1 == nums[i])maxLen = max(maxLen,i-left);
            else left = i;
        }
        return maxLen;
    }
};

 
##[连续的子数组和](https://leetcode-cn.com/problems/continuous-subarray-sum)    
class Solution {
public:
    bool checkSubarraySum(vector<int>& nums, int k) 
    {
        if (nums.size() < 2)return false;
        vector<int> dp(nums.size(),0);
        for(int i = 0;i < nums.size();++i)
        {
            dp[i] = nums[i];//断开
            for(int j = i+1; j < nums.size();++j)
            {
                dp[j] = dp[j-1] + nums[j];
                if ((dp[j] == k) || (k != 0 && dp[j] % k == 0))return true;
            }
        }
        return false;
    }
};

##[数组拆分 I](https://leetcode-cn.com/problems/array-partition-i)    
class Solution {
public:
    int arrayPairSum(vector<int>& nums) {
        sort(nums.begin(),nums.end());
        int sum(0);
        for(int i =0;i < nums.size();i += 2)
        {
            sum += nums[i];
        }
        return sum;
    }
};

##[错误的集合](https://leetcode-cn.com/problems/set-mismatch)    
class Solution {
public:
    vector<int> findErrorNums(vector<int>& nums) {
        if (nums.size() == 0)return {};
        vector<int> res;
        unordered_map<int,int> m;
        for(auto n:nums)
        {
            m[n]++;
        }
        for(int i = 1;i <=nums.size();++i)
        {
            if (m[i] > 1)res.insert(res.begin(),i);
            else if (m[i] == 0)res.push_back(i);
        }
        return res;
    }
};
##[数组的度](https://leetcode-cn.com/problems/degree-of-an-array)    
class Solution {
public:
    int findShortestSubArray(vector<int>& nums) {
        unordered_map<int, int> start;
        unordered_map<int, int> end;
        unordered_map<int, int> count;
        int maxCount = 0;
        for (int i = 0; i < nums.size(); i++){
            if (!count[nums[i]]){
                start[nums[i]] = i;
            }
            ++count[nums[i]];
            end[nums[i]] = i;
            maxCount = max(maxCount, count[nums[i]]);
        }
        int minLength = INT_MAX;
        for (auto it : count){
            if (it.second == maxCount){
                minLength = min(minLength,end[it.first]-start[it.first]+1);
            }
        }
        return minLength;
    }
};

##[划分为k个相等的子集](https://leetcode-cn.com/problems/partition-to-k-equal-sum-subsets)    
class Solution {
public:
    bool canPartitionKSubsets(vector<int>& nums, int k) {
        int sum = accumulate(nums.begin(), nums.end(), 0);
        if (sum % k != 0) return false;
        vector<bool> visited(nums.size(), false);
        return dfs(nums, k, sum / k, 0, 0, visited);
    }
    bool dfs(vector<int>& nums, int k, int target, int start, int curSum, vector<bool>& visited) {
        if (k == 1) return true;
        if (curSum == target) return dfs(nums, k - 1, target, 0, 0, visited);//下一个
        if( curSum > target)return false;
        for (int i = start; i < nums.size(); ++i) {
            if (visited[i]) continue;
            visited[i] = true;
            if (dfs(nums, k, target, i + 1, curSum + nums[i], visited))
           {
                visited[i] = false;
               return true;
           }
            visited[i] = false;
        }
        return false;
    }
};

##[转置矩阵](https://leetcode-cn.com/problems/transpose-matrix)    
class Solution {
public:
    vector<vector<int>> transpose(vector<vector<int>>& A) 
    {
        if (A.size() == 0|| A[0].size() == 0)return {};
        int m = A.size();
        int n = A[0].size();
        vector<vector<int>> B(n,vector<int>(m,0));
        for(int i = 0;i < m;++i)
        {
            for(int j = 0; j < n;++j)
            {
                B[j][i] = A[i][j];
            }
        }
        return B;
    }
};

##[最长连续递增序列](https://leetcode-cn.com/problems/longest-continuous-increasing-subsequence)    
class Solution {
public:
    int findLengthOfLCIS(vector<int>& nums) 
    {
        if (nums.size() <= 1)return nums.size();
        int global(INT_MIN);    
        int local(0);
        for(int i = 1;i < nums.size();++i)
        {
            if (nums[i] > nums[i-1])
            {
                local++;
            }
            else
            {
                global = max(global,local+1);
                local = 0;
            }
        }
        global = max(global,local+1);;
        return global;
    }
};
 
##[最短无序连续子数组](https://leetcode-cn.com/problems/shortest-unsorted-continuous-subarray)    
class Solution {
public:
    int findUnsortedSubarray(vector<int>& nums) {
        vector<int> nums2 = nums;  
        sort(nums.begin(),nums.end());  
        int begin = -1;  
        int end = -1;  
        for(int i = 0;i < nums.size(); i++)  
        {  
            if(nums[i] != nums2[i])   
            {  
                if(begin == -1)  
                    begin = i;  
                end = i;  
            }  
        }  
        return end == -1 ? 0 :end - begin + 1;  
    }
};

##[任务调度器](https://leetcode-cn.com/problems/task-scheduler)    
 
因为任务的等待时间，制约的就是数量最大的任务，时间=（该类任务等待时间 + 1） *
（该类任务数量 -1） + 该类任务同样数量的任务的类数  ，或者任务数中的较大的
class Solution {
public:
    int leastInterval(vector<char>& tasks, int n) {
        vector<int> mVec(26,0);
        for(auto &val:tasks) {
            mVec[val-'A']++;
        }
        sort(mVec.begin(),mVec.end());
        int i(0);
        while(i < mVec.size() && mVec.back()==mVec[mVec.size() - 1 - i])i++;
        return max((mVec.back()-1)*(n+1)+i,(int)tasks.size());
    }
};
 

#字符串
 
##(排列)
 
##[下一个更大元素 III](https://leetcode-cn.com/problems/next-greater-element-iii)
class Solution {
public:
    //只是为了测试next_permutation的实现，next_permutation 可以是stl的
    static bool next_permutation(string::iterator start,string::iterator end)
    {
        auto cur = end - 1,pre= cur - 1;
        while(cur > start && *pre >= *cur) cur--,pre--;//找到后缀的升序（为了替换）
        if(cur <= start) return false;
        for(cur=end-1;*cur <= *pre;cur--);//找到比*pre大的
        swap(*cur,*pre);
        reverse(pre+1,end);//逆序到正序
        return true;
    }
    int nextGreaterElement(int n) {
        string s = to_string(n);
        sort(s.begin(), s.end());
        while(next_permutation(s.begin(), s.end())) 
        {
            int tmp = atoll(s.c_str());
            if (tmp > n)return tmp;
        } 
        return -1;
    }
    /*
    next_permutation:
    1 2 3
    1 3 2
    */
};

##(切分)

##[简化路径](https://leetcode-cn.com/problems/simplify-path) 
 
字节流的分割和遍历
class Solution {
public:
    string simplifyPath(string path) {
        vector<string> vec;
        stringstream ss(path);
        string tmp;
        while(getline(ss,tmp,'/'))//从前往后 取出所有的 子路径
        {
            if(tmp == "" || tmp == ".") continue;
            if(tmp == "..")
            {
                if (vec.size() > 0) vec.pop_back();
            }
            else vec.push_back(tmp);
        }
        string res;
        for(const auto &str:vec) res += "/" + str;
        return res.size() ? res:"/";
    }
};

##（字符数组）

##[压缩字符串](https://leetcode-cn.com/problems/string-compression)    
class Solution {
public:
    int compress(vector<char>& chars) 
    {
        if (chars.size() == 0)return 0;
        char c = chars[0];
        int num(1),left(0),cnt(0);
        for(int i = 1;i <= chars.size();++i)
        {
            if (i == chars.size() || chars[i] != c)
            {
                chars[left++] = c;
                if (num > 1)
                {
                    string n = to_string(num);
                    for(int k = 0;k < n.size();++k)
                    {
                        chars[left+k] = n[k]; 
                    }
                    cnt += n.size()+1;
                    left += n.size();
                }
                else  
                {
                    ++cnt;
                }
                c = i < chars.size() ? chars[i]:0;   
                num = 1;
            }
            else
            {
                ++num;
            }
        }
        return cnt;
    }
};

##(其他)

##[基本计算器 II](https://leetcode-cn.com/problems/basic-calculator-ii)    

class Solution {
public:
    int calculate(string s) {
        char op = '+';
        stack<long> sk;
        long num(0);
        for(int i = 0;i <= s.size();++i)
        {
            if (s[i] >= '0' && s[i] <= '9')
            {
                num = num * 10 + s[i] - '0';
            }
            else if (s[i] != ' '|| s[i] == 0)//符号或者结束
            {
                //检查上个数之前的符号
                if (op == '+')sk.push(num);
                else if (op == '-') sk.push(-num);
                else if (op == '*') 
                {
                    sk.top() *=  num;
                }
                else if (op == '/')
                {
                    sk.top() /= num;
                }
                op = s[i];//新的符号或者结束
                num = 0;
            }
        }
        num = 0;
        while(sk.size())
        {
            num += sk.top();//累加
            sk.pop();
        }
        return num;
    }
};

##[最后一个单词的长度](https://leetcode-cn.com/problems/length-of-last-word)    

class Solution {
public:
    int lengthOfLastWord(string s) {
        if(s.size() == 0)  return 0;  
        int sLen = s.size() -1;  
        while(sLen >= 0 && s[sLen]==' ') sLen--;  
        int nCount = 0;  
        for(int i = sLen;i>=0 && s[i] != ' ';i--) nCount++;  
        return nCount;  
    }
};

##[实现strStr()](https://leetcode-cn.com/problems/implement-strstr)    

class Solution {
public:
    int strStr(string haystack, string needle) {
        if (needle.size() == 0)return 0;
        if (needle.size()> haystack.size())return -1;
        for(int i = 0;i < haystack.size() - needle.size()+1;++i)
        {
            if (haystack.substr(i,needle.size()) == needle)return i;
        }
        return -1;
    }
};

##[验证回文串](https://leetcode-cn.com/problems/valid-palindrome)   

class Solution {
public:
    bool isPalindrome(string s) {
        int left = 0,right = s.size()-1;
        while(left <= right)
        {
            while (left <= right && !isalpha(s[left]) && !isdigit(s[left]) )//跳过非字母和数字（只考虑字母和数字）
            {
                ++left;
            }
            while (left <= right && !isalpha(s[right]) && !isdigit(s[right]) )//跳过非字母和数字（只考虑字母和数字）
            {
                --right;
            }
            if (left > right)break;
            if (tolower(s[left]) != tolower(s[right]))return false;//tolower不会处理数字
            ++left,--right;
        }
        return true;
    }
};

##[报数](https://leetcode-cn.com/problems/count-and-say)    
 
##[字符串相乘](https://leetcode-cn.com/problems/multiply-strings)    

class Solution {
public:
    string multiply(string num1, string num2) 
    {
        string res;
        vector<int> mul(num1.size() + num2.size(),0);
        for(int i = 0;i < num1.size();++i)
        {
            for(int j = 0;j < num2.size();++j)
            {
                mul[i + j] += (num1[num1.size() - 1 - i] - '0') * (num2[num2.size() - 1 - j] - '0');
                //printf("1) %d %d   %d\n",i,j,mul[i+j]);
            }
        }
        for(int i = 0,c = 0,tmp = 0;i < mul.size();++i)
        {
            tmp = mul[i];
            mul[i] = (tmp + c) % 10;
            c = (tmp + c) /10;
        }
        int i = mul.size() -1; 
        while(mul[i] == 0 && i > 0)--i;
        for(;i >= 0;--i)
        {
            //printf("2)%d\n",mul[i]);
            res.push_back('0' + mul[i]);
        }
        return res;
    }
};

##[找不同](https://leetcode-cn.com/problems/find-the-difference)    
 
class Solution {
public:
    char findTheDifference(string s, string t) {
        int m[128] = {0};
        for(int i = 0;i< s.size();++i)m[s[i]]++;
        for(int i = 0;i< t.size();++i)
        {
            m[t[i]]--;
            if (m[t[i]] < 0) return t[i];
        }
        return 0;
    }
};
 
##[去除重复字母](https://leetcode-cn.com/problems/remove-duplicate-letters)
 
贪心计算字符
class Solution {
public:
    string removeDuplicateLetters(string s) {
        int m[256] = {0}, visited[256] = {0};
        string res = "0";
        for (auto a : s) ++m[a];//计数，为了判断字符数
        for (auto a : s) {
            --m[a];
            if (visited[a])
continue;//同样的字符前面访问过的就不再访问，因为没意义
            while (a < res.back() && m[res.back()])
{//较小的，后面还有该字符，则被移除（贪心）
                visited[res.back()] = 0;
                res.pop_back();
            }
            res += a;//加入新的字符
            visited[a] = 1;
        }
        return res.substr(1);
    }
};
 
##[最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring)    
 
class Solution {
public:
    int begin = 0;int maxLen = 0;
    string longestPalindrome(string s) {
        int len = s.length();
        if (len < 2)
            return s;
        for (int i = 0; i < len-1; i++) {
             extendPalindrome(s, i, i);  //assume odd length
             extendPalindrome(s, i, i+1); //assume even length.
        }
        return s.substr(begin, maxLen);
    }
    void extendPalindrome(const string& s, int left, int right) {
        while (left >= 0 && right < s.length() && s[left] == s[right]) {
            left--;right++;
        }
        if (maxLen < right - left - 1) {
            begin = left + 1;
            maxLen = right - left - 1;
        }
    }
};
 
##[反转字符串](https://leetcode-cn.com/problems/reverse-string)    
 
class Solution {
public:
    void reverseString(vector<char>& s) {
        for(int  b = 0, e = s.size()-1;b < e;++b,--e)
        {//利用异或的特性：相同的数异或会抵消掉
            s[b] ^= s[e];
            s[e] ^= s[b];
            s[b] ^= s[e];
        }
    }
};
 
##[反转字符串中的元音字母](https://leetcode-cn.com/problems/reverse-vowels-of-a-string)    
 
##[至少有K个重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-with-at-least-k-repeating-characters)    
 
##[字符串中的第一个唯一字符](https://leetcode-cn.com/problems/first-unique-character-in-a-string)    
 
##[字符串解码](https://leetcode-cn.com/problems/decode-string)    
 
##[找到字符串中所有字母异位词](https://leetcode-cn.com/problems/find-all-anagrams-in-a-string)    
 
##[无重复字符的最长子串](https://leetcode-cn.com/problems/longest-substring-without-repeating-characters)
 
class Solution {
public:
  int lengthOfLongestSubstring(string s)
  {
      if (s.size() <= 1)return s.size();
      vector<int> mark(128,-1);
      int global(0),left(0),right(0);
      for(;right < s.size();++right)
      {
          if (mark[s[right]] >= 0)
          {
              global = max(global,right - left);
              //cout << mark[s[right]] << " " << s[right] << endl;
              left = max(left,mark[s[right]]+1);//跳过无重复
              //left = mark[s[right]]+1;
          }
          mark[s[right]] = right;
      }
      global = max(global,right - left);
      return global;
  }
};
 
##[有效的字母异位词](https://leetcode-cn.com/problems/valid-anagram)    
 
class Solution {
public:
    bool isAnagram(string s, string t) 
    {
        if (s.size() != t.size())return false;
        if (s.size() == 0)return true;
        unordered_map<char,int> m1;
        unordered_map<char,int> m2;
        for(auto& c:s)
        {
            m1[c]++;
        }
        for(auto& c:t)
        {
            m2[c]++;
        }
        for(auto& iter:m1)
        {
            if (iter.second != m2[iter.first])return false;
        }
        return true;
    }
};
 
##[Fizz Buzz](https://leetcode-cn.com/problems/fizz-buzz)    
 
class Solution {
public:
    vector<string> fizzBuzz(int n) {
        vector<string> res;
        for(int i = 1;i <= n;++i)
        {
            if (i % 3 == 0 && i % 5 == 0)
            {
                res.push_back("FizzBuzz");
            }
            else if (i % 3 == 0)
            {
                res.push_back("Fizz");
            }
            else if (i % 5 == 0)
            {
                res.push_back("Buzz");
            }
            else
            {
                res.push_back(to_string(i));
            }
        }
        return res;
    }
};
 
##[字符串相加](https://leetcode-cn.com/problems/add-strings)    
 
class Solution {
public:
    string addStrings(string num1, string num2) {
        string res;
        int i(0),c(0),n;
        for(;i < num1.size() || i < num2.size();++i)
        {
            if (i < num1.size() && i < num2.size())
            {
                n = (num1[num1.size()-1-i] - '0')+ (num2[num2.size()-1-i] - '0') + c ; 
            }
            else if (i < num1.size())
            {
                n = (num1[num1.size()-1-i] - '0')+ c ; 
            }
            else
            {
                n = (num2[num2.size()-1-i] - '0') + c ; 
            }
            c = n /10;
            res.push_back('0' + n % 10);
        } 
        if (c)res.push_back('0' + c);
        reverse(res.begin(),res.end());
        return res;
    }
};
 
##[字符串中的单词数](https://leetcode-cn.com/problems/number-of-segments-in-a-string)    
 
class Solution {
public:
    int countSegments(string s) {
        int res = 0, n = s.size();
        for (int i = 0; i < n; ++i) {
            if (s[i] == ' ') continue;
            ++res;
            while (i < n && s[i] != ' ') ++i;
        }
        return res;
    }
};
 
##[最长回文串](https://leetcode-cn.com/problems/longest-palindrome)    
 
##[重复的子字符串](https://leetcode-cn.com/problems/repeated-substring-pattern)    
 
class Solution {
public:
    bool repeatedSubstringPattern(string s) {
        string newStr = s + s;
        newStr = newStr.substr(1, 2 * s.size() - 2);
        return newStr.find(s) != -1;
    }
};
 
##[键盘行](https://leetcode-cn.com/problems/keyboard-row)    
 
class Solution {
public:
    vector<string> findWords(vector<string>& words) {
        string arr1 = "qwertyuiop";
        string arr2 = "asdfghjkl";
        string arr3 = "zxcvbnm";
        unordered_map<char,int> m;
        for(auto c:arr1) m[c] = 1;
        for(auto c:arr2) m[c] = 2;
        for(auto c:arr3) m[c] = 3;
        vector<string> res;
        for(auto s:words)
        {
            int last(0), index(0);
            for(;index < s.size();++index) 
            {
                int i = m[tolower(s[index])];
                if (!i || (last && last != i))break;
                if (!last) last = i;
            }
            if (index == s.size())res.emplace_back(s);
        }
        return res;
    }
};
 
##[检测大写字母](https://leetcode-cn.com/problems/detect-capital)    
 
class Solution {
public:
    bool detectCapitalUse(string word) {
        int cnt = 0, n = word.size();
        for (int i = 0; i < n; ++i) {
            if (word[i] <= 'Z') ++cnt;
        }
        return cnt == 0 || cnt == n || (cnt == 1 && word[0] <= 'Z');
    }
};
 
##[单词替换](https://leetcode-cn.com/problems/replace-words)    
 
class Solution {
public:
    string replaceWords(vector<string>& dict, string sentence) {
        string res;  
        unordered_set<string> wordSet(dict.begin(),dict.end());  
        vector<string> words;
        stringstream ss(sentence);
        string w;
        while(getline(ss, w, ' '))
        {
            words.emplace_back(w);
        }
        for(int i=0;i<words.size();i++){  
            string word=words[i];  
            for(int j = 1; j <= word.length() && j <= 100;j++)
            {  
                const string &subWord=word.substr(0,j);  
                if(wordSet.count(subWord)){  
                    word = subWord;
                    break;  
                }  
            }  
            res += word + " ";
        }  
        res.pop_back();
        return res;
    }
};
 
##[词典中最长的单词](https://leetcode-cn.com/problems/longest-word-in-dictionary)
 
class Solution {
public:
    string longestWord(vector<string>& words) {
        string res = "";
        unordered_set<string> s;
        sort(words.begin(), words.end());
        for (string word : words) {
            if (word.size() == 1 || s.count(word.substr(0, word.size() - 1))) {
                res = (word.size() > res.size()) ? word : res;
                s.insert(word);
            }
        }
        return res;
    }
};
 
##[仅仅反转字母](https://leetcode-cn.com/problems/reverse-only-letters)    
 
class Solution {
public:
    string reverseOnlyLetters(string S) 
    {
        if (S.size() == 0)return S;
        string s2;
        for(int i = 0;i < S.size();++i)
        {
            if (isalpha(S[i]))
            {
                s2.push_back(S[i]);
            }
        }
        reverse(s2.begin(),s2.end());
        for(int i = 0,cnt = 0;i < S.size();++i)
        {
            if (isalpha(S[i]))
            {
                S[i] = s2[cnt++];
            }
        }
        return S;
    }
};
 
##[反转字符串 II](https://leetcode-cn.com/problems/reverse-string-ii)    

class Solution {
public:
    string reverseStr(string s, int k) {
        for(int i = 0;i < s.size();i += 2 * k)
        {
            reverse(s.begin()+i,s.begin()+ min(i + k,(int)s.size()));
        }
        return s;
    }
};
 
##[反转字符串中的单词 III](https://leetcode-cn.com/problems/reverse-words-in-a-string-iii)    
 
class Solution {
public:
    string reverseWords(string s) {
        for(int i = 0,left = 0;i <= s.size();++i)//最后一个字节为0
        {
            if (s[i] == ' ' || s[i] == 0)
            {
                reverse(s.begin()+left,s.begin()+i);
                while(s[i] == ' ' && i < s.size()) ++i;
                left = i;
            }
        }
        return s;
    }
};
 
##[实现一个魔法字典](https://leetcode-cn.com/problems/implement-magic-dictionary)    
 
class MagicDictionary {
public:
    /** Initialize your data structure here. */
    MagicDictionary() {
    }
    /** Build a dictionary through a list of words */
    void buildDict(vector<string> dict) {
        for (int i = 0; i < dict.size(); i++) {
            words.insert(dict[i]);    
         }
    }
    /** Returns if there is any word in the trie that equals to the given word after modifying exactly one character */
    bool search(string word) {
        for (int i = 0; i < word.size(); i++) {
            char wi = word[i];
            for (char ch = 'a'; ch <= 'z'; ch++) {
                if (ch == wi) continue;
                word[i] = ch;
                if (words.count(word)) return true;
            }
            word[i] = wi;
        }
        return false;
    }
    unordered_set<string> words;
};
 
##[验证回文字符串 Ⅱ](https://leetcode-cn.com/problems/valid-palindrome-ii)    
class Solution {
public:
    bool validPalindrome(string s) {
        int i = 0, j = s.length() - 1, count = 0;  
        while (i < j)  
        {  
            if (s[i] == s[j])  
            {  
                i++;  
                j--;  
            }  
            else  
            {  
                if (++count > 1) return false;  
                if (s[i + 1] != s[j] && s[i] != s[j - 1]) return false;  
                if (s[i + 1] == s[j])  
                {  
                    if (j > i + 2 && s[i + 2] != s[j - 1])
                    {
                    }
                    else
                    {
                        ++i;
                        continue;
                    }
                }  
                --j;  
            }  
        }  
        return true; 
    }
};

##[字母大小写全排列](https://leetcode-cn.com/problems/letter-case-permutation)    
 
class Solution {
public:
    vector<string> res;
    vector<string> letterCasePermutation(string S) {
        if (S.size() == 0) return {};
        dfs(S,0);
        return res;
    }
    void dfs(string s,int pos)
    {
        if (pos >= s.size())return;
        if (res.size() == 0 || res[res.size()-1] != s)res.push_back(s);
        dfs(s,pos + 1);
        if (s[pos] >= 'A' &&  s[pos] <= 'Z')
        {
            s[pos] = s[pos] + 'a' - 'A';
            res.push_back(s);
            dfs(s,pos + 1);
        }
        else if (s[pos] >= 'a' &&  s[pos] <= 'z')
        {
            s[pos] = s[pos] + 'A' - 'a';
            res.push_back(s);
            dfs(s,pos + 1);
        }
    }
};
 
##[旋转字符串](https://leetcode-cn.com/problems/rotate-string)    
 
class Solution {
public:
    bool rotateString(string A, string B) {
        return (A.length() == B.length()) && ((A + A).find(B) != string::npos);
    }
};
 
##[字符的最短距离](https://leetcode-cn.com/problems/shortest-distance-to-a-character)    
 
class Solution {
public:
    vector<int> shortestToChar(string S, char C) {
        vector<int> cV;
        for(int i = 0;i < S.size();++i) if (S[i] == C)cV.emplace_back(i);
        int index = 0;
        vector<int> res;
        for(int i = 0;i < S.size();++i) 
        {
            if (index < cV.size() -1 && abs(cV[index] - i) > abs(cV[index + 1] - i))
            {
                ++index;
            }
            res.emplace_back(abs(cV[index] - i));
        }
        return res;
    }
};
 
#哈希表

 
##[验证外星语词典](https://leetcode-cn.com/problems/verifying-an-alien-dictionary)    
 
class Solution {
public:
    bool isAlienSorted(vector<string>& words, string order) {
        int m[26];
        for (int i = 0; i < 26; i++) {
            m[order[i] - 'a'] = i;
        }
        vector<string> newWords;
        for (string w: words) 
        {
            for (int i = 0; i < w.size(); i++)
            {
                w[i] = 'a' + m[w[i] - 'a'];
            }
            newWords.push_back(w);
        }
        for (int i = 1; i < newWords.size(); i++)
            if (newWords[i] < newWords[i-1])
                return false;
        return true;
    }
};
 
##[键值映射](https://leetcode-cn.com/problems/map-sum-pairs)    
 
class MapSum {
public:
    /** Initialize your data structure here. */
    MapSum() {
    }
    void insert(string key, int val) {
        m[key] = val; 
    }
    int sum(string prefix) {
        int sum = 0;
        for (auto s : m) {
            if (prefix == s.first.substr(0,prefix.size()))
            {
                sum += s.second;
            }
        }
        return sum;
    }
    unordered_map<string,int> m;
};
 
#二叉树
 
常用dfs或者bfs，前序、中序、后序遍历，dfs一般需要递归

 
##[求根到叶子节点数字之和](https://leetcode-cn.com/problems/sum-root-to-leaf-numbers)    
 
class Solution {
public:
    vector<int> sums;
    int sumNumbers(TreeNode* root) {
        if (!root)return 0;
        dfs(root,0);
        return accumulate(sums.begin(),sums.end(),0);
    }
    void dfs(TreeNode* root,int num)
    {
        if (!root->left && !root->right)
        {
            sums.push_back(num * 10 + root->val);
            return;
        }
        if (root->left)dfs(root->left,num * 10 + root->val);
        if (root->right)dfs(root->right,num * 10 + root->val);
    }
};
 
##[完全二叉树的节点个数](https://leetcode-cn.com/problems/count-complete-tree-nodes)    
 
class Solution {
public:
    int countNodes(TreeNode* root) {
      if(root == NULL) return 0;
        int ldepth(0),rdepth(0);
        TreeNode *left = root,*right = root;
        while (left){left = left->left;++ldepth;}
        while (right){right = right->right;++rdepth;}
        if(ldepth == rdepth) return (1 << ldepth) - 1;
        return countNodes(root->left)+countNodes(root->right)+1;
    }
};
 
##[二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum)    
 
class Solution {
public:
    int maxPathSum(TreeNode* root) {
        int ma = INT_MIN;
        if (!root)return ma;
        maxTree(root,ma);
        return ma;
    }
    int maxTree(TreeNode* root,int &ma) 
    {
        if (!root)return 0;
        int leftPath = maxTree(root->left,ma);
        int rightPath = maxTree(root->right,ma);
        int v = max(root->val,root->val + max(leftPath,0)+ max(rightPath,0));//本树路径最大值
        ma = max(ma,v);
        return max(0,max(root->val + leftPath,root->val + rightPath));//边路径最大值
    }
};
 
##[二叉树的右视图](https://leetcode-cn.com/problems/binary-tree-right-side-view)    bfs 
 
class Solution {
public:
    vector<TreeNode*> level[2];
    int index;
    vector<int> res;
    vector<int> rightSideView(TreeNode* root) {
        if (!root)return res;
        index = 0;
        level[index].push_back(root);
        bfs();
        return res;
    }
    void bfs()
    {
        if (level[index].size() == 0)return;
        for(auto n:level[index])
        {
            if (n->left)level[1-index].push_back(n->left);
            if (n->right)level[1-index].push_back(n->right);
        }
        res.push_back(level[index][level[index].size()-1]->val);
        level[index].clear();
        index = 1 - index;
        bfs();
    }
};
 
##[路径总和](https://leetcode-cn.com/problems/path-sum)    
 
class Solution {
public:
    bool hasPathSum(TreeNode* root, int sum) {
        if (!root)return false;
        return dfs(root,0,sum);
    }
    bool dfs(TreeNode* root,int s,int sum)
    {
        if (!root)return false;
        if (!root->left && !root->right)
        {
            return root->val + s == sum;
        }    
        return dfs(root->left,root->val + s,sum)||dfs(root->right,root->val + s,sum);
    }
};
 
##[路径总和 II](https://leetcode-cn.com/problems/path-sum-ii)  dfs
 
//遍历节点，选择或者不选择
class Solution {
public:
    vector<vector<int>> pathSum(TreeNode* root, int sum) {
        vector<int> path;
        dfs(root,path,0,sum);
        return res;
    }
    void dfs(TreeNode* root,vector<int> &path,int s ,int sum)
    {
        if (!root)return;
        path.push_back(root->val);//选择
        if (!root->left && !root->right && root->val + s == sum)
        {
            res.push_back(path);
            path.pop_back();
            return;
        }
        dfs(root->left,path,root->val + s,sum);
        dfs(root->right,path,root->val + s,sum);
        path.pop_back();//不选择，不选择需要放在最后边，因为前面需要遍历
    }
    vector<vector<int>> res;
};
 
##[二叉树展开为链表](https://leetcode-cn.com/problems/flatten-binary-tree-to-linked-list)    
 
二叉树与链表关系的处理
void flatten(TreeNode* root) {
        while (root) {  
            if (root->left) {  
                TreeNode* pre = root->left;  
                while (pre->right)  
                    pre = pre->right;  
                pre->right = root->right;  
                root->right = root->left;  
                root->left = NULL;  
            }  
            root = root->right;  
        }  
    }
 
##[填充每个节点的下一个右侧节点指针](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node)    
  
class Solution {
public:
    void connect(TreeLinkNode *root) {
        if (root == NULL)
            return;
        TreeLinkNode* pre = root;
        TreeLinkNode* cur = NULL;
        while (pre)
        {
            cur = pre;
            while (cur)
            {
                if (cur->left)
                    cur->left->next = cur->right;
                if (cur->right && cur->next)
                    cur->right->next = cur->next->left;
                cur = cur->next;
            }
            pre = pre->left;
        }
    }
};
 
##[填充每个节点的下一个右侧节点指针 II](https://leetcode-cn.com/problems/populating-next-right-pointers-in-each-node-ii)    
  
class Solution {
public:
    Node* connect(Node  *root) {
        if (!root)return root;
        vector<Node*> v[2];
        int index = 0;
        v[index].push_back(root);
        while(v[index].size() > 0)
        {
            for(int i = 0;i < v[index].size();++i)
            {
                if (i < v[index].size()-1)
                {
                    v[index][i]->next = v[index][i+1];
                }
                if(v[index][i]->left) v[1-index].push_back(v[index][i]->left);
                if(v[index][i]->right) v[1-index].push_back(v[index][i]->right);
            }
            v[index].clear();
            index = 1 - index;
        }
        return root;
    }
};
 
##[二叉树的前序遍历](https://leetcode-cn.com/problems/binary-tree-preorder-traversal)    
 
class Solution {
public:
    vector<int> preorderTraversal(TreeNode* root) {
        if(!root)return {};
        vector<int> res;
        stack<TreeNode*> s;
        TreeNode* p = root;
        while(p ||s.size())
        {
            while(p)
            {
                res.push_back(p->val);
                s.push(p);
                p = p->left;
            }
            p = s.top();
            s.pop();
            p = p->right;
        }
        return res;
    }
};
 
##[二叉树的后序遍历](https://leetcode-cn.com/problems/binary-tree-postorder-traversal)    
 
class Solution {
public:
    vector<int> postorderTraversal(TreeNode* root) 
    {
        if (!root)return {};
        TreeNode* p = root;
        vector<int> res;
        stack<TreeNode*> s;
        while(p || s.size())
        {
            while(p)
            {
                res.push_back(p->val);//先序
                s.push(p);
                p = p->right;//跟先序的相反
            }
            p = s.top();
            s.pop();
            p = p->left;//跟先序的相反
        }
            reverse(res.begin(),res.end());//反序
        return res;
    }
};

 
##[翻转二叉树](https://leetcode-cn.com/problems/invert-binary-tree)    
 
class Solution {
public:
    TreeNode* invertTree(TreeNode* root) {
        if (!root)return root;
        q.emplace(root);
        while (q.size() > 0)
        {
            TreeNode* f = q.front();
            swap(f->left,f->right);
            if (f->left)
            q.emplace(f->left);  
            if (f->right)
            q.emplace(f->right);  
            q.pop();
        }
        return root;
    }
    queue<TreeNode*> q;
};
 
##[二叉树的序列化与反序列化](https://leetcode-cn.com/problems/serialize-and-deserialize-binary-tree)
 
//先序遍历序列化，对应的就可以使用dfs来反序列化了
class Codec {
public:
    // Encodes a tree to a single string.
    string serialize(TreeNode* root)//"[1,2,3,\#,\#,4,5]"
    {
        if (!root)return "";
        string res("[");
        //先序遍历
        stack<TreeNode*> sk;
        sk.push(root);
        while(sk.size())
        {
            TreeNode* p = sk.top();
            sk.pop();
            if (!p)
            {
                res.append("\#,");
            }
            else
            {
                res.append(to_string(p->val) + ",");
                sk.push(p->right);//有否子节点也压入
                sk.push(p->left);
            }
            //printf("%u\\n",sk.size());
        }
        res.pop_back();
        res.append("]");
        //printf("%s\\n",res.c_str());
        return res;
    }
    // Decodes your encoded data to tree.
    TreeNode* deserialize(string data) {
        if (data.size() <= 2)return NULL;
        vector<string> total;
        data.erase(data.begin());
        data.pop_back();
        string str;
        stringstream in(data);
        while(getline(in, str,','))
        {
            total.push_back(str);
        }
        int index(0);
        TreeNode* root = GetTree(total,index);
        return root;
    }
    TreeNode* GetTree(vector<string> &total,int &index)//先序遍历的方式解析
    {
        if (index == total.size())return NULL;
        if (total[index] == "\#"){++index;return NULL;}//遇到\#表示该子树已结束
        //处理当前节点
        int num(0);
        int op = total[index][0] == '-' ? -1:1;
        for(int i = (op == -1) ? 1:0;i < total[index].size();++i)
        {
            num = num * 10 + total[index][i] - '0';
        }
        TreeNode* node = new TreeNode(num * op);
        ++index;
        node->left = GetTree(total,index);
        node->right = GetTree(total,index);
        return node;
    }
};
 
##[二叉搜索树中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst)
 
因为是二叉搜索树，使用中序遍历就可以，就是从小到大。使用栈来遍历
class Solution {
public:
    int kthSmallest(TreeNode* root, int k)
    {
        //bst,中序遍历方式
        if (!root)return 0;
        stack<TreeNode*> sk;
        TreeNode* p = root;
        int cnt(0);
        while(p || sk.size())
        {
            while(p)
            {
                sk.push(p);
                p = p->left;
            }
            p = sk.top();
            sk.pop();
            if (++cnt == k)return p->val;
            p = p->right;
        }
        return 0;
    }
};
 
##[二叉搜索树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-search-tree)
 
因为是二叉搜索树，则可以根据值选择哪边，在中间的为本节点，大了或小了就选边，然后遍历下去
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q)
    {
        while(root)
        {
            if (!root || root == p|| root == q)
            {
                break;
            }
            if (max(p->val,q->val) < root->val)
            {
                root = root->left;
            }
            else if  (min(p->val,q->val) > root->val)
            {
                root = root->right;
            }
            else break;
        }
        return root;
    }
};
 
##[二叉树的最近公共祖先](https://leetcode-cn.com/problems/lowest-common-ancestor-of-a-binary-tree) 
 
因为是二叉树，求的是最近的，递归下去获取需求的节点，在两边的就是本节点，在一边的就是该边返回的
class Solution {
public:
    TreeNode* lowestCommonAncestor(TreeNode* root, TreeNode* p, TreeNode* q)
{
        if (!root || root == p|| root == q) return root;
        TreeNode* left = lowestCommonAncestor(root->left,p,q);
        TreeNode* right = lowestCommonAncestor(root->right,p,q);
        if (left && right) return root;
        return left ? left:right;
    }
};
 
##[左叶子之和](https://leetcode-cn.com/problems/sum-of-left-leaves)    
 
##[二叉树的中序遍历](https://leetcode-cn.com/problems/binary-tree-inorder-traversal)    
 
##[不同的二叉搜索树 II](https://leetcode-cn.com/problems/unique-binary-search-trees-ii)    
 
##[不同的二叉搜索树](https://leetcode-cn.com/problems/unique-binary-search-trees)    
 
##[验证二叉搜索树](https://leetcode-cn.com/problems/validate-binary-search-tree)    
 
##[恢复二叉搜索树](https://leetcode-cn.com/problems/recover-binary-search-tree)    
 
##[相同的树](https://leetcode-cn.com/problems/same-tree)    
 
##[对称二叉树](https://leetcode-cn.com/problems/symmetric-tree)    
 
##[二叉树的层次遍历](https://leetcode-cn.com/problems/binary-tree-level-order-traversal)    
 
##[二叉树的锯齿形层次遍历](https://leetcode-cn.com/problems/binary-tree-zigzag-level-order-traversal)    
 
##[二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree)    
 
##[从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal)    
 
class Solution {
public:
    unordered_map<int,int> inmap;
    TreeNode* buildTree(vector<int>& preorder, vector<int>& inorder) {
        if (preorder.size() == 0 || inorder.size() == 0|| preorder.size() != inorder.size())return NULL;
        for(int i = 0;i < inorder.size();++i)inmap[inorder[i]] = i;
        return build(preorder,0, inorder.size() - 1, inorder,0, inorder.size() - 1);
    }
    TreeNode *build(vector<int> &pre, int b1, int e1, vector<int> &in,int b2, int e2) {
        if (b1 > e1 || b2 > e2) return NULL;
        TreeNode *root = new TreeNode(pre[b1]);
        int mid2 = inmap[pre[b1]];
        int len2 = mid2 - b2;
        root->left = build(pre,b1 + 1, b1 + len2, in, b2, mid2 - 1);
        root->right = build(pre,b1 + len2 + 1, e1, in,mid2 + 1, e2);
        return root;
    }
};
 
##[从中序与后序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-inorder-and-postorder-traversal)    
 
class Solution {
public:
	unordered_map<int, int> inmap;
    TreeNode* buildTree(vector<int>& inorder, vector<int>& postorder) {
        if(inorder.size() == 0 || postorder.size() == 0 || inorder.size() != postorder.size())
            return NULL;
        for(int i = 0; i < inorder.size(); i ++) inmap[inorder[i]] = i;
        return dfs(inorder, 0, inorder.size() - 1, postorder,0,postorder.size() - 1);
    }
    TreeNode* dfs(vector<int>& inorder, int b1, int e1, vector<int>& postorder,int b2, int e2) {
        if(e1 < b1 || e2 < b2)return NULL;
        TreeNode* root = new TreeNode(postorder[e2]);
        int mid1 = inmap[postorder[e2]];
        int len = e1 - mid1;
        root->left = dfs(inorder, b1, mid1 - 1, postorder,b2, e2 - 1 - len);
		root->right = dfs(inorder, mid1 + 1, e1, postorder,e2 - len, e2 - 1);
        return root;
    }
};
 
##[二叉树的层次遍历 II](https://leetcode-cn.com/problems/binary-tree-level-order-traversal-ii)    
 
class Solution {
public:
    vector<vector<int>> levelOrderBottom(TreeNode* root) {
        if (!root)return {};
        vector<vector<int>> res;
        vector<int> tmp;
        list<TreeNode*> l;
        l.push_back(root);
        while(l.size())
        {
            int c = l.size();
            while(c--)
            {
                tmp.push_back(l.front()->val);
                if (l.front()->left)l.push_back(l.front()->left);
                if (l.front()->right)l.push_back(l.front()->right);
                l.pop_front();
            }
            res.push_back(tmp);
            tmp.clear();
        }
        reverse(res.begin(),res.end());
        return res;
    }
};
 
##[将有序数组转换为二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-array-to-binary-search-tree)    
 
class Solution {
public:
    TreeNode* sortedArrayToBST(vector<int>& nums) {
        return dfs(nums,0,nums.size()-1);
    }
    TreeNode* dfs(vector<int>& nums,int left,int right)
    {
        if (left > right) return NULL;  
        int mid = (left + right) / 2;  
        TreeNode* root = new TreeNode(nums[mid]);  
        root->left = dfs(nums, left, mid - 1);  
        root->right = dfs(nums, mid + 1, right);  
        return root;  
    }
};

 
##[有序链表转换二叉搜索树](https://leetcode-cn.com/problems/convert-sorted-list-to-binary-search-tree)    
 
class Solution {
public:
    vector<ListNode*> nodeslist;
    TreeNode* sortedListToBST(ListNode* head) {
        if(head == NULL) {
             return NULL;
         }
         while(head) {
             nodeslist.push_back(head);
             head = head->next;
         }
         return constructTree(0, nodeslist.size() - 1);
     }
     TreeNode* constructTree(int start, int end) {
         if(start > end) {
             return NULL;
         }
         int mid = (start + end) >>1;
         TreeNode* root = new TreeNode(nodeslist[mid]->val);
         root->left = constructTree(start, mid - 1);
         root->right = constructTree(mid + 1, end);
         return root;
     }
};

 
##[平衡二叉树](https://leetcode-cn.com/problems/balanced-binary-tree)    

class Solution {
public:  
    int cntHeight(TreeNode *node) {  
        if(node == NULL) return 0;  
        int l = cntHeight(node->left);  
        if (l < 0)return -1;
        int r = cntHeight(node->right);  
        if (r < 0)return -1;
        if(abs(l-r) > 1) return -1; //不平衡
        return max(l, r) + 1;  
    }  
    bool isBalanced(TreeNode *root) {    
        if (!root)return true; 
        return cntHeight(root) >= 0;
    }  
};

 
##[二叉树的最小深度](https://leetcode-cn.com/problems/minimum-depth-of-binary-tree)    
 
class Solution {
public:
    int minDepth(TreeNode* root) {
        if (!root)return 0;
        if (!root->left)return minDepth(root->right) + 1;
        if (!root->right)return minDepth(root->left) + 1;
        return min(minDepth(root->left),minDepth(root->right)) + 1;
    }
};

 
##[二叉树的所有路径](https://leetcode-cn.com/problems/binary-tree-paths)    
 
class Solution {
    vector<string>  ps;
public:
    vector<string> binaryTreePaths(TreeNode* root) {
        string s;
        dfs(root,s);
        return ps;
    }
    void dfs(TreeNode* root,string s)
    {
        if (!root)return;
        if (!root->left && !root->right)
        {
            s += to_string(root->val);
            ps.push_back(s);
            return;
        }
        s += to_string(root->val);
        if (root->left)
        {
            dfs(root->left,s + "->");
        }
        if (root->right)
        {
            dfs(root->right,s + "->");
        } 
    }
};

 
##[路径总和 III](https://leetcode-cn.com/problems/path-sum-iii)    
 
class Solution {
public:
    int pathSum(TreeNode* root, int sum) {
        if (!root) return 0;
        int res = findPath(root, 0, sum) + pathSum(root->left, sum) + pathSum(root->right, sum);
        return res;
     }
     int findPath(TreeNode* node, int curSum, int sum) {
         if (!node) return 0;
         curSum += node->val;
         return (curSum == sum) + findPath(node->left, curSum, sum) + findPath(node->right, curSum, sum);
     }
};

 
##[找树左下角的值](https://leetcode-cn.com/problems/find-bottom-left-tree-value)    
 
class Solution {
public:
    int findBottomLeftValue(TreeNode* root) {
        if (!root)return 0;
        res = index = 0;
        v[index].emplace_back(root);
        bfs();
        return res;
    }
    void bfs()
    {
        if (v[index].size() == 0)return;
        for(auto node:v[index])
        {
            if (node->left)v[1-index].emplace_back(node->left);
            if (node->right)v[1-index].emplace_back(node->right);
        }
        if (v[1-index].size() == 0)
        {
            res = v[index][0]->val;
            return;
        }
        v[index].clear();
        index = 1 - index;
        bfs();
    }
    int res;
    vector<TreeNode*> v[2];
    int index;
};

 
##[二叉搜索树中的众数](https://leetcode-cn.com/problems/find-mode-in-binary-search-tree)    
 
class Solution {
public:
    unordered_map<int,int> m;
    vector<int> findMode(TreeNode* root) {
        vector<int> res;
        if (!root)return res;
        stack<TreeNode*> s;
        s.push(root);
        int maxCount(0);
        while(s.size() > 0)
        {
            root = s.top();s.pop();
            ++m[root->val];
            if (m[root->val] > maxCount)maxCount = m[root->val];
            if (root->right)s.push(root->right);
            if (root->left)s.push(root->left);
        }
        for(auto it:m)
        {
            if (it.second == maxCount)
            {
                res.push_back(it.first);
            }
        }
        return res;
    }
};

 
##[N叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-n-ary-tree)    
 
class Solution {
public:
    int maxDepth(Node* root) {
        if (!root)return 0;
        vector<Node*> l[2];
        int index = 0;
        l[index].push_back(root);
        int res = 0;
        while(l[index].size())
        {
            //printf("%d\n",l[index].size());
            ++res;
            for(auto n:l[index])
            {
                if (n)
                {
                    for(auto sub:n->children)
                    if (sub)l[1-index].push_back(sub);
                }
            }
            l[index].clear();
            index = 1 - index;
        }
        return res;
    }
};

 
##[二叉树的直径](https://leetcode-cn.com/problems/diameter-of-binary-tree)    
 
class Solution {
public:
    int diameterOfBinaryTree(TreeNode* root) {
        if (!root) return 0;
        int res = getHeight(root->left) + getHeight(root->right);
        return max(res, max(diameterOfBinaryTree(root->left), diameterOfBinaryTree(root->right)));
    }
    int getHeight(TreeNode* node)
    {
        if (!node) return 0;
        if (m.count(node)) return m[node];
        int h = 1 + max(getHeight(node->left), getHeight(node->right));
        return m[node] = h;
    }
    unordered_map<TreeNode*, int> m;
};

 
##[最长同值路径](https://leetcode-cn.com/problems/longest-univalue-path)    
 
class Solution {
public:
    int res = 0;
    int longestUnivaluePath(TreeNode* root) {
        dfs(root, 0);
		return res;
	}
	int dfs(TreeNode* root, int n) {
		if (!root) return 0;
		int left = dfs(root->left, root->val);
		int right = dfs(root->right, root->val);
		res = max(res, left + right);//路径
		return root->val == n ? max(left, right) + 1 : 0;//路径
	}
};

 
##[两数之和 IV - 输入 BST](https://leetcode-cn.com/problems/two-sum-iv-input-is-a-bst)    
 
class Solution {
public:
    bool findTarget(TreeNode* root, int k) {
        if(!root) return false;
        unordered_map<int,int> m;
        stack<TreeNode* > s;
        TreeNode* p = root;
        while(p || s.size())
        {
            while(p)
            {
                s.push(p);
                p = p->left;
            }
            p = s.top();
            s.pop();
            if (m[k - p->val])
            {
                return true;
            }
            m[p->val] = 1;
            p = p->right;
        }
        return false;
    }
};

 
##[二叉树最大宽度](https://leetcode-cn.com/problems/maximum-width-of-binary-tree)    
 
class Solution {
public:
    int widthOfBinaryTree(TreeNode* root) {
        long double maxWidth = 0;
        vector<long double> start;
        dfs(root, 0, 1, start, maxWidth);
        return maxWidth;
    }
    void dfs(TreeNode* root, int depth, long double index, vector<long double>& start, long double& maxWidth)
    {
        if (root)
        {
            if (depth >= start.size())
                start.push_back(index);
            maxWidth = max(maxWidth, index - start[depth] + 1);
            dfs(root->left, depth + 1, 2 * index, start, maxWidth);
            dfs(root->right, depth + 1, 2 * index + 1, start, maxWidth);
        }
    } 
};

 
##[修剪二叉搜索树](https://leetcode-cn.com/problems/trim-a-binary-search-tree)    
 
class Solution {
public:
    TreeNode* trimBST(TreeNode* root, int L, int R) {
        if (!root)
        return NULL;
        if (root->val < L) {
            return trimBST(root->right, L, R);
        }
        else if (root->val > R) {
            return trimBST(root->left, L, R);
        }
        else {
            root->left = trimBST(root->left, L, R);
            root->right = trimBST(root->right, L, R);
            return root;
        }
    }
};

 
##[二叉树的层平均值](https://leetcode-cn.com/problems/average-of-levels-in-binary-tree)    
 
class Solution {
public:
    vector<double> averageOfLevels(TreeNode* root) {
        if (!root)return {};
        index = 0;
        dataList[index].push_back(root);
        bfs();
        return res;
    }
    void bfs()
    {
        if (dataList[index].size() == 0)return;
        double avg(0.0);
        for(int i = 0;i < dataList[index].size();++i)
        {
            TreeNode* node = dataList[index][i];
            avg += node->val;
            if (node->left) dataList[1-index].push_back(node->left);
            if (node->right) dataList[1-index].push_back(node->right);
        }
        res.push_back(avg/dataList[index].size());
        dataList[index].clear();
        index = 1-index;
        bfs();
    }
    int index;
    vector<TreeNode*> dataList[2];
    vector<double> res;
};

 
##[二叉搜索树的最小绝对差](https://leetcode-cn.com/problems/minimum-absolute-difference-in-bst)    
 
class Solution {
public:
    int getMinimumDifference(TreeNode* root) {
        long diff = INT_MAX, pre = INT_MIN;  
        stack<TreeNode*> stk;  
        TreeNode* p = root;  
        //中序遍历
        while(p || !stk.empty()) {  
            while(p) { 
                stk.push(p);  
                p = p->left; 
            }  
            p = stk.top();  
            stk.pop();  
            diff = min(diff, p->val - pre);  
            pre = p->val;  
            p = p->right;   
        }  
        return diff;  
    }
};

 
##[把二叉搜索树转换为累加树](https://leetcode-cn.com/problems/convert-bst-to-greater-tree)    
 
class Solution {
public:
    TreeNode* convertBST(TreeNode* root) {
        if (!root)return root;
        int sum(0);
        convertBST(root,sum);
        return root;
    }
    void convertBST(TreeNode* root,int &s)
    {
        if (!root)return; 
        convertBST(root->right,s);
        root->val += s;
        s = root->val;
        convertBST(root->left,s);
    }
};

 
##[二叉搜索树中的搜索](https://leetcode-cn.com/problems/search-in-a-binary-search-tree)    
 
class Solution {
public:
    TreeNode* searchBST(TreeNode* root, int val) {
        if (!root) return NULL;
        TreeNode*  p = root;
        while(p)
        {
            if (p->val == val)break;
            else if (p->val > val)p = p->left; 
            else p = p->right; 
        }
        return p;
    }
};

 
##[二叉搜索树结点最小距离](https://leetcode-cn.com/problems/minimum-distance-between-bst-nodes)    
 
class Solution {
public:
    int minDiffInBST(TreeNode* root) {
        if (!root)return 0;
        stack<TreeNode*> s;
        long m1(INT_MIN),m2(INT_MIN),diff(INT_MAX);
        TreeNode* p = root;
        while(p||s.size())
        {
            while(p)
            {
                s.push(p);
                p = p->left;
            }
            p = s.top();
            s.pop();
            m1 = m2;
            m2 = p->val;
            diff = min(diff,m2 - m1);
            p = p->right;
        }
        return diff;
    }
};

 
##[二叉树的坡度](https://leetcode-cn.com/problems/binary-tree-tilt)    
 
class Solution {
public:
    int findTilt(TreeNode* root) {
        if (NULL == root)return 0;
        int sum = 0;
        dfs(root,sum);
        return sum;
    }
    void dfs(TreeNode* n,int &sum)
    {
        if (NULL == n)return;
        int l = 0;int r = 0;
        subtree(n->left,l);
        subtree(n->right,r);
        sum += abs(l - r);
        dfs(n->left,sum);
        dfs(n->right,sum);
    }
    void subtree(TreeNode* n,int &s)
    {
        if (NULL == n)return;
        s += n->val;
        if (n->left)subtree(n->left,s);
        if (n->right)subtree(n->right,s);
    }
};

 
##[另一个树的子树](https://leetcode-cn.com/problems/subtree-of-another-tree)    
 
class Solution {
public:
    list<TreeNode*> l;
    bool isSubtree(TreeNode* s, TreeNode* t) {
        list<TreeNode*> sl;
        sl.push_back(s);
        while(sl.size() > 0)
        {
            s = sl.front();
            sl.pop_front();
            if (isSame(s,t))return true;
            if (s->left)sl.push_back(s->left);
            if (s->right)sl.push_back(s->right);
        }
        return false;
    }
    bool isSame(TreeNode* p, TreeNode* q) 
    {
        l.clear();
        l.push_back(p);
        l.push_back(q);
        while(l.size() > 1)
        {
            p = l.front();
            l.pop_front();
            q = l.front();
            l.pop_front();
            if (p || q) 
            {
                if (!p || !q) return false;    
                if (p->val != q->val) return false;    
                if ((p->left && !q->left) || (p->right && !q->right)) return false;    
                l.push_back(p->left);
                l.push_back(q->left);
                l.push_back(p->right);
                l.push_back(q->right);
            }
        }
        return true;
    }
};

 
##[根据二叉树创建字符串](https://leetcode-cn.com/problems/construct-string-from-binary-tree)    
 
class Solution {
public:
    string tree2str(TreeNode* t) {
        if (!t) return "";
        string res = "";
        helper(t, res);
        return string(res.begin() + 1, res.end() - 1);
    }
    void helper(TreeNode* t, string& res) {
        if (!t) return;
        res += "(" + to_string(t->val);
        if (!t->left && t->right) res += "()";
        helper(t->left, res);
        helper(t->right, res);
        res += ")";
    }
};

 
##[合并二叉树](https://leetcode-cn.com/problems/merge-two-binary-trees)    
 
class Solution {
public:
    TreeNode* mergeTrees(TreeNode* t1, TreeNode* t2) {
        if (t1 == NULL) {
            return t2;
        } else if (t2 == NULL) {
            return t1;
        } else {
            TreeNode* newNode = new TreeNode(t1->val + t2->val);
            newNode->left = mergeTrees(t1->left, t2->left);
            newNode->right = mergeTrees(t1->right, t2->right);
            return newNode;
        }
    }
};

#多线程
##[按序打印](https://leetcode-cn.com/problems/print-in-order/)    
class Foo {
    mutex l1;
    mutex l2;
public:
    Foo() {
        // Initialize user defined data here.
        l1.lock();
        l2.lock();
    }
    void first(function<void()> printFirst) {
        // printFirst() outputs "first". Do not change or remove this line.
        printFirst();
        l1.unlock();
    }
    void second(function<void()> printSecond) {
        l1.lock();
        // printSecond() outputs "second". Do not change or remove this line.
        printSecond();
        l2.unlock();
    }
    void third(function<void()> printThird) {
        l2.lock();
        // printThird() outputs "third". Do not change or remove this line.
        printThird();
    }
};


##[交替打印FooBar](https://leetcode-cn.com/problems/print-foobar-alternately/)    
class FooBar {
private:
    int n;
    mutex l1;
    mutex l2;
public:
    FooBar(int n) {
        this->n = n;
        l2.lock();
    }
    void foo(function<void()> printFoo) {
        for (int i = 0; i < n; i++) {
        	// printFoo() outputs "foo". Do not change or remove this line.
        	l1.lock();
            printFoo();
            l2.unlock();
        }
    }
    void bar(function<void()> printBar) {
        for (int i = 0; i < n; i++) {
        	// printBar() outputs "bar". Do not change or remove this line.
        	l2.lock();
            printBar();
            l1.unlock();
        }
    }
};

##[打印零与奇偶数](https://leetcode-cn.com/problems/print-zero-even-odd/)    
class ZeroEvenOdd {
private:
    int n;
    mutex z;
    mutex e;
    mutex o;
public:
    ZeroEvenOdd(int n) {
        this->n = n;
        e.lock();
        o.lock();
    }
    // printNumber(x) outputs "x", where x is an integer.
    void zero(function<void(int)> printNumber) {
        for (int i = 1; i <= n; i++) {
            z.lock();
            printNumber(0);
            if (i % 2) o.unlock();
            else e.unlock();
        }
    }
    void even(function<void(int)> printNumber) {
        for (int i = 2; i <= n; i+=2) {
            e.lock();
            printNumber(i);
            z.unlock();
        }
    }
    void odd(function<void(int)> printNumber) {
        for (int i = 1; i <= n; i+=2) {
            o.lock();
            printNumber(i);
            z.unlock();
        }
    }
};

##[H2O 生成](https://leetcode-cn.com/problems/building-h2o/)    
class H2O {
    mutex h;
    mutex o;
    int n;
public:
    H2O() {
        n = 2;
        o.lock();
    }
    void hydrogen(function<void()> releaseHydrogen) {
        // releaseHydrogen() outputs "H". Do not change or remove this line.
        h.lock();
        if (n > 0)
        {
            releaseHydrogen();
            n--;
            if (n > 0)h.unlock();
            else o.unlock();
        }
    }
    void oxygen(function<void()> releaseOxygen) {
        // releaseOxygen() outputs "O". Do not change or remove this line.
        o.lock();
        if (n == 0)
        {
            releaseOxygen();
            n = 2;
            h.unlock();
        }
    }
};

#shell
##[统计词频](https://leetcode-cn.com/problems/word-frequency/)   
grep -oE  '[a-z]+' words.txt |sort|uniq -c |sort -r|awk '{print $2" "$1}'

##[有效电话号码](https://leetcode-cn.com/problems/valid-phone-numbers/)   

^(\([0-9]{3}\) ){1}[0-9]{3}-[0-9]{4}$ 匹配形如(123) 456-7890的电话号码
^([0-9]{3}-){2}[0-9]{4}$ 匹配形如987-123-4567的电话号码

grep -oE '^\([0-9]{3}\) [0-9]{3}-[0-9]{4}$|^[0-9]{3}-[0-9]{3}-[0-9]{4}$' file.txt 

##[转置文件](https://leetcode-cn.com/problems/transpose-file/)   

一共执行了三次for循环,每次分别读取一行的内容。

第一次读取的是第一行的内容 name age: 
NF等于2,NR等于1,然后s[1] = name , s[2] = age。

第二次读取的是第二行的内容 alice 21: 
NF等于2,NR等于2,然后s[1] = name alice ,s[2] = age 21。
第三次读取的是第三行的内容 ryan 30: 
NF等于2,NR等于2,然后s[1] = name alice ryan,s[2] = age 21 30。

最后执行END里面的,将其输出即可。

awk '{for (i = 1;i <= NF;++i) { if (NR == 1) s[i] = $i; else s[i] = s[i] " " $i; }}  END { for (i = 1;i <= NF;++i) {print s[i];} } ' file.txt  

##[第十行](https://leetcode-cn.com/problems/tenth-line/)  

awk 'NR == 10' file.txt
