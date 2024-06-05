# [Leetcode] 56. Merge Intervals


https://leetcode.com/problems/merge-intervals/description/

<!--more-->

먼저 정렬하고 시작하면 되는데,, 처음에 안하고 하다가 시간 다 뺏김

예시로 `[[1,3],[2,6],[8,10],[15,18]]` 라면 먼저 [1,3], [2,6]을 비교 하자.

이미 0의 인덱스로 정렬되어있다고 가정하고 겹치려면 [2,6]의 0인덱스인 2와 [1,3]의 1인덱스인 3을 비교 하면 된다. 즉 3이 더 크기 때문에 두 array는 겹친다고 판단

[1, max(3,6)] 으로 merge 하면됨

```go
func merge(intervals [][]int) [][]int {
    sort.Slice(intervals, func(i, j int) bool {
        return intervals[i][0] < intervals[j][0]
    })
    
    for i := 1 ; i < len(intervals) ; i++ {

        if intervals[i][0] <= intervals[i - 1][1] {
            intervals[i] = []int{intervals[i-1][0], max(intervals[i-1][1], intervals[i][1])}
            
            intervals = append(intervals[:i - 1], intervals[i:]...)
            i--
        }
    } 

    return intervals
}

func max(a, b int) int {
    if a > b {
        return a
    } else {
        return b
    }
}
```
