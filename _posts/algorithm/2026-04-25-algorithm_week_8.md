---
title: 二分探索
published: true
categories: [algorithm]
---


## <span style="color:#802548">_実装ー二分探索を再帰で実装_</span>

- 整列された前提で探索をする
- 左と右は先頭と末尾になる
- 二分探索のベースケースは左が右を超えるとき
- 中央値はオーバーフローにならないように以下とする
- 中央値よを基準にして片方は参考する必要がなくなるため、そのままスキップする
- 結局中央値が対象値と同じくなることがコンピューター的な正解となる

```go
func binarySearchRecursive(arr []int, left, right, target int) int {
    if left > right {
        return -1
    }

    mid := left + (right-left)/2

    if arr[mid] == target {
        return mid
    } else if arr[mid] < target {
        return binarySearchRecursive(arr, mid+1, right, target)
    } else {
        return binarySearchRecursive(arr, left, mid-1, target)
    }
}

func main() {
    arr := []int{1, 3, 5, 7, 9, 11}

    index := binarySearchRecursive(arr, 0, len(arr)-1, 7)

    fmt.Println(index) // Output: 3
}
```


## <span style="color:#802548">_実装ー二分探索を反復文で実装_</span>

- できなかったら止める（ベースケース）の真逆の条件でループを回す
- 再帰と同じく右と左の中で片方だけ必要なため、選ばれなかった片方はスキップする

```go
func binarySearch(arr []int, target int) int {
    left, right := 0, len(arr)-1

    for left <= right {
        mid := left + (right-left)/2

        if arr[mid] == target {
            return mid
        } else if arr[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }

    return -1 // not found
}
```

- 二分探索が線形探索より性能に優れる理由は時間計算量に起因する
    - Linear search → O(n)
    - Binary search → O(log n)
        - Linear search → up to 1,000,000 checks
        - Binary search → about 20 checks

```text
 1 回目 → n/2
 2 回目 → n/4
 3 回目 → n/8
 k 回目 → n / 2^k

n/2^k = 1;
n=2^k;
k=log2​n
```


## <span style="color:#802548">_実装ー中央値計算する際異にオーバーフローを防ぐロジック_</span>
- 以下のように実装したら、オーバーフローになってしまう
- If left and right are very large integers, their sum can exceed the maximum value of int.

```go
mid := (left + right) / 2
left = 2,000,000,000
right = 2,100,000,000

//left + right --> overflow
```

- なので、以下のようにする

```go
mid := left + (right-left)/2
```




## <span style="color:#802548">_深く探求ーlower_boundとupper_bound実装_</span>

- lower boundはtarget未満ではない値が最初に現れる位置

```text
arr = [1, 2, 2, 2, 4, 5]

target = 2  → lower_bound = 1
target = 3  → lower_bound = 4
target = 0  → lower_bound = 0
target = 6  → lower_bound = 6 (end)
```

- lower_boundは >= targetであるため、そうでない場合、arr[mid] < targetはもう１度繰り返して探す
- lower boundは最終的には右と左が同じくなるが、慣例上、左を返す
    - それに加えて、左には条件を満たす最初のインデックスという意味も込められているので、左にする
    - ただ、これは繰り返しの条件が left < rightのみの話

    Why right = mid in the else?

The else means:

arr[mid] > target

So:

👉 mid is a valid candidate for upper bound
👉 BUT there might be an earlier one on the left

So we do:

right = mid

which means:

"keep mid, and continue searching left"



```go
func lowerBound(arr []int, target int) int {
    left, right := 0, len(arr)

    for left < right {
        mid := left + (right-left)/2

        if arr[mid] < target {
            left = mid + 1
        } else {
            right = mid
        }
    }

    return left
}
```                 

- 以下も同じでが、そこに潜んでいる考えが違う
- 

```java
static int lowerBound(int[] arr, int target) {
    int low = 0, high = arr.length - 1;
    int res = arr.length;
    
    while (low <= high) {
        int mid = low + (high - low) / 2;
        
        // If arr[mid] >= target, then mid can be the
        // lower bound, so update res to mid and
        // search in left half, i.e. [lo...mid-1]
        if (arr[mid] >= target) {
            res = mid;
            high = mid - 1;
        }
        
        // If arr[mid] < target, then lower bound
        // cannot lie in the range [lo...mid] so
        // search in right half, i.e. [mid+1...hi]
        else {
            low = mid + 1;
        }
    }
    return res;
}

public static void main(String[] args) {
    int[] arr = {2, 3, 7, 10, 11, 11, 25};
    int target = 9;
    
    System.out.println(lowerBound(arr, target));
}
```


Key Differences
1. Need for res
Your version → ✅ needs res
Boundary version → ❌ no extra variable

👉 Less state = fewer bugs

2. Edge Case Handling

Your version must carefully initialize:

int res = n; // or -1 depending on definition

Otherwise:

If no element ≥ target → ❌ wrong result

The boundary version:

automatically returns n (insertion point)
no special handling needed
3. Loop Condition
Your version → low <= high
Boundary version → left < right

👉 The second one guarantees:

no infinite loop
clean convergence to a single point
4. Conceptual Difference

Your approach is:

"Find answer while searching"

Boundary approach is:

"Find the boundary directly"

This boundary idea is extremely powerful:

[ < target | >= target ]
            ^



- upper boundは targetより大きい値が最初に現れる位置

```text
arr = [1, 2, 2, 2, 4, 5]

target = 2

lower_bound → 1（最初の「2」）--> bound is index, not value
upper_bound → 4（最初の「2より大きい値」= 4）
```


- upper_bound > targetであるため、そうでない場合、arr[mid] <= targetはもう１度繰り返して探す
- upper_boundは最終的には右と左が同じくなるが、慣例上、左を返す
    - それに加えて、左には条件を満たす最初のインデックスという意味も込められているので、左にする
    - ただ、これは繰り返しの条件が left < rightのみの話
```go
func upperBound(arr []int, target int) int {
    left, right := 0, len(arr)

    for left < right {
        mid := left + (right-left)/2

        if arr[mid] <= target {
            left = mid + 1
        } else {
            right = mid
        }
    }

    return left
}
```


- lower_bound / upper_boundまで来たら、実は二分探索はただの値を探すのではなく、上記の条件が満たされる領域を探すことだということに気づく


## <span style="color:#802548">_深く探求ー1番目の登場位置探す_</span>

```go
func firstOccurrence(arr []int, target int) int {
    left, right := 0, len(arr)-1
    ans := -1

    for left <= right {
        mid := left + (right-left)/2

        if arr[mid] == target {
            ans = mid        // record answer
            right = mid - 1  // move LEFT to find earlier one
        } else if arr[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }

    return ans
}
```


```go
func firstOccurrence(arr []int, target int) int {
    left, right := 0, len(arr)

    for left < right {
        mid := left + (right-left)/2

        if arr[mid] < target {
            left = mid + 1
        } else {
            right = mid
        }
    }

    if left < len(arr) && arr[left] == target {
        return left
    }
    return -1
}
```


```go
import "sort"

i := sort.Search(len(arr), func(i int) bool {
    return arr[i] >= target
})

if i < len(arr) && arr[i] == target {
    return i
}
return -1
```

