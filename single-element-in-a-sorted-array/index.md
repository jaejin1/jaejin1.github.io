# [Leetcode] 540. Single Element in a Sorted Array


https://leetcode.com/problems/single-element-in-a-sorted-array/description/

<!--more-->

O(log n) 시간 복잡도에 O(1) 으로 풀어야한다는 걸 보자마자 binary search로 탐색 해야겠다는 걸 알아 차렸지만..

생각 외로 시간이 오래걸림

나는 중간 값(무조건 홀수이기 때문에)을 먼저 보고 nums[len(nums/2)] 중간 값의 양옆을 비교 하면서 반씩 쪼개 갔는데

솔루션을 보니 0 인덱스를 left, last 인덱스를 right로 주고 풀더라 

뭐든 결국 쪼개서 log n 시간복잡도를 나오게 하는건 맞지만 조건을 덜 주면서 간결하게 짜려면 솔루션 방법대로 할 순 있을까 싶다

```go
func singleNonDuplicate(nums []int) int {
    existinLeft := false

    for len(nums) > 1 {
        middle := nums[len(nums)/2]

        if nums[len(nums)/2 - 1] == middle {
            // 같은게 중앙 왼쪽
            existinLeft = true
        } else if nums[len(nums)/2 + 1] == middle {
            // 같은게 중앙 오른쪽
            existinLeft = false
        } else {
            // 가운데 값이 한개의 값
            return middle
        }

        if existinLeft {
            if (len(nums)/2 - 1) % 2 == 0 {
                nums = nums[len(nums)/2 + 1:]
            } else {
                nums = nums[:len(nums)/2 - 1]
            }
        } else {
            if (len(nums)/2) % 2 == 0 {
                nums = nums[len(nums)/2 + 2:]
            } else {
                nums = nums[:len(nums)/2]
            }
        }
    }
    return nums[0]
}
```
