### 剑指offer







### 前端算法

##### [剑指 Offer II 087. 复原 IP ](https://leetcode-cn.com/problems/0on3uN/)

```js
var restoreIpAddresses = function(s) {
    let l=s.length
    let ans=[]
    //i,j,k表示.分割的位置
    for(let i=1;i<4;i++){
        for(let j=i+1;j<i+4;j++){
            for(let k=j+1;k<j+4;k++){
                let a=s.slice(0,i)
                let b=s.slice(i,j)
                let c=s.slice(j,k)
                let d=s.slice(k,l)
                if(check(a)&&check(b)&&check(c)&&check(d)){
                    ans.push(a+'.'+b+'.'+c+'.'+d)
                }
            }
        }
    }
    function check(str){
        let num=parseInt(str)
        //如果转成num后再转回字符串不一样 说明可能是0开头的
        if(num+''!=str) return false
        if(num>=0&&num<=255) return true
        return false
    }
    return ans
};
```

### 热题100

###### [17. 电话号码的字母组合](https://leetcode-cn.com/problems/letter-combinations-of-a-phone-number/)

``` js
let button=['','','abc','def','ghi','jkl','mno','pqrs','tuv','wxyz']
let res=[]
var letterCombinations = function(digits) {
    if(digits=='') return[]
    res=[]
    let size=digits.length
    let ans=""
    mydfs(0,size,ans,digits)
    return res
};
function mydfs(cur,size,ans,digits){
    if(ans.length==size) return res.push(JSON.parse(JSON.stringify(ans)))
    let str=button[digits[cur]]
    for(let i=0;i<str.length;i++){
        //concat要重新赋值回ans
        ans=ans.concat(str[i])
        mydfs(cur+1,size,ans,digits)
        //切掉最后一个字符
        ans=ans.slice(0,ans.length-1)
    }
}
```

###### [31. 下一个排列](https://leetcode-cn.com/problems/next-permutation/)

```js
//思路 先从右往左找到第一个降序的数，然后再从右往左找比这个数大的数，交换后排序
var nextPermutation = function(nums) {
    let pre=nums[nums.length-1]
    let mark=0
    for(let i=nums.length-2;i>=0;i--){
        if(i==-1) return nums.sort()
        let cur=nums[i]
        if(cur<pre){
            for(let j=nums.length-1;j>=i;j--){
                if(nums[j]>cur){
                    nums[i]=nums[j]
                    nums[j]=cur
                    break
                }
            }
            mark=i+1
            break
        }else{
            pre=cur
        }
    }
    for(let i=mark;i<nums.length;i++){
        for(let j=mark;j<nums.length+i-mark-1;j++){
            if(nums[j]>nums[j+1]){
                let temp=nums[j]
                nums[j]=nums[j+1]
                nums[j+1]=temp
            }
        }
    }
    return n
```

###### 11.  [盛最多水的容器](https://leetcode-cn.com/problems/container-with-most-water/)

```js
var maxArea = function(height) {
    let left=0
    let right=height.length-1
    let ans=0
    while(left<right){
        ans=Math.max(ans,(right-left)*Math.min(height[right],height[left]))
        if(height[left]<height[right]){
            left++
        }else{
            right--
        }
    }
    return ans
};
```

###### [48. 旋转图像](https://leetcode-cn.com/problems/rotate-image/)

```js
var rotate = function(matrix) {
    const n=matrix[0].length
    //先上下翻转
    for(let i=0;i<Math.floor(n/2);i++){
        for(let j=0;j<n;j++){
            [matrix[i][j],matrix[n-i-1][j]]=[matrix[n-i-1][j],matrix[i][j]]
        }
    }
    //再沿着主对角线翻转
    for(let i=0;i<n;i++){
        //j<i！！！
        for(let j=0;j<i;j++){
            [matrix[i][j],matrix[j][i]]=[matrix[j][i],matrix[i][j]]
        }
    }
};
```

###### 49 [字母异位词分组](https://leetcode-cn.com/problems/group-anagrams/)

