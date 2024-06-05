# [Leetcode] 46. Permutations


https://leetcode.com/problems/permutations/description/

<!--more-->

보통 array의 모든 경우의 수를 구하는 문제가 나오면 backtracking을 생각하게 된다..

처음 작성할때는 tempList를 list에 그냥 append를 했었는데 몇가지 경우의 수에서 틀린 답을 뱉었는데

```go
if len(tempList) == len(nums) {
    *list = append(*list, tempList)
}
```

go내부에서 slice는 주소값을 참조하고 있어서 값이 변하면 영향을 받을 꺼고 이를 위해 다음과 같이 새로운 메모리 영역을 할당하고 복사하면 문제 없이 통과했다.

```go
if len(tempList) == len(nums) {
    p := make([]int, len(tempList))
    copy(p, tempList)

    *list = append(*list, p)
}
```

전체 코드에서 또한 같은 값이 있는 부분을 contains 함수를 따로 정의해서 체크 했었는데 tempList를 계속 탐색하는 것보다 map을 하나 만들고 구현하면 어떤 차이가 있을까?

밑의 코드로 제출하면 leetcode시스템에서 runtime 0ms, memory 2.7MB 사용한다 나왔다. 

```go
func permute(nums []int) [][]int {
    var res [][]int

    backtracking(&res, []int{}, nums)
    return res
}

func backtracking(list *[][]int, tempList []int, nums []int) {
    if len(tempList) == len(nums) {
        p := make([]int, len(tempList))
        copy(p, tempList)

        *list = append(*list, p)
        return
    } else {
        for _, num := range nums {
            if contains(tempList, num) {
                continue
            }
            tempList = append(tempList, num)
            backtracking(list, tempList, nums)
            tempList = tempList[:len(tempList) - 1]
        }
    }

    return
}

func contains(s []int, sub int) bool {
    for _, v := range s {
        if v == sub {
            return true
        }
    }
    return false
}
```

이를 used라는 map을 하나 정의해서 contains 함수를 안돌게 변경하면?

runtime 3ms, memory 2.7 ~ 2.8MB 를 사용한다.

메모리는 약간 증가할 수 있다 생각했는데 왜 시간이 늘었지? map을 사용해서 반복을 줄였는데?

Constraints를 확인하니 num의 길이의 최대가 6이라고 한다. 

예전 기억으로 검색하는 범위가 일정 수준보다 작으면 hash 값으로 찾는 것보다 반복으로 돌려서 찾는게 빠르다는 것이 생각나네..

뭐 큰 차이는 아니니까 밑의 코드로 제출

```go
func permute(nums []int) [][]int {
    var res [][]int

    backtracking(&res, []int{}, nums, map[int]struct{}{})
    return res
}

func backtracking(list *[][]int, tempList []int, nums []int, used map[int]struct{}) {
    if len(tempList) == len(nums) {
        p := make([]int, len(tempList))
        copy(p, tempList)

        *list = append(*list, p)
        return
    } else {
        for _, num := range nums {
            if _, ok := used[num]; !ok {
                used[num] = struct{}{}

                tempList = append(tempList, num)
                backtracking(list, tempList, nums, used)

                delete(used, num)
                tempList = tempList[:len(tempList) - 1]
            }
        }
    }
    return
}
```
