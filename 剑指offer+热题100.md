### 剑指offer



splice() 方法用于添加或删除**数组**中的元素 **改变原始数组** **会返回删除的元素数组**

```js
var fruits = ["Banana", "Orange", "Apple", "Mango"];
fruits.splice(2,0,"Lemon","Kiwi");
fruits 输出结果：
Banana,Orange,Lemon,Kiwi,Apple,Mango
```





##### [剑指 Offer II 09用两个栈实现队列](https://leetcode-cn.com/problems/yong-liang-ge-zhan-shi-xian-dui-lie-lcof/)

```js
var CQueue = function() {
    this.inStack=[]
    this.ouStack=[]
};

/** 
 * @param {number} value
 * @return {void}
 */
CQueue.prototype.appendTail = function(value) {
    this.inStack.push(value)
};

/**
 * @return {number}
 */
CQueue.prototype.deleteHead = function() {
    if(!this.ouStack.length){
        if(!this.inStack.length){
            return -1
        }
        while(this.inStack.length){
            this.ouStack.push(this.inStack.pop())
        }
    }
    return this.ouStack.pop()
};

```

##### [剑指 Offer 14- I. 剪绳子](https://leetcode-cn.com/problems/jian-sheng-zi-lcof/)

```js
var cuttingRope = function(n) {
    let dp=new Array(n+1).fill(1)
    dp[2]=1
    for(let i=3;i<=n;i++){
        for(let j=1;j<i;j++){
            dp[i]=Math.max(dp[i],Math.max(j*(i-j),j*dp[i-j]))
        }
    }
    return dp[n]
};

//大数情况下
var cuttingRope = function(n) {
    let a=parseInt(n/3)
    let b=n%3
    let ans=1
    if(n<=3){return n-1}
    if(b==0){
        ans=myPow(3,a)
    }else if(b==1){
        ans=myPow(3,a-1)*4
    }else if(b==2){
        ans=myPow(3,a)*2
    }
    return ans%1000000007
};
function myPow(a,b){
    let ans=1
    for(let i=0;i<b;i++){
        ans*=a
        ans%=1000000007
    }
    return ans
}
```



##### [剑指 Offer 15. 二进制中1的个数](https://leetcode-cn.com/problems/er-jin-zhi-zhong-1de-ge-shu-lcof/)

```js
var hammingWeight = function(n) {
    let ans=0
    for(let i=0;i<32;i++){
        if(n&(1<<i)){
            ans++
        }
    }
    return ans
};
//加速方法
var hammingWeight = function(n) {
    let ans=0
    while(n){
        n&=(n-1)
        ans++
    }
    return ans
};
```



##### 快速幂  [剑指 Offer 16. 数值的整数次方](https://leetcode-cn.com/problems/shu-zhi-de-zheng-shu-ci-fang-lcof/) 

```js
var myPow = function(x, n) {
    if(n==0)return 1
    return n>0?quickPow(x,n):1.0/quickPow(x,-n)
};
function quickPow(x,n){
    let xx=x
    let ans=1.0
    while(n>0){
        if(n&1){
            ans*=xx
        }
        xx*=xx
        n/=2
    }
    return ans
}
```



##### [剑指 Offer 18. 删除链表的节点](https://leetcode-cn.com/problems/shan-chu-lian-biao-de-jie-dian-lcof/)

```js
var deleteNode = function(head, val) {
    let dummyNode=new ListNode(-1)
    dummyNode.next=head
    let pre=dummyNode
    while(head!=null){
        let next=head.next
        if(head.val==val){
            pre.next=next
            break
        }
        pre=head
        head=next
    }
    return dummyNode.next
};
```



#### 树的递归

##### [剑指 Offer 26. 树的子结构](https://leetcode-cn.com/problems/shu-de-zi-jie-gou-lcof/)

```js
var isSubStructure = function(A, B) {
    if(A==null||B==null) return false
    if(A.val==B.val&&helper(A.left,B.left)&&helper(A.right,B.right)){
        return true
    }else{
        return isSubStructure(A.left,B)||isSubStructure(A.right,B)
    }
};
function helper(A,B){
    //先判断B是不是null 是的话说明匹配完了  return true
    if(B==null) return true
    if(A==null) return false
    if(A.val==B.val){
        return helper(A.left,B.left)&&helper(A.right,B.right)
    }else{
        return false
    }
}
```

