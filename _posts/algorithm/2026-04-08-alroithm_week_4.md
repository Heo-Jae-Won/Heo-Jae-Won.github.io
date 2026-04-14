---
title: マージソート
published: true
categories: [algorithm]
---


## <span style="color:#802548">_実装ーマージソート昇順_</span>
- 最初に工夫して書いたソースコードだけど、正しく動作しない
- 前半部と後半部が分かれるのはわかるので、折半にすることまでは思いついたが、ベースケースの定義が全然間違っている
- なのでmergeSort()を置く位置も間違えてしまった
- その上に、CallStackがソースコードで展開するとどのようにメソッドが重なるのか理解をちゃんとしてなかった

```java
public static void mergeSort(int[] arr) {
    System.out.println(arr.length);
    int[] beforeArray = new int[arr.length / 2];
    int[] afterArray = new int[arr.length / 2];

    for (int i = 0; i < arr.length / 2; i++) {
        if(arr.length == 1) {
            return;
        }
        beforeArray[i] = arr[i];

        if(i == arr.length / 2 - 1) {
            System.out.println(Arrays.toString(beforeArray));
            mergeSort(beforeArray);
        }

    }

    for (int i = arr.length /2 ; i < arr.length ; i++) {
        if(arr.length == 1) {
            return;
        }
        afterArray[i] = arr[i];

        
        if(i == arr.length - 1 ) {
            System.out.println(Arrays.toString(afterArray));
            mergeSort(afterArray);
        }

    }
}
```


- 前のアルゴリズムを復習しながら、ベースケースの意味をしっかり理解し、修正を加えた
- 今回は、コールスタックを手書きしながら、どのように展開されるのかを確認しつつソースコードを作成した
- このようにして、前半部と後半部に分けるところまではできるようになった


```go
func MergeSort(arr []int) {
	if len(arr) == 1 {
		return
	}

	beforeArray := []int{}
	afterArray := []int{}

	for i := 0; i < len(arr)/2; i++ {
		beforeArray = append(beforeArray, arr[i])
	}

	for i := len(arr); i > len(arr)/2; i-- {
		afterArray = append(afterArray, arr[i-1])
	}
	fmt.Println(beforeArray)
	fmt.Println(afterArray)
	MergeSort(beforeArray)
	MergeSort(afterArray)
}
```

- 後ろから要素を取るのは順序が変わってしまうため、以下のように直した

```go
for i := len(arr) / 2; i < len(arr); i++ {
    afterArray = append(afterArray, arr[i])
}
```


- ただし、マージのロジックがどうしても思いつかなかった
- マージするには別途の関数が必要だという結論には至ったが、以前の配列などをどのように引き継いで扱うのかが分からなかった
- ChatGPTに聞いたところ、変数に再帰関数を代入するという発想が必要だった
- 再帰関数を変数の代入にも使えるなんて、思ったこともなかったので、勉強になった

```text
mergeSort(arr):
    if length <= 1:
        return arr

    mid = middle
    left = mergeSort(left half)
    right = mergeSort(right half)

    return merge(left, right)


merge(left, right):
    result = []

    while both have elements:
        take smaller one and append

    append remaining elements

    return result1
```

- ChatGPTにもらった疑似コードのおかげで、基本的な枠組みは完成した。

```go
func MergeSort(arr []int) []int {
	if len(arr) == 1 {
		return arr
	}

	beforeArray := []int{}
	afterArray := []int{}

	for i := 0; i < len(arr)/2; i++ {
		beforeArray = append(beforeArray, arr[i])
	}

	for i := len(arr) / 2; i < len(arr); i++ {
		afterArray = append(afterArray, arr[i])
	}
	fmt.Println(beforeArray)
	fmt.Println(afterArray)
	beforeArray = MergeSort(beforeArray)
	afterArray =  MergeSort(afterArray)

	return merge(beforeArray, afterArray)
}
```

- 上のソースコードは以下のような syntactic sugarが存在する
そして、スライスが空の場合に備えて、ベースケースの条件が変わっていることが分かる


```go
func MergeSort(arr []int) []int {
    if len(arr) <= 1 {
        return arr
    }

    mid := len(arr) / 2

    left := MergeSort(arr[:mid])
    right := MergeSort(arr[mid:])

    return merge(left, right)
}
```


