---
title: "Some Useful Helper Functions in Go for Leetcode -- Linked List, Tree, Graph"
data: 2025-04-01
---

## Assertion

```go
func assertEq[T comparable](a, b T) {
	if a != b {
		fmt.Printf("Failed: %v != %v\n", a, b)
	} else {
		fmt.Printf("Ok: %v\n", a)
	}
}
```

## Linked List

```go
// LeetCode linked list definition
type ListNode struct {
	Val  int
	Next *ListNode
}

// create a linked list with an array of ints
func makeList(vals []int) *ListNode {
	head := &ListNode{0, nil}
	prev := head
	for _, v := range vals {
		prev.Next = &ListNode{v, nil}
		prev = prev.Next
	}
	return head.Next
}

// print a linked list
func printList(head *ListNode) {
	fmt.Print("[")
	for head != nil {
		fmt.Printf("%d, ", head.Val)
		head = head.Next
	}
	fmt.Println("]")
}
```

## Binary Tree

```go
// LeetCode binary tree definition
type TreeNode struct {
	Val   int
	Left  *TreeNode
	Right *TreeNode
}

// create a bianry tree given an array of `int` or `nil`
// e.g. root := makeBinaryTree([]any{1, 2, 3, nil, nil, 4, 5})
// will produce a binary tree:
//           1
//     2           3
// nil   nil     4   5
func makeBinaryTree(nodes []any) *TreeNode {
	var root *TreeNode
	queue := []*TreeNode{}
	for i, n := range nodes {
		var cur *TreeNode
		if n != nil {
			cur = &TreeNode{
				Val:   n.(int),
				Left:  nil,
				Right: nil,
			}
		}
		queue = append(queue, cur)

		for queue[0] == nil {
			queue = queue[1:]
		}

		if i == 0 {
			root = cur
		} else if i%2 != 0 {
			queue[0].Left = cur
		} else {
			queue[0].Right = cur
			queue = queue[1:]
		}
	}
	return root
}

// print a bianry tree
func printBinaryTree(root *TreeNode) {
	if root == nil {
		return
	}
	fmt.Println(root.Val)
	printBinaryTree(root.Left)
	printBinaryTree(root.Right)
}
```