##### [剑指 Offer 28. 对称的二叉树](https://leetcode-cn.com/problems/dui-cheng-de-er-cha-shu-lcof/)

```js
var isSymmetric = function(root) {
    return helper(root,root)
};
function helper(left,right){
    if(!left&&!right) return true
    if(!left||!right) return false
    return left.val==right.val&&helper(left.left,right.right)&&helper(left.right,right.left)
}
```



##### [剑指 Offer 29. 顺时针打印矩阵](https://leetcode-cn.com/problems/shun-shi-zhen-da-yin-ju-zhen-lcof/)

```js
let ans=[]
var spiralOrder = function(matrix) {
    ans=[]
    let diretion=1
    let mark=new Array(matrix.length).fill(false).map(()=>new Array(matrix[0].length).fill(false))
    for(let i=0;i<matrix.length;i++){
        if(mark[i][i]==false){
            mark[i][i]=true
            dfs(i,i,diretion,matrix,mark)
        }
    }
    return ans
};
function dfs(x,y,diretion,matrix,mark){
    ans.push(matrix[x][y])
    if(diretion%4==1){
        if(y+1<matrix[0].length&&mark[x][y+1]==false){
            mark[x][y+1]=true
            dfs(x,y+1,diretion,matrix,mark)
        }else{
            diretion++
        }
    }
    if(diretion%4==2){
        if(x+1<matrix.length&&mark[x+1][y]==false){
            mark[x+1][y]=true
            dfs(x+1,y,diretion,matrix,mark)
        }else{
            diretion++
        }
    }
    if(diretion%4==3){
        if(y-1>=0&&mark[x][y-1]==false){
            mark[x][y-1]=true
            dfs(x,y-1,diretion,matrix,mark)
        }else{
            diretion++
        }
    }
    if(diretion%4==0){
        if(x-1>=0&&mark[x-1][y]==false){
            mark[x-1][y]=true
            dfs(x-1,y,diretion,matrix,mark)
        }else{
            diretion++
        }
    }
}
```



##### [剑指 Offer 30. 包含min函数的栈](https://leetcode-cn.com/problems/bao-han-minhan-shu-de-zhan-lcof/)

```js
var MinStack = function() {
    this.stack=[]
    this.minStack=[]
    this.minn=Number.MAX_VALUE
};

MinStack.prototype.push = function(x) {
    this.stack.push(x)
    if(x<this.minn){
        this.minStack.push(x)
        this.minn=x
    }else{
        this.minStack.push(this.minn)
    }
};


MinStack.prototype.pop = function() {
    this.stack.pop()
    this.minStack.pop()
    this.minn=this.minStack[this.minStack.length-1]
    if(this.stack.length==0){
        this.minn=Number.MAX_VALUE
    }
};


MinStack.prototype.top = function() {
    return this.stack[this.stack.length-1]
};


MinStack.prototype.min = function() {
    return this.minStack[this.minStack.length-1]
};

```



##### [剑指 Offer 31. 栈的压入、弹出序列](https://leetcode-cn.com/problems/zhan-de-ya-ru-dan-chu-xu-lie-lcof/)

```js
//直接模拟压栈和出栈操作
var validateStackSequences = function(pushed, popped) {
    let stack=[]
    let k=0
    for(let i=0;i<pushed.length;i++){
        stack.push(pushed[i])
        if(pushed[i]==popped[k]){
            k++
            stack.pop()
        }
        while(stack[stack.length-1]==popped[k]&&stack.length>0){
            k++
            stack.pop()
        }
    }
    if(stack.length==0){
        return true
    }else{
        return false
    }
};
```



##### [剑指 Offer 32 - II. 从上到下打印二叉树 II](https://leetcode-cn.com/problems/cong-shang-dao-xia-da-yin-er-cha-shu-ii-lcof/)

```js
var levelOrder = function(root) {
    let queue=[]
    let ans=[]
    queue.push(root)
    while(queue.length!=0){
        let ceng=[]
        while(queue.length!=0){
            ceng.push(queue.shift())
        }
        let temp=[]
        for(let i=0;i<ceng.length;i++){
            if(ceng[i]!=null){
                temp.push(ceng[i].val)
                queue.push(ceng[i].left)
                queue.push(ceng[i].right)
            }
        }
        if(temp.length!=0)
        ans.push(temp)
    }
    return ans
};
```



