---
title: leet-code之旅
tags:
  - 刷题
---

# 正式开启刷leet-code

### 第225.用两个队列实现栈

问题：

请你仅使用两个队列实现一个后入先出（LIFO）的栈，并支持普通栈的全部四种操作（push、top、pop 和 empty）。

实现 MyStack 类：

void push(int x) 将元素 x 压入栈顶。
int pop() 移除并返回栈顶元素。
int top() 返回栈顶元素。
boolean empty() 如果栈是空的，返回 true ；否则，返回 false 。


#### 我的代码：
```
type MyStack struct {
    list1 := list.New()
    list2 := list.New()
}


func Constructor() MyStack {
    MyStack.Push(x)
    MyStack.Push(x)
    MyStack.Push(x)
    MyStack.Pop()
    MyStack.Top()
    MyStack.empty()
}


func (this *MyStack) Push(x int)  {
    this.list1.PushBack(x)
    temp := x
    this.list2.PushFront(temp)
}


func (this *MyStack) Pop() int {
   temp = this.list2.Value
   e = e.Next
}


func (this *MyStack) Top() int {
    temp = this.list2.Value
    return temp
}


func (this *MyStack) Empty() bool {
    if this.list2.Value == nil
    return false
}


/**
 * Your MyStack object will be instantiated and called as such:
 * obj := Constructor();
 * obj.Push(x);
 * param_2 := obj.Pop();
 * param_3 := obj.Top();
 * param_4 := obj.Empty();
 */
 ```


#### 虚伪的正确的代码：
```
package main

import (
"container/list"
)

type MyStack struct {
list1 *list.List
list2 *list.List
}


func Constructor() MyStack {
return MyStack {
list1: list.New(),
list2: list.New(),
}
}


func (this *MyStack) Push(x int)  {
this.list1.PushBack(x)
temp := x
this.list2.PushFront(temp)
}


func (this *MyStack) Pop() int {
if this.list2.Len() == 0 {
return -1 // 如果stack为空返回-1或者其他适当的值
}
temp := this.list2.Front()
this.list2.Remove(temp)
return temp.Value.(int)
}


func (this *MyStack) Top() int {
if this.list2.Len() == 0 {
return -1 // 如果栈为空则返回-1或者其他适当的值
}
temp := this.list2.Front().Value(int)
return temp
}


func (this *MyStack) Empty() bool {
return this.list2.Len() == 0
}


/**
* Your MyStack object will be instantiated and called as such:
* obj := Constructor();
* obj.Push(x);
* param_2 := obj.Pop();
* param_3 := obj.Top();
* param_4 := obj.Empty();
  */
```

使用go语言内置的list包 也就是双向链表的操作
运用到了go语言的断言 `this.list2.Front().Value.(int)`

但是 虽然通过了 但是思路应该是错了哈哈哈哈 ！ 这段代码并没有用到队列的性质而是双链表。。。。并没有遵守题目的规则
代码问题：

而只是使用了两个链表，其中 list2 实际上扮演了“栈”的角色。这样的话，list2 单独一个链表就能实现后进先出，不需要 list1 的辅助。

#### 真正的正确代码

```
package main

import "container/list"

type MyStack struct {
	queue1 *list.List
	queue2 *list.List
}

func Constructor() MyStack {
	return MyStack{
		queue1: list.New(),
		queue2: list.New(),
	}
}

func (this *MyStack) Push(x int) {
	this.queue1.PushBack(x)
}

func (this *MyStack) Pop() int {
	if this.queue1.Len() == 0 {
		return -1
	}

	for this.queue1.Len() > 1 {
		front := this.queue1.Front()
		this.queue1.Remove(front)
		this.queue2.PushBack(front.Value)
	}

	top := this.queue1.Front()
	this.queue1.Remove(top)

	this.queue1, this.queue2 = this.queue2, this.queue1
	return top.Value.(int)
}

func (this *MyStack) Top() int {
	if this.queue1.Len() == 0 {
		return -1
	}

	for this.queue1.Len() > 1 {
		front := this.queue1.Front()
		this.queue1.Remove(front)
		this.queue2.PushBack(front.Value)
	}

	top := this.queue1.Front()
	this.queue2.PushBack(top.Value)

	this.queue1, this.queue2 = this.queue2, this.queue1
	return top.Value.(int)
}

func (this *MyStack) Empty() bool {
	return this.queue1.Len() == 0
}

```
---

### 第20题有效括号

问题：

给定一个只包括 '('，')'，'{'，'}'，'['，']' 的字符串 s ，判断字符串是否有效。

有效字符串需满足：

左括号必须用相同类型的右括号闭合。
左括号必须以正确的顺序闭合。
每个右括号都有一个对应的相同类型的左括号。

#### 解答过程
```

func isValid(s string) bool {
    // 创建一个栈来存储左括号
    stack := make([]rune, 0)
    // 括号的对应关系
    pairs := map[rune]rune{
        ')': '(',
        '}': '{',
        ']': '[',
    }
    // 遍历字符串s
    for _, char := range s {
        // 如果是右括号
        if _, exists := pairs[char]; exists {
            // 取出栈顶元素，如果栈为空或者不匹配则返回false
            if len(stack) == 0 || stack[len(stack)-1] != pairs[char] {
                return false
            }
            // 否则弹出栈顶元素
            stack = stack[:len(stack)-1]
        } else {
            // 如果是左括号，压入栈中
            stack = append(stack, char)
        }
    }
    // 如果栈为空，则所有括号正确匹配
    return len(stack) == 0
}


```
栈的问题

---
### 字节刷题（队列）相关：
问题：

