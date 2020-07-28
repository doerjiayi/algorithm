
##[二叉树的最大深度](https://leetcode-cn.com/problems/maximum-depth-of-binary-tree)    

给定一个二叉树，找出其最大深度。
二叉树的深度为根节点到最远叶子节点的最长路径上的节点数。
说明: 叶子节点是指没有子节点的节点。

示例：
给定二叉树 [3,9,20,null,null,15,7]，
    3
   / \
  9  20
    /  \
   15   7

返回它的最大深度 3 。

class Solution {
public:
    int maxDepth(TreeNode* root) {
        if(!root)return 0;
        return max(maxDepth(root->left) + 1,maxDepth(root->right) + 1); 
    }
};


##[两数相加](https://leetcode-cn.com/problems/add-two-numbers)    
给出两个 非空 的链表用来表示两个非负的整数。其中，它们各自的位数是按照 逆序 的方式存储的，并且它们的每个节点只能存储 一位 数字。

如果，我们将这两个数相加起来，则会返回一个新的链表来表示它们的和。

您可以假设除了数字 0 之外，这两个数都不会以 0 开头。

示例：

输入：(2 -> 4 -> 3) + (5 -> 6 -> 4)
输出：7 -> 0 -> 8
原因：342 + 465 = 807

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


##[4. 寻找两个正序数组的中位数](https://leetcode-cn.com/problems/median-of-two-sorted-arrays/)

给定两个大小为 m 和 n 的正序（从小到大）数组 nums1 和 nums2。

请你找出这两个正序数组的中位数，并且要求算法的时间复杂度为 O(log(m + n))。

你可以假设 nums1 和 nums2 不会同时为空。

示例 1:

nums1 = [1, 3]
nums2 = [2]

则中位数是 2.0


示例 2:

nums1 = [1, 2]
nums2 = [3, 4]

则中位数是 (2 + 3)/2 = 2.5

class Solution {
public:
    double findMedianSortedArrays(vector<int>& nums1, vector<int>& nums2) 
    {
        if (nums1.size() == 0 && nums2.size() == 0)return 0.0;
        vector<int> total;
        int i = 0,j = 0;
        while (i < nums1.size() && j < nums2.size())//遍历了一边nums1和nums2，复杂度为m + n
        {
            if (nums1[i] < nums2[j])total.emplace_back(nums1[i++]);
            else total.emplace_back(nums2[j++]);
        }
        for(;j < nums2.size();++j)total.emplace_back(nums2[j]);
        for(;i < nums1.size();++i)total.emplace_back(nums1[i]);
        if (total.size() % 2 == 1)//奇数个
        {
            return total[total.size() / 2];
        }
        //偶数个
        return (double)((double) total[total.size() / 2] + (double)total[(total.size() / 2) -1])/2;
    }
};


##[最长回文子串](https://leetcode-cn.com/problems/longest-palindromic-substring)    
给定一个字符串 s，找到 s 中最长的回文子串。你可以假设 s 的最大长度为 1000。
示例 1：
输入: "babad"
输出: "bab"
注意: "aba" 也是一个有效答案。

示例 2：
输入: "cbbd"
输出: "bb"

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

复杂度 O(N * N)

拓展一下：
https://leetcode-cn.com/problems/longest-palindromic-substring/solution/zhong-xin-kuo-san-dong-tai-gui-hua-by-liweiwei1419/

dp的做法，但是性能不很好
class Solution {
public:
    string longestPalindrome(string s) {        //动态规划
        int n = s.size();
	    if (n < 2)  return s;         //字符串只有一个字符，直接返回
	    int maxL = 1, begin = 0, l, r,i;//maxL记录LPS的长度，begin记录LPS在字符串中的起始位置,l,r分别为左右指针
	    vector<vector<bool>> dp(n,vector<bool>(n,true));    //构造bool型二维向量,dp表
	    for (r = 1; r < n; r++) {       
		    for (l = 0; l < r; l++)
		    {
			    if (s[l] != s[r])//子串首尾字符不相同，直接为false
			    	dp[l][r] = false;
			    else if (r - l < 3)//字串首尾相同，若长度小于3，直接为true，否则利用状态转移方程
				    dp[l][r] = true;
			    else
			    	dp[l][r] =  dp[l + 1][r - 1];
			    if (dp[l][r] && (r - l + 1 > maxL)) {//如果是回文字符串且长度大于已知回文子串长度，那么进行更新
			    	maxL = r - l + 1;
			    	begin = l;
			    }              
		    }
	    }
	    return s.substr(begin, maxL);
    }
};

第一种做法
通过  56 ms   6.8 MB  Cpp
第二种做法
通过  1200 ms 16.8 MB  Cpp

##[整数反转](https://leetcode-cn.com/problems/reverse-integer)    
给出一个 32 位的有符号整数，你需要将这个整数中每位上的数字进行反转。
示例 1:
输入: 123
输出: 321

 示例 2:
输入: -123
输出: -321

示例 3:
输入: 120
输出: 21

注意:
假设我们的环境只能存储得下 32 位的有符号整数，则其数值范围为 [−231,  231 − 1]。请根据这个假设，如果反转后整数溢出那么就返回 0。

class Solution {
public:
    int reverse(int x) {
        int res = 0;
        while (x != 0) {
            if (abs(res) > INT_MAX / 10) return 0;
            res = res * 10 + x % 10;
            x /= 10;
        }
        return res;
    }
};


##[字符串转换整数 (atoi)](https://leetcode-cn.com/problems/string-to-integer-atoi)    
请你来实现一个 atoi 函数，使其能将字符串转换成整数。
首先，该函数会根据需要丢弃无用的开头空格字符，直到寻找到第一个非空格的字符为止。接下来的转化规则如下：
	如果第一个非空字符为正或者负号时，则将该符号与之后面尽可能多的连续数字字符组合起来，形成一个有符号整数。
	假如第一个非空字符是数字，则直接将其与之后连续的数字字符组合起来，形成一个整数。
	该字符串在有效的整数部分之后也可能会存在多余的字符，那么这些字符可以被忽略，它们对函数不应该造成影响。