##### [剑指 Offer 33. 二叉搜索树的后序遍历序列](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-de-hou-xu-bian-li-xu-lie-lcof/)

```js
var verifyPostorder = function(postorder) {
    return recur(postorder,0,postorder.length-1)
};
function recur(postorder,i,j){
    if(i>=j) return true
    let pos=i
    //找到比root小的
    while(postorder[pos]<postorder[j]){
        pos++
    }
    //看右边是不是都比根节点大
    let pps=pos
    while(postorder[pos]>postorder[j]){
        pos++
    }
    //如果pos走到了j&&左子树也符合 右子树也符合 则为真 ，记得pos-1是因为闭区间pos++了  j-1是去掉根节点
    return pos==j&&recur(postorder,i,pos-1)&&recur(postorder,pos,j-1)
}
```

##### [剑指 Offer 34. 二叉树中和为某一值的路径](https://leetcode-cn.com/problems/er-cha-shu-zhong-he-wei-mou-yi-zhi-de-lu-jing-lcof/)

```js
//难度更高请看 https://leetcode-cn.com/problems/path-sum-iii/
var pathSum = function(root, target) {
    let map=new Map()
    let ans=[]
    dfs(root,target,[],0)
    function dfs(root,target,arr,sum){
        if(root==null) return 0
        sum+=root.val
        arr.push(root.val)
        if(sum==target&&root.left==null&&root.right==null){
            ans.push(JSON.parse(JSON.stringify(arr)))
        }
        let arr1=JSON.parse(JSON.stringify(arr))
        dfs(root.left,target,arr,sum)
        dfs(root.right,target,arr1,sum)
    }
    return ans
};
```



##### [剑指 Offer 35. 复杂链表的复制](https://leetcode-cn.com/problems/fu-za-lian-biao-de-fu-zhi-lcof/)

```js
//hash表解法
var copyRandomList = function(head) {
    let cur=head
    let map=new Map()
    while(cur!=null){
        let node=new Node(cur.val)
        map.set(cur,node)
        cur=cur.next
    }
    cur=head
    while(cur!=null){
        let node=map.get(cur)
        node.next=map.get(cur.next)||null
        node.random=map.get(cur.random)||null
        cur=cur.next
    }
    return map.get(head)
}

var copyRandomList = function(head) {
    if(head==null) return 
    let cur=head
    //复制链表
    while(cur!=null){
        let node=new Node(cur.val)
        node.next=cur.next
        cur.next=node
        cur=node.next
    }
    //补上random指针
    cur=head
    while(cur!=null){
        node=cur.next
        if(cur.random!=null){
            node.random=cur.random.next
        }
        cur=node.next
    }
    //拆分链表
    cur=head.next
    let pre=head
    let newHead = cur
    while(cur.next!=null){
        pre.next=pre.next.next
        cur.next=cur.next.next
        cur=cur.next
        pre=pre.next
    }
    pre.next=null
    return newHead
}
```



##### [剑指 Offer 36. 二叉搜索树与双向链表](https://leetcode-cn.com/problems/er-cha-sou-suo-shu-yu-shuang-xiang-lian-biao-lcof/)

```js
let pre=null,head=null
var treeToDoublyList = function(root) {
    pre=null
    head=null
    if(root==null) return null
    dfs(root)
    pre.right=head //进行头节点和尾节点的相互指向，这两句的顺序也是可以颠倒的
    head.left=pre
    return head
};
function dfs(root){
    if(root==null) return null
    dfs(root.left)
    //pre用于记录双向链表中位于cur左侧的节点，即上一次迭代中的cur,当pre==null时，cur左侧没有节点,即此时cur为双向链表中的头节点
    if(pre==null){
        head=root
    }
    //反之，pre!=null时，cur左侧存在节点pre，需要进行pre.right=cur的操作。
    else{
        pre.right=root
    }
    root.left=pre//pre是否为null对这句没有影响,且这句放在上面两句if else之前也是可以的
    pre=root//pre指向当前的cur
    dfs(root.right)//全部迭代完成后，pre指向双向链表中的尾节点
}
```



