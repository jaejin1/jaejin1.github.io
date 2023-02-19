# [Leetcode] 17. Letter Combinations of a Phone Number


https://leetcode.com/problems/letter-combinations-of-a-phone-number/description/

<!--more-->



```go
func letterCombinations(digits string) []string {
    res := []string{}
    if digits == "" {
        return res
    }

    digitsToLetters := map[string][]string{
        "2": { "a","b","c" },
        "3": { "d","e","f" },
        "4": { "g","h","i" },
        "5": { "j","k","l" },
        "6": { "m","n","o" },
        "7": { "p","q","r", "s" },
        "8": { "t","u","v" },
        "9": { "w","x","y", "z" },
    }


    letters := [][]string{}

    for _, digit := range digits {
        letters = append(letters, digitsToLetters[string(digit)])
    }


    for lettersIndex, letter := range letters {
        if lettersIndex == 0 {
            res = letter
            continue
        }

        temp := []string{}
        for _, r := range res { 
            for _, l := range letter {
                temp = append(temp, r + l)
            }
        }

        res = temp
    }

    return res
}
```