注意：假如该字符串中的第一个非空格字符不是一个有效整数字符、字符串为空或字符串仅包含空白字符时，则你的函数不需要进行转换，即无法进行有效转换。
在任何情况下，若函数不能进行有效的转换时，请返回 0 。
提示：
	本题中的空白字符只包括空格字符 ' ' 。
	假设我们的环境只能存储 32 位大小的有符号整数，那么其数值范围为 [−231,  231 − 1]。如果数值超过这个范围，请返回  INT_MAX (231 − 1) 或 INT_MIN (−231) 。

示例 1:
输入: "42"
输出: 42


示例 2:
输入: "   -42"
输出: -42
解释: 第一个非空白字符为 '-', 它是一个负号。
     我们尽可能将负号与后面所有连续出现的数字组合起来，最后得到 -42 。


示例 3:
输入: "4193 with words"
输出: 4193
解释: 转换截止于数字 '3' ，因为它的下一个字符不为数字。


示例 4:
输入: "words and 987"
输出: 0
解释: 第一个非空字符是 'w', 但它不是数字或正、负号。
     因此无法执行有效的转换。

示例 5:

输入: "-91283472332"
输出: -2147483648
解释: 数字 "-91283472332" 超过 32 位有符号整数范围。 
     因此返回 INT_MIN (−231) 。

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

##[回文数](https://leetcode-cn.com/problems/palindrome-number)    
判断一个整数是否是回文数。回文数是指正序（从左向右）和倒序（从右向左）读都是一样的整数。

示例 1:
输入: 121
输出: true


示例 2:
输入: -121
输出: false
解释: 从左向右读, 为 -121 。 从右向左读, 为 121- 。因此它不是一个回文数。


示例 3:
输入: 10
输出: false
解释: 从右向左读, 为 01 。因此它不是一个回文数。


进阶:
你能不将整数转为字符串来解决这个问题吗？

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

##[盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water)
给你 n 个非负整数 a1，a2，...，an，每个数代表坐标中的一个点 (i, ai) 。在坐标内画 n 条垂直线，垂直线 i 的两个端点分别为 (i, ai) 和 (i, 0)。找出其中的两条线，使得它们与 x 轴共同构成的容器可以容纳最多的水。

说明：你不能倾斜容器，且 n 的值至少为 2。

矮边是乘积的成员，而宽度是一直减少的，贪心移动矮边

class Solution {
public:
    int maxArea(vector<int>& height) {
        int left = 0;int right = height.size()-1;
        int nMax = 0;
        while(left<right){
            nMax = max(nMax, min(height[left], height[right])*(right-left));
            if(height[left] < height[right])left++;
            else right--;
        }
        return nMax;
    }
};


##[最长公共前缀](https://leetcode-cn.com/problems/longest-common-prefix)    
编写一个函数来查找字符串数组中的最长公共前缀。

如果不存在公共前缀，返回空字符串 ""。

示例 1:
输入: ["flower","flow","flight"]
输出: "fl"


示例 2:
输入: ["dog","racecar","car"]
输出: ""
解释: 输入不存在公共前缀。


说明:
所有输入只包含小写字母 a-z 。

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


##[三数之和](https://leetcode-cn.com/problems/3sum)    

给你一个包含 n 个整数的数组 nums，判断 nums 中是否存在三个元素 a，b，c ，使得 a + b + c = 0 ？请你找出所有满足条件且不重复的三元组。
注意：答案中不可以包含重复的三元组。

示例：
给定数组 nums = [-1, 0, 1, 2, -1, -4]，
满足要求的三元组集合为：
[
  [-1, 0, 1],
  [-1, -1, 2]
]

class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
        if (nums.size() < 3) return {};
        sort(nums.begin(), nums.end());
        unordered_map<int,int> map;
        for (int i = 0; i < nums.size(); i++) {
            map[nums[i]] = i;
        }
        vector<vector<int> > ret;
        for (int i = 0; i < nums.size() - 2 && nums[i] <= 0; i++) {//第一个数
            if (i > 0 && nums[i] == nums[i-1]) continue;//去重
            for (int j = i + 1; j < nums.size() - 1; j++) {//第二个数
                if (j > i + 1 && nums[j] == nums[j-1]) continue;//去重
                int third = -(nums[i] + nums[j]);
                auto it = map.find(third);//有第三个数，且在后面
                if (it != map.end() && it->second > j) {
                    vector<int> r = {nums[i],nums[j],third};
                    ret.push_back(r);
                }
            }
        }
        return ret;
    }
};

class Solution {
public:
    vector<vector<int>> threeSum(vector<int>& nums) {
            if (nums.size() < 3) return {};
            sort(nums.begin(), nums.end());
            vector<vector<int> > ret;
            int n = nums.size();
            for (int i = 0; i < n - 2 && nums[i] <= 0; i++) {//第一个数
                if (i > 0 && nums[i] == nums[i-1]) continue;//去重，第一个不跟上次相同
                int L = i + 1;
                int R = n - 1;
                while(L < R)
                {
                    if(nums[i]+nums[L]+nums[R]==0)
                    {
                        ret.push_back({nums[i],nums[L],nums[R]});
                        ++L;
                        while(L<R && nums[L]==nums[L-1])//去重，第二个不跟上次相同
                        {
                        	++L;
                        }
                        --R;
                        while(L<R && nums[R]==nums[R+1])//去重，第三个不跟上次相同
                        {
                        	--R;
                        }
                    }
                    else if(nums[i]+nums[L]+nums[R]>0)
                    {
                        --R;
                    }
                    else
                    {
                        ++L;
                    }
                }
            }
            return ret;
        }
};

第二种方案要优于第一种，减少了构造哈希map和查询哈希map的过程。
第一种 468 ms 21.4 MB Cpp
第二种 156 ms 19.6 MB Cpp

##[最接近的三数之和](https://leetcode-cn.com/problems/3sum-closest)   

