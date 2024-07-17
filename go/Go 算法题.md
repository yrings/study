# Go 算法题

## ACM模式

### 构造二叉树

```
输入: 1 null 2 3
输出: [1,2,3]
```



``` go
// @File : main
// @Author : yrings
// @Time : 2024/7/16
package main

import (
	"bufio"
	"container/list"
	"fmt"
	"os"
	"strconv"
	"strings"
)

type TreeNode struct {
	Val   int
	Right *TreeNode
	Left  *TreeNode
}

func main() {
	in := bufio.NewScanner(os.Stdin)

	for in.Scan() {
		data := strings.Split(in.Text(), " ")
		root := new(TreeNode)
		st := list.New()
		st.PushBack(root)
		index := 1
		val, _ := strconv.Atoi(data[0])
		root.Val = val
		for st.Len() > 0 && index < len(data) {
			node := st.Remove(st.Front()).(*TreeNode)
			if index < len(data) && data[index] != "null" {
				v, _ := strconv.Atoi(data[index])
				node.Left = &TreeNode{Val: v}
				st.PushBack(node.Left)
			}
			index++
			if index < len(data) && data[index] != "null" {
				v, _ := strconv.Atoi(data[index])
				node.Right = &TreeNode{Val: v}
				st.PushBack(node.Right)
			}
			index++
		}
		result := preorderTraversal(root)
		fmt.Println("result:", result)
	}
}

func preorderTraversal(root *TreeNode) []int {
	ans := []int{}

	if root == nil {
		return ans
	}
	st := list.New()
	st.PushBack(root)

	for st.Len() > 0 {
		node := st.Remove(st.Back()).(*TreeNode)
		ans = append(ans, node.Val)
		if node.Right != nil {
			st.PushBack(node.Right)
		}
		if node.Left != nil {
			st.PushBack(node.Left)
		}
	}
	return ans
}

```



* ### 读取数字

  > 要读取一行数字，计算它们的和。给出多行数据。

  #### 知道每行数字数量

  数量固定为二

  ```go
  func main() {
      a := 0
      b := 0
      for {
          n, _ := fmt.Scanln(&a, &b)
          if n == 0 {
              break
          } else {
              Add(a,b)
          }
      }
  }
  func Add(numbers ... int){
      sum := 0
      for _, number := range numbers {
          sum +=  number
      }
      fmt.Println(sum)
  }
  ```

  #### 不知道每行数字数量(整行读取)

  > 输入描述: 输入数据有多组, 每行表示一组输入数据。 每行不定有n个整数，空格隔开。(1 <= n <= 100)。 输出描述: 
  >
  > 每组数据输出求和的结果 
  >
  > 输入例子1: 
  >
  > 1 2 3 
  >
  > 4 5 
  >
  > 0 0 0 0 0 
  >
  > 输出例子1: 
  >
  > 6 
  >
  > 9 
  >
  > 0

  ```go
  func main() {
  	in := bufio.NewScanner(os.Stdin)
  	for in.Scan() {
  		// 通过空格将它们分开分割并存入返回一个切片
  		data := strings.Split(in.Text(), " ")
  		AddBySlice(data)
  	}
  }
  func AddBySlice(numbers []string) {
  	sum := 0
  	for _, number := range numbers {
  		val, _ := strconv.Atoi(number)
  		sum += val
  	}
  	fmt.Println(sum)
  }
  ```

* ### 读取字符串

  > 输入描述:
  > 多个测试用例，每个测试用例一行。
  >
  > 每行通过空格隔开，有n个字符，n＜100
  >
  > 输出描述:
  > 对于每组测试用例，输出一行排序过的字符串，每个字符串通过空格隔开
  >
  > 输入例子1:
  > a c bb
  > f dddd
  > nowcoder
  >
  > 输出例子1:
  > a bb c
  > dddd f
  > nowcoder

  读取多行字符串

  ```go
  func main() {
      in := bufio.NewScanner(os.Stdin)
      for in.Scan() {
          data := strings.Split(in.Text(), " ")
          // 对切片进行排序
          sort.Strings(data)
          // 将切片连接成字符串
          fmt.Println(strings.Join(data, " "))
      }
  }
  ```

  



## 一、数组

### 二分法

```go
func search(nums []int, target int) int {
    left := 0
    right := len(nums) - 1
    index := (right - left) / 2
    for left <= right{
        index = (right + left) / 2 
        if nums[index] == target {
            return index
        } else if nums[index] > target{
            right = index - 1
        } else {
            left = index + 1
        }
    }
     return -1
}
```



