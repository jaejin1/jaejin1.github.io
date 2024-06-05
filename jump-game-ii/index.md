# [Leetcode] 45. Jump Game II


https://leetcode.com/problems/jump-game-ii/description/

<!--more-->

전형적인 DP문제 라고 생각했다. 

minimum number를 구하는 문제이기 때문에 min number를 저장하는 dp list를 만들고 

O(n + m)?이 되도록 진행해보았다.

```go
func jump(nums []int) int {
    dp := make([]int, len(nums))

    for index := 0; index < len(nums); index++ {
        
        for i := 1; i < nums[index] + 1; i++ {
            
            if len(dp) == index + i {
                break
            }

            if dp[index + i] == 0 {
                dp[index + i] = dp[index] + 1
            } else if dp[index + i -1] + 1 < dp[index + i] {
                dp[index + i] = dp[index + i -1]
            } 
        }
    }

    return dp[len(dp) - 1]
}
```
