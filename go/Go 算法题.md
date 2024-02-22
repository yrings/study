# Go 算法题

## ACM模式

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



## 二、二叉树

### 迭代遍历

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