### 移除元素

> 给你一个数组 `nums` 和一个值 `val`，你需要 **[原地](https://baike.baidu.com/item/原地算法)** 移除所有数值等于 `val` 的元素，并返回移除后数组的新长度。
>
> 不要使用额外的数组空间，你必须仅使用 `O(1)` 额外空间并 **[原地 ](https://baike.baidu.com/item/原地算法)修改输入数组**。
>
> 元素的顺序可以改变。你不需要考虑数组中超出新长度后面的元素

* #### 思路

  * solution 1:两个for循环遍历

    时间复杂度 O（n ^ 2）

    空间复杂度O（1）

  * 双指针

    双指针法（快慢指针法）： **通过一个快指针和慢指针在一个for循环下完成两个for循环的工作。**

    定义快慢指针

    - 快指针：寻找新数组的元素 ，新数组就是不含有目标元素的数组
    - 慢指针：指向更新 新数组下标的位置

    时间复杂度：O(n)

    空间复杂度：O(1)

```go
func removeElement(nums []int, val int) int {
    fastIndex := 0
    slowIndex := 0
    for ; fastIndex < len(nums); fastIndex++ {
        if nums[fastIndex] != val {
            nums[slowIndex] = nums[fastIndex]
            slowIndex++
        }
    }
    return slowIndex
}
```

第二种方法：

```go
func removeElement(nums []int, val int) int {
    left := 0
    right := len(nums)
    for left < right{
        if nums[left] == val{
                nums[left] = nums[right-1]
                right--

        }else{
            left++
        }
    }
    return left
}
```

## 二、链表

### 206. 反转链表

![](image\206.翻转链表.gif)

```go
func reverseList(head *ListNode) *ListNode {
    var pre *ListNode
    cur := head
    for cur != nil{
        temp := cur.Next
        cur.Next = pre
        pre = cur
        cur = temp
    }
    return pre
}
```

### 24. 两两交换链表中的节点

```go
func swapPairs(head *ListNode) *ListNode {
    if head == nil || head.Next == nil {
        return head
    }
    var pre *ListNode
    save := head.Next
    cur := head
    next := head.Next
    for {
        if pre != nil {
            pre.Next = next
        }
        cur.Next = next.Next
        next.Next = cur
        if cur.Next == nil || cur.Next.Next == nil {
            break
        }
        pre = cur
        cur = cur.Next
        next = cur.Next
    }
    return save
}
```



## 三、二叉树

### 后序遍历（迭代法）

再来看后序遍历，先序遍历是中左右，后续遍历是左右中，那么我们只需要调整一下先序遍历的代码顺序，就变成中右左的遍历顺序，然后在反转result数组，输出的结果顺序就是左右中了，如下图：

### 迭代遍历

前序遍历

```go
type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

func preorderTraversal(root *TreeNode) []int {
	ans := []int{}

	if root == nil {
		return ans
	}

	st := list.New()
	st.PushBack(root)

	for st.Len() > 0 {
		node := st.Remove(st.Back()).(*TreeNode)
		
		ans = append(ans, node.Val)
		if node.Right != nil {
			st.PushBack(node.Right)
		} 
		if node.Left  != nil {
			st.PushBack(node.Left)
		}
	}
	return ans
}
```

中序遍历

```go
```



### 根据中序遍历和后序遍历得到二叉树

 思路可有以下优化点

* 哈希表存储中序`inorder[]`数值对应下标，避免暴力遍历找

* 递归过程中参数避免传数组，用`left、rightleft、rightleft、right`下标去代替



```go
/**
 * Definition for a binary tree node.
 * type TreeNode struct {
 *     Val int
 *     Left *TreeNode
 *     Right *TreeNode
 * }
 */
func buildTree(inorder []int, postorder []int) *TreeNode {
    var f func(ins , pos , len int) *TreeNode
    // 取头去尾
    f = func (ins , pos , len int) *TreeNode{
        if len == 0 {
            return nil
        }
        tree := &TreeNode{}
        tree.Val = postorder[pos + len - 1]
        if len == 1 {
            return tree
        }
        i := 0
        for i < len && tree.Val != inorder[i + ins] {
            i++
        }
        tree.Left = f(ins, pos, i)
        tree.Right = f(ins + i + 1, pos + i, len - 1 - i)
        return tree
    }
    return f(0,0,len(inorder))
}
```



### 根据前后序遍历构造二叉树

```go
 // 后序遍历的倒序的第二个一定是根节点的右节点吗？
 // 通过与前序遍历的第二个数进行比较，不相等的话一定有右子树; 
 // 相等的话只有一个子树，左右都有可能。这时是分辨不出来的，可以忽略
 // 建立一个hash表来存储前序中值的索引，快速定位，避免暴力遍历

func constructFromPrePost(preorder []int, postorder []int) *TreeNode {
	var recurtion func(preS, preE, posS, posE int) *TreeNode
	indexMap := make(map[int]int)
	for i := 0; i < len(preorder); i++ {
		indexMap[preorder[i]] = i
	}
	fmt.Println(indexMap)
	// 取头取尾
	recurtion = func(preS, preE, posS, posE int) *TreeNode {
		nodeLen := preE - preS + 1
		node := &TreeNode{}
		node.Val = preorder[preS]
		if nodeLen <= 0 {
			return nil
		}
		if nodeLen == 1 {
			return node
		}
		if postorder[posE-1] == preorder[preS+1] {
			node.Left = recurtion(preS+1, preE, posS, posE-1)
		} else {
			index := indexMap[postorder[posE-1]]
			leftEndIndex := index - 1 - (preS + 1) + posS
			rightStartIndex := posE - 1 + index - preE

			node.Left = recurtion(preS+1, index-1, posS, leftEndIndex)
			node.Right = recurtion(index, preE, rightStartIndex, posE-1)
		}
		return node
	}
	res := recurtion(0, len(preorder)-1, 0, len(preorder)-1)
	return res
}
```

### 2476. 二叉搜索树最近节点查询

> 迭代中序遍历 + map索引

```go
func closestNodes(root *TreeNode, queries []int) [][]int {
    result := make([][]int, len(queries))
    indexMap := make(map[int][]int,0)
    
    for i := 0; i < len(queries); i++ {
        _, ok := indexMap[queries[i]] 
        if ok {
            indexMap[queries[i]] = append(indexMap[queries[i]], i)
        } else {
            indexMap[queries[i]] = make([]int, 0)
            indexMap[queries[i]] = append(indexMap[queries[i]], i)
        }
        result[i] = make([]int, 2)
        result[i][0] = -1
        result[i][1] = -1
    }
    sort.Ints(queries)
    st := list.New()
    cur := root
    index := 0
    lastVal := -1
    
    for cur != nil || st.Len() > 0 {
        if index == len(queries) {
            return result
        }
        if cur != nil {
            st.PushBack(cur)
            cur = cur.Left
        } else {
            cur = st.Remove(st.Back()).(*TreeNode)
            for index < len(queries){
                if queries[index] == cur.Val {
                    for r := 0; r < len(indexMap[queries[index]]); r++ {
                        i := indexMap[queries[index]][r]
                        result[i][1] = cur.Val
                        result[i][0] = cur.Val
                    }
                    index += len(indexMap[queries[index]])
                } else if queries[index] < cur.Val {
                    for r := 0; r < len(indexMap[queries[index]]); r++ {
                        i := indexMap[queries[index]][r]
                        result[i][1] = cur.Val
                        result[i][0] = lastVal
                    }
                    index += len(indexMap[queries[index]])
                } else {
                    break
                }
            }
            lastVal = cur.Val
            cur = cur.Right
        }
    }
    if index != len(queries) {
        for j := index; j < len(queries);j++ {
            k := indexMap[queries[j]][0]
            result[k][0] = lastVal
        }
    }
    return result
}
```

### 235. 二叉搜索树的最近公共祖先

给定一个二叉搜索树, 找到该树中两个指定节点的最近公共祖先。

[百度百科](https://baike.baidu.com/item/最近公共祖先/8918834?fr=aladdin)中最近公共祖先的定义为：“对于有根树 T 的两个结点 p、q，最近公共祖先表示为一个结点 x，满足 x 是 p、q 的祖先且 x 的深度尽可能大（**一个节点也可以是它自己的祖先**）。”

* #### 思路

  由于二叉搜索树是有序的，所以只要找到一个节点的值在这两个节点数值的中间，就可以保证这个节点一定是**最近公共祖先**。

```go
func lowestCommonAncestor(root, p, q *TreeNode) *TreeNode {
    st := list.New()
    st.PushBack(root)
    if p.Val > q.Val {
        p,q = q,p
    }
    node := &TreeNode{}


    for st.Len() != 0 {
        node = st.Remove(st.Back()).(*TreeNode)
        if (node.Val > p.Val && node.Val < q.Val) || node.Val == p.Val || node.Val == q.Val {
                break
        } else if node.Val < p.Val{
            if node.Right != nil {
                st.PushBack(node.Right)
            }
        } else if node.Val > q.Val{
            if node.Left != nil {
                st.PushBack(node.Left)
            }
        }
    }
    return node
}
```



### 2673. 使二叉树所有路径值相等的最小代价

> 给你一个整数 `n` 表示一棵 **满二叉树** 里面节点的数目，节点编号从 `1` 到 `n` 。根节点编号为 `1` ，树中每个非叶子节点 `i` 都有两个孩子，分别是左孩子 `2 * i` 和右孩子 `2 * i + 1` 。
>
> 树中每个节点都有一个值，用下标从 **0** 开始、长度为 `n` 的整数数组 `cost` 表示，其中 `cost[i]` 是第 `i + 1` 个节点的值。每次操作，你可以将树中 **任意** 节点的值 **增加** `1` 。你可以执行操作 **任意** 次。
>
> 你的目标是让根到每一个 **叶子结点** 的路径值相等。请你W返回 **最少** 需要执行增加操作多少次。
>
> **注意：**
>
> - **满二叉树** 指的是一棵树，它满足树中除了叶子节点外每个节点都恰好有 2 个子节点，且所有叶子节点距离根节点距离相同。
> - **路径值** 指的是路径上所有节点的值之和。

* ### 思路

  对于任一叶结点，它的值为 xxx，它的兄弟节点的值为 yyy。可以发现，对于树上的其余节点，它们要么同时是这两个叶节点的祖先，要么同时不是这两个叶节点的祖先。对这些节点进行一次操作，要么同时增加了根到这两个叶节点的路径值 111，要么没有任何效果。因此，要想使得根到这两个叶节点的路径值相等，我们只能增加 xxx 和 yyy 本身。

  ```go
  func minIncrements(n int, cost []int) (ans int) {
  	for i := n / 2; i > 0; i-- { // 从最后一个非叶节点开始算
  		left, right := cost[i*2-1], cost[i*2]
  		if left > right { // 保证 left <= right
  			left, right = right, left
  		}
  		ans += right - left // 两个子节点变成一样的
  		cost[i-1] += right // 累加路径和
  	}
  	return
  }
  ```




### 49.字母异位词分组

> 给你一个字符串数组，请你将 **字母异位词** 组合在一起。可以按任意顺序返回结果列表。
>
> **字母异位词** 是由重新排列源单词的所有字母得到的一个新单词。

* **思路**

  通过桶排序，创建一个map，以`[26]int`数组为key，然后遍历所有字符串，统计每个字符出现的次数，如果两个字符串能够满足条件，他们的key就会相同，就添加到对于key的后边就可以满足进行分组了。

```go
func groupAnagrams(strs []string) [][]string {
    m := make(map[[26]int][]string)
    for _, v := range strs {
        cnt := [26]int{}
        for _, c := range v {
            cnt[c-'a']++
        }
        m[cnt] = append(m[cnt], v)
    }
    ans := make([][]string, 0, len(strs))
    for _, v := range m {
        ans = append(ans, v)
    }
    return ans
}
```



### 2369. 检查数组是否存在有效划分

```go
func validPartition(nums []int) bool {
    n := len(nums)
    // dp[i] 表示前i个节点是否至少存在一个有效划分
    dp := make([]bool, n + 1)
    dp[0] = true

    for i := 1; i <= n; i++ {
        if i >= 2 {
            dp[i] = dp[i - 2] && validTwo(nums[i - 1], nums[i - 2])
        }
        if i >= 3 {
            dp[i] = dp[i] || (dp[i - 3] && validThree(nums[i - 3], nums[i - 2], nums[i - 1]))
        }
    }
    return dp[n]
}

func validTwo(num1, num2 int) bool{
    return num1 == num2
}

func validThree(num1, num2, num3 int) bool{
    return (num1 == num2 && num1 == num3) || (num1 + 1 == num2 && num2 + 1 == num3)
}
```



### 2368. 受限条件下可以到达节点的数目

**方法一：深度优先搜索**

```go
func reachableNodes(n int, edges [][]int, restricted []int) int {
    isRestricted := make([]int, n)
    for _, x := range restricted {
        isRestricted[x] = 1
    }
    // 创建一个二维数组，存储每个节点可达的节点
    g := make([][]int, n)
    for _, v := range edges {
        g[v[0]] = append(g[v[0]], v[1])
        g[v[1]] = append(g[v[1]], v[0])
    }

    cnt := 0
    var dfs func(int, int)
    dfs = func(x, f int) {
        cnt++
        for _, y := range g[x] {
            if y != f && isRestricted[y] != 1 {
                dfs(y, x)
            }
        }
    }
    dfs(0, -1)
    return cnt
}
```

**方法二：并查集**

忽略限制节点，将树变成若干个连通块，然后计算的就是0号所在的连通块。



### 225. 用队列实现栈

```go
// 队列1用于缓存当前栈中元素，队列2用于添加元素的载体
type MyStack struct {
    queue1, queue2 []int
}

/** Initialize your data structure here. */
func Constructor() (s MyStack) {
    return
}

/** Push element x onto stack. */
func (s *MyStack) Push(x int) {
    s.queue2 = append(s.queue2, x)
    for len(s.queue1) > 0 {
        s.queue2 = append(s.queue2, s.queue1[0])
        s.queue1 = s.queue1[1:]
    }
    s.queue1, s.queue2 = s.queue2, s.queue1
}

/** Removes the element on top of the stack and returns that element. */
func (s *MyStack) Pop() int {
    v := s.queue1[0]
    s.queue1 = s.queue1[1:]
    return v
}

/** Get the top element. */
func (s *MyStack) Top() int {
    return s.queue1[0]
}

/** Returns whether the stack is empty. */
func (s *MyStack) Empty() bool {
    return len(s.queue1) == 0
}
```



### 2386. 找出数组的第 K 大和

> 首先，我们找到最大的子序和 mxmxmx，即所有正数之和。
>
> 可以发现，其他子序列的和，都可以看成在这个最大子序列和之上，减去其他部分子序列之和得到。因此，我们可以将问题转换为求第 kkk 小的子序列和。
>
> 只需要将所有数的绝对值升序排列，之后建立小根堆，存储二元组 (s,i)(s, i)(s,i)，表示当前和为 sss，且下一个待选择的数字的下标为 iii 的子序列。
>
> 每次取出堆顶，并放入两种新情况：一是再选择下一位，二是选择下一位并且不选择本位。
>
> 由于数组是从小到大排序，这种方式能够不重不漏地按序遍历完所有的子序列和。

```go
func kSum(nums []int, k int) int64 {
	mx := 0
	for i, x := range nums {
		if x > 0 {
			mx += x
		} else {
			nums[i] *= -1
		}
	}
	sort.Ints(nums)
	h := &hp{{0, 0}}
	for k > 1 {
		k--
		p := heap.Pop(h).(pair)
		if p.i < len(nums) {
			heap.Push(h, pair{p.sum + nums[p.i], p.i + 1})
			if p.i > 0 {
				heap.Push(h, pair{p.sum + nums[p.i] - nums[p.i-1], p.i + 1})
			}
		}
	}
	return int64(mx) - int64((*h)[0].sum)
}

type pair struct{ sum, i int }
type hp []pair

func (h hp) Len() int           { return len(h) }
func (h hp) Less(i, j int) bool { return h[i].sum < h[j].sum }
func (h hp) Swap(i, j int)      { h[i], h[j] = h[j], h[i] }
func (h *hp) Push(v any)        { *h = append(*h, v.(pair)) }
func (h *hp) Pop() any          { a := *h; v := a[len(a)-1]; *h = a[:len(a)-1]; return v }
```



### 189. 轮转数组

```go
func reverse(a []int) {
    for i, n := 0, len(a); i < n/2; i++ {
        a[i], a[n-1-i] = a[n-1-i], a[i]
    }
}
func rotate(nums []int, k int) {
    k %= len(nums)
    reverse(nums)
    reverse(nums[:k])
    reverse(nums[k:])
}
```



### 2313. 买木头块

> ### 动态规划/记忆化搜索

给你两个整数 `m` 和 `n` ，分别表示一块矩形木块的高和宽。同时给你一个二维整数数组 `prices` ，其中 `prices[i] = [hi, wi, pricei]` 表示你可以以 `pricei` 元的价格卖一块高为 `hi` 宽为 `wi` 的矩形木块。

每一次操作中，你必须按下述方式之一执行切割操作，以得到两块更小的矩形木块：

- 沿垂直方向按高度 **完全** 切割木块，或
- 沿水平方向按宽度 **完全** 切割木块

在将一块木块切成若干小木块后，你可以根据 `prices` 卖木块。你可以卖多块同样尺寸的木块。你不需要将所有小木块都卖出去。你 **不能** 旋转切好后木块的高和宽。

请你返回切割一块大小为 `m x n` 的木块后，能得到的 **最多** 钱数。

注意你可以切割木块任意次。

```go
func sellingWood(m int, n int, prices [][]int) int64 {
    value := make(map[[2]int]int, 0)
    memo := make([][]int64, m + 1)
    for i := range memo {
        memo[i] = make([]int64, n+1)
        for j := range memo[i] {
            memo[i][j] = -1
        }
    }

    var dfs func(int, int) int64
    dfs = func(x, y int) int64 {
        if memo[x][y] != -1 {
            return memo[x][y]
        }

        var ret int64
        if val, ok := value[[2]int{x, y}]; ok {
            ret = int64(val)
        }
        if x > 1 {
            for i := 1; i < x; i++ {
                ret = max(ret, dfs(i, y) + dfs(x - i, y))
            }
        }
        if y > 1 {
            for j := 1; j < y; j++ {
                ret = max(ret, dfs(x, j) + dfs(x, y - j))
            }
        }
        memo[x][y] = ret
        return ret
    }

    for _, price := range prices {
        value[[2]int{price[0], price[1]}] = price[2]
    }
    return dfs(m, n)
}
```





### 406.根据身高重建队列

> 假设有打乱顺序的一群人站成一个队列，数组 `people` 表示队列中一些人的属性（不一定按顺序）。每个 `people[i] = [hi, ki]` 表示第 `i` 个人的身高为 `hi` ，前面 **正好** 有 `ki` 个身高大于或等于 `hi` 的人。
>
> 请你重新构造并返回输入数组 `people` 所表示的队列。返回的队列应该格式化为数组 `queue` ，其中 `queue[j] = [hj, kj]` 是队列中第 `j` 个人的属性（`queue[0]` 是排在队列前面的人）。

按照身高排序之后，优先按身高高的people的k来插入，后序插入节点也不会影响前面已经插入的节点，最终按照k的规则完成了队列。

```go
func reconstructQueue(people [][]int) [][]int {
    // 先将身高从大到小排序，确定最大个子的相对位置
    sort.Slice(people, func(i, j int) bool {
        if people[i][0] == people[j][0] {
            return people[i][1] < people[j][1]   // 当身高相同时，将K按照从小到大排序
        }
        return people[i][0] > people[j][0]     // 身高按照由大到小的顺序来排
    })

    // 再按照K进行插入排序，优先插入K小的
	for i, p := range people {
		copy(people[p[1]+1 : i+1], people[p[1] : i+1])  // 空出一个位置
		people[p[1]] = p
	}
	return people
}
```



### 343.整数拆分

> 给定一个正整数 n，将其拆分为至少两个正整数的和，并使这些整数的乘积最大化。 返回你可以获得的最大乘积

```go
func integerBreak(n int) int {
    dp := make([]int, n + 1)
    dp[0] = 0
    dp[1] = 1
    dp[2] = 1
    for i := 3; i <= n; i++ {
        for j := 2; j < i; j++ {
            num1 := i / j
            num2 := i - num1
            dp[i] = max(dp[i], max(dp[num1], num1 )* max(dp[num2], num2))
        }
    }
    return dp[n]

}
```



### 435. 无重叠区间

![](.\image-al\20230201164134.png)

**我来按照右边界排序，从左向右记录非交叉区间的个数。最后用区间总数减去非交叉区间的个数就是需要移除的区间个数了**。

区间4结束之后，再找到区间6，所以一共记录非交叉区间的个数是三个。

总共区间个数为6，减去非交叉区间的个数3。移除区间的最小数量就是3。

```go
func eraseOverlapIntervals(intervals [][]int) int {
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][1] < intervals[j][1]
    })
    res := 1
    for i := 1; i < len(intervals); i++ {
        if intervals[i][0] >= intervals[i-1][1] {
            res++
        } else {
            intervals[i][1] = min(intervals[i - 1][1], intervals[i][1])
        }
    }
    return len(intervals) - res
}
```



### 96. 不同的二叉搜索树

![](.\image-al\20210107093226241.png)

首先一定是遍历节点数，从递归公式：dp[i] += dp[j - 1] * dp[i - j]可以看出，节点数为i的状态是依靠 i之前节点数的状态。

```go
func numTrees(n int) int {
    if n <= 2 {
        return n
    }
    dp := make([]int, n + 1)
    dp[0] = 1
    dp[1] = 1
    dp[2] = 2
    for i := 3; i <= n; i++ {
        for j := 1; j <= i; j++ {
            dp[i] += dp[j - 1] * dp[i - j]
        }
    }
    return dp[n]
}
```



### 763. 划分字母区间

第一步：记录所有字母最后出现的索引

第二步：遍历字符串，维护片段最大索引

```go
func partitionLabels(s string) []int {
    res := make([]int, 0)
    // 记录所有字母出现的最后索引
    m := make(map[string]int)
    for i := 0; i < len(s); i++ {
        m[string(s[i])] = i
    }
    
    for index := 0; index < len(s);{
        lastIndex := index
        for index <= lastIndex {
            lastIndex = max(lastIndex, m[string(s[index])])
            index++
        }
        length := lastIndex
        for j := 0; j < len(res); j++ {
            length -= res[j]
        }
        res = append(res, length + 1)
    }
    return res
}
```



### 背包问题

![](.\image-al\20210117171307407.png)



```go
package main

import "fmt"

func test_wei_bag_problem1(weight, value []int, bagWeight int) int {
	if bagWeight <= 0 {
		return 0
	}
	dp := make([][]int, len(weight))
	for i := 0; i < len(weight); i++ {
		dp[i] = make([]int, bagWeight+1)
	}
	// 初始化
	for j := bagWeight; j >= weight[0]; j-- {
		dp[0][j] = dp[0][j-weight[0]] + value[0]
	}
	// 递推公式
	for j := 0; j <= bagWeight; j++ {
		for i := 1; i < len(weight); i++ {
			if weight[i] > j {
				dp[i][j] = dp[i-1][j]
			} else {
				dp[i][j] = max(dp[i-1][j], dp[i-1][j-weight[i]]+value[i])
			}
		}
	}
	return dp[len(weight)-1][bagWeight]
}

func max(a, b int) int {
    if a > b {
        return a
    } else {
        return b
    }
}

func main() {
	var M, N int
	_, err := fmt.Scanln(&M, &N)
	if err != nil {
		return
	}
	weight := make([]int, M)
	for i := 0; i < M; i++ {
		if _, er := fmt.Scan(&weight[i]); er != nil {
			return
		}
	}
	value := make([]int, M)
	for i := 0; i < M; i++ {
		if _, er := fmt.Scan(&value[i]); er != nil {
			return
		}
	}
	//fmt.Printf("M: %v, N: %v, weight:%v, value: %v\n", M, N, weight, value)
	maxValue := test_wei_bag_problem1(weight, value, N)
	fmt.Println(maxValue)
}
```





### 738. 单调递增数字

贪心

```go
func monotoneIncreasingDigits(n int) int {
    arr := make([]int, 0)
    for n > 0 {
        y := n % 10
        n = n / 10
        if y == 0 {
            y = 9
            for i := 0; i < len(arr); i++ {
                arr[i] = 9 
            }
            n -= 1
        }
        if len(arr) > 0 && y > arr[len(arr) - 1] {
            y -= 1
            for i := 0; i < len(arr); i++ {
                arr[i] = 9 
            }
            if y == 0 {
                n -= 1
                y = 9
            }
        }
        arr = append(arr, y)
    }
    res := 0
    for i := 0; i < len(arr); i++ {
        t := int(math.Pow(10, float64(i)))
        res += arr[i] * t
    }
    return res
}
```





### 416. 分割等和子集

01背包问题，一维滚动数组

```go
func canPartition(nums []int) bool {
    target := 0
    for i := 0; i < len(nums); i++ {
        target += nums[i]
    }
    if target % 2 == 1 {
        return false
    } else {
        target = target / 2
    }
    dp := make([]int, target + 1)

    for i := 0; i < len(nums); i++ {
        for j := target; j >= nums[i]; j-- {
            dp[j] = max(dp[j], dp[j - nums[i]] + nums[i])
        }
    }
    return dp[target] == target
}
```



### 968. 监控二叉树



