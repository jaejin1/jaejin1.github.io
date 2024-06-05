# [Leetcode] 1456. Maximum Number of Vowels in a Substring of Given Length


https://leetcode.com/problems/maximum-number-of-vowels-in-a-substring-of-given-length/description/

<!--more-->

처음엔 s = "abciiidef", k = 3 이라고 하면 3개씩 잘라가면서 모음을 확인하고 max값을 찾았는데 당연히 시간 초과 발생함.

max값을 찾는 것이고 3개라는 길이가 정해져 있으므로 굳이 매번 모음을 확인 할 필요없이 s 값을 처음부터 진행하면서 counting 시작하고 맨 끝에 하나, 맨 뒤 하나 모음을 확인하며 count를 하고 max 값이랑 비교

```go
func maxVowels(s string, k int) int {
    index := k
    dpString := s[:k]

    count := checkExist(dpString)
    max := count
    
    for index < len(s) {
        lastString := string(s[index])

        count += checkExist(lastString)
        count -= checkExist(string(dpString[0]))

        dpString = dpString[1:] + lastString

        if max < count {
            max = count
        }
        index++
    }

    return max
}

func checkExist(s string) int {
    vowel := []string{"a", "e", "i", "o", "u"}
    count := 0
    
    for _, n := range s {
        for _, v := range vowel {
            if v == string(n) {
                count++
                break
            }
        }
    }

    return count
}
```