- 次の問題は、どうやってマージするかだが、比較してマージするかとマージしてから比較するか、どちらを選ぶべきか悩んでいた
- 既存の整列アルゴリズムと同じようにするには、比較しながら整列してマージする必要があると考え、その方針で進めた
    - ただし、ロジックがなかなか思いつかなかった
    - のため、ChatGPTにロジックを教えてもらった
- まず、2つのスライスをマージする際は、それぞれの先頭要素を比較し、小さい方を結果の配列に追加する
- その際、結果の配列も常に先頭から順に最小値が並ぶようにする
- 要素を追加したら、その要素はもう不要になるため、対応するインデックスを進める
- もう一方の配列の要素の方が小さい場合は、そちらを結果に追加する
- この処理を繰り返すと、どちらか一方の配列の要素がすべてなくなるまでループが続く
- 最後に、残った配列の要素をそのまま結果の配列に追加する
    - 残っている要素がどちらの配列にあるか分からないため、それぞれに対してループ処理を用意する
    - 条件を満たさないループは実行されないため、パフォーマンスやロジックには影響しない

```go
func merge(left, right []int) []int {
    result := []int{}

    i := 0
    j := 0

    //while
    for i < len(left) && j < len(right) {
        if left[i] < right[j] {
            result = append(result, left[i])
            i++
        } else {
            result = append(result, right[j])
            j++
        }
    }

    // 最後の要素を処理
    for i < len(left) {
        result = append(result, left[i])
        i++
    }

    for j < len(right) {
        result = append(result, right[j])
        j++
    }

    return result
}
```

- 時間計算量は n x logn. 
    - 分割する過程が lognで、半分にすることを1になるまでに続くとlog2nだからだ
    - 一方、マージする過程が nで、要素を数分を演算するからだ

- わからなかったポイント
    - コールスタックの理解が足りていなかった
    - 再帰についての応用が足りていなかった
    - マージする際、最小限とか最大限などは各配列の先頭に来ること
    - 最後の要素を処理すること





## <span style="color:#802548">_実装ーマージソート降順_</span>

- 今回は降順を実装してみた
- 分割する過程は変わらない

```java
public int[] mergeSort(int[] arr) {
    if (arr.legnth <= 1 ) {
        return arr;
    }

    int[] leftArray;
    for(int i = 0; i < arr.legnth /2; i++) {
        leftArray = leftArray[i];    
    }

    int[] rightArray;
    for(int i = arr.length / 2; i < arr.length; i++) {
        rightArray = rightArray[i - arr.length / 2]
    }

    leftArray = mergeSort(leftArray);
    rigthArray = mergeSort(rightArray);

    return merge(leftArray, rightArray);
}
```

- ただ、Javaで配列を初期化するためには、最初に初期容量を決めなければならない
- そのため、以下のように変更する

```java
int mid = arr.length / 2;
int[] leftArray = new int[mid];
int[] rightArray = new int[arr.length - mid];

// Copy elements into left array
for (int i = 0; i < mid; i++) {
    leftArray[i] = arr[i];
}

// Copy elements into right array
for (int i = mid; i < arr.length; i++) {
    rightArray[i - mid] = arr[i];
}
```

- 降順のマージソートもあまり違くなるところはない

```java
public int[] merge(int[] leftArray, int[] rightArray) {
    int[] newArray = new int[leftArray.length + rightArray.length];
    int i = 0;
    int j = 0;
    int size = 0;
    while (i < leftArray.length && j < rightArray.length) {
        if (leftArray[i] > rightArray[j]) {
            newArray[size++] = leftArray[i++];
        } else {
            newArray[size++] = rightArray[j++];
        }
    }

    while (i < leftArray.length) {
        newArray[size++] = leftArray[i++];
    }

    while (j < rightArray.length) {
        newArray[size++] = rightArray[j++];
    }

    return newArray;
}
```


## <span style="color:#802548">_実装ー文字列マージソート_</span>

- 文字列の場合でも、基本的な考え方はあまり変わらない
- ただし、比較に使うメソッドが変わるだけである
- 以下は昇順

```java
if (leftArray[i].compareTo(rightArray[j]) > 0) {
    newArray[size++] = leftArray[i++];
} else {
    newArray[size++] = rightArray[j++];
}
```



## <span style="color:#802548">_深く探求ーStringとIntの違い_</span>

- 左と右の位置が入れ替わったら、intとStringは安定性の側面で結果が異なる
- intの場合は、別に安定性に問題がない
- それは int は別のメタデータなどを持ってないからだ