```js
var groupAnagrams = function(strs) {
    let mymap=new Map()
    //使用for of取出每个str
    for(let str of strs){
        let arr=Array.from(str)
        //tea、eta转成aet统一标识
        arr.sort()
        let strr=arr.join('')
        //查map，有就返回，无就返回空数组
        let list=mymap.get(strr)?mymap.get(strr):[]
        //推入新Str
        list.push(str)
        //map重新设置
        mymap.set(strr,list)
    }
    return Array.from(mymap.values())
};
```





###### 56 [合并区间](https://leetcode-cn.com/problems/merge-intervals/)

```js
var merge = function(intervals) {
    //排序
    intervals.sort((a,b)=>{
        return a[0]-b[0]
    })
    //先记录pre
    let pre=intervals[0][1]
    let ans=[]
    ans.push(intervals[0])
    for(let i=1;i<intervals.length;i++){
        if(intervals[i][0]<=pre){
            //ans末尾的区间
            let prenode=ans.pop()
            prenode.pop()
            //去重
            prenode.push(intervals[i][1]>pre?intervals[i][1]:pre)
            //记录
            ans.push(prenode)
        }else{
            //不重叠 直接推入
        ans.push(intervals[i])
        }
        //记录最后一个区间的结束时间
        pre=ans[ans.length-1][1]
    }
    return ans 
};
```



###### 105 [从前序与中序遍历序列构造二叉树](https://leetcode-cn.com/problems/construct-binary-tree-from-preorder-and-inorder-traversal/)

```js
var buildTree = function(preorder, inorder) {
    return dfs(preorder,inorder)
};
function dfs(preorder,inorder){
    if(preorder.length==0) return null
    let node=new TreeNode(preorder[0])
    if(preorder.length==1) return node
    let i
    for(i=0;i<inorder.length;i++){
        if(inorder[i]==preorder[0]) break 
    }
    let leftInorder=inorder.slice(0,i)
    let rightInorder=inorder.slice(i+1)
    let leftPreoder=preorder.slice(1,i+1)
    let rightPreoder=preorder.slice(i+1)
    node.left=dfs(leftPreoder,leftInorder)
    node.right=dfs(rightPreoder,rightInorder)
    return node
}
//自己写的解法
var buildTree = function(preorder, inorder) {
    return myBuildTree(preorder,inorder)
};
function myBuildTree(preorder,inorder){
    if(preorder.length==0||inorder.length==0) return null
    let root=preorder[0]
    preorder.shift()
    let node=new TreeNode(root)
    let pos=inorder.indexOf(root)
    let left=inorder.slice(0,pos)
    let right=inorder.slice(pos+1)
    node.left=myBuildTree(preorder,left)
    node.right=myBuildTree(preorder,right)
    return node
}
```



```js
//方法1 先序遍历后存在List再for循环修改

//方法2 先序遍历的同时改变节点并且通过栈保存右节点
var flatten = function(root) {
    let stack=[]
    if(root==null) return 
    //使用栈
    stack.push(root)
    let pre=null
    while(stack.length!=0){
        let cur=stack.pop()
        if(pre!=null){
            pre.left=null
            pre.right=cur
        }
        let left=cur.left
        let right=cur.right
        //先入栈右节点，因为先进后出
        if(right!=null) stack.push(right)
        if(left!=null) stack.push(left)
        pre=cur
    }
};
```







###### 128 [最长连续序列](https://leetcode-cn.com/problems/longest-consecutive-sequence/)

```js
var longestConsecutive = function(nums) {
    //使用set查找更快
    if(nums.length==0) return 0
    let mySet=new Set(nums) //Set去重
    let ans=1
    let count=1
    for(let i=0;i<nums.length;i++){
        let cur=nums[i]
        //记得初始化count=1
        count=1
        //如果有更小的元素，那么更小元素开始递增肯定长度更长
        if(mySet.has(cur-1)){
            continue
        }
        //否则开始找递增序列
        while(mySet.has(cur+1)){
            count++
            if(count>ans){
                ans=count
            }
            cur++
        }
    }
    return ans
};
```



###### 139 [单词拆分](https://leetcode-cn.com/problems/word-break/)

