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
    - 線形探索 → O(n)
    - 二分探索 → O(log n)
        - 線形探索 → 1,000,000 回
        - 二分探索 → 20 回

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

- lower_boundは arr[i] >= targetであるが、その値が始まる領域の境界値だと思えばいい
- 探す条件は二分探索と同じ
    - arr[mid] < targetはlower_boundの反対条件なので、左はもう見ることもないので、右に行く
        - 従って、left = mid + 1
    - しかし、else(arr[mid] >= target )の場合は、midも候補としてありえるので、midを保ちつつ左に行く
        - 従って、rigth = mid
- lower boundは最終的には右と左が同じくなるが、慣例上、左を返す
    - それに加えて、左には条件を満たす最初のインデックスという意味も込められているので、左にする

- left < rightの場合は、右側は考慮されない
- そのため、mid が右のインデックスになる可能性も考えなければならない
    - right = mid 
- right = mid - 1になってしまうと、[left, mid-1)になるため、正確な領域の設定ができなくなる

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
- これはもっと直感敵に理解しやすいが、領域という概念が中心ではない
- lower_boundはarr[i] >= targetであるため、必ずその条件で見つかる
- そのため、else文では見つからない
- low < highであった条件式が low <= highに変わったことが見られる
- left <= rightは、二分探索とロジックが同じなので、理解しやすいが、余計な変数が必要とされる

```java
static int lowerBound(int[] arr, int target) {
    int low = 0, high = arr.length - 1;
    int res = arr.length;
    
    while (low <= high) {
        int mid = low + (high - low) / 2;
        
        if (arr[mid] >= target) {
            res = mid;
            high = mid - 1;
        } else {
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

- [left, right)と[left, right]はロジックが違くなる
- [left, right)のほうがよりソースコードとしては完成度が高い
    - 余計な変数が必要
    - 無限ルールを防げる

- upper boundは targetより大きい値が最初に現れる位置

```text
arr = [1, 2, 2, 2, 4, 5]

target = 2

lower_bound → 1（最初の「2」）--> bound is index, not value
upper_bound → 4（最初の「2より大きい値」= 4）
```


- upper_boundは arr[i] > targetであるが、その値より大きい値が始まる領域の境界値だと思えばいい
- 探す条件は二分探索と同じ
    - arr[mid] <= targetはupper_boundの反対条件なので、左はもう見ることもないので、右に行く
        - 従って、left = mid + 1
    - しかし、else(arr[mid] > target )の場合は、midも候補としてありえるので、midを保ちつつ左に行く
        - 従って、rigth = mid
- upper_boundは最終的には右と左が同じくなるが、慣例上、左を返す
    - それに加えて、左には条件を満たす最初のインデックスという意味も込められているので、左にする

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

- 二分探索とほぼ同じだが、最初の位置を求めてるため、mid値を見つけても、即座にリターンするわけにはいかない
- もっと左に移動して探し続けて最初の位置を返す

```go
func firstOccurrence(arr []int, target int) int {
    left, right := 0, len(arr)-1
    ans := -1

    for left <= right {
        mid := left + (right-left)/2

        if arr[mid] == target {
            ans = mid        
            right = mid - 1  
        } else if arr[mid] < target {
            left = mid + 1
        } else {
            right = mid - 1
        }
    }

    return ans
}
```


- 同じ結果を作る関数だが、領域を決める bound類のロジックに近い


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


- go langのAPIで以下のように簡潔にできる

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