```java
if(rightArray[i] > leftArray[j] ) {
    newArray[size++] = rightArray[i++]
} else {
    newArray[size++] = leftArray[j++]
}
```

- Stringは少し違う
- Stringみたいなクラスはメタデータを持っている場合が多い
- 以下のようにすると、不安定のアルゴリズムになってしまう

```java
if (rightArray[i].compareTo(leftArray[j]) > 0) {
    newArray[size++] = rightArray[i++];
} else {
    newArray[size++] = leftArray[j++];
}
```

- 以下のString配列があると仮定してみた

```java
String[] arr = {"apple_1", "banana_2", "apple_3", "cherry_4"};
```

- 右側を基準とする場合、While文が進むにつれて状況は次のようになる。
- apple_1 が apple_3 より先にあったにもかかわらず、その順序が乱れてしまった。

```text
["apple_1", "banana_2", "apple_3", "cherry_4"]
          /                        \
["apple_1", "banana_2"]       ["apple_3", "cherry_4"]
     /         \                 /           \
["apple_1"] ["banana_2"]   ["apple_3"]   ["cherry_4"]
    \           /                \            /
["banana_2", "apple_1"]     ["cherry_4"] + ["apple_3"] 
            \                       /
["cherry_4", "banana_2", "apple_3", "apple_1"]

```

- そのため、安全性を保証するためには、左を基準にしたほうがいい
- 特に複数のフィールドを持つクラスの場合、不安定性の問題が顕著に現れる
- 1が先頭にあるはずなのに、３が先頭になってしまった

```text
Item[] arr = { ("apple", 1), ("banana", 2), ("apple", 3) }

ソート後の結果
("apple", 3), ("apple", 1), ("banana", 2)
```

## <span style="color:#802548">_深く探求ー部分的なマージソート_</span>

- 部分的にすると、JavaのAPIであるcopyOfRangeを使うと、便利になる

```java
public void executePartialMergeSort(int[] arr, int n, int y) {
    int[] arr = {1,1,2,5,6,8,3,6,11,20,25};

    int[] sub = Arrays.copyOfRange(arr, n, j); 

    sub = mergeSort(sub); 

    for (int i = 0; i < sub.length; i++) {
        arr[n + i] = sub[i];
    }
}
```


```C#
static void ExecutePartialMergeSort(int[] arr, int n, int y)
{
    int[] sub = arr.Skip(n).Take(y - n).ToArray();

    sub = MergeSort(sub);

    for (int i = 0; i < sub.Length; i++)
    {
        arr[n + i] = sub[i];
    }
}
```


```go
func ExecutePartialMergeSort(arr []int, n int, y int) {
    
	sub := append([]int(nil), arr[n:y]...)

	// Step 2: Sort the subarray using merge sort (descending)
	sub = MergeSort(sub)

	// Step 3: Put it back into original array
	for i := 0; i < len(sub); i++ {
		arr[n+i] = sub[i]
	}
}
```


## <span style="color:#802548">_振り返り_</span>
- 再帰を多彩の活用に気づけなかった


```go
func MergeSort(arr []int) []int {
    .
    .
    .
    beforeArray = MergeSort(beforeArray)
    afterArray =  MergeSort(afterArray)
}
```

- 再帰関数が１つだけで済むと勘違いしていた

```go
func MergeSort(arr []int) []int {
    .
    .
    .
	beforeArray = MergeSort(beforeArray)
	afterArray =  MergeSort(afterArray)

	return merge(beforeArray, afterArray)
}
```


- 必ず先頭のインデックスには最小とか最大の数値が並ぶことがわからなかった
- 振り子のように、左と右のスライスを行ったり来たりすることをどうやって実装するのかピンとこなかった

```go
func merge(left, right []int) []int {
    result := []int{}

    i := 0
    j := 0

    //while
    for i < len(left) && j < len(right) {
        if left[i] < right[j] {
            result = append(result, left[i])
            i++
        } else {
            result = append(result, right[j])
            j++
        }
    }
    .
    .
    .

    return result
}
```


- 最後の値は触れられてない状態なので、以下の通り、最後の値を入れる処理が必要だと思えなかった

```go
// 最後の要素を処理
for i < len(left) {
    result = append(result, left[i])
    i++
}

for j < len(right) {
    result = append(result, right[j])
    j++
}
```