给定一个包括 n 个整数的数组 nums 和 一个目标值 target。找出 nums 中的三个整数，使得它们的和与 target 最接近。返回这三个数的和。假定每组输入只存在唯一答案。

示例：
输入：nums = [-1,2,1,-4], target = 1
输出：2
解释：与 target 最接近的和是 2 (-1 + 2 + 1 = 2) 。

提示：
	3 <= nums.length <= 10^3
	-10^3 <= nums[i] <= 10^3
	-10^4 <= target <= 10^4

 
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
            while(left < right)
            {
                int s = nums[i] + nums[left] + nums[right];
                if (s == target)return s;
                int d = abs(target - s);
                if (d < diff)
                {
                    res = s;
                    diff = d;
                }
                if (s < target)
                {
                    ++left;
                    while (left < right && nums[left] == nums[left - 1])++left; 
                }
                else 
                {
                    --right;
                    while (left < right && nums[right] == nums[right + 1])--right; 
                }
            }
        }
        return res;
    }
};

第二种性能要优化一些
第一种 24 ms9.4 MB Cpp
第二种 12 ms10.2 MBCpp

##[有效的括号](https://leetcode-cn.com/problems/valid-parentheses)

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串，判断字符串是否有效。

有效字符串需满足：
	左括号必须用相同类型的右括号闭合。
	左括号必须以正确的顺序闭合。


注意空字符串可被认为是有效字符串。

示例 1:
输入: "()"
输出: true


示例 2:
输入: "()[]{}"
输出: true


示例 3:
输入: "(]"
输出: false


示例 4:
输入: "([)]"
输出: false


示例 5:
输入: "{[]}"
输出: true

利用栈的特性和括号顺序的特性

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

##[合并两个有序链表](https://leetcode-cn.com/problems/merge-two-sorted-lists)    
将两个升序链表合并为一个新的 升序 链表并返回。新链表是通过拼接给定的两个链表的所有节点组成的。 
示例：

输入：1->2->4, 1->3->4
输出：1->1->2->3->4->4

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
 合并 k 个排序链表，返回合并后的排序链表。请分析和描述算法的复杂度。

示例:
输入:
[
  1->4->5,
  1->3->4,
  2->6
]
输出: 1->1->2->3->4->4->5->6

   考虑分治的思想来解这个题（类似归并排序的思路）。把这些链表分成两半，如果每一半都合并好了，那么我就最后把这两个合并了就行了。这就是分治法的核心思想。
但是这道题由于存的都是指针，就具有了更大的操作灵活性，可以不用递归来实现分治。就是先两两合并后在两两合并。。。一直下去直到最后成了一个。（相当于分治算法的那棵二叉树从底向上走了）。
第一次两两合并是进行了k/2次，每次处理2n个值,即2n * k/2 = kn 次比较。
第二次两两合并是进行了k/4次，每次处理4n个值,即4n * k/4 = kn 次比较。
。。。
最后一次两两合并是进行了k/(2^logk)次（=1次），每次处理2^logK  * N个值（kn个），即1*kn= kn 次比较。
所以时间复杂度：
O(KN* logK)
空间复杂度是O(1)。
class Solution {
public:
    shared_ptr<ListNode> dummy = make_shared<ListNode>();
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
        ListNode *tmp = dummy.get();
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
};

##[删除排序数组中的重复项](https://leetcode-cn.com/problems/remove-duplicates-from-sorted-array)    
class Solution {
public:
    int removeDuplicates(vector<int>& nums) 
    {//直接在nums上赋值
        int len = nums.size();
        if (len <= 1)return len;
        int count = 1;
        for (int i = 1; i < len; i++) //从第二个数开始
        {
            if (nums[i] != nums[count-1]) 
            {
                nums[count++] = nums[i];
            }
        }
        return count;
    }
};

##[搜索旋转排序数组](https://leetcode-cn.com/problems/search-in-rotated-sorted-array)    
假设按照升序排序的数组在预先未知的某个点上进行了旋转。
( 例如，数组 [0,1,2,4,5,6,7] 可能变为 [4,5,6,7,0,1,2] )。

搜索一个给定的目标值，如果数组中存在这个目标值，则返回它的索引，否则返回 -1 。
你可以假设数组中不存在重复的元素。
你的算法时间复杂度必须是 O(log n) 级别。

示例 1:
输入: nums = [4,5,6,7,0,1,2], target = 0
输出: 4
示例 2:

输入: nums = [4,5,6,7,0,1,2], target = 3
输出: -1

class Solution {
public:
    int search(vector<int>& nums, int target) {
        int left = 0,right = nums.size() -1,mid;
        while(left <= right)
        {
            mid = left + (right-left)/2;
            if (nums[mid] == target)return mid;
            if (nums[mid] < nums[right])
            {
                if (nums[mid] < target && target <= nums[right])
                {
                    left = mid + 1;
                }
                else right = mid -1;
            }
            else //if (nums[left] < nums[mid])
            {
                if (nums[left] <= target && target < nums[mid])
                {
                    right = mid - 1;
                }
                else left = mid + 1;
            }
        }
        return -1;
    }
};

##[字符串相乘](https://leetcode-cn.com/problems/multiply-strings)    
给定两个以字符串形式表示的非负整数 num1 和 num2，返回 num1 和 num2 的乘积，它们的乘积也表示为字符串形式。

示例 1:
输入: num1 = "2", num2 = "3"
输出: "6"

示例 2:
输入: num1 = "123", num2 = "456"
输出: "56088"

说明：
	num1 和 num2 的长度小于110。
	num1 和 num2 只包含数字 0-9。
	num1 和 num2 均不以零开头，除非是数字 0 本身。
	不能使用任何标准库的大数类型（比如 BigInteger）或直接将输入转换为整数来处理。

class Solution {
public:
    string multiply(string num1, string num2) 
    {
        string res;
        vector<int> mul(num1.size() + num2.size(),0);// 从低位到高位
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
            res.push_back('0' + mul[i]);// 从高位到低位
        }
        return res;
    }
};


##[全排列](https://leetcode-cn.com/problems/permutations)  

给定一个 没有重复 数字的序列，返回其所有可能的全排列。