小R的随机播放顺序
问题描述
小有一个特殊的随机播放规则。他首先播放歌单中的第一首歌，播放后将其从歌
单中移除。如果歌单中还有歌曲，则会将当前第一首歌移到最后一首。这个过程会
一直重复，直到歌单中没有任何歌曲。
例如，给定歌单[5,3,2,1,4]，真实的播放顺序是[5,2,4,1,3]。
保证歌曲中的id两两不同。
测试样例
样例1：
输入：n=5,a=[5,3,2,1,4]输出：[5,2,4,1,3]
样2：
输入：n=4,a=[4,1,3,2]输出：[4,3,1,2]
样3：
输入：n=6,a=[1,2,3,4,5,6]输出：[1,3,5,2,6,4]

看完题之后的思路 ： 就是队列问题 如何操作栈 在文中也就是实现： 先出栈-》再执行出栈入栈-》再出栈 这是目前简单的思路
然后搜索Go语言队列的相关操作 用slice实现队列：或者 list实现队列

#### 题解：
```
package main

import "fmt"

func solution(n int, a []int) []int {
    result := []int{}  // 用于存储播放顺序

    for len(a) > 0 {
        // 播放第一首歌并加入到结果中
        result = append(result, a[0])
        // 移除播放的歌曲
        a = a[1:]
        
        // 如果歌单还有剩余，将当前第一首歌移到最后
        if len(a) > 0 {
            a = append(a[1:], a[0])
        }
    }

    return result
}

func main() {
    fmt.Println(fmt.Sprintf("%v", solution(5, []int{5, 3, 2, 1, 4})) == fmt.Sprintf("%v", []int{5, 2, 4, 1, 3}))
    fmt.Println(fmt.Sprintf("%v", solution(4, []int{4, 1, 3, 2})) == fmt.Sprintf("%v", []int{4, 3, 1, 2}))
    fmt.Println(fmt.Sprintf("%v", solution(6, []int{1, 2, 3, 4, 5, 6})) == fmt.Sprintf("%v", []int{1, 3, 5, 2, 6, 4}))
}

```
看到题解我蒙了：我在思考的时候确实想如何实现这个队列并且如何实现添加元素并且删除元素  

但是 在这道题中，Go 切片的特性其实已经很好地帮你实现了这些操作。切片的 append 和切片操作符 a[1:] 直接完成了队列的“入队”和“出队”需求。只是在简化的问题上，利用 Go 自带的切片操作让解决方案更简洁、易懂。

---

#### 12.2 日
##### 重刷两数之和
###### 我的代码


```go
func twoSum(nums []int, target int) []int {
    k := make(map[int]int)
    for i, v := range nums {
        other := target - v
        if _ , ok := map[other]; ok {
            return {i, map[nums[i]]}
        }
        m[nums[i]] = i 
    }
    return nil
}
```

###### 更正后：

```go
func twoSum(nums []int, target int) []int {
    k := make(map[int]int)
    for i, v := range nums {
        other := target - v
        if idx , ok := k[other]; ok {
            return []int{i, idx}
        }
        k[v] = i 
    }
    return nil
}
```
---

##### 删除有序数组中的重复项

###### 错误的代码
```go
func removeDuplicates(nums []int) int {
    
    k := make([]int,10)
    for i:=0;i<len(nums);i++ {
        for j:=i; j<len(nums); j++{
            if nums[i] == nums[j] {
                nums[i] = nums[j]
                continue
            } 
                k[i] = nums[i]
                
                break
        } 
    }
    return len(k)
}

```

题目要求：
题目需要在原地删除重复项，使用 O(1) 额外空间，返回去重后数组的长度。去重后的元素必须保留在数组的前部，后续位置可以随意。
代码中存在的问题：

额外空间不符合要求：你使用了 k 数组存储去重后的结果，违背了 O(1) 额外空间的限制。
逻辑问题：双重循环实际上是在尝试复制和比较元素，但逻辑上没有保证正确的去重。
continue 和 break 的混乱：循环控制没有明确目的，导致部分逻辑跳过了关键步骤。
返回值错误：你返回的是辅助数组 k 的长度，而不是修改后数组 nums 的有效长度。

###### 正确解法：

```go
func removeDuplicates(nums []int) int {
    if len(nums) == 0{
        return 0
    }
    var k = 0
    for i :=1; i<len(nums); i++ {
        if nums[k] != nums [i] {
            k++
            nums[k] = nums[i]
        }
    }
    return k+1

  
}
```
---

##### 27 移除元素

###### 错误代码
```go
func removeElement(nums []int, val int) int {
    var length = 0
    for i := length; i<len(nums); i++ {
        if val == nums[length] {
            var k = 0
        k = nums[length]
        length ++
        }
    
    }
        return length
}

```
不出意外又是错误的代码
k 被赋值：
k = nums[length]
然而，k 在后续的代码中并没有被用于任何地方，它没有被打印、返回、存储或进一步处理。只是简单地在 if 语句内被赋值。

k 的作用无效： k 是在 if 语句中声明并赋值的，但它没有发挥任何作用。Go 编译器会在编译时检查到这一点，并给出警告，因为声明了一个变量却没有用到。

###### 正确代码：
```go

func removeElement(nums []int, val int) int {
    var length = 0
    for i := 0; i < len(nums); i++ {
        if nums[i] != val {
            nums[length] = nums[i]
            length++
        }
    }
    return length
}

```
思路是 把不相等的元素按照顺序放到开头 而我是想覆盖相同的元素 处理不了这个撮箕


28 找出字符串中的第一个匹配项下标

题目： 

