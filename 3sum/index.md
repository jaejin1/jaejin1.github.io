# [Leetcode] 15. 3Sum


https://leetcode.com/problems/3sum/description/

<!--more-->

```go
func threeSum(nums []int) [][]int {
    var result [][]int
    sort.Ints(nums)

    for i := 0; i < len(nums) - 2; i++ {
        left := i + 1
        right := len(nums) - 1
        
        for left < right {
            sum := nums[i] + nums[left] + nums[right]
            if sum < 0 {
                left += 1
            } else if sum > 0 {
                right -= 1
            } else {
                result = append(result, []int{nums[i], nums[left], nums[right]})

                for left < right && nums[left] == nums[left + 1] {
                    left += 1
                }

                for left < right && nums[right] == nums[right - 1] {
                    right -= 1
                }

                left += 1
                right -= 1
            }
        }
        for i + 1 < len(nums) && nums[i+1] == nums[i] {
            i += 1
        }
    }


    return result
}
```
