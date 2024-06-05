# [Leetcode] 33. Search in Rotated Sorted Array


https://leetcode.com/problems/search-in-rotated-sorted-array/description/

<!--more-->

이것도 log(n) 보니 Binary Search로 해야겠다는 생각을 했고

이미 정렬이 되어 있으나 pivot index에 의해 possibly rotated 되어 있을 수도 있다고 한다.

생각을 해보다가 Binary Search를 두번 돌리는데, 먼저 rotated되기 전 0 index를 찾는거 예를들어 `[4,5,6,7,0,1,2]` 라면 index ==> 4 

그 후 target이 주어진 배열의 마지막 보다 크다면 rotated되어 왼쪽 부분, 작다면 오른쪽에 있다고 생각했다. 

즉 첫번째 search로 target이 5라면 `[4,5,6,7,0,1,2]` -> `[4,5,6,7]` 에서 찾으면 될 거라고 생각했다. (rotated된 부분은 날려버리면 이미 정렬되어 있는 것이기 때문에)

`[4,5,6,7]` 에서 한번더 search로 target이 5인 index, 1을 찾으면 된다.

마지막으로 rotated 배열에서 앞부분을 자르게 되면 결과 값에 자른 만큼 크기를 더하기 위해서 `return result + left` 로 처리해준 것을 볼 수 있다.

```go
func search(nums []int, target int) int {
    left, right := 0, len(nums) - 1
    result := 0

    max := nums[len(nums) - 1]
    for left < right {
        mid := (right - left) / 2 + left

        if left != 0 {
            if nums[left] < nums[left-1] {
                break
            }
        }

        if nums[mid] < max {
            right = mid
        } else {
            left = mid + 1
        }
    }
   
    if nums[left] <= target && max >= target {
        nums = nums[left:]
        result += left
    } else {
        nums = nums[:left]
    }

    // example
    // [4,5,6,7,0,1,2] -> [0,1,2]
    // [6,7,0,1,2,3,4,5] -> [6,7]

    left, right = 0, len(nums) - 1
    for left < right {
        mid := (right - left) / 2 + left

         if nums[mid] >= target {
            right = mid
        } else {
            left = mid + 1
        }
    }

    if len(nums) == 0 || nums[left] != target {
        return -1
    }
    
    return result + left
}
```