示例:

输入: [1,2,3]
输出:
[
  [1,2,3],
  [1,3,2],
  [2,1,3],
  [2,3,1],
  [3,1,2],
  [3,2,1]
]

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

##[最大子序和](https://leetcode-cn.com/problems/maximum-subarray)  

给定一个整数数组 nums ，找到一个具有最大和的连续子数组（子数组最少包含一个元素），返回其最大和。

示例:
输入: [-2,1,-3,4,-1,2,1,-5,4],
输出: 6
解释: 连续子数组 [4,-1,2,1] 的和最大，为 6。


进阶:
如果你已经实现复杂度为 O(n) 的解法，尝试使用更为精妙的分治法求解。

贪心算法

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
            if (sum < 0)sum = 0;//小于0的就不需要了，可以去掉
        }
        return res;
    }
};
通过 4 ms 7.1 MB Cpp

dp算法
class Solution
{
public:
    int maxSubArray(vector<int> &nums)
    {
        int numsSize = int(nums.size());
        if (numsSize == 0)return 0;
        //因为只需要知道dp的前一项，可以用int代替一维数组
        int dp(nums[0]),result = dp;
        for (int i = 1; i < numsSize; i++)
        {
            dp = max(dp + nums[i], nums[i]);
            result = max(result, dp);
        }
        return result;
    }
};

通过 4 ms 6.9 MB Cpp

贪心算法看上去更简洁一点

##[螺旋矩阵](https://leetcode-cn.com/problems/spiral-matrix)    

给定一个包含 m x n 个元素的矩阵（m 行, n 列），请按照顺时针螺旋顺序，返回矩阵中的所有元素。

示例 1:

输入:
[
 [ 1, 2, 3 ],
 [ 4, 5, 6 ],
 [ 7, 8, 9 ]
]
输出: [1,2,3,6,9,8,7,4,5]


示例 2:

输入:
[
  [1, 2, 3, 4],
  [5, 6, 7, 8],
  [9,10,11,12]
]
输出: [1,2,3,4,8,12,11,10,9,5,6,7]

考察理解行和列的边界条件

class Solution {
public:
    vector<int> spiralOrder(vector<vector<int>>& matrix) {
       if (matrix.size() == 0 || matrix[0].size() == 0)return {};
        vector<int> ret;
        int row(matrix.size() - 1),col(matrix[0].size() - 1),x(0),y(0),i(0);
        for (; x <= row && y <= col; x++, y++,row--,col--)
        {
            for(i=y ; i<=col ; ++i)//首行
            {
                ret.push_back(matrix[x][i]);
            }
            for (i = x + 1; i <= row; ++i)//最右列(上到下，需要跳过第一个)
            {
                ret.push_back(matrix[i][col]);
            }
            for (i = col - 1; i >= y && x != row; --i)//最底行(右到左，需要跳过第一个，判断重复行)
            {
                ret.push_back(matrix[row][i]);
            }
            for (i = row - 1; i > x && y != col; --i)//最左列(下到上，需要跳过第一个，判断重复列)
            {
                ret.push_back(matrix[i][y]);
            }
        }
        return ret;
    }
};

##[螺旋矩阵 II](https://leetcode-cn.com/problems/spiral-matrix-ii)    
 
给定一个正整数 n，生成一个包含 1 到 n2 所有元素，且元素按顺时针顺序螺旋排列的正方形矩阵。

示例:

输入: 3
输出:
[
 [ 1, 2, 3 ],
 [ 8, 9, 4 ],
 [ 7, 6, 5 ]
]

 
class Solution {
public:
    vector<vector<int>> generateMatrix(int n) 
    {
        if (n <= 0)return {};
        vector<vector<int>> res(n,vector<int>(n,0));
        int row(n-1),col(n-1),x(0),y(0),cnt(0),i(0);
        for(;x <= row && y <= col;++x,++y,--row,--col)
        {
            for(i = y;i <= col;++i)
            {
                res[x][i] = ++cnt;//printf("1)%d %d %d\n",x,i,cnt);
            }
            for(i = x+1;i <= row;++i)
            {
                res[i][col] = ++cnt;//printf("2)%d %d %d\n",i,col,cnt);
            }
            for(i = col-1;i >= y && x != row;--i)
            {
                res[row][i] = ++cnt;//printf("3)%d %d %d\n",row,i,cnt);
            }
            for(i = row-1;i >= x + 1&& y != col;--i)
            {
                res[i][y] = ++cnt;//printf("4)%d %d %d\n",i,y,cnt);
            }
        }
        return res;
    }
};

##[旋转链表](https://leetcode-cn.com/problems/rotate-list)    

给定一个链表，旋转链表，将链表每个节点向右移动 k 个位置，其中 k 是非负数。

示例 1:

输入: 1->2->3->4->5->NULL, k = 2
输出: 4->5->1->2->3->NULL
解释:
向右旋转 1 步: 5->1->2->3->4->NULL
向右旋转 2 步: 4->5->1->2->3->NULL


示例 2:

输入: 0->1->2->NULL, k = 4
输出: 2->0->1->NULL
解释:
向右旋转 1 步: 2->0->1->NULL
向右旋转 2 步: 1->2->0->NULL
向右旋转 3 步: 0->1->2->NULL
向右旋转 4 步: 2->0->1->NULL


