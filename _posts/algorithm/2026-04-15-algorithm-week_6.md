---
title: Heap sort
published: true
categories: [algorithm]
---

## <span style="color:#802548">_実装ーheap sort_</span>
- 最初ダイアグラムとかを見たが、さっぱり理解ができなかった
- なので、少しヒントをくれと言ってルールや原則なども貰ったけど、それでも何も理解できなかった

```text
Araryはツリーとして捉えられる

MaxHeapを作る 
MAX値を末尾に移動させる
heapify
繰り返し
```

## <span style="color:#802548">_実装ー配列をツリーとして捉える_</span>
- 上の文章の意味が当時はいったい何を指してるのかわからなかった全投げするしかなかった
- それでソースコードをもらったが、これを見ても全く理解ができなかったので、もう1度CHATGPTに詳しく聞いた
- 詳しく聞いてからは、配列は特別なツリーの表現として捉えられるという文章の意味が分かった
- こういう風に解釈すれば、配列もツリーとして捉えられる

```go
largest := i
left := 2*i + 1
right := 2*i + 2
```

## <span style="color:#802548">_実装ーＭＡＸＨＥＡＰを作る_</span>
- それを理解してから、MaxHeapというものを調べた
- 最初は MaxHeapを作る
- MaxHeapを作るためのループの対象となるインデックスは子がある親要素だけだ
- そのため、n/2 -1になる
- そこからMaxHeapを作る関数を各親に対して呼び出す

```go
for i := n/2 - 1; i >= 0; i-- {
	heapify(arr, n, i)
}
```

- 1番大きいものをわかる必要がない
- 1番大きいのを親として仮定してみて、そうでなかったら、子と親を入れ替えながら、一番大きいものを上にあげるロジックだ
- そのため、親に対しての子たちの要素のインデックスを計算する必要がある
	- それが left, right
- ただ、最後の親とかの場合、Rightなどの子たちが存在しない場合もあるし、Leftの子たちも存在しない場合もある
- そのため、スワップのロジックはまず、スワップの範囲で行われることを確実にする
	- それが left < n と rigth < n
- 子たちが親より大きいかったら、子たちのインデックスを代入して、あとで変える
	- それが  largest = left、　largest = right、 if largest != i { arr[i], arr[largest] = arr[largest], arr[i]
- 入れ替わったインデックスによって、変わったインデックスも親の可能性があるので、もう1度確認して heapifyする
	- heapify(arr, n, largest)

```go
func heapify(arr []int, n int, i int) {
	largest := i
	left := 2*i + 1
	right := 2*i + 2

	if left < n && arr[left] > arr[largest] {
		largest = left
	}

	if right < n && arr[right] > arr[largest] {
		largest = right
	}

	if largest != i {
		arr[i], arr[largest] = arr[largest], arr[i]
		heapify(arr, n, largest)
	}
}
```

## <span style="color:#802548">_実装ー最大値を末尾に移動させてから安定化_</span>
- 次は以下の段階に進む
- nがサイズなので、ー１してインデックスと合わせる
- MAXになった最初の値と iインデックスを入れ替えながら、最大値をデータを末尾に移動させる
- その移動によって乱れた順序を修復するため、heapify() を実行する
- 最後の i = 0はもうソートされてる状態なので、ループを回す必要がない

```go
// 2. Extract elements one by one
for i := n - 1; i > 0; i-- {
	// Move current root to end
	arr[0], arr[i] = arr[i], arr[0]

	// Heapify reduced heap
	heapify(arr, i, 0)
}
```

- もう順序通りになって移動させる必要がなくなった要素を除いてまだソートされてないものだけを対象とする
	- left < n, right < n
- 最初にMaxHeapを作る際に left, right < n の役割とは違くなる
	- あの時の役割は、インデックスがない子要素に対するアクセスを防ぐためだった

```go
func heapify(arr []int, n int, i int) {
	largest := i
	left := 2*i + 1
	right := 2*i + 2

	if left < n && arr[left] > arr[largest] {
		largest = left
	}

	if right < n && arr[right] > arr[largest] {
		largest = right
	}

	if largest != i {
		arr[i], arr[largest] = arr[largest], arr[i]
		heapify(arr, n, largest)
	}
}
```

- HeapSortのメリットは以下の通り
	- メモリの利用率が低い
		- Nodeとかポインターとかがない
	- Cache-friendly
		- メモリ上で連続して配置されている
	- インデックスだけ計算すればいい
- HeapSortのデメリットは以下の通り
	- 安定的でない


## <span style="color:#802548">_実装ー時間計算量_</span>
- MAXHEAPを作る過程が O(n logn)のように見えるかもしれない
	- しかし、要素ごとに同じ必要で計算してるわけではない
	- 下の子要素鯛はやすいコストで計算する
		- 葉ノードは高さ0なので、heapify()が呼び出されても演算が行われない
	- 上の親要素は高いコストで計算する
		- 一番上位に近いノードは高さが大きくて演算が多く行われる
- そのため、全部合わせたら、平均 O(n)になる

```text
        ●        ← height 3 (1 node)
      ●   ●      ← height 2 (2 nodes)
     ● ● ● ●     ← height 1 (4 nodes)
    ● ● ● ● ● ● ● ●  ← height 0 (8 nodes, leaves)
```

- MAX HEAPから最大値を取り出すソートフェーズは以下のソースコード

```java
for (int i = n - 1; i > 0; i--) {
    swap(arr[0], arr[i]);
    heapify(arr, i, 0);
}
```

- スワップはo(1)で、heapify()はさっき言った通りo(logn)
- そこでFor文で n回繰り返すので o(n logn)になる

```java
swap O(1)
heapify O(log n)
loop runs n - 1 times ≈ n
```

- O(n) + O(n log n) は漸近的に O(n log n) に等しい


## <span style="color:#802548">_実装ーheap sort降順_</span>

- arr[left] < arr[largest]と arr[right] < arr[largest]のように矢印の方向だけ変えればいい
- それと誤解を払しょくするため、変数名をlargestからsmallestに変える

```go
package main

import "fmt"

// Heapify function (fix subtree rooted at i)
func heapify(arr []int, n int, i int) {
	smallest := i
	left := 2*i + 1
	right := 2*i + 2

	if left < n && arr[left] < arr[smallest] {
		smallest = left
	}

	if right < n && arr[right] < arr[smallest] {
		smallest = right
	}

	if smallest != i {
		arr[i], arr[smallest] = arr[smallest], arr[i]
		heapify(arr, n, smallest)
	}
}

// Heap Sort
func heapSort(arr []int) {
	n := len(arr)

	// 1. Build Max Heap
	for i := n/2 - 1; i >= 0; i-- {
		heapify(arr, n, i)
	}

	// 2. Extract elements one by one
	for i := n - 1; i > 0; i-- {
		// Move current root to end
		arr[0], arr[i] = arr[i], arr[0]

		// Heapify reduced heap
		heapify(arr, i, 0)
	}
}

func main() {
	arr := []int{4, 10, 3, 5, 1}

	fmt.Println("Before:", arr)
	heapSort(arr)
	fmt.Println("After: ", arr)
}
```

