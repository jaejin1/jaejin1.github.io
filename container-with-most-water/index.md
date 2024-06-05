# [Leetcode] 11. Container With Most Water


https://leetcode.com/problems/container-with-most-water/description/

<!--more-->


```go
func maxArea(height []int) int {
    max := 0
    left := 1
    right := len(height)

    for left < right {
        y := min(height[left - 1], height[right - 1])
        x := right - left
        area := x * y
        
        if max < area {
            max = area
        }

        if height[left - 1 ] <= height[right - 1] {
            left += 1
        } else {
            right -= 1
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