```js
var wordBreak = function(s, wordDict) {
    let dp=new Array(s.length+1).fill(false)
    dp[0]=true
    //动态规划 dp[i]表示s字符串到i-1位置是否能拼接出来
    for(let i=1;i<=s.length;i++){
        for(let j=0;j<i;j++){
            if(dp[j]==true&&wordDict.includes(s.slice(j,i))){
                dp[i]=true
            }
        }
    }
    return dp[s.length]
};
```





###### 148 [排序链表](https://leetcode-cn.com/problems/sort-list/)

```js
//自顶向下的归并排序
var sortList = function(head) {
    return toSortList(head,null)
};
function toSortList(head,tail){
    if(head==null) return null
    if(head.next==tail){
        head.next=null
        return head
    }
    let slow=head,fast=head
    while(fast!=tail&&fast.next!=tail){
        slow=slow.next
        fast=fast.next.next
    }
    if(fast!=tail){
        slow=slow.next
    }
    let mid=slow
    return myMerge(toSortList(head,mid),toSortList(mid,tail))
}
function myMerge(head1,head2){
    let newHead=new ListNode(-1)
    let temp=newHead
    let temp1=head1,temp2=head2
    while(temp1!=null&&temp2!=null){
        if(temp1.val<temp2.val){
            temp.next=temp1
            temp1=temp1.next
        }else{
            temp.next=temp2
            temp2=temp2.next
        }
        temp=temp.next
    }
    if(temp1!=null) temp.next=temp1
    if(temp2!=null) temp.next=temp2
    return newHead.next
}
```



###### 152 [乘积最大子数组](https://leetcode-cn.com/problems/maximum-product-subarray/) 

```js
var maxProduct = function(nums) {
    let minArr=new Array(nums.length).fill(nums[0])
    let maxArr=new Array(nums.length).fill(nums[0])
    for(let i=1;i<nums.length;i++){
        //如果这个数是正数，那么找之前最大的，   如果是负数 找之前负数最大的(也就是正数最小的)
        maxArr[i]=Math.max(maxArr[i-1]*nums[i],Math.max(nums[i],nums[i]*minArr[i-1]))
        minArr[i]=Math.min(minArr[i-1]*nums[i],Math.min(nums[i],nums[i]*maxArr[i-1]))
    }
    let res=nums[0]
    for(let i=0;i<nums.length;i++){
        res=Math.max(res,maxArr[i])
    }
    return res
};
```



###### 207 [课程表](https://leetcode-cn.com/problems/course-schedule/) 

```js
var canFinish = function(numCourses, prerequisites) {
    let inDegree=new Array(numCourses).fill(0) // 入度数组
    const map={} // 邻接表
    for(let i=0;i<prerequisites.length;i++){
        inDegree[prerequisites[i][0]]++ // 求课的初始入度值
        // 当前课是否存在于邻接表
        if(map[prerequisites[i][1]]){
            map[prerequisites[i][1]].push(prerequisites[i][0])
        }else{
            map[prerequisites[i][1]]=[prerequisites[i][0]]
        }
    }
    let queue=[]
    let ans=0
    // 所有入度为0的课入列
    for(let i=0;i<numCourses;i++){
        if(inDegree[i]==0){
            queue.push(i)
        }
    }
    while(queue.length!=0){
        let first=queue.shift()
        if(map[first]){
            for(let i=0;i<map[first].length;i++){
                // 依赖它的后续课的入度-1
                let item=map[first][i]
                inDegree[item]--
                // 如果因此减为0，入列
                if(inDegree[item]==0){
                    queue.push(item)
                }
            }
        }
        ans++
    }
    return ans==numCourses
};
```



###### 221 [最大正方形](https://leetcode-cn.com/problems/maximal-square/)