输入：haystack = "sadbutsad", needle = "sad"
输出：0
解释："sad" 在下标 0 和 6 处匹配。
第一个匹配项的下标是 0 ，所以返回 0 。
示例 2：

输入：haystack = "leetcode", needle = "leeto"
输出：-1
解释："leeto" 没有在 "leetcode" 中出现，所以返回 -1 。


```go
 var num int
func strStr(haystack string, needle string) int {   
    for i:=0; i<len(haystack); i++ {
        for j := 0; j<len(needle); j++ {
            if needle[j] == haystack[i]{
                num++
                i++
                 if num == len(needle){
                    return i-len(needle)
                } 
            }
            
       }   
 }
          return -1
    }
```
只通过了一个用例， 你要反思了 只通过一个用例，说明了你的想法是错的

暴力解法：

```go
func strStr(haystack string, needle string) int {
    if len(needle) == 0 { // 空字符串直接返回 0
        return 0
    }
    for i := 0; i <= len(haystack)-len(needle); i++ {
        j := 0
        for j < len(needle) && haystack[i+j] == needle[j] {
            j++
        }
        if j == len(needle) {
            return i // 完全匹配，返回起始索引
        }
    }
    return -1 // 未找到匹配
}
```
多多练习， haystack[i+j] == needle[j] 这里是很巧妙的地方，同时比对了两个字符串， i的位置就是起始位置， 如果整个字符串都在内部匹配完成了，说明i就是起始位置
记录需要对比的字符串长度

---

##### 58题： 最后一个单词的长度

示例 1：

输入：s = "Hello World"
输出：5
解释：最后一个单词是“World”，长度为 5。
示例 2：

输入：s = "   fly me   to   the moon  "
输出：4
解释：最后一个单词是“moon”，长度为 4。
示例 3：

输入：s = "luffy is still joyboy"
输出：6
解释：最后一个单词是长度为 6 的“joyboy”。

我的代码：

```go
var length int
func lengthOfLastWord(s string) int {
    for i := len(s)-1; i>0; i-- {
        if s[i] != ' ' {
            for s[i] != ' '{
                length++
                i--
            }
            return length
        }
}
return 0
}
```
还是错了 但是错的不太离谱，简单的呗GPT加了一点就过了

```go
func lengthOfLastWord(s string) int {
    length := 0
    for i := len(s) - 1; i >= 0; i-- { // 修正条件为 i >= 0
        if s[i] != ' ' {
            for i >= 0 && s[i] != ' ' { // 避免索引越界
                length++
                i--
            }
            return length
        }
    }
    return 0 // 如果没有找到任何单词，返回 0
}
```
---

但是在其中话是有一个有意思的事情

```go

func lengthOfLastWord(s string) int {
    length := 0
    for i := len(s)-1; i>=0; i-- {
        if s[i] != ' ' {
            for s[i] != ' ' &&  i >= 0{
                length++
                i--
            }
            return length
        }
}
return 0
}
```
代码我是这么写的 跟上面的没什么区别，但是没有通过 我又询问到底是为什么

关键在于顺序 以防止越界  for s[i] != ' ' &&  i >= 0

内层循环条件：