连成一圈再断开，注意计算断开位置
class Solution {
public:
    ListNode* rotateRight(ListNode* head, int k) {
        if (!head || !head->next) return head;
		// 获取长度和末尾节点tail
		int len = 1;
		ListNode* tail = head;
		while (tail->next) {tail = tail->next;len++;}
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
##[不同路径](https://leetcode-cn.com/problems/unique-paths)    
一个机器人位于一个 m x n 网格的左上角 （起始点在下图中标记为“Start” ）。
机器人每次只能向下或者向右移动一步。机器人试图达到网格的右下角（在下图中标记为“Finish”）。

问总共有多少条不同的路径？
例如，上图是一个7 x 3 的网格。有多少可能的路径？

示例 1:

输入: m = 3, n = 2
输出: 3
解释:
从左上角开始，总共有 3 条路径可以到达右下角。
1. 向右 -> 向右 -> 向下
2. 向右 -> 向下 -> 向右
3. 向下 -> 向右 -> 向右

示例 2:
输入: m = 7, n = 3
输出: 28

提示：
	1 <= m, n <= 100
	题目数据保证答案小于等于 2 * 10 ^ 9

class Solution {
public:
    int uniquePaths(int m, int n) {
        vector<vector<int>> dp(m,vector<int>(n,1));  
        for(int i=1; i<m;i++)//直接从第二行、第二列开始计算
        {  
            for(int j=1;j<n;j++)
            {  
                dp[i][j]=dp[i-1][j]+dp[i][j-1];  //dp,每个格子来源于左边和上边的格子之和
            }  
        }  
        return dp[m-1][n-1];  
    }
};

##[爬楼梯](https://leetcode-cn.com/problems/climbing-stairs)    

假设你正在爬楼梯。需要 n 阶你才能到达楼顶。

每次你可以爬 1 或 2 个台阶。你有多少种不同的方法可以爬到楼顶呢？

注意：给定 n 是一个正整数。

示例 1：

输入： 2
输出： 2
解释： 有两种方法可以爬到楼顶。
1.  1 阶 + 1 阶
2.  2 阶

示例 2：

输入： 3
输出： 3
解释： 有三种方法可以爬到楼顶。
1.  1 阶 + 1 阶 + 1 阶
2.  1 阶 + 2 阶
3.  2 阶 + 1 阶

class Solution {
public:
    int climbStairs(int n) {
        if (n <= 2)return n;
        int vec[3] = {1,2,0};//初始化值（n 为1和2 ）
        for(int i=3; i<=n; i++)//计算 n -2 次
        {//递推公式为： vec[n] = vec[n-1] + vec[n-2]
            vec[2] = vec[0]+vec[1];
            vec[0] = vec[1];//移动变量
            vec[1] = vec[2];
        }
        return vec[2];
    }
};

##[子集](https://leetcode-cn.com/problems/subsets)   

给定一组不含重复元素的整数数组 nums，返回该数组所有可能的子集（幂集）。

说明：解集不能包含重复的子集。

示例:

输入: nums = [1,2,3]
输出:
[
  [3],
  [1],
  [2],
  [1,2,3],
  [1,3],
  [2,3],
  [1,2],
  []
]

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
##[合并两个有序数组](https://leetcode-cn.com/problems/merge-sorted-array)    
给你两个有序整数数组 nums1 和 nums2，请你将 nums2 合并到 nums1 中，使 nums1 成为一个有序数组。

说明:
	初始化 nums1 和 nums2 的元素数量分别为 m 和 n 。
	你可以假设 nums1 有足够的空间（空间大小大于或等于 m + n）来保存 nums2 中的元素。

示例:
输入:
nums1 = [1,2,3,0,0,0], m = 3
nums2 = [2,5,6],       n = 3

输出: [1,2,2,3,5,6]
 
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
        while(j >= 0)nums1[k--] = nums2[j--];
    }
};
##[格雷编码](https://leetcode-cn.com/problems/gray-code)

格雷编码是一个二进制数字系统，在该系统中，两个连续的数值仅有一个位数的差异。

给定一个代表编码总位数的非负整数 n，打印其格雷编码序列。即使有多个不同答案，你也只需要返回其中一种。

格雷编码序列必须以 0 开头。

 

示例 1:

输入: 2
输出: [0,1,3,2]
解释:
00 - 0
01 - 1
11 - 3
10 - 2

对于给定的 n，其格雷编码序列并不唯一。
例如，[0,2,3,1] 也是一个有效的格雷编码序列。

00 - 0
10 - 2
11 - 3
01 - 1

示例 2:

输入: 0
输出: [0]
解释: 我们定义格雷编码序列必须以 0 开头。
     给定编码总位数为 n 的格雷编码序列，其长度为 2n。当 n = 0 时，长度为 20 = 1。
     因此，当 n = 0 时，其格雷编码序列为 [0]。

初始为一个0，在之前数字的基础上，从后往前遍历，在高位加1位处理，则每次跟之前只会相差一位，例如
00 - 0
01 - 1
11 - 3
10 - 2

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
##[买卖股票的最佳时机](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock)

给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。
如果你最多只允许完成一笔交易（即买入和卖出一支股票一次），设计一个算法来计算你所能获取的最大利润。
注意：你不能在买入股票前卖出股票。

示例 1:
输入: [7,1,5,3,6,4]
输出: 5
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 5 天（股票价格 = 6）的时候卖出，最大利润 = 6-1 = 5 。
     注意利润不能是 7-1 = 6, 因为卖出价格需要大于买入价格；同时，你不能在买入前卖出股票。

示例 2:
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
 
class Solution {
public:
    int maxProfit(vector<int>& prices) {
        int res(0);
        int mi(INT_MAX);
        for(int i = 0;i < prices.size();++i)
        {
            mi = min(prices[i],mi);//之前的最低价格
            res = max(res,prices[i]-mi);//当前价格减去之前的最低价格
        }
        return res;
    }
};
##[买卖股票的最佳时机 II](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-ii)    

给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。
设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。
注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

示例 1:

输入: [7,1,5,3,6,4]
输出: 7
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。
示例 2:

输入: [1,2,3,4,5]
输出: 4
解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。
     因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。
示例 3:

输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。
 
提示：

1 <= prices.length <= 3 * 10 ^ 4
0 <= prices[i] <= 10 ^ 4


给定一个数组，它的第 i 个元素是一支给定股票第 i 天的价格。
设计一个算法来计算你所能获取的最大利润。你可以尽可能地完成更多的交易（多次买卖一支股票）。
注意：你不能同时参与多笔交易（你必须在再次购买前出售掉之前的股票）。