```js
//dp[i][j]为i,j为右下角的最大正方形边长
var maximalSquare = function(matrix) {
    let dp=new Array(matrix.length).fill(0).map(()=>new Array(matrix[0].length).fill(0))
    let ans=0
    for(let i=0;i<matrix.length;i++){
        dp[i][0]=Number(matrix[i][0])
        ans=Math.max(ans,dp[i][0])
    }
    for(let j=0;j<matrix[0].length;j++){
        dp[0][j]=Number(matrix[0][j])
        ans=Math.max(ans,dp[0][j])
    }
    for(let i=1;i<matrix.length;i++){
        for(let j=1;j<matrix[0].length;j++){
            //如果该处为0 则dp[i][j]不可能为右下角的最大正方形
            if(matrix[i][j]==0){
                dp[i][j]=0
            }else{
                dp[i][j]=Math.min(dp[i-1][j],dp[i][j-1],dp[i-1][j-1])+1
                //如果为1 则取dp[i-1][j],dp[i][j-1],dp[i-1][j-1]最小
            }
            ans=Math.max(ans,dp[i][j])
        }
    }
    return ans*ans
};
```



###### 238 [除自身以外数组的乘积](https://leetcode-cn.com/problems/product-of-array-except-self/)

```js
var productExceptSelf = function(nums) {
    let left=new Array(nums.length)
    let right=new Array(nums.length)
    let ans=new Array(nums.length)
    left[0]=1
    for(let i=1;i<nums.length;i++){
        left[i]=left[i-1]*nums[i-1]
    }
    right[nums.length-1]=1
    for(let i=nums.length-2;i>=0;i--){
        right[i]=right[i+1]*nums[i+1]
    }
    //i左边乘积*i右边乘积
    for(let i=0;i<nums.length;i++){
        ans[i]=left[i]*right[i]
    }
    return ans
};

//优化空间复杂度后
var productExceptSelf = function(nums) {
    let left=new Array(nums.length)
    let right=1
    left[0]=1
    for(let i=1;i<nums.length;i++){
        left[i]=left[i-1]*nums[i-1]
    }
    //正常算出left[i]
    //ans[i]=left[i]*right 压缩right[i]
    for(let i=nums.length-1;i>=0;i--){
        left[i]=left[i]*right
        right*=nums[i]
    }
    return left
};
```



###### 240 [搜索二维矩阵 II](https://leetcode-cn.com/problems/search-a-2d-matrix-ii/)

```js
//二分法搜
var searchMatrix = function(matrix, target) {
    for(let row of matrix){
        let index=search(row,target)
        if(index>=0){
            return true
        }
    }
    return false
    function search(nums,target){
        let left=0
        let right=nums.length-1
        while(left<=right){
            let mid=Math.floor((left+(right-left)/2))
            if(nums[mid]==target){
                return mid
            }else if(nums[mid]>target){
                right=mid-1
            }else if(nums[mid]<target){
                left=mid+1
            }
        }
        return -1
    }
};
//快速搜
var searchMatrix = function(matrix, target) {
    let m=matrix.length
    let n=matrix[0].length
    let x=0
    let y=n-1
    while(x<m&&n>=0){
        if(matrix[x][y]==target){
            return true
        }else if(matrix[x][y]>target){
            y--
        }else{
            x++
        }
    }
    return false
};
```



###### 279 [完全平方数](https://leetcode-cn.com/problems/perfect-squares/)

```js
var numSquares = function(n) {
    let f=new Array(n+1).fill(0)
    for(let i=1;i<=n;i++){
        let minn=Number.MAX_VALUE
        for(let j=1;j*j<=i;j++){
            minn=Math.min(minn,f[i-j*j])
        }
        f[i]=minn+1
    }
    return f[n]
};

//或者使用找硬币方法
let size=parseInt(Math.sqrt(n))
    let coins=new Array(size+1).fill(0)
    for(let i=1;i<=size;i++){
        coins[i]=i*i
    }
    let dp=new Array(n+1).fill(n)
    dp[0]=0
    for(let i=1;i<=n;i++){
        for(let j=1;j<=size;j++){
            if(coins[j]<=i){
                dp[i]=Math.min(dp[i-coins[j]]+1,dp[i])
            }
        }
    }
    return dp[n]
```



###### 287 [寻找重复数](https://leetcode-cn.com/problems/find-the-duplicate-number/)