for s[i] != ' ' && i >= 0 {
当 i 等于 0 时，s[i] 可能会访问索引 0 的内容，然后才进行 i >= 0 的检查。由于条件的顺序，s[i] 的访问可能在 i < 0 时执行，导致索引越界。也就是说：

问题在于：

在执行 s[i] != ' ' 这个条件时，i 可能会变成负数（比如 i == 0 时，接下来你做 i--，i 会变成 -1）。
但是，i >= 0 的条件是后面的检查，也就是说，在 i-- 之后，s[i] != ' ' 已经执行了，导致 i == -1 时，程序尝试访问 s[-1]，而 Go 语言数组下标不能为负数，这样会导致 索引越界 错误。

---

---

#### 21 合并两个有序链表：

l1 = [1,2,4], l2 = [1,3,4]
输出：[1,1,2,3,4,4]
示例 2：

输入：l1 = [], l2 = []
输出：[]
示例 3：

输入：l1 = [], l2 = [0]
输出：[0]

我写的又臭又长的代码

```go

/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */

func mergeTwoLists(list1 *ListNode, list2 *ListNode) *ListNode {
    list3 := &ListNode{
    Val: 0,
    Next: nil,
}
   if list1 == nil {
        return list2
    }
    if list2 == nil {
        return list1
    }

    for list1.Val <= list2.Val {
        if list1.Next == nil {
            for list2.Next != nil{
                   list3.Val = list2.Val 
                   list2 = list2.Next 
            }  
        }else {
        list3.Val = list1.Val
        list3 = list3.Next
        list1 = list1.Next
        }  if list1.Next == nil {
            for list1.Next != nil {
                list3.Val = list1.Val
                list1 = list1.Next
            }
        }else {
        list3.Val = list2.Val
        list3 = list3.Next
        list2 = list2.Next
        }
       
    }

    for list1.Val >= list2.Val {
       
       
    }
return list3
}
```

疑惑： 

你说得对，list3 := &ListNode{} 确实只初始化了一个节点，但这个节点的作用是 哨兵节点 (dummy node)，用来简化链表操作和边界条件处理。

哨兵节点的主要特点是：

它本身并不存储有效数据（值通常是默认值，比如 0）。
它的作用是作为链表的起点，使链表构造和返回操作更加简单和一致。
以下是对你的疑问的详细解释：

1. 为什么哨兵节点只初始化一个节点却能构造完整链表？
   哨兵节点的作用只是提供链表的起点，后续节点是通过动态创建并连接到它的 Next 指针上构造的。例如：

list3 := &ListNode{}  // 初始化哨兵节点
current := list3      // current 指向哨兵节点

current.Next = &ListNode{Val: 1} // 创建第一个有效节点
current = current.Next           // 移动指针到新节点

current.Next = &ListNode{Val: 2} // 创建第二个有效节点
current = current.Next           // 移动指针到新节点
这段代码最终形成的链表结构是：

scss
复制代码
哨兵节点 (值: 0) -> 节点1 (值: 1) -> 节点2 (值: 2)
返回值时，通过 list3.Next 跳过哨兵节点，直接指向第一个有效节点。




正确思维：

期望行为：应该创建一个新节点并将其连接到 current.Next。  我没有理解怎么初始化一个链表，并为后续创建新的结点添加值

emm 正解


#### 正确代码：
```go
func mergeTwoLists(list1, list2 *ListNode) *ListNode {
    dummy := ListNode{} // 用哨兵节点简化代码逻辑
    cur := &dummy // cur 指向新链表的末尾
    for list1 != nil && list2 != nil {
        if list1.Val < list2.Val {
            cur.Next = list1 // 把 list1 加到新链表中
            list1 = list1.Next
        } else { // 注：相等的情况加哪个节点都是可以的
            cur.Next = list2 // 把 list2 加到新链表中
            list2 = list2.Next
        }
        cur = cur.Next
    }
    // 拼接剩余链表
    if list1 != nil {
        cur.Next = list1
    } else {
        cur.Next = list2
    }
    return dummy.Next
}
```

这段代码中直接使用 list1 或 list2 而不是 list1.Val 是因为它要把整个节点连接到新链表中，而不仅仅是节点的值。

区别
list1.Val 仅获取当前节点的值。

如果只使用 Val，你只能获得一个值，无法直接将这个值插入链表中，还需要手动创建一个新节点。
list1 是指针，代表当前节点。

直接使用 list1 表示把当前节点（包括其 Val 和 Next 指针）插入到新链表中，效率更高且代码更简单。

---

---
今天又重新做了一下20题 有效括号

刚开始我在想，电脑应该怎么区分左括号和右括号这两个是怎么合并的？ 绞尽脑汁。 最终思想就是遍历字符串， 将左括号放入栈中 将栈顶元素和当前元素作比较 如果相同弹出括号，成功的标志是栈中元素为空

---
232 用两个栈实现队列

["MyQueue", "push", "push", "peek", "pop", "empty"]
[], [1], [2], [], [], []
输出：
[null, null, null, 1, 1, false]

```go
type MyQueue struct {
    Rear *MyQueue
    Head *MyQueue
    size int
}


func Constructor() MyQueue {
    Stack1 := []int
    Stack2 := []int
    
}


func (this *MyQueue) Push(x int)  {
    Stack1 := []int
    Stack2 := []int
    Stack1 = append(Stack1,x)
    legth := len(Stack1)
    v := Stack1[l-1]
}


func (this *MyQueue) Pop() int {
    Stack1 = Stack1(:len(Stack1)-1)
    Stack2 = append(Stack2,)
}


func (this *MyQueue) Peek() int {
    
}


func (this *MyQueue) Empty() bool {
    
}


/**
 * Your MyQueue object will be instantiated and called as such:
 * obj := Constructor();
 * obj.Push(x);
 * param_2 := obj.Pop();
 * param_3 := obj.Peek();
 * param_4 := obj.Empty();
 */
```
看到代码有点不知所措了，在每个地方应该放入什么 两个栈实现队列：思路我是有的 把数字放入其中一个栈中，然后在弹出来，放入第二个栈中，然后在第二个栈中弹出来就是队列

思路是对的

```go
type MyQueue struct {
    Stack1 []int
    Stack2 []int
}


func Constructor() MyQueue {
    return MyQueue{}
}


func (this *MyQueue) Push(x int)  {
    this.Stack1 = append (this.Stack1,x)
}


func (this *MyQueue) Pop() int {
    if len(this.Stack2) == 0 && len(this.Stack1) == 0  {
        return -1
    }

    if len(this.Stack1) != 0 {

        for len(this.Stack1) > 0 {
    length1 := len(this.Stack1)-1
    val := this.Stack1[length1]
    this.Stack1 = this.Stack1[:length1]
    this.Stack2 = append(this.Stack2,val)
        }
   
    }
    length2 := len(this.Stack2)-1
    result := this.Stack2[length2]
    this.Stack2 = this.Stack2[:length2]
   
    return result
}


func (this *MyQueue) Peek() int {
    
    if len(this.Stack1) == 0 && len(this.Stack2) == 0 {
        return -1
    }
    if len(this.Stack1) != 0 {

        for len(this.Stack1) >0 {
            
    length1 := len(this.Stack1)-1
    val := this.Stack1[length1]
    this.Stack1 = this.Stack1[:length1]
    this.Stack2 = append(this.Stack2,val)
        }
   
    }
   
    length2 := len(this.Stack2) -1

    return this.Stack2[length2]
}


func (this *MyQueue) Empty() bool {
    return len(this.Stack1) == 0 && len(this.Stack2) == 0
}


/**
 * Your MyQueue object will be instantiated and called as such:
 * obj := Constructor();
 * obj.Push(x);
 * param_2 := obj.Pop();
 * param_3 := obj.Peek();
 * param_4 := obj.Empty();
 */

```
重新写了一遍 用例没有完全通过 

[null,null,null,null,null,1,null,5,2,3,4]

预期结果
[null,null,null,null,null,1,null,2,3,4,5]

有两个地方出错导致没有通过  if len(this.Stack1) != 0  条件判断句 改成 ==0 就通过了 思考为什么

---
383.赎金信
参考两数之和的哈希表用法

我写出了以下代码 

```go
func canConstruct(ransomNote string, magazine string) bool {
    m := make(map[byte]int)
    for i:=1 ; i<len(ransomNote); i++ {
        if _,ok := m[ransomNote[i]]; ok{
            continue
        }
        m[ransomNote[i]] = i 
    }

    for k:=1; k<len(magazine); k++ {
        if _, ok := m[magazine[k]]; ok {
            continue
        } 
        if k == len(magazine) {
            return true
        }
    }
     return false
}
```
但是没有通过，应该是情况考虑错了

```go
func canConstruct(ransomNote string, magazine string) bool {
    m := make(map[byte]int)
    for i:=0; i<len(magazine); i++ {
        if _,ok := m[magazine[i]]; ok{
            continue
        }
        m[magazine[i]] = i 
    }

    for k:=0; k<len(ransomNote); k++ {
        if _, ok := m[ransomNote[k]]; ok {
            continue
        } else {
            return false
        }
    }
     return true
}
```
最终修改成这样 题目中话有一个条件 字母无法重复使用！！！这点给忘记给忘记了
问了这一点！ 关键就是对于字符的统计！



正确代码：
```go
func canConstruct(ransomNote string, magazine string) bool {
    // 用一个 map 记录 magazine 中每个字符出现的次数
    m := make(map[byte]int)
    for i := 0; i < len(magazine); i++ {
        m[magazine[i]]++
    }

    // 检查 ransomNote 中的每个字符是否在 magazine 中有足够的次数
    for i := 0; i < len(ransomNote); i++ {
        if m[ransomNote[i]] > 0 {
            m[ransomNote[i]]-- // 使用一个字符
        } else {
            return false // 如果次数不足，无法构造
        }
    }

    return true // 所有字符都能满足
}

```

我的疑惑：我理解前两个a会增到2 但是第三个字符是b的时候为什么会从0开始计算起而不是2

这是因为在 Go 的 map 中，每个键（key）是独立的，互相之间没有关联。

在你的例子中，当处理第三个字符 b 时：

之前已经对 a 进行了计数，所以 m['a'] = 2。
但是 b 是一个新的键，之前并没有在 map 中出现过。
当访问 m['b'] 时，由于 map 默认返回未初始化键的值为类型的 零值（对于 int 类型来说，零值是 0），所以 m['b'] = 0。
然后执行 m['b']++，将其从 0 增加到 1。

---
141.环形链表

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func hasCycle(head *ListNode) bool {
    if head == nil {
        return false
    }
    m := make(map[*ListNode]int)
    for head != nil {
        m[head]++
        if _,ok := m[head]; ok {
            return true
        }
        head = head.Next
    }
    return false
}
```
又没有一次性通过 条件感太差了！

啊居然自己改对了！！！

调整了一下顺序，

```go
/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func hasCycle(head *ListNode) bool {
    if head == nil {
        return false
    }
    m := make(map[*ListNode]int)
    for head != nil {
        m[head]++
        head = head.Next
        if _,ok := m[head]; ok {
            return true
        }
    }
    return false
}

```
在其中出现的错误： map中的地址应该是结构体指针 而不是 *int 

---

如何在go中使用中序遍历

94题：

```go
func inorderTraversal(root *TreeNode) []int {
    result := []int{} // 用于存储中序遍历的结果

    // 定义递归函数
    var inorder func(node *TreeNode)
    inorder = func(node *TreeNode) {
        if node == nil { // 递归终止条件：当前节点为空
            return
        }
        inorder(node.Left)           // 遍历左子树
        result = append(result, node.Val) // 访问当前节点
        inorder(node.Right)          // 遍历右子树
    }

    inorder(root) // 从根节点开始中序遍历
    return result // 返回结果
}


```

inorder = func(node *TreeNode) {
if node == nil { // 递归终止条件：当前节点为空
return
} 这块使用我不太清楚 我还以为是对原函数递归

---
933.最近的请求次数

写一个 RecentCounter 类来计算特定时间范围内最近的请求。

请你实现 RecentCounter 类：

RecentCounter() 初始化计数器，请求数为 0 。
int ping(int t) 在时间 t 添加一个新请求，其中 t 表示以毫秒为单位的某个时间，并返回过去 3000 毫秒内发生的所有请求数（包括新请求）。确切地说，返回在 [t-3000, t] 内发生的请求数。
保证 每次对 ping 的调用都使用比之前更大的 t 值。

输入：
"RecentCounter", "ping", "ping", "ping", "ping"]

输出：


解释：
RecentCounter recentCounter = new RecentCounter();
recentCounter.ping(1);     // requests = [1]，范围是 [-2999,1]，返回 1
recentCounter.ping(100);   // requests = [1, 100]，范围是 [-2900,100]，返回 2
recentCounter.ping(3001);  // requests = [1, 100, 3001]，范围是 [1,3001]，返回 3
recentCounter.ping(3002);  // requests = [1, 100, 3001, 3002]，范围是 [2,3002]，返回 3

读题读了一会才读懂，最后反应过来应该就是返回 在范围内的ping次数 需要用一个切片（其实应该是队列思想 入队 出队不符合条件的 返回符合条件的数量）
（这在go语言中就可以用切片来简化操作 返回 符合条件的ping次数 也就是 队列中剩下的长度 len（））

正确解答

```go
type RecentCounter struct {
    count []int
}


func Constructor() RecentCounter {
    return RecentCounter {
        count: []int{}, 
    }
}


func (this *RecentCounter) Ping(t int) int {
    this.count = append(this.count,t)


    //重点怎么移除不符合条件的 一个个对比
    for len(this.count)>0 && this.count[0]<t-3000 {
        this.count = this.count[1:]
    }
    return len(this.count)
}


/**
 * Your RecentCounter object will be instantiated and called as such:
 * obj := Constructor();
 * param_1 := obj.Ping(t);
 */
```

1.学会初始化结构体
2.学会使用切片思想实现 队列 栈等 加强对切片的操作


---

12.17日 开始执行树和递归

104 二叉树的最大深度
110 平衡二叉树

104代码

```go

func maxDepth(root *TreeNode) int {
	if root == nil {
		return 0
	}
	var l, r = maxDepth(root.Left), maxDepth(root.Right)
	if l > r {
		return l + 1
	} else {
		return r + 1
	}
}
```
这段代码我是抄的 我对递归的思想还是不太理解

递归的 隐式深度记录 是通过调用栈实现的：

每次递归调用时，程序会进入一个新的函数调用，并等待其返回值。
当递归调用到底（即遇到叶子节点或 nil 节点）时，递归开始返回。
每一层递归通过返回值，将子树的深度信息逐层传递回上一层。
可以理解为：

每个函数调用相当于“记住”当前节点的状态，等待左右子树的深度计算完成后，才计算当前节点的深度并返回。

递归的返回值 自然累加 深度的方式，不需要额外的变量来显式记录深度。


#### 49.字母异位词分组

**输入:** strs = `["eat", "tea", "tan", "ate", "nat", "bat"]`
**输出:** [["bat"],["nat","tan"],["ate","eat","tea"]]

思路： 
1. 为每个单词实施排序 使单词从小到大
2. 将排序的单词存入到数据结构中
3. 将每一个单词和数据结构中的单词比较 如果 相同 则分为一组 

```go
func groupAnagrams(strs []string) [][]string {

    m := map[string][]string{} //map的用法！

    for _, s := range strs {

        t := []byte(s)

        slices.Sort(t)

        sortedS := string(t)

        m[sortedS] = append(m[sortedS], s) // sortedS 相同的字符串分到同一组

    }

    return slices.Collect(maps.Values(m))

}
```

标准解法：

对字符串的操作不熟练！，见到字符串有些发懵

在 Go 中，字符串是不可变的，而字节切片是可变的

##### map的用法
```go
m := map[string][]string{}
```

##### string操作示例：

```go
s := "hello"
t := []byte(s)  // t = [104 101 108 108 111]
```

- `slices.Sort` 是 Go 1.21 引入的一个泛型函数，用于对切片进行排序'
- `maps.Values` 是 Go 1.21 引入的一个泛型函数，用于从一个映射（`map`）中提取所有的值。
- 它接受一个映射 `m` 作为参数，并返回一个包含所有值的切片。

**`append(m[sortedS], s)` 的作用**

- `append` 函数用于将 `s` 添加到 `m[sortedS]` 的末尾。
    
- 如果 `m[sortedS]` 不存在（即 `sortedS` 是第一次出现），`m[sortedS]` 会返回一个空的切片（`nil`），然后 `append` 会创建一个新的切片并将 `s` 添加进去。
    
- 如果 `m[sortedS]` 已经存在，`append` 会将 `s` 追加到已有的切片中。


这个方法好厉害！

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

`ans := make([][]string, 0, len(strs))`   - 预分配容量的空切片



#### 19 删除链表的倒数第N个节点

```go
/**

 * Definition for singly-linked list.

 * type ListNode struct {

 *     Val int

 *     Next *ListNode

 * }

 */

func removeNthFromEnd(head *ListNode, n int) *ListNode {

    length := 0

    cur := head

    for cur != nil {

        length ++

        cur = cur.Next

    }

  

    if n == length {

        return head.Next

    }

  

    number := length - n

  

    prev := head

    for i := 0; i < number - 1; i++ {

        prev = prev.Next

    }

    prev.Next = prev.Next.Next

  

 return head

}
```

思路 根据倒数 推到整数

我觉得很直白但是肯定不是最佳做法

写的时候边界条件忘记了


#### 两数相加

```go
/**

 * Definition for singly-linked list.

 * type ListNode struct {

 *     Val int

 *     Next *ListNode

 * }

 */

func addTwoNumbers(l1 *ListNode, l2 *ListNode) *ListNode {

    l3 := &ListNode{}

    carry := 0

    cur := l3

    for l1 != nil || l2 != nil || carry != 0 {

        s := carry

        if l1 != nil {

            s += l1.Val

        }

        if l2 != nil {

            s += l2.Val

        }

        carry = s / 10

        cur.Next = &ListNode{Val: s % 10,Next: nil }

        cur = cur.Next

        if l1 != nil {

            l1 = l1.Next

    }

        if l2 != nil {

            l2 = l2.Next

        }

}

return l3.Next

}
```


借鉴的代码 有思路 但是写不出来 尤其是 `for l1 != nil || l2 != nil || carry != 0`




---

2025.4.6

### 560 和为k的子数组

```go
func subarraySum(nums []int, k int) int {

    count := 0

    preSum := 0

    hash := map[int]int{0:1}

  

    for i := 0; i < len(nums); i++{

        preSum += nums[i]

  

        if (hash[preSum - k] > 0) {

            count += hash[preSum - k]

    }

    hash[preSum] ++

  

}

return  count

}
```

前缀和＋哈希表

看代码还是可以理解的但是还是想不到

暴力求和

```go
func subarraySum(nums []int, k int) int {
    Sumlength := 0
for i := 0; i< len(nums); i++ {
   currentSum := nums[i]
    if currentSum == k {
        Sumlength ++
    } 
for j := i + 1; j < len(nums); j ++ {
    currentSum += nums[j]
    if  currentSum == k {
        Sumlength ++
    }
}
}
return Sumlength
}
```

很慢 并且第一次写的时候并没有处理好逻辑 对于break 和 continue的使用有点乱了！



### 230.二叉搜索树


~~~
二叉搜索树（BST，Binary Search Tree），也称[二叉排序树]或二叉查找树。  
二叉搜索树：一棵二叉树，可以为空；如果不为空，满足以下性质：

1. 非空左子树的所有键值小于其根结点的键值。
2. 非空右子树的所有键值大于其根结点的键值。
3. 左、右子树都是二叉搜索树。

~~~


#### 理解二叉树的前中后序遍历的作用： 

中序可以将二叉树从小到大排序


#### 理解GO中的闭包性质

函数字面量（匿名函数)是_闭包_：它们可以引用在周围函数中定义的变量。然后，这些变量在周围的函数和函数字面量之间共享，只要它们还可以访问，它们就会继续存在。

闭包 = 函数 + 引用环境


```go
/**

 * Definition for a binary tree node.

 * type TreeNode struct {

 *     Val int

 *     Left *TreeNode

 *     Right *TreeNode

 * }

 */

func kthSmallest(root *TreeNode, k int) (ans int) {

    var dfs func(*TreeNode)

    dfs = func (node *TreeNode){

        if node == nil {

        return

    }

    dfs(node.Left)

    k --

    if k == 0 {

    ans = node.Val

    return

    }

    dfs(node.Right)

    }

    dfs(root)

    return

}
```


### 11.  盛最多水的容器

给定一个长度为 `n` 的整数数组 `height` 。有 `n` 条垂线，第 `i` 条线的两个端点是 `(i, 0)` 和 `(i, height[i])` 。

找出其中的两条线，使得它们与 `x` 轴共同构成的容器可以容纳最多的水。

返回容器可以储存的最大水量。

[1,8,6,2,5,4,8,3,7]
**输出：**49 
**解释：**图中垂直线代表输入数组 [1,8,6,2,5,4,8,3,7]。在此情况下，容器能够容纳水（表示为蓝色部分）的最大值为 49。

```go
func maxArea(height []int) int {

    max := 0

    for i := 0; i < len(height)-1; i++ {

        for j := i + 1; j<len(height); j++ {

            min := min(height[i], height[j])

            maxVolume := (j - i) * min

            if max <maxVolume {

                max = maxVolume

            }

        }

    }  

    return max

}

  

func min(a, b int) int {

    if a < b {

        return a

    }

    return b

}
```

效率特别低哈哈哈


```go
func maxArea(height []int) int {

    left := 0

    right := len(height) - 1

    max := 0

  

    for  left < right {

    min := min(height[left],height[right])

    width := right - left

    maxVolume := min * width

    if max < maxVolume {

        max = maxVolume

    }

    // 移动指针（保留较高的边）

        if height[left] < height[right] {

            left++

        } else {

            right--

        }

    }

            return max

    }

  

func min(a, b int) int {

    if a <= b {

        return a

    }

    return b

}
```

改进双指针


### 56 合并区间

```go
func merge(intervals [][]int) [][]int {

    slices.SortFunc(intervals, func(p, q []int) int { return p[0] - q[0] })

    var results [][]int

    results = append(results, intervals[0]) // 先加入第一个区间

    for i := 1 ; i< len(intervals); i ++ {

        curr := intervals[i]

        last := results[len(results) - 1]

        if curr[0] <= last[1] {

            if curr[1] > last[1] {

                last[1] = curr[1]

            } else {

                results = append(results,curr)

            }

        }

    }

    return results

}
```


这是按照我的思路 写出来的 但是最终结果没有通过！ 只通过了14个用例

```go
func merge(intervals [][]int) [][]int {

    slices.SortFunc(intervals, func(p, q []int) int { return p[0] - q[0] })

    var results [][]int

    results = append(results, intervals[0]) // 先加入第一个区间

    for i := 1 ; i< len(intervals); i ++ {

        curr := intervals[i]

        lastIndex := len(results) - 1

        if curr[0] <= results[lastIndex][1] {

            if curr[1] > results[len(results) - 1][1]{

               results[lastIndex][1] = curr[1]

            } else {

                results = append(results,curr)

            }

        }

    }

    return results

}
```



**else 放错位置了！！！！！**

**你对排序算法的理解！ 掌握了几种呢？**


## 138 随机链表的复制 

理解深拷贝！

```go
  
  

func copyRandomList(head *Node) *Node {

    if head == nil {

        return nil

    }

    for curr := head; curr != nil; {

        newNode := &Node{Val: curr.Val}

        newNode.Next = curr.Next

        curr.Next = newNode

        curr = newNode.Next

}

    for curr := head; curr != nil; curr = curr.Next.Next {

        if curr.Random != nil {

            curr.Next.Random = curr.Random.Next

        }

    }

     newHead := head.Next

    for oldNode, newNode := head, head.Next; oldNode != nil; {

        oldNode.Next = newNode.Next

        if newNode.Next != nil {

            newNode.Next = newNode.Next.Next

        }

         newNode = newNode.Next

         oldNode = oldNode.Next

    }

    return newHead

}
```

为什么需要 newHead := head.Next 不是有 oldNode, newNode := head,head.Next这个了吗


```go
newHead := head.Next

for oldNode, newNode := head, head.Next; oldNode != nil; {

oldNode.Next = newNode.Next

if newNode.Next != nil {

newNode.Next = newNode.Next.Next

}

oldNode = oldNode.Next

newNode = newNode.Next

}

return newHead
```

为什么不能返回 newNode呢？？ 为什么会表示newNode未定义



# 如何设计特定的时间复杂度？

## 158.寻找旋转排序数组中的最小值

你必须设计一个时间复杂度为 `O(log n)` 的算法解决此问题。



```go
func findMin(nums []int) int {

    left := 0;

    right := len(nums) - 1

  

    min := nums [right]

    for left < right {    

        mid := left + (right - left) / 2

        if nums[mid] > nums[right] {

        left = mid + 1

    } else {

        right = mid

    }  

    if nums [mid] < min {

        min = nums[mid]

    }

}

    return min

}
```


二分法：

## 33.搜索旋转排序数组




### 437 路径总和

```go
/**

 * Definition for a binary tree node.

 * type TreeNode struct {

 *     Val int

 *     Left *TreeNode

 *     Right *TreeNode

 * }

 */

func pathSum(root *TreeNode, targetSum int) int {

    if root == nil {

        return 0

    }

  

    // 计算从当前节点出发的路径数

    result := dfs(root, targetSum)

  

    // 递归左子树和右子树

    result += pathSum(root.Left, targetSum)

    result += pathSum(root.Right, targetSum)

  

    return result

}

  

// 从当前节点出发，计算符合条件的路径数

func dfs(root *TreeNode, targetSum int) int {

    if root == nil {

        return 0

    }

  

    count := 0

    if root.Val == targetSum {

        count = 1

    }

  

    // 递归检查左子树和右子树

    count += dfs(root.Left, targetSum - root.Val)

    count += dfs(root.Right, targetSum - root.Val)

  

    return count

}
```




### 3. 无重复字符的最长字串



```go
func lengthOfLongestSubstring(s string) int {
    left := 0
    max := 0
    length := 0;
    windows := make(map[rune]int)

    for key , letter := range s {
        windows[letter] ++
        length ++
        if windows[letter] > 1 {
            left ++
            max = length 
            length = 0 
            for letter := range windows {
                windows[letter] --
            }
        }
        max = length
}
    return max - 1
} 
```


思路一： 根据下标索引

```go
func lengthOfLongestSubstring(s string) int {

    left := 0

    maxLength := 0

    windows := make(map[rune]int)

  

    for right,letter := range s {

        if pos, exists := windows[letter]; exists && pos >= left {

    left = pos + 1

}

      windows[letter] = right

      currLength := right - left + 1

      if currLength > maxLength {

       maxLength =  currLength

      }

}

   return maxLength

}
```


思路二

根据出现次数

```go
func lengthOfLongestSubstring(s string) int {

    left := 0

    maxLength := 0

    charCount := make(map[rune]int)

  

    for right, char := range s {

        charCount[char]++

        for charCount[char] > 1 {

            charCount[rune(s[left])]--

            left++                    

        }

        if right - left + 1 > maxLength {

            maxLength = right - left + 1

        }

    }

    return maxLength

}
```



### 1493 删掉一个元素以后全为1的最长子数组

思路1：以0的个数作为窗口

```go
func longestSubarray(nums []int) int {
    left, zeroCount := 0, 0
    maxLen := 0

    for right := 0; right < len(nums); right++ {
        if nums[right] == 0 {
            zeroCount++
        }

        for zeroCount > 1 {
            if nums[left] == 0 {
                zeroCount--
            }
            left++
        }

        maxLen = max(maxLen, right-left)
    }

    return maxLen
}

func max(a, b int) int {
    if a > b {
        return a
    }
    return b
}

```

我的思路：让两个被隔开的 1 段连接起来
（但是我没有写出来）

```go
func longestSubarray(nums []int) int {

    maxLen := 0

    prevOnes := 0

    currOnes := 0
    
    zeroSeen := false

    for _, num := range nums {

        if num == 1 {

            currOnes++

        } else {

            // 碰到 0 时，尝试拼接前后 1 段

            maxLen = max(maxLen, prevOnes+currOnes)

            prevOnes = currOnes // 当前段变成“上一段”

            currOnes = 0         // 重置当前段

            zeroSeen = true
        }
    }

    // 最后还要比一次（可能最后一段是 1）
    if zeroSeen {
        maxLen = max(maxLen, prevOnes+currOnes)
    } else {
        // 全是 1 的情况，必须删掉一个
        maxLen = currOnes - 1
    }
        return maxLen
}
```



#### 34 在排序数组中查找元素的第一个位置和最后一个位置

给你一个按照非递减顺序排列的整数数组 `nums`，和一个目标值 `target`。请你找出给定目标值在数组中的开始位置和结束位置。

如果数组中不存在目标值 `target`，返回 `[-1, -1]`。

你必须设计并实现时间复杂度为 `O(log n)` 的算法解决此问题。



```go
func searchRange(nums []int, target int) []int {

    return []int{findLeft(nums, target), findRight(nums, target)}

}

  

func findLeft(nums []int, target int) int {

    left, right := 0, len(nums)-1

    res := -1

    for left <= right {

        mid := left + (right-left)/2

        if nums[mid] == target {

            res = mid        // 记录结果

            right = mid - 1  // 继续往左找

        } else if nums[mid] < target {

            left = mid + 1

        } else {

            right = mid - 1

        }

    }

    return res

}

  

func findRight(nums []int, target int) int {

    left, right := 0, len(nums)-1

    res := -1

    for left <= right {

        mid := left + (right-left)/2

        if nums[mid] == target {

            res = mid        // 记录结果

            left = mid + 1   // 继续往右找

        } else if nums[mid] < target {

            left = mid + 1

        } else {

            right = mid - 1

        }

    }

    return res

}
```