##### [剑指 Offer 38. 字符串的排列](https://leetcode-cn.com/problems/zi-fu-chuan-de-pai-lie-lcof/)

```js
var permutation = function(s) {
    let ret=[]
    let memo=new Array(s.length).fill(false)
    dfs(s,[])
    function dfs(s,res){
        if(s.length==res.length){
            //如果不重复才加入res
            if(!ret.includes(res.join(''))){
                ret.push(res.join(''))
            }
            return 
        }
        //for循环如果该元素没用过就用
        for(let i=0;i<s.length;i++){
            if(memo[i]==false){
                res.push(s[i])
                memo[i]=true
                dfs(s,res)
                memo[i]=false
                res.pop(s[i])
            }
        }
    }
    return ret
};
```



##### [剑指 Offer 44. 数字序列中某一位的数字](https://leetcode-cn.com/problems/shu-zi-xu-lie-zhong-mou-yi-wei-de-shu-zi-lcof/)

```js
var findNthDigit = function(n) {
    let digit=1 //位数
    let start=1 //开始的数字
    let count=9 //该位数下有多少个数字
    while(n>count){
        n-=count
        digit++
        start*=10 //开始数字是 1 10 100 1000
        count=digit*start*9 //数字个数
    }
    //找到是哪个数
    let num=start+Math.floor((n-1)/digit)
    //找到是这个数字的哪一位
    return String(num)[Math.floor((n-1)%digit)]
};
```







##### [剑指 Offer 45. 把数组排成最小的数](https://leetcode-cn.com/problems/ba-shu-zu-pai-cheng-zui-xiao-de-shu-lcof/)

```js
var minNumber = function(nums) {
    nums.sort((a,b)=>{
        return (a+''+b)-(b+''+a)
    })
    return nums.join('')
};
```



##### [剑指 Offer 46. 把数字翻译成字符串](https://leetcode-cn.com/problems/ba-shu-zi-fan-yi-cheng-zi-fu-chuan-lcof/)

```js
//注意点1 num是数字 所以要先string化
// dp[i]=dp[i-1]+dp[i-2]  或  dp[i]=dp[i-1]
var translateNum = function(num) {
    if(String(num).length==0){
        return 0
    }
    let dp=new Array(String(num).length+1).fill(0)
    dp[1]=1
    let temp=Number(String(num)[0]+String(num)[1])
    //如果在10到25之间 那么说明可以翻译 for循环里相同
    if (temp >= 10 && temp <= 25) {
        dp[2]=2
    }else{
        dp[2]=1
    }
    for(let i=3;i<=String(num).length;i++){
        temp=Number(String(num)[i-2]+String(num)[i-1])
        if (temp >= 10 && temp <= 25) {
            dp[i]=dp[i-1]+dp[i-2]
        }else{
            dp[i]=dp[i-1]
        }
    }
    return dp[String(num).length]
};
```



##### [剑指 Offer 48. 最长不含重复字符的子字符串](https://leetcode-cn.com/problems/zui-chang-bu-han-zhong-fu-zi-fu-de-zi-zi-fu-chuan-lcof/)

```js
var lengthOfLongestSubstring = function(s) {
    //用队列来进行滑动窗口写法
    let ans=1
    if(s.length==0) return 0
    let queue=[]
    let ch=s[0]
    let pos=1
    queue.push(ch)
    while(pos<s.length){
        ch=s[pos]
        if(queue.includes(ch)){
            while(queue.includes(ch)&&queue.length){
                queue.shift()
            }
        }
        queue.push(ch)
        if(queue.length>ans){
            ans=queue.length
        }
        pos++
    }
    return ans
};
```



##### [剑指 Offer 49. 丑数](https://leetcode-cn.com/problems/chou-shu-lcof/)

```js
//看题解
var nthUglyNumber = function(n) {
    let a,b,c
    a=b=c=0
    let dp=new Array(n).fill(0)
    dp[0]=1
    for(let i=1;i<n;i++){
        let n1=dp[a]*2
        let n2=dp[b]*3
        let n3=dp[c]*5
        dp[i]=Math.min(n1,n2,n3)
        if(dp[i]==n1) a++
        if(dp[i]==n2) b++
        if(dp[i]==n3) c++
    }
    return dp[n-1]
};
```