```js
var findDuplicate = function(nums) {
    for(let i=0;i<nums.length;i++){
        while(nums[i]!=i+1){
            //该步操作时将3换到位置3 如果位置3就是3那么说明重复了
            if(nums[i]!=nums[nums[i]-1]){
                swap(nums,i,nums[i]-1)
            }else{
                return nums[i]
            }
        }
    }
    function swap(nums,left,right){
        let temp=nums[left]
        nums[left]=nums[right]
        nums[right]=temp
    }
};
```



######  309 [最佳买卖股票时机含冷冻期](https://leetcode-cn.com/problems/best-time-to-buy-and-sell-stock-with-cooldown/)

```js
var maxProfit = function(prices) {
    let f=new Array(prices.length).fill(0).map(()=>new Array(3).fill(0))
    f[0][0]=-prices[0]
    // f[i][0]: 手上持有股票的最大收益
    // f[i][1]: 手上不持有股票，并且处于冷冻期中的累计最大收益
    // f[i][2]: 手上不持有股票，并且不在冷冻期中的累计最大收益
    let n=prices.length
    for(let i=1;i<n;i++){
        f[i][0]=Math.max(f[i-1][0],f[i-1][2]-prices[i])
        f[i][1]=f[i-1][0]+prices[i]
        f[i][2]=Math.max(f[i-1][1],f[i-1][2])
    }
    return Math.max(f[n-1][1],f[n-1][2])
};
```



######  394 [字符串解码](https://leetcode-cn.com/problems/decode-string/)

```js
var decodeString = function(s) {
    let stack=[]
    let ans=''
    for(let ch of s){
        if(ch==']'){
            let memo=''
            while(stack.length!=0){
                let item=stack.pop()
                if(item=='[') break 
                memo=item.concat(memo)
            }
            let num=''
            while(stack[stack.length-1]>='0'&&stack[stack.length-1]<=9){
                let item=stack.pop()
                num=item.concat(num)
            }
            let temp=''
            for(let i=0;i<num;i++){
                temp=temp.concat(memo)
            }
            stack.push(temp)
        }else{
            stack.push(ch)
        }
    }
    return stack.join('')
};
```



###### 406 [根据身高重建队列](https://leetcode-cn.com/problems/queue-reconstruction-by-height/)

```js
//插队法
var reconstructQueue = function(people) {
    //身高从高到低 相等情况由第二个参数由低到高
    people.sort((a,b)=>{
        if(a[0]!=b[0]){
            return b[0]-a[0]
        }else{
            return a[1]-b[1]
        }
    })
    let ans=[]
    for(let p of people){
        ans.splice(p[1],0,p)
    }
    return ans
};
```



##### 前缀和

###### 437 [路径总和 III](https://leetcode-cn.com/problems/path-sum-iii/)

```js
//前缀和
var pathSum = function(root, targetSum) {
    let mymap=new Map()
    mymap.set(0,1)
    return dfs(root,0,targetSum)
    function dfs(root,curr,targetSum){
        if(root==null){
            return 0
        }
        curr+=root.val
        let ret=0
        ret=(mymap.get(curr-targetSum)||0)
        mymap.set(curr,(mymap.get(curr)||0)+1)
        ret+=dfs(root.left,curr,targetSum)
        ret+=dfs(root.right,curr,targetSum)
        mymap.set(curr,(mymap.get(curr))-1)
        return ret
    }
    
};
```

###### 560 [和为 K 的子数组](https://leetcode-cn.com/problems/subarray-sum-equals-k/)

```js
//前缀和
var subarraySum = function(nums, k) {
    let prefix=new Array(nums.length).fill(0)
    prefix[0]=nums[0]
    let map=new Map()
    map.set(prefix[0],1)
    let ans=0
    if(prefix[0]==k) ans++
    for(let i=1;i<nums.length;i++){
        prefix[i]=prefix[i-1]+nums[i]
        if(prefix[i]==k) ans++
        map.get(prefix[i]-k)?ans+=map.get(prefix[i]-k):0
        map.get(prefix[i])?map.set(prefix[i],map.get(prefix[i])+1):map.set(prefix[i],1)
    }
    return ans
};
```