示例 1:
输入: [7,1,5,3,6,4]
输出: 7
解释: 在第 2 天（股票价格 = 1）的时候买入，在第 3 天（股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     随后，在第 4 天（股票价格 = 3）的时候买入，在第 5 天（股票价格 = 6）的时候卖出, 这笔交易所能获得利润 = 6-3 = 3 。

示例 2:
输入: [1,2,3,4,5]
输出: 4
解释: 在第 1 天（股票价格 = 1）的时候买入，在第 5 天 （股票价格 = 5）的时候卖出, 这笔交易所能获得利润 = 5-1 = 4 。
     注意你不能在第 1 天和第 2 天接连购买股票，之后再将它们卖出。
     因为这样属于同时参与了多笔交易，你必须在再次购买前出售掉之前的股票。

示例 3:
输入: [7,6,4,3,1]
输出: 0
解释: 在这种情况下, 没有交易完成, 所以最大利润为 0。

提示：
	1 <= prices.length <= 3 * 10 ^ 4
	0 <= prices[i] <= 10 ^ 4

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
##[二叉树中的最大路径和](https://leetcode-cn.com/problems/binary-tree-maximum-path-sum)    
给定一个非空二叉树，返回其最大路径和。
本题中，路径被定义为一条从树中任意节点出发，达到任意节点的序列。该路径至少包含一个节点，且不一定经过根节点。

示例 1:
输入: [1,2,3]
       1
      / \
     2   3

输出: 6

示例 2:
输入: [-10,9,20,null,null,15,7]

   -10
   / \
  9  20
    /  \
   15   7

输出: 42
 
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
        int leftPath = maxTree(root->left,ma);//左边、右边路径大小最大路径和
        int rightPath = maxTree(root->right,ma);
        int v = max(root->val,root->val + max(leftPath,0)+ max(rightPath,0));//本节点树的最大路径和
        ma = max(ma,v);
        return max(0,max(root->val + leftPath,root->val + rightPath));//返回本节点开始的边路径最大值（或者可以不使用本节点）
    }
};
##[只出现一次的数字](https://leetcode-cn.com/problems/single-number)
给定一个非空整数数组，除了某个元素只出现一次以外，其余每个元素均出现两次。找出那个只出现了一次的元素。

说明：
你的算法应该具有线性时间复杂度。 你可以不使用额外空间来实现吗？

示例 1:
输入: [2,2,1]
输出: 1

示例 2:
输入: [4,1,2,1,2]
输出: 4

异或两相同数会抵消，剩下的就是一次的那个数了

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
##[环形链表](https://leetcode-cn.com/problems/linked-list-cycle)
给定一个链表，判断链表中是否有环。
为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。

示例 1：
输入：head = [3,2,0,-4], pos = 1
输出：true
解释：链表中有一个环，其尾部连接到第二个节点。

使用快慢指针的方式遍历判断

class Solution {
public:
    bool hasCycle(ListNode *head)
    {
        ListNode *slow = head,*fast = head;
        while(fast && fast->next)//每次需要判断后面两个节点
        {
            if (slow == fast->next || slow == fast->next->next)return true;
            slow = slow->next,fast = fast->next->next;
        }
        return false;
    }
};
##[环形链表 II](https://leetcode-cn.com/problems/linked-list-cycle-ii)
 
给定一个链表，返回链表开始入环的第一个节点。 如果链表无环，则返回 null。
为了表示给定链表中的环，我们使用整数 pos 来表示链表尾连接到链表中的位置（索引从 0 开始）。 如果 pos 是 -1，则在该链表中没有环。
说明：不允许修改给定的链表。

使用快慢指针找到重叠，重置快指针，再找到重叠点就是入环点

class Solution {
public:
    ListNode *detectCycle(ListNode *head) {
        if( !head || !head->next )return NULL;  
        ListNode* fp = head,* sp = head;  
        while( fp  && fp->next )
        {  
            sp = sp->next;  
            fp = fp->next->next;  
            if( fp == sp )//使用快慢指针找到重叠
            {   
                break;  
            }  
        }  
        if( !fp || !fp->next )return NULL;  
        fp = head;//重置快指针  
        while( fp != sp )//再找到重叠点就是入环点
        {  
            sp = sp->next;  
            fp = fp->next;  
        }  
        return sp;  
    }
};
##[LRU缓存机制](https://leetcode-cn.com/problems/lru-cache)

运用你所掌握的数据结构，设计和实现一个  LRU (最近最少使用) 缓存机制。它应该支持以下操作： 获取数据 get 和 写入数据 put 。

获取数据 get(key) - 如果关键字 (key) 存在于缓存中，则获取关键字的值（总是正数），否则返回 -1。
写入数据 put(key, value) - 如果关键字已经存在，则变更其数据值；如果关键字不存在，则插入该组「关键字/值」。当缓存容量达到上限时，它应该在写入新数据之前删除最久未使用的数据值，从而为新的数据值留出空间。

进阶:
你是否可以在 O(1) 时间复杂度内完成这两种操作？

示例:
LRUCache cache = new LRUCache( 2 /* 缓存容量 */ );

cache.put(1, 1);
cache.put(2, 2);
cache.get(1);       // 返回  1
cache.put(3, 3);    // 该操作会使得关键字 2 作废
cache.get(2);       // 返回 -1 (未找到)
cache.put(4, 4);    // 该操作会使得关键字 1 作废
cache.get(1);       // 返回 -1 (未找到)
cache.get(3);       // 返回  3
cache.get(4);       // 返回  4


 （缓存）利用哈希表的快速访问，记录的是链表迭代器，因为迭代器需要被移动；链表的访问顺序判断访问热度
 
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
            l.splice(l.begin(),l,it->second);//找到的迭代器放到列表前面去
            return it->second->second;//返回值（迭代器的内容不会变，只是他在列表的位置变了）
        }
        return -1;
    }  
    void put(int key, int value)
    {  
        auto it = m.find(key);
        if (it != m.end())
        {
            l.splice(l.begin(),l,it->second);//找到的迭代器放到列表前面去
            it->second->second = value;//修改迭代器的值
        }
        else
        {
            if (l.size() + 1 > s)
            {
                m.erase(l.back().first);// 超过容量的需要移除列表最后面的迭代器，和哈希表对应的键
                l.pop_back();
            }
            l.push_front(make_pair(key,value));//插入列表的前面
            m[key] = l.begin();//更新哈希表的键对应的列表迭代器
        }
    }  