##### [剑指 Offer 50. 第一个只出现一次的字符](https://leetcode-cn.com/problems/di-yi-ge-zhi-chu-xian-yi-ci-de-zi-fu-lcof/)

```js
var firstUniqChar = function(s) {
    //lodash中计算频次的函数
    const frequency=_.countBy(s)
    for(const item of Array.from(s)){
        //如果频次为1 则返回
        if(frequency[item]==1){
            return item
        }
    }
    return ' '
};
```



##### [剑指 Offer 52. 两个链表的第一个公共节点](https://leetcode-cn.com/problems/liang-ge-lian-biao-de-di-yi-ge-gong-gong-jie-dian-lcof/)

```js
var getIntersectionNode = function(headA, headB) {
    if(headA==null||headB==null){
        return null
    }
    let p1=headA
    let p2=headB
    while(p1!=p2){
        p1=p1==null?headB:p1.next
        p2=p2==null?headA:p2.next
    }
    return p1
};
```



##### [剑指 Offer 53 - I. 在排序数组中查找数字 I](https://leetcode-cn.com/problems/zai-pai-xu-shu-zu-zhong-cha-zhao-shu-zi-lcof/)

```js
//使用二分搜索左边界法
var search = function(nums, target) {
    let left=0
    let right=nums.length-1
    while(left<=right){
        let mid=left+Math.floor((right-left)/2)
        if(nums[mid]==target){
            right=mid-1
        }else if(nums[mid]>target){
            right=mid-1
        }else if(nums[mid]<target){
            left=mid+1
        }
    }
    if(nums[left]!=target||left>=nums.length){
        return 0
    }else{
        let count=0
        for(let i=left;i<nums.length;i++){
            if(nums[i]==target){
                count++
            }
        }
        return count
    }
};
```



##### [剑指 Offer 53 - II. 0～n-1中缺失的数字](https://leetcode-cn.com/problems/que-shi-de-shu-zi-lcof/)

```js
//二分法
var missingNumber = function(nums) {
    let left=0
    let right=nums.length-1
    while(left<=right){
        let mid=left+Math.floor((right-left)/2)		
        if(nums[mid]==mid){
            left=mid+1
        }else{
            right=mid-1
        }
    }
    return left
};
```



##### [剑指 Offer 55 - II. 平衡二叉树](https://leetcode-cn.com/problems/ping-heng-er-cha-shu-lcof/)

```js
var isBalanced = function(root) {
    let ans=dfs(root)
    function dfs(root){
        if(root==null){
            return 0
        }
        let left=dfs(root.left)
        let right=dfs(root.right)
        if(left==-1||right==-1||Math.abs(left-right)>1){
            return -1
        }else{
            return Math.max(left,right)+1
        }
    }
    if(ans==-1){
        return false
    }else{
        return true
    }
};
```



##### [剑指 Offer 56 - II. 数组中数字出现的次数 II](https://leetcode.cn/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-ii-lcof/)

```js
//方法1 hash表 同下

//方法2 分组异或， 找到一个异或为1的位 然后根据这位去分组 分组后就ok
var singleNumbers = function(nums) {
    let yiHuo=0
    for(let i=0;i<nums.length;i++){
        let n=nums[i]
        yiHuo^=n
    }
    let flag=1
    while((flag&yiHuo)==0){
        flag<<=1
    }
    let a=0
    let b=0
    for(let n of nums){
        if(flag&n){
            a^=n
        }else{
            b^=n
        }
    }
    return [a,b]
};
```



##### [剑指 Offer 56 - I. 数组中数字出现的次数](https://leetcode.cn/problems/shu-zu-zhong-shu-zi-chu-xian-de-ci-shu-lcof/)

