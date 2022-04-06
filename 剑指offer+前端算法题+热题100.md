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