private:  
    unordered_map<int,list<pair<int,int>>::iterator> m;//key value (iter)
    list<pair<int,int>> l;//key value
    int s;
};

本题的拓展思维：
有的需求是需要增加时间检查的，哈希表的值就是时间，键就是对象的id，加入时检查容量，再定时检查对象的时间，需要回收的移动到回收的集合对象里
方案如下：
typedef uint32 unsigned int;
template<typename ELEM,typename ELEM_LIST = std::list<std::pair<ELEM,uint32>>>
struct OnlineLRUCache {
	explicit OnlineLRUCache(uint32 capacity = 10000,uint32 expired = 60 * 10):m_capacity(capacity),m_expired(expired){}
	void SetConfig(uint32 capacity,uint32 expired)
	{
		m_capacity = capacity;
		m_expired = expired;
	}
	uint32 Get(const ELEM& dataid)//key(dataid),value(time)
    {
		m_setExpiredData.erase(dataid);
        auto it = m_mapData.find(dataid);
        if (it != m_mapData.end())
        {
        	m_listData.splice(m_listData.begin(),m_listData,it->second);
            return it->second->second;
        }
        return 0;
    }
    void Put(const ELEM& dataid, uint32 value = ::time(nullptr))//key (data id),value time
    {
    	m_setExpiredData.erase(dataid);
        auto it = m_mapData.find(dataid);
        if (it != m_mapData.end())
        {
        	m_listData.splice(m_listData.begin(),m_listData,it->second);
            it->second->second = value;
        }
        else
        {
            if (m_listData.size() + 1 > m_capacity)
            {
            	m_mapData.erase(m_listData.back().first);
            	m_setExpiredData.insert(m_listData.back().first);
            	m_listData.pop_back();
            }
            m_listData.push_front(std::make_pair(dataid,value));
            m_mapData[dataid] = m_listData.begin();
        }
    }
    std::unordered_set<ELEM> m_setExpiredData;
    void CheckOvertime(uint32 value = ::time(nullptr))//检查超时的对象
	{
		while (m_listData.size() > 0)
		{
			if (m_listData.back().second + m_expired < value)
			{
				m_mapData.erase(m_listData.back().first);
				m_setExpiredData.insert(m_listData.back().first);
				m_listData.pop_back();
			}
			else
			{
				break;
			}
		}
	}
private:
    ELEM_LIST m_listData;//key (dataid),value(time,单位秒)
    std::unordered_map<ELEM,typename ELEM_LIST::iterator> m_mapData;//key value (iter)
    uint32 m_capacity = 10000;
    uint32 m_expired = 60 * 10;//超时时间，单位秒
};

##[排序链表](https://leetcode-cn.com/problems/sort-list)   
 
在 O(n log n) 时间复杂度和常数级空间复杂度下，对链表进行排序。

示例 1:
输入: 4->2->1->3
输出: 1->2->3->4


示例 2:
输入: -1->5->3->4->0
输出: -1->0->3->4->5

冒泡排序方案，复杂度 O(N*N),冒泡的时间复杂度是不能满足题目要求的，只是比较简单

通过 1152 ms 15.3 MB

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

二路归并排序，从底到顶来合并
通过 68 ms  15.5 MB Cpp
从运行结果来看，归并排序要好很多

class Solution {
public:
    ListNode* sortList(ListNode* head) {
        if (!head) return head;
        ListNode dummyHead(0);
        dummyHead.next = head;
        auto p = head;
        int length = 0;
        while (p) {++length;p = p->next;}
        for (int size = 1; size < length; size <<= 1) {//size是每次需要合并的分段的长度1、2、4、、
            auto cur = dummyHead.next;
            auto tail = &dummyHead;
            while (cur) {
                auto left = cur;
                auto right = cut(left, size); // left->@->@ right->@->@->@...
                if (right)
                {
                    cur = cut(right, size); // left->@->@ right->@->@  cur->@->...  
                    tail->next = merge(left, right);//合并左边与右边，追加到前面
                    while (tail->next) tail = tail->next;
                }
                else
                {
                    tail->next = left;
                    break;
                }
            }
        }
        return dummyHead.next;
    }
    ListNode* cut(ListNode* head, int n) {//按长度分割
        if (!head) return nullptr;
        while (--n && head->next) head = head->next;//最后那个节点
        auto next = head->next;
        head->next = nullptr;
        return next;
    }
    ListNode* merge(ListNode* l1, ListNode* l2) {//合并两个排好序的l链表，复杂度是长度
        ListNode dummyHead(0);
        auto p = &dummyHead;
        while (l1 && l2) {
            if (l1->val < l2->val) {
                p->next = l1;
                p = l1;
                l1 = l1->next;       
            } else {
                p->next = l2;
                p = l2;
                l2 = l2->next;
            }
        }
        p->next = l1 ? l1 : l2;
        return dummyHead.next;
    }
};

##[最小栈](https://leetcode-cn.com/problems/min-stack)    
 
设计一个支持 push ，pop ，top 操作，并能在常数时间内检索到最小元素的栈。
	push(x) —— 将元素 x 推入栈中。
	pop() —— 删除栈顶的元素。
	top() —— 获取栈顶元素。
	getMin() —— 检索栈中的最小元素。

示例:

输入：
["MinStack","push","push","push","getMin","pop","top","getMin"]
[[],[-2],[0],[-3],[],[],[],[]]

输出：
[null,null,null,null,-3,null,0,-2]

解释：
MinStack minStack = new MinStack();
minStack.push(-2);
minStack.push(0);
minStack.push(-3);
minStack.getMin();   --> 返回 -3.
minStack.pop();
minStack.top();      --> 返回 0.
minStack.getMin();   --> 返回 -2.

提示：
	pop、top 和 getMin 操作总是在 非空栈 上调用。