```js
//方法1 hash表
var singleNumbers = function(nums) {
    let map={}
    for(let i=0;i<nums.length;i++){
        let n=nums[i]
        if(map[n]){
            delete map[n]
        }else{
            map[n]=1
        }
    }
    return Object.keys(map)
};
//方法2 统一二进制每一位上1个个数后%3
var singleNumber = function(nums) {
    let count=new Array(32).fill(0)
    for(let i=0;i<nums.length;i++){
        let n=nums[i]
        for(let j=0;j<32;j++){
            count[j]+=n&1
            n=n>>1
        }
    }
    let res
    for(let i=0;i<32;i++){
        res<<=1
        res|=count[31-i]%3
    }
    return res
};
```



##### [剑指 Offer 59 - II. 队列的最大值](https://leetcode.cn/problems/dui-lie-de-zui-da-zhi-lcof/)

```js
var MaxQueue = function() {
    this.queue=[]
    this.maxqueue=[]
};

MaxQueue.prototype.max_value = function() {
    return this.maxqueue.length?this.maxqueue[0]:-1
};

MaxQueue.prototype.push_back = function(value) {
    this.queue.push(value)
    while(this.maxqueue.length&&value>this.maxqueue[this.maxqueue.length-1]){
        this.maxqueue.pop()
    }
    this.maxqueue.push(value)
    return null
};

MaxQueue.prototype.pop_front = function() {
    if(this.queue.length==0){
        return -1
    }
    let value=this.queue.shift()
    if(value==this.maxqueue[0]){
        this.maxqueue.shift()
    }
    return value
};

```



##### [剑指 Offer 61. 扑克牌中的顺子](https://leetcode.cn/problems/bu-ke-pai-zhong-de-shun-zi-lcof/)

```js
var isStraight = function(nums) {
    let set=new Set() //用set看有无重复的元素
    let min=14
    let max=0
    for(let item of nums){
        if(item==0) continue //如果是0直接跳过
        min=Math.min(item,min)
        max=Math.max(item,max)
        if(set.has(item)){
            return false
        }else{
            set.add(item)
        }
    }
    //如果max-min<5说明可以是顺子 不能判断为4 因为存在11 0 0 0 0 的情况
    return max-min<5
};
```



##### [剑指 Offer 62. 圆圈中最后剩下的数字](https://leetcode.cn/problems/yuan-quan-zhong-zui-hou-sheng-xia-de-shu-zi-lcof/)

```js
var lastRemaining = function(n, m) {
    let x=0
    for(let i=2;i<=n;i++){
        x=(x+m)%i
    }
    return x
};//约瑟夫环问题 看题解
```



##### [剑指 Offer 64. 求1+2+…+n](https://leetcode.cn/problems/qiu-12n-lcof/)

```js
var sumNums = function(n) {
    let A=n;
    let B=n+1;
    let ans=0;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    (B&1)&&(ans+=A);
    A<<=1;
    B>>=1;
    return ans>>=1
};
```



##### [剑指 Offer 65. 不用加减乘除做加法](https://leetcode.cn/problems/bu-yong-jia-jian-cheng-chu-zuo-jia-fa-lcof/)

```js
var add = function(a, b) {
    let c
    while(b!=0){
        c=(a&b)<<1 //进位
        a=a^b //非进位和
        b=c //进位
    }
    return a
};
```



##### [剑指 Offer 66. 构建乘积数组](https://leetcode.cn/problems/gou-jian-cheng-ji-shu-zu-lcof/)

```js
//整体思路 两个前缀和
var constructArr = function(a) {
    let left=new Array(a.length).fill(1)
    let right=new Array(a.length).fill(1)
    left[0]=1
    right[a.length-1]=1
    for(let i=1;i<a.length;i++){
        left[i]=left[i-1]*a[i-1]
    }
    for(let i=a.length-2;i>=0;i--){
        right[i]=right[i+1]*a[i+1]    
    }
    let ans=new Array(a.length).fill(1)
    for(let i=0;i<a.length;i++){
        ans[i]=left[i]*right[i]
    }
    return ans
};
```





##### [剑指 Offer 67. 把字符串转换成整数](https://leetcode.cn/problems/ba-zi-fu-chuan-zhuan-huan-cheng-zheng-shu-lcof/)

