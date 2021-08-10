<!-- GFM-TOC -->
* [总结] (##总结)
* [Rotate Image](##RotateImage)
* [Subset等一系列问题](##Subset等一系列问题)
* [BitOperator]( ##BitOperator)
* [LinkedListCycle](##LinkedListCycle)
* [LRU](##LRU)
* [RotateArray](##RoateArray)
* [CountPrime](##CountPrime)
* [ReverseLinkedList](##ReverseLinkedList)
* [Palindrome Linked List](##PalindromeLinkedList)
* [Sliding Window Maximum](##SlidingWindowMaximum)
* [Serialize and Deserialize Binary Tree](##SerializeandDeserializeBinaryTree)
* [KMP](##KMP)
* [Longest Palindromic Subsequence](##LongestPalindromicSubsequence)
<!-- GFM-TOC -->
## 总结
如果碰到一些比较傻的[valid number](https://leetcode.com/submissions/detail/362311611/)的问题，试着把一个大问题分解成比较小的问题，然后用小问题把复杂的问题变得简单.  
这题就是把大问题变成用e分割的问题，然后对每一个分割的部分求解。每个部分又可以成为几个正负号，几个dot，有没有其他字符之类的自问题。  


## [RotateImage](https://leetcode.com/problems/rotate-image/)
Roate image 可以先transpose，then reverse

```c++
    void rotate(vector<vector<int>>& matrix) {
        int n = matrix.size();
        for( int i = 0; i < n; i++ )
            for( int j = i; j < n; j++ ){
                int tmp = matrix[j][i];
                matrix[j][i] = matrix[i][j];
                matrix[i][j] = tmp;
            }
        for( auto& m : matrix )
            reverse( m.begin(), m.end() );
                
    }
```
```c++
  public void rotate(int[][] matrix) {
    int n = matrix.length;
    for (int i = 0; i < (n + 1) / 2; i ++) {
      for (int j = 0; j < n / 2; j++) {
        int temp = matrix[n - 1 - j][i];
        matrix[n - 1 - j][i] = matrix[n - 1 - i][n - j - 1];
        matrix[n - 1 - i][n - j - 1] = matrix[j][n - 1 -i];
        matrix[j][n - 1 - i] = matrix[i][j];
        matrix[i][j] = temp;
      }
    }
  }

```

## [Subset等一系列问题](https://leetcode.com/problems/subsets/discuss/27281/A-general-approach-to-backtracking-questions-in-Java-(Subsets-Permutations-Combination-Sum-Palindrome-Partitioning))

这些都是同类型问题，用backtracking
1. [subset](https://leetcode.com/problems/subsets-ii/)， sort array，recursion的for loop，起始为idx，从idx+1开始，如果数字相同，continue。  if( i != idx && nums[i] == nums[i-1] )
```c++
    void help( vector<int>& nums, vector<vector<int>>& res, vector<int>& tmp, int idx )
    {
        res.push_back( tmp );
        for( int i = idx; i < nums.size(); i++ )
        {
            if( i != idx && nums[i] == nums[i-1] )
                continue;
            tmp.push_back( nums[i] );
            help( nums, res, tmp, i + 1 );
            tmp.pop_back();
        }
    }
```
2. [Permutation](https://leetcode.com/submissions/detail/120310729/) sort array，recursion的for loop，起始为0，记录current idx是否used，并且记录本次for loop是否用过这个value，如果用过，就跳过。
```c++
        for( int i = 0; i < nums.size(); i++ )
        {
            if( !used[i] )
            {
                tmp.push_back( nums[i] );
                used[i] = true;
                help( res, nums, used, tmp );
                used[i] = false;
                tmp.pop_back();
                while( i < nums.size() - 1 && nums[i] == nums[i+1] )
                {
                    i++;
                }
            }
        }
```
3. [ConbinationSum](https://leetcode.com/submissions/detail/120189136/), 和subset类似 sort array，recursion的for loop，起始为idx，从idx+1开始，如果数字相同，continue。  if( i != idx && nums[i] == nums[i-1] )
4. [Palindrome Partition](https://leetcode.com/submissions/detail/121567071/), 是find paralindrom 和backtrack的结合。把parlindrom放到一个array里，再backtrack.

## BitOperator
1. [1U](https://stackoverflow.com/questions/2128442/what-does-1u-x-do) means 00000000000000000000000000000001
2. [change a bit](https://stackoverflow.com/questions/6916974/change-a-bit-of-an-integer)
    * Or bit: x |= (1u << 3);
    * clear bit: x &= ~(1u<<3)
    * XOR bit: x ^=( 1u <<3 )

## [LinkedListCycle](https://leetcode.com/problems/linked-list-cycle/solution/)
首先搞head，然后fast是head->next。 while fast != slow
```c++
    bool hasCycle(ListNode *head) {
        if( !head ) return false;
        ListNode* slow = head;
        ListNode* fast = head->next;
        while( slow != fast ){
            if( !fast || fast->next == nullptr )
                return false;
            slow = slow->next;
            fast = fast->next->next;
        }
        return true;
    }
```

## [LRU](https://leetcode.com/problems/lru-cache/)
1. 需要double linked list 和 map of LS iterator and value
2. using PAIR = pair<list::iterator, int>
3. list erase can only erase iterator
4. iterator的最后一位 prev( iterator.end())
5. 需要删除最后一位，和最前一位。所以需要list 和map互相可以找到。list 记录both key 和value，然后用map的key来找到他们

```c++
using PAIR = pair<list<int>::iterator, int> ;
class LRUCache {
public:
    LRUCache(int capacity) {
        sz = capacity;
    }
    
    int get(int key) {
        if( mp.count( key ) ){
            ls.erase( mp[key].first );
            ls.push_front( key );
            int value = mp[key].second;
            mp[key] = make_pair( ls.begin(), value );
            return value;
        }
        return -1;
    }
    
    void put(int key, int value) {
        if( mp.count( key ) > 0 )
            ls.erase( mp[key].first );
        if( ls.size() >= sz ){
            mp.erase( ls.back() );
            ls.pop_back();
        }
        ls.push_front( key );
        mp[key] = make_pair( ls.begin(), value );
    }
    int sz;
    list<int> ls;
    unordered_map<int,PAIR> mp;
};
```

## [RotateArray](https://leetcode.com/problems/rotate-array/solution/)
注意reverse 的end point是你想要的那个数组加一. 你想reverse 0 to k-1, reverse( nums.begin(), nums.begin() + k )
1. 先reverse 整个
2. reverse first k
3. reverse n - k

```c++
    void rotate(vector<int>& nums, int k) {
        int n = nums.size();
        k %= n;
        reverse( nums.begin(), nums.end() );
        reverse( nums.begin(), nums.begin() + k);
        reverse( nums.begin() + k, nums.end() );   
    }
```
## [CountPrime](https://leetcode.com/problems/count-primes/)
1. 如果这个数是prime，那么把这个prime的倍数全部设成 non-prime
1. 取倍数的时候要主要不要overflow
```c++
    int countPrimes(int n) {
        int res = 0;
        vector<bool> vec( n, true );
        for( int i = 2; i < n; i++ )
        {
            if( vec[i] )
            {
                res++;
                long long tmp = i + i;
                while( tmp < n )
                {
                    vec[tmp] = false;
                    tmp = tmp + i;
                }
            }
        }
        return res;
    }
```

## [ReverseLinkedList](https://leetcode.com/problems/reverse-linked-list/submissions/)
1. Reverse LinkedList iteration method 只要两个variable
2. recursive method 需要记得不当前的next point to nullptr

```c++
    ListNode* reverseList(ListNode* head) {
        ListNode* pre = nullptr;
        ListNode* cur = head;
        while( cur ){
            ListNode* tmp = cur->next;
            cur->next = pre;
            pre = cur;
            cur = tmp;
        }
        return pre;
    }
```

## [PalindromeLinkedList](https://leetcode.com/problems/palindrome-linked-list/)
三种解法
1. copy array
2. reverse last half of linked list
3. recursion
```c++
public:
    bool isPalindrome(ListNode* head) {
        front = head;
        return helper( head );
        
    }
    
    bool helper( ListNode* head ){
        if( head ){
            if( !helper( head->next ) ) return false;
            if( front->val != head->val ) return false;
            front = front->next;
        }
        return true;
    }
    
private:
    ListNode* front;
};
```

## [SlidingWindowMaximum](https://leetcode.com/problems/sliding-window-maximum/description/)
两种方法
1. k个element 形成一个tree，每次insert/delete一个element，timestamp O(nlogk)
    1. 注意，multiset erase 会删除所有 value，删除时要 set.erase( set.find(val))
2. 做一个list,只存at most k index。每一次clear list，把小于current value的index都clear掉，最后再加上current index。最大值就是first item in list

```c++
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        multiset<int> st;
        int i = 0;
        while( i < k && i < nums.size() ){
            st.insert( nums[i]);
            i++;
        }
        auto iter = prev( st.end());
        vector<int> res{ *iter };
        for( ; i < nums.size(); i++ ){
            st.erase( st.find( nums[i-k] ) );
            st.insert( nums[i] );
            res.push_back( * prev( st.end() ) );
        }
        return res;
    }
```

```c++
    vector<int> maxSlidingWindow(vector<int>& nums, int k) {
        int i = 0;
        int j = 0;
        vector<int> res;
        for( int i = 0; i < nums.size(); i++ ){
            PushQue( nums[i] );
            if( i + 1 < k ) continue;
            
            if( i + 1 != k )
                PopQue( nums[j++] );
            res.push_back( qu.front() );
        }
        return res;
    }
    void PushQue( int k ){
        while( !qu.empty() && k > qu.back() )
            qu.pop_back();
        qu.push_back(k);
    }
    void PopQue( int k ){
        if( k == qu.front() )
            qu.pop_front();
    }
private:
    deque<int> qu;
```

## [SerializeandDeserializeBinaryTree](https://leetcode.com/problems/serialize-and-deserialize-binary-tree/)
用string stream
```c++
    // Encodes a tree to a single string.
    string serialize(TreeNode* root) {
        string str = "";
        SHelper( root, str );
        return str;
    }
    void SHelper(TreeNode* root, string& str ){
        if(!root ){
            str += "# ";
            return;
        }
        str += to_string( root->val ) + " ";
        SHelper( root->left, str );
        SHelper( root->right, str );
        
    } 

    // Decodes your encoded data to tree.
    TreeNode* deserialize(string data) {
        stringstream ss( data );
        return DHelper( ss );
    }
    TreeNode* DHelper( stringstream& ss ){
        string str;
        ss >> str;
        if( str == "#" ) return nullptr;
        TreeNode* node = new TreeNode( stoi(str) );
        node->left = DHelper( ss );
        node->right = DHelper( ss );
        return node;
    }
```

## [KMP](https://leetcode.com/problems/repeated-substring-pattern/solution/)
KMP stores the length of the longest prefix that is also a suffix.
1. 生成一个为n的vector
2. i 递增，j = dp[i-1]
3. 如果j不为0，s[i] != s[j], j = dp[j-1]
4. 如果s[i] == s[j], j++
得到table
5. 如果 n % （ n - dp[n-1] ) && dp[n-1] > 0, 则为真

```c++
 bool repeatedSubstringPattern(string s) {
        // KMP stores the length of the longest prefix that is also a suffix.
        int n = s.size();
        vector<int> dp( n, 0);
        for( int i = 1; i < n; i++ ){
            int j = dp[i-1];
            while( j > 0 && s[i] != s[j] )
                j = dp[j-1];
            if( s[i] == s[j] ) j++;
            dp[i] = j;
        }
        return dp[n-1] != 0 && ( n % (n-dp[n-1]) == 0 );
    }
```
其他类似问题
1. [Implement strStr()](https://leetcode.com/problems/implement-strstr/)
    ```c++
        int strStr(string haystack, string needle) {
            if( needle == "" ) return 0;
            auto table = GetTable( needle );
            auto res = FindMatch( table, haystack, needle );
            return res.size() > 0 ? res[0] : -1;
        }
        
        vector<int> GetTable( string s ){
            int n = s.size();
            vector<int> dp(n, 0);
            for( int i = 1; i < n; i++ ){
                int j = dp[i-1];
                while( j > 0 && s[i] != s[j] )
                    j = dp[j-1];
                if( s[i] == s[j] ) j++;
                dp[i] = j;
            }
            return dp;
        }
        
        vector<int> FindMatch( vector<int>& dp, string s, string p ){
            vector<int> res;
            int i = 0, j = 0;
            int n = s.size();
            for( ; i < n; i++ ){
                while( j > 0 && s[i] != p[j] )
                    j = dp[j-1];
                if( s[i] == p[j] ) j++;
                if( j == p.size() ) {
                    res.push_back( i - p.size() + 1);
                    j = dp[j-1];
                }
            }
            return res;
        }
    ```
1. [Longest Happy Prefix](https://leetcode.com/problems/longest-happy-prefix/)

## [LongestPalindromicSubsequence](https://leetcode.com/problems/longest-palindromic-subsequence/)
Parlindrom 的题大部分可以用O(n^2)来解决，这题 dp[i][i] == 1, if( s[i] == s[j] ) dp[i][j] = dp[i+1][j-1] + 2, else dp[i][j] = max( dp[i+1][j], dp[i][j-1]).