需要两个栈，一个正常压入数据，一个是只会压入越来越小的数据。
 
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

##[相交链表](https://leetcode-cn.com/problems/intersection-of-two-linked-lists)    
 
编写一个程序，找到两个单链表相交的起始节点。
注意：
	如果两个链表没有交点，返回 null.
	在返回结果后，两个链表仍须保持原有的结构。
	可假定整个链表结构中没有循环。
	程序尽量满足 O(n) 时间复杂度，且仅用 O(1) 内存。

以下只要遍历一次两个链表，但是需要 O(N)的空间复杂度，不符合题目要求

class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if (!headA || !headB)return nullptr;
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
            if (m[tmp] > 0)return tmp;//相交节点
            tmp = tmp->next;
        }
        return nullptr;
    }
};

通过 136 ms 22.2 MB Cpp
 
以下只要 O(1) 空间复杂度，最多每个列表遍历两次，也算是O(n) 时间复杂度。
class Solution {
public:
    ListNode *getIntersectionNode(ListNode *headA, ListNode *headB) {
        if (!headA || !headB)return nullptr;
        int lenA(0),lenB(0);
        ListNode *curA = headA, *curB = headB;
        for(;curA;curA = curA->next,lenA++);
        for(;curB;curB = curB->next,lenB++);
        curB = headB;
        curA = headA;
        if (lenB > lenA)
        {
            int diff = lenB - lenA;
            //printf("diff %d \n",diff);
            while(diff--)
            {
                curB = curB->next;
            }
        }
        else if (lenA > lenB)
        {
            int diff = lenA - lenB;
            while(diff--)
            {
                curA = curA->next;
            }
        }
        while(curA && curB)
        {
            if (curA == curB)return curA;
            curA = curA->next;
            curB = curB->next;
        }
        return nullptr;
    }
};
 
通过 52 ms 14.5 MB Cpp
从结果上看，第二种时间复杂度和空间复杂度都更好一点，可见哈希表查询、插入也有一定的时间消耗

##[多数元素](https://leetcode-cn.com/problems/majority-element)    
 
给定一个大小为 n 的数组，找到其中的多数元素。多数元素是指在数组中出现次数大于 ⌊ n/2 ⌋ 的元素。
你可以假设数组是非空的，并且给定的数组总是存在多数元素。

示例 1:

输入: [3,2,3]
输出: 3

示例 2:

输入: [2,2,1,1,1,2,2]
输出: 2

因为多数元素是存在的，并且大于n/2，则计数器能选拔出多数元素来。

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
##[反转链表](https://leetcode-cn.com/problems/reverse-linked-list)    
 
反转一个单链表。

示例:

输入: 1->2->3->4->5->NULL
输出: 5->4->3->2->1->NULL

进阶:
你可以迭代或递归地反转链表。你能否用两种方法解决这道题？

迭代的方案
class Solution {
public:
    ListNode* reverseList(ListNode* head) 
    {
        if (!head || !head->next)return head;
        ListNode* pre = nullptr;
        ListNode* cur = head;
        while(cur)
        {
            ListNode* next = cur->next;//先记录下一个节点
            cur->next = pre;//连接
            pre = cur;//移动
            cur = next;
        }
        return pre;
    }
};

通过 12 ms 8.3 MB Cpp
##[数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array) 

在未排序的数组中找到第 k 个最大的元素。请注意，你需要找的是数组排序后的第 k 个最大的元素，而不是第 k 个不同的元素。

示例 1:

输入: [3,2,1,5,6,4] 和 k = 2
输出: 5


示例 2:

输入: [3,2,3,1,2,4,5,5,6] 和 k = 4
输出: 4

说明: 

你可以假设 k 总是有效的，且 1 ≤ k ≤ 数组的长度。

需要使用最小堆，才能找出第几大的。priority_queue默认是最大堆，priority_queue<int>默认是最大堆,等同于 priority_queue<int, vector<int>, less<int> >
定义最小堆,使用priority_queue<int,vector<int>,greater<int>>

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

给定一个整数数组，判断是否存在重复元素。
如果任意一值在数组中出现至少两次，函数返回 true 。如果数组中每个元素都不相同，则返回 false 。

示例 1:

输入: [1,2,3,1]
输出: true

示例 2:

输入: [1,2,3,4]
输出: false

示例 3:

输入: [1,1,1,3,3,4,3,2,4,2]
输出: true

使用哈希表来判断重复

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
##[二叉搜索树中第K小的元素](https://leetcode-cn.com/problems/kth-smallest-element-in-a-bst)
 
给定一个二叉搜索树，编写一个函数 kthSmallest 来查找其中第 k 个最小的元素。

说明：
你可以假设 k 总是有效的，1 ≤ k ≤ 二叉搜索树元素个数。

示例 1:

输入: root = [3,1,4,null,2], k = 1
   3
  / \
 1   4
  \
   2
输出: 1

示例 2:

输入: root = [5,3,6,2,4,null,null,1], k = 3
       5
      / \
     3   6
    / \
   2   4
  /
 1
输出: 3

进阶：
如果二叉搜索树经常被修改（插入/删除操作）并且你需要频繁地查找第 k 小的值，你将如何优化 kthSmallest 函数？

因为是二叉搜索树，使用中序遍历就可以，就是从小到大。使用栈来实现中序遍历。

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
                sk.push(p);//使用栈来实现中序遍历，二叉搜索树可以实现从小到大遍历
                p = p->left;//一直放左边
            }
            p = sk.top();
            sk.pop();
            if (++cnt == k)return p->val;
            p = p->right;
        }
        return 0;
    }
};
##[2的幂](https://leetcode-cn.com/problems/power-of-two)
 

class Solution {
public:
//n & (n-1) 就是为了 去掉最右边的位数 1.剩下的为0则为2 的幂
bool isPowerOfTwo(int n) {
	return (n>0) && !(n & (n-1));
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
##[Nim游戏](https://leetcode-cn.com/problems/nim-game)    
class Solution {
public:
    bool canWinNim(int n) {
        return n % 4 != 0;
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