```js
//待优化
var strToInt = function(str) {
    let started=false
    let flag=true
    let left=0
    let right=0
    str=str.trim()
    for(let i=0;i<str.length;i++){
        let ch=str[i]
        if(started==false&&(ch=='+'||ch=='-')){
            started=true
            flag=ch=='+'?true:false
            left=i+1
            if(Number(str[left])!=str[left]||str[left]==' '){
                return 0
            }
            continue
        }
        if(started==false&&Number(str[left])==str[left]){
            left=i
            started=true
            continue
        }
        if(started==false&&Number(str[left])!=str[left]){
            return 0
        }
        if(started==true&&Number(str[i])==str[i]&&str[i]!=' '){
            right=i
            continue
        }
        if(started==true&&Number(str[i])!=str[i]||str[i]==' '){
            if(str.slice(left,right+1)>=Math.pow(2,31)||str.slice(left,right+1)<Math.pow(-2,31))
                return flag?Math.pow(2,31)-1:'-'+Math.pow(2,31)
            return flag?str.slice(left,right+1):'-'+str.slice(left,right+1)
        }
    }
    if(str.slice(left,right+1)>=Math.pow(2,31)||str.slice(left,right+1)<Math.pow(-2,31))
                return flag?Math.pow(2,31)-1:'-'+Math.pow(2,31)
            return flag?str.slice(left,right+1):'-'+str.slice(left,right+1)
};
```





##### [239. 滑动窗口最大值](https://leetcode.cn/problems/sliding-window-maximum/)

```js
//同算法小抄笔记中的
var maxSlidingWindow = function(nums, k) {
    let ans=[]
    let queue=[]
    for(let i=0;i<nums.length;i++){
        if(i-queue[0]>k-1){
            queue.shift()
        }
        if(queue.length){
            while(nums[i]>nums[queue[queue.length-1]]){
                queue.pop()
            }
        }
        queue.push(i)
        if(i>=k-1){
            ans.push(nums[queue[0]])
        }
    }
    return ans
};
```



##### 

```js
var dicesProbability = function(n) {
    //初始值 1/6的概率
    //从0开始的数组
    let dp1=new Array(6).fill(1/6)
    for(let i=2;i<=n;i++){
        //因为dp1[i]的概率只会影响dp2[i]到dp2[i+5]
        let dp2=new Array(i*5+1).fill(0)
        //外层是dp1的每一个情况的概率
        for(let j=0;j<dp1.length;j++){
            //这个是筛子的长度
            for(let k=0;k<6;k++){
                dp2[j+k]+=dp1[j]/6
            }
        }
        //更新
        dp1=dp2
    }
    return dp1
};
```









##### [正则表达式匹配](https://leetcode-cn.com/problems/regular-expression-matching/)

```js
var isMatch = function(s, p) {
    let dp=new Array(s.length+1).fill(false).map(()=>new Array(p.length+1).fill(false))
    dp[0][0]=true
    for(let i=0;i<=s.length;i++){
        for(let j=1;j<=p.length;j++){
            if(p[j-1]=='*'){
                //看j-2也就是*前第两个位置
                dp[i][j]=dp[i][j-2]
                if(myMatch(s,p,i,j-1)){
                    dp[i][j]=dp[i][j]||dp[i-1][j]
                }
            }else{
                //不是* 则直接看i,j是否匹配
                if(myMatch(s,p,i,j)){
                    dp[i][j]=dp[i-1][j-1]
                }
            }
        }
    }
    return dp[s.length][p.length]
};
function myMatch(s,p,i,j){
    if(i==0){
        return false
    }
    if(p[j-1]=='.'){
        return true
    }
    return s[i-1]==p[j-1]
}
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
        //如果长度为1的数组
        if(i==-1) return nums.sort()
        let cur=nums[i]
        if(cur<pre){
            for(let j=nums.length-1;j>=i;j--){
                if(nums[j]>cur){
                    nums[i]=nums[j]
                    nums[j]=cur
                    mark=i+1
                    break
                }
            }
            break
        }else{
            pre=cur
        }
    }
    //从mark开始排序 这里用的是冒泡 可以用双指针
    for(let i=mark;i<nums.length;i++){
        for(let j=mark;j<nums.length-i+mark-1;j++){
            if(nums[j]>nums[j+1]){
                [nums[j],nums[j+1]]=[nums[j+1],nums[j]]
            }
        }
    }
};
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



###### [数组中的第K个最大元素](https://leetcode-cn.com/problems/kth-largest-element-in-an-array/)

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