###### 647 [回文子串](https://leetcode-cn.com/problems/palindromic-substrings/)

```js
//和找最长回文子串一样
var countSubstrings = function(s) {
    let ans=0
    for(let i=0;i<s.length;i++){
        ans+=findStrings(i,i,s)
        if(i<s.length-1)
        ans+=findStrings(i,i+1,s)
    }
    return ans
};
function findStrings(left,right,s){
    let count=0
    while(left>=0&&right<s.length&&s[left]==s[right]){
        left--
        right++
        count++
    }
    return count
}
```



###### 621 [任务调度器](https://leetcode-cn.com/problems/task-scheduler/)

```js
var leastInterval = function(tasks, n) {
    //统计每个任务次数
    let freq=_.countBy(tasks)
    //有几种任务
    let m=Object.keys(freq).length
    //下次可执行的时间
    let nextValid=new Array(m).fill(1)
    //每种任务剩余一个
    let rest=Object.values(freq)
    let time=0
    for(let i=0;i<tasks.length;i++){
        time++
        let minNextValid=Number.MAX_VALUE
        //找到最快能执行的任务
        for(let j=0;j<m;j++){
            if(rest[j]>0){
                minNextValid=Math.min(minNextValid,nextValid[j])
            }
        }
        time=Math.max(time,minNextValid)
        //找到最快能执行的任务中 剩余数最多的
        let best=-1
        for(let j=0;j<m;j++){
            if(rest[j]&&nextValid[j]<=time){
                if(best==-1||rest[j]>rest[best]){
                    best=j
                }
            }
        }
        nextValid[best]=time+n+1
        rest[best]--
    }
    return time
};
```



###### 581 [最短无序连续子数组](https://leetcode-cn.com/problems/shortest-unsorted-continuous-subarray/)

```js
//解法1 先排序 然后left right 双指针
//解法2 一次遍历 找到start、end 
//start表示左边第一个不满足升序的元素，end同理
var findUnsortedSubarray = function(nums) {
    let len=nums.length
    let maxx=nums[0]
    let minn=nums[len-1]
    let end=-1
    let start=0
    for(let i=0;i<len;i++){
        if(nums[i]>=maxx){
            maxx=nums[i]
        }else{
            end=i
        }
        if(nums[len-i-1]<=minn){
            minn=nums[len-i-1]
        }else{
            start=len-i-1
        }
    }
    return end-start+1
};
```







### 排序算法

##### 堆排序

参考文献：https://leetcode-cn.com/problems/kth-largest-element-in-an-array/solution/xie-gei-qian-duan-tong-xue-de-ti-jie-yi-kt5p2/

```js

 // 整个流程就是上浮下沉
var findKthLargest = function(nums, k) {
   let heapSize=nums.length
    buildMaxHeap(nums,heapSize) // 构建好了一个大顶堆
    // 进行下沉 大顶堆是最大元素下沉到末尾
    for(let i=nums.length-1;i>=nums.length-k+1;i--){
        swap(nums,0,i)
        --heapSize // 下沉后的元素不参与到大顶堆的调整
        // 重新调整大顶堆
         maxHeapify(nums, 0, heapSize);
    }
    return nums[0]
   // 自下而上构建一颗大顶堆
   function buildMaxHeap(nums,heapSize){
     for(let i=Math.floor(heapSize/2)-1;i>=0;i--){
        maxHeapify(nums,i,heapSize)
     }
   }
   // 从左向右，自上而下的调整节点
   function maxHeapify(nums,i,heapSize){
       let l=i*2+1
       let r=i*2+2
       let largest=i
       if(l < heapSize && nums[l] > nums[largest]){
           largest=l
       }
       if(r < heapSize && nums[r] > nums[largest]){
           largest=r
       }
       if(largest!==i){
           swap(nums,i,largest) // 进行节点调整
           // 继续调整下面的非叶子节点
           maxHeapify(nums,largest,heapSize)
       }
   }
   function swap(a,  i,  j){
        let temp = a[i];
        a[i] = a[j];
        a[j] = temp;
   }
};
```

