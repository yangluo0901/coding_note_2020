

## Priority of Bit operators

| Precedence | Operator                                         | Associativity |
| ---------- | ------------------------------------------------ | ------------- |
| 1          | ~(Bitwise negation)                              | Right to left |
| 2          | <<(Bitwise Left Shift) , >>(Bitwise Right Shift) | Left to Right |
| 3          | & (Bitwise AND)                                  | Left to Right |
| 4          | ^(Bitwise XOR)                                   | Left to Right |
| 5          | \| (Bitwise Or)                                  | Left to Right |

## Set range of bits as 1

```java
range = (((1 << (l - 1)) - 1) ^ ((1 << (r)) - 1));
```



### check bit

`& 1`  will check if origin bit is 1 or not, `n >> index & 1` will check bit at index, `0100 >> 2 & 1` --> `0001 & 0001 = 0001 = 1`

`| 1` will set bit, `0011 | 0010 = 0010`

`& 0` will set bit into 0

`| 0` will check bit.



### Gray code

1. #### **Convert Binary number to Gray number**: 

   + keep the **Most Significant Bit** (MSB);
   + XOR each pair of adjacent bits

   ##### **<u>Example: convert A= `110111`,</u>**

   + the most left (5th from right side) bit `1` is the MSB, the **Res** = `1 _ _ _ _ _ `
   + then `A[5] ^ A[4] = 1 ^ 1 = 0`,  **Res** = `1 0 _ _ _ _ `
   + `A[4] ^ A[3] = 1 ^ 0 = 1`,  **Res** = `1 0 1 _ _ _ `
   + `A[3] ^ A[2] = 0 ^ 1 = 1`,  **Res** = `1 0 1 1 _ _ `
   + `A[2] ^ A[1] = 1 ^ 1 = 0`,  **Res** = `1 0 1 1 0 _ `
   + `A[1] ^ A[0] = 1 ^ 1 = 0`,  **Res** = `1 0 1 1 0 0`

   ##### **<u>formula</u>**

   ​	we noticed that `A[i] ^ A[i - 1]`, then formula is `A ^ A >> 1`

   ```java
   int binary2Gray(int n){
       return n ^ n >> 1;
   }
   ```

   

2. #### Convert Gray to Binary

   + keep the MSB
   + XOR the MSB with the next bit, get the new bit, lets call it `nb`
   + XOR the `nb` with the next bit, get new `nb`
   + keep doing until the end

```java
int gray2Binary(int n){
    int mask = n >> 1;
    while(mask > 0){
        n = n ^ mask;
        mask = mask >> 1;
    }
    return n;
}
```

## Use bitmask as memo

[464. Can I Win -- Medium](https://leetcode.com/problems/can-i-win/)





## To be categorized

[136. Single Number -- Easy](https://leetcode.com/problems/single-number)

[137. Single Number II -- Medium](https://leetcode.com/problems/single-number-ii/)

```java
 public int singleNumber(int[] nums, k) { // every number occurs k times excepts for one
        /*
            11011
            10001
            11011
            11011
            if we add all digit at the ith positio, and sum % 3 = 0
            bit[0] = 1 + 1 + 1 + 1 = 4 % 3 = 1, this 1 belongs to the single number at 0th
            bit[1] = 1 + 0 + 1 + 1 = 3 % 3 = 0, digit at the single number 1th is 0
            bit[2] = 0 + 0 + 0 + 0 = 0 % 3 = 0, digit at the single number 2th is 0
            bit[3] = 1 + 0 + 1 + 1 = 3 % 3 = 0, digit at the single number 3th is 0
            bit[4] = 1 + 1 + 1 + 1 = 4 % 3 = 1, this 1 belongs to the single number at 4th
            single number is 10001
        */
        int ans = 0;
        // for each position
        for(int i = 0; i < 32; i++){
            // add digit at ith position for each number
            int count = 0;
            for(int j = 0; j < nums.length; j++){
                if((nums[j] >> i & 1) == 1){
                    count ++;
                }
                if(count % k != 0){// there is an extra, mod must be either 1 or 0
                    ans |= 1 << i;
                }
            }
        }
        return ans;
    }
```
