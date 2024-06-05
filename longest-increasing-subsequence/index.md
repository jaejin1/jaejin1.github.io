# [Leetcode] 300. Longest Increasing Subsequence


https://leetcode.com/problems/longest-increasing-subsequence/description/

<!--more-->

이것도 결국 개수를 구하는 거기 때문에 DP로 접근을 했다.

주어진 nums를 하나씩 보면서 지나온 배열의 최대 값을 저장하면서 진행

```go
func lengthOfLIS(nums []int) int {
    dp := make([]int, len(nums))

    res := 0

    for i, num := range nums {
        max := 0
        maxIndex := 0

        for j := 0; j < i; j++ {

            if nums[j] < num && max < dp[j] {
                max = dp[j]
                maxIndex = j
            }
        }
        
        if max != 0 {
            dp[i] = dp[maxIndex] + 1
        } else {
            dp[i] = 1
        }

        if res < dp[i] {
            res = dp[i]
        }
    }

    return res
}
```
