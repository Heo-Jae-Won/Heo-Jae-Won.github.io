---
title: クイックソート
published: true
categories: [algorithm]
---

## <span style="color:#802548">_実装ークイックソートOOP版_</span>
- クイックソートはマージソートと同じ仕組みだと思い、そういう方向で進んでいった
- なので、マージソートのように半分に分割して、新しい配列に再起関数を代入する形だとてっきり思った
- だが、ピボットになるインデックスをどうやって決めればいいのか、それが全然わからなかった
- 結局、すべてAIに頼るしかなかった
- その結果として得られたのが以下のロジックである
- ここではLowとHighというポインターみたいな概念が必要だった

 ```java
 public class QuickSort {

    public static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            int pivotIndex = partition(arr, low, high);

            quickSort(arr, low, pivotIndex - 1);   // left side
            quickSort(arr, pivotIndex + 1, high);  // right side
        }
    }

    private static int partition(int[] arr, int low, int high) {
        int pivot = arr[high]; 
        int i = low - 1; 
        for (int j = low; j < high; j++) {
            if (arr[j] < pivot) {
                i++;
                swap(arr, i, j);
            }
        }

        swap(arr, i + 1, high);
        return i + 1;
    }

    private static void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```

- ベースケースが見えなくなる短所はあるので、以下のように明確にする
- ベースケースの意味はlowとhighが一緒の場合は、分割が意味がなくなるため、必要ではないということ
- ピボットインデックスを選択したら、左と右を分けるパーティションを作ってスワップする
- 最初は メソッドの名前を見ても全然意味が分からなかった

```java
public static void quickSort(int[] arr, int low, int high) {
    if (low >= high) {
        return;
    } 
    int pivotIndex = partition(arr, low, high);

    quickSort(arr, low, pivotIndex - 1);   // left side
    quickSort(arr, pivotIndex + 1, high);  // right side
}
```

- このメソッドの役割は決まったピボットに応じて、ピボットを境界線に大きいものと小さいものを分けることがある
- そうやって分けてからそこで次のピボットを決めるという責任もある
- OOPでは２つの責任を避けることがおすすめされる
- そのため、２つのメソッドに分ける
- 時間計算量は n log nじゃなくて、2 * n log nになるけど、O（nlogn）には間違えない
- ただ、大量のデータになると、そういった余分も大事になるため、古典的なクイックソートを使うことになる

```java
public static void quickSort(int[] arr, int low, int high) {
    if (low >= high) return;

    int pivotIndex = calculatePivotIndex(arr, low, high);

    rearrangeAroundPivot(arr, low, high, pivotIndex);

    quickSort(arr, low, pivotIndex - 1);
    quickSort(arr, pivotIndex + 1, high);
}

```


- 実際のメソッドを見てみよう
- ピボットのインデックスを計算するロジックは、自分より小さいやつを数えることだ
- 概念的には、それで得られたインデックスに挿入する形だけど、プログラミング的にはスワップで実装する
- For文の範囲としては、Highは除外される
    - その理由は、上でPivotとして最後のインデックスを使ったからだ
- 重複の場合も扱えるよう、＜＝になっている

```java
// Responsibility 1: Calculate where the pivot will end up
private static int calculatePivotIndex(int[] arr, int low, int high) {
    int pivot = arr[high];
    int notBiggerCount = 0;

    for (int i = low; i < high; i++) {
        if (arr[i] <= pivot) {
            notBiggerCount++;
        }
    }

    return low + notBiggerCount;
}
```

- ピボットをピボットのインデックスに移動させる
- 左からのピボットに移動する i の値と、右からピボットに移動しながら、スワップの対象となる値を探す
- ピボットを軸にスワップするインデックスを見つけたら、スワップし、スワップが終わった後、次のインデックスに移動する
- ダブルWhile文だといっても、時間計算量はO(n)になる
    - 巻き戻しができない仕組みであるため、ダブルFor文みたいに最初から要素を見直すことではないからだ

```java
private static void rearrangeAroundPivot(int[] arr, int low, int high, int pivotIndex) {
    int pivot = arr[high];

    swap(arr, pivotIndex, high);

    int i = low;
    int j = high;

    //  ソートが終わったら、i と　j が
    while (i < pivotIndex && j > pivotIndex) {

        //ピボットより小さい値がない場合は、該当するインデックスをスキップする
        while (arr[i] <= pivot && i < pivotIndex) {
            i++;
        }

        //ピボットより大きい値がない場合は、該当するインデックスをスキップする
        while (arr[j] > pivot && j > pivotIndex) {
            j--;
        }

        //i と j がまだピボットにたどり着けなかったということはスワップに対象になる要素があること
        if (i < pivotIndex && j > pivotIndex) {
            swap(arr, i, j);
            i++;
            j--;
        }
    }
}
```


- 例えば、次の配列があったとしたら、一回目で以下のようになる
    - ７がピボットになる
    - ７より小さい数値の分をピボットのインデクスにしてそこに挿入する（実際にはスワップ）
    - ４，２，３は自分自身とスワップすることになるから、不要なスワップをしてしまうが、単純スワップは操作のコストが低いかつロジックの簡潔さを保てる
    - １１に当たって、i は増えるけど、Leftというポインターは増えないから、次の変換対象になる小さいRight側の１に当たったら、１１と１のスワップができる

```text
{4,2,3,11,8,9,1,7}
        |
{4,2,3,11,7,9,1,8}
        |
{4,2,3,1,7,9,11,8}
```


## <span style="color:#802548">_実装ークイックソートLomuto版_</span>

- 上の２つに分かれてるメソッドを１つにまとめると以下になる
- partitionを分ける過程でボットを中心に右と左に分けることになる

```java
public class QuickSortLomuto {

    public static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            int p = partition(arr, low, high);
            quickSort(arr, low, p - 1);
            quickSort(arr, p + 1, high);
        }
    }
}
```

- i はピボットよりも小さい数値が入る領域の空間を確保するインデックスになる
    - そのため、最初はアクセスできないインデックスから始まる
    - 小さい領域を1つ広げるかつ小さい値を左に入れるための if文
        - いったん全部小さいと大きい領域に分けてから再起しながらソートする

```java
private static int partition(int[] arr, int low, int high) {
    int pivot = arr[high];
    int i = low - 1; // boundary of smaller elements

    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(arr, i, j);
        }
    }

    // place pivot
    swap(arr, i + 1, high);
    return i + 1;
}
```

- 領域分けが終わったら、以下の状態になる
- ピボットを本来あるべき位置に配置するため、i と ピボットをスワップする
- i = 小さい値の最後の indexなので、pivot の場所はその1つ右

```text
[ 小さい値 ] [ 大きい値 ] pivot
            i
```


- 時間計算量は以下になる
- パーティションを作る動作は最悪のケースでもO(n)になる

```java
for (int j = low; j < high; j++) {
    if (arr[j] < pivot) {
        i++;
        swap(arr, i, j);
    }
}
swap(arr, i + 1, high);
```

- ただ、再起は違う
- 普通のケースではT(n) = T(n/2) + T(n/2) + O(n)なので、O(n log n)になる
- 最悪の場合は、分けることが１つ筒になってしまい、T(n) = T(n-1) + O(n)になってしまい、o(n^2)になる
    - 特にとっくにソートされた配列
    - 逆にソートされた配列
    - ピボットの選択が間違えた場合

## <span style="color:#802548">_深く探究ー3点中央値_</span>

- ピボットの選択による最悪なケースを減らすため、中央値を探す方法もある
- こうやって中央値に近い可能性を高めて最悪のケースになる可能性を減らす
- 最初の段階に中央値に近い値を探して、末尾に置く

```java
private static int partition(int[] arr, int low, int high) {
    int mid = (low + high) / 2;
    if (arr[low] > arr[mid]) swap(arr, low, mid);
    if (arr[low] > arr[high]) swap(arr, low, high);
    if (arr[mid] > arr[high]) swap(arr, mid, high);
    swap(arr, mid, high);

    int pivot = arr[high];
    int i = low - 1; 

    for (int j = low; j < high; j++) {
        if (arr[j] < pivot) {
            i++;
            swap(arr, i, j);
        }
    }

    swap(arr, i + 1, high);
    return i + 1;
}
```


- 以下を実行すると、arr[low] <= arr[mid]になる

```java
int mid = (low + high) / 2;
if (arr[low] > arr[mid]) swap(arr, low, mid);
```

- 以下を実行すると、arr[low] <= arr[high]

```java
if (arr[low] > arr[high]) swap(arr, low, high);
```
- lowはmidとhighどちらと比べても小さくなった
- 以下を実行すると、arr[mid] <= arr[high]

```java
if (arr[mid] > arr[high]) swap(arr, mid, high);
```


- すなわち、以下の連続の if文で arr[low] ≤ arr[mid] ≤ arr[high]になる

```java
if (arr[low] > arr[mid]) swap(arr, low, mid);
if (arr[low] > arr[high]) swap(arr, low, high);
if (arr[mid] > arr[high]) swap(arr, mid, high);
```

- midとhighをスワップすることで、中央値を末尾に配置できる

```java
swap(arr, mid, high);
```



## <span style="color:#802548">_深く探究ークイックソートHoare版_</span>
- Lomuto版じゃなくて、Hoare版もある
    - ピボットを最後に配置せず、ひとえに境界だけを設定する
    - 左と右の比較を同時に進行して、間違えたら修正

```text
[ smaller-or-equal | larger-or-equal ]
          ↑
         boundary (p)
```

- そのため、再帰するインデックスも変わる

```go
//Lomuto
quickSort(arr, low, p - 1);        
quickSort(arr, p + 1, high);

//Hoare
quickSort(arr, low, p);        
quickSort(arr, p + 1, high);
```

- 小さい領域だけじゃなくて、大きい領域も新しくできた
    - j がそれを代表する
- 小さい慮域でピボットより小さい値の場合はスキップする
- 大きいの場合は、真逆のロジックで動作する
- 両方で間違えた値を見つけたら、スワップする
- i が　jと交差する瞬間、パーティション処理が終わったということなので、ソートを抜けるベースケースになる

```java
public class QuickSortHoare {

    public static void quickSort(int[] arr, int low, int high) {
        if (low < high) {
            int p = partition(arr, low, high);
            quickSort(arr, low, p);        // ⚠️ different!
            quickSort(arr, p + 1, high);
        }
    }

    private static int partition(int[] arr, int low, int high) {
        int pivot = arr[low];
        int i = low - 1;
        int j = high + 1;

        while (true) {
            do {
                i++;
            } while (arr[i] < pivot);

            do {
                j--;
            } while (arr[j] > pivot);

            if (i >= j) return j;

            swap(arr, i, j);
        }
    }
}
```


- Lomutoよりはスワップの回数が少ないため、より速い
    - 両方で間違えたときだけスワップするので、回数が少ない
- それでも、時間計算は同じ

```text
| Case    | Complexity |
| ------- | ---------- |
| Best    | O(n log n) |
| Average | O(n log n) |
| Worst   | O(n²)      |
```


- goのバージョンは以下の通り

```go
package main

import "fmt"

func quickSort(arr []int, low, high int) {
	if low < high {
		p := partition(arr, low, high)
		quickSort(arr, low, p)     // ⚠️ important: p is included
		quickSort(arr, p+1, high)
	}
}

func partition(arr []int, low, high int) int {
	pivot := arr[low]

	i := low - 1
	j := high + 1

	for {
		// move i from left
		for {
			i++
			if arr[i] >= pivot {
				break
			}
		}

		// move j from right
		for {
			j--
			if arr[j] <= pivot {
				break
			}
		}

		// pointers crossed
		if i >= j {
			return j
		}

		swap(arr, i, j)
	}
}

func swap(arr []int, i, j int) {
	arr[i], arr[j] = arr[j], arr[i]
}

func main() {
	arr := []int{10, 7, 8, 9, 1, 5}

	quickSort(arr, 0, len(arr)-1)

	fmt.Println(arr)
}
```


- バランスのいいピボットを選ぶため、以下のようなことがおすすめされる

```java
// ３点中央点
int mid = (low + high) / 2;
if (arr[low] > arr[mid]) swap(arr, low, mid);
if (arr[low] > arr[high]) swap(arr, low, high);
if (arr[mid] > arr[high]) swap(arr, mid, high);

int pivot = arr[mid];

// ランダム
int pivot = arr[low + rand() % (high - low + 1)];
```




## <span style="color:#802548">_深く探究ークイックソート３Way_</span>

- Hoare, Lomutoでは重複された値が多いとき、最悪のケースに陥りやすい
- 3wayはピボットが最後でなく、最初から始める
- 重複の値をきちんと扱うため、領域を３つに分ける
    - lt (less-than boundary)
    - i (current index)：
    - gt (greater-than boundary)
- lomutoとは違って、ピボットが 1 から始まるので、

```java
public static void quickSort3Way(int[] arr, int low, int high) {
    if (low >= high) return;

    int pivot = arr[low];
    int lt = low;      // < ピボット
    int i = low + 1;   // ピボット
    int gt = high;     // > ピボット
}
```

- 最初は以下のような状態

```text
[ P | ? ? ? ? ? ]
      ↑ 未知
```

- どんどん動きながら、以下のように未知の領域を減らす

```text
[ < | = = | ? ? | > ]
              ↑ shrinking
```


- そのためのロジックは以下の通り
    - 小さい値があったら、値をスワップする
        - スワップしたら、左から未知領域が縮めるので、ltを++する
        - スワップされたら、整頓されたので、現在のインデックス i を次のインデックスに安全に進める 
    - 大きい値があったら、値をスワップする
        - スワップしたら、右から未知領域が縮めるので、gtを--する
        - スワップしても、i は動かないが、理由はまだ右からピボットに向けてきてるインデックスはチェックされてないからだ
    - どちらにも当てはまらなかった場合、重複なので、インデックスをスキップする

```java
while (i <= gt) {
    if (arr[i] < pivot) {
        swap(arr, lt++, i++);
    } else if (arr[i] > pivot) {
        swap(arr, i, gt--);
    } else {
        i++;
    }
}
```

- lt がピボットの領域、つまり重複の領域なので、 lt - 1までにすると、小さい値の領域になる
- gt もピボットの領域、つまり重複の領域なので、gt + 1からすると、大きい値の領域になる
- クイックソートの作業をもう１度繰り返す

```java
quickSort3Way(arr, low, lt - 1);
quickSort3Way(arr, gt + 1, high);
```

## <span style="color:#802548">_深く探究ークイックソートハイブリッド_</span>

- ハイブリットのクイックソートは以下のようになる
    - サイズが少ない場合は、挿入ソートが効率高いので、挿入ソートを行う
    - 少なくない場合、クイックソートを行う
    - ピボットは３点中央値で最悪のケースに陥る可能性を減らす
        - 重複値が多い場合、３way方法で進める
        - でなければ、Hoare方法で進める


```java
public class HybridQuickSort {

    public static void quickSort(int[] arr, int low, int high) {
        if (low >= high) return;

        // Small array optimization (optional but common)
        if (high - low < 16) {
            insertionSort(arr, low, high);
            return;
        }

        // Step 1: median-of-three pivot selection
        int pivot = medianOfThree(arr, low, high);

        // Step 2: detect duplicates (simple heuristic)
        if (isDuplicateHeavy(arr, low, high, pivot)) {
            // Use 3-way partition
            int[] range = partition3Way(arr, low, high, pivot);
            quickSort(arr, low, range[0] - 1);
            quickSort(arr, range[1] + 1, high);
        } else {
            // Use Hoare partition
            int p = partitionHoare(arr, low, high, pivot);
            quickSort(arr, low, p);
            quickSort(arr, p + 1, high);
        }
    }

    // -------- median-of-three --------
    private static int medianOfThree(int[] arr, int low, int high) {
        int mid = (low + high) / 2;

        if (arr[low] > arr[mid]) swap(arr, low, mid);
        if (arr[low] > arr[high]) swap(arr, low, high);
        if (arr[mid] > arr[high]) swap(arr, mid, high);

        return arr[mid]; // median value
    }

    // -------- Hoare partition --------
    private static int partitionHoare(int[] arr, int low, int high, int pivot) {
        int i = low - 1;
        int j = high + 1;

        while (true) {
            do { i++; } while (arr[i] < pivot);
            do { j--; } while (arr[j] > pivot);

            if (i >= j) return j;

            swap(arr, i, j);
        }
    }

    // -------- 3-way partition --------
    private static int[] partition3Way(int[] arr, int low, int high, int pivot) {
        int lt = low;
        int i = low;
        int gt = high;

        while (i <= gt) {
            if (arr[i] < pivot) {
                swap(arr, lt++, i++);
            } else if (arr[i] > pivot) {
                swap(arr, i, gt--);
            } else {
                i++;
            }
        }

        return new int[]{lt, gt};
    }

    // -------- Duplicate detection --------
    private static boolean isDuplicateHeavy(int[] arr, int low, int high, int pivot) {
        int count = 0;
        int sampleSize = Math.min(10, high - low + 1);

        for (int i = low; i < low + sampleSize; i++) {
            if (arr[i] == pivot) count++;
        }

        return count > sampleSize / 2;
    }

    // -------- Insertion sort --------
    private static void insertionSort(int[] arr, int low, int high) {
        for (int i = low + 1; i <= high; i++) {
            int key = arr[i];
            int j = i - 1;

            while (j >= low && arr[j] > key) {
                arr[j + 1] = arr[j];
                j--;
            }

            arr[j + 1] = key;
        }
    }

    // -------- swap --------
    private static void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```


## <span style="color:#802548">_深く探究クイックソートダブルピボット_</span>

- 実際Javaでは、上のようなクイックソートじゃなく、ダブルピボットで基本型のデータをソートするアルゴリズムを実装してる
- 領域を作るロジック事態は3wayと同じ
- しかし、ダブルピボットになるため、領域を決めるWhile文以前の前処理と以後の後処理が違う

```java
public class DualPivotQuickSort {

    public static void quickSort(int[] arr, int low, int high) {
        if (low >= high) return;

        // Ensure pivot1 <= pivot2
        if (arr[low] > arr[high]) {
            swap(arr, low, high);
        }

        int pivot1 = arr[low];
        int pivot2 = arr[high];

        int lt = low + 1;   // boundary for < pivot1
        int gt = high - 1;  // boundary for > pivot2
        int i = lt;

        while (i <= gt) {

            if (arr[i] < pivot1) {
                swap(arr, i, lt);
                lt++;
                i++;
            } else if (arr[i] > pivot2) {
                swap(arr, i, gt);
                gt--;
                // ⚠️ don't increment i here
            } else {
                i++;
            }
        }

        // Place pivots in correct positions
        lt--;
        gt++;

        swap(arr, low, lt);
        swap(arr, high, gt);

        // Recursively sort three parts
        quickSort(arr, low, lt - 1);     // < pivot1
        quickSort(arr, lt + 1, gt - 1);  // between
        quickSort(arr, gt + 1, high);    // > pivot2
    }

    private static void swap(int[] arr, int i, int j) {
        int temp = arr[i];
        arr[i] = arr[j];
        arr[j] = temp;
    }
}
```

- まず、小さいピボットと大きいピボットを決める
- そのあと、ピボットを除いて慮域を設定する
- 慮域の探索のための i は ピボットを除いた先頭から始まる

```java
// Ensure pivot1 <= pivot2
if (arr[low] > arr[high]) {
    swap(arr, low, high);
}

int pivot1 = arr[low];
int pivot2 = arr[high];

int lt = low + 1;   // boundary for < pivot1
int gt = high - 1;  // boundary for > pivot2
int i = lt;
```

- 領域を決めるロジックは３Wayと変わらない

```java
while (i <= gt) {
    if (arr[i] < pivot1) {
        swap(arr, i, lt);
        lt++;
        i++;
    } else if (arr[i] > pivot2) {
        swap(arr, i, gt);
        gt--;
    } else {
        i++;
    }
}
```


- While文が終わった後では、lt と gtはピボットの中間の領域の境界値
    - lt は --して、境界値の直前の値にする
    - gt は ++して、境界値の直後の値にする
- ピボットが配置されべきのインデックスである lt と gtに調整が終わったので、スワップする


```java
lt--;
gt++;

swap(arr, low, lt);
swap(arr, high, gt);
```

- そうすることで、以下の通りになる

```text
[ < pivot1 | pivot1 | middle | pivot2 | > pivot2 ]
```


- 小さい領域、中間領域、大きい領域に分けて１度クイックソートを行う
    - low から lt-1
        - 1番目のピボットより小さい要素の集まり
    - lt+1 から gt-1
        - 1番目のピボットと2番目のピボットの間の要素の集まり
    - gt+1 から high
        - ２番目のピボットより大きい要素の集まり
    - lt と gt はピボットであるため、ソート範囲には含まれない。

```java
quickSort(arr, low, lt - 1);     // < pivot1
quickSort(arr, lt + 1, gt - 1);  // between
quickSort(arr, gt + 1, high);    // > pivot2
```


- ダブルピボットが優れている理由
    - １度に3つの領域に分割できる
        - より多くのパーティションに分けることで、再帰呼び出しの回数が減る
    - データの偏りが抑えられる
        - ピボットを2つ使うことで、データをより均等に分割できる確率が高まる
        - これにより、再帰ツリーがより「浅く」なります。
    - キャッシュとメモリ効率が良い
        - 1回のパーティション処理でより多くの要素を処理できるため、CPU観点での局所が向上する

## <span style="color:#802548">_振り返り_</span>

- OOP版　ー＞ Lomuto版 ー＞ 中央値探索　ー＞ Hoare版 ー＞ ３Way版 ー＞ ハイブリッド版 ー＞ Javaのダブルピボット

- OOP版は理解しやすいだけに、効率は悪い
- LomotoはOOPより効率は高いけど、ピボットの選択が最悪のケースに陥る可能性がわりとある
- 中央値はピボットの選択で最悪のケースに陥りにくいが、Lomuto自体がスワップが多く、スワップで遅い
- スワップの回数を抑えるため、Hoare版を取り入れる
- 重複の可能性のあるロジックについてLomutoもHoareも効率が優れてないので、３Wayの方法を作成
- 全てを踏まえて、堅牢なハイブリット版を導入する
- ピボットを１つじゃなくて、２つにするダブルピボット版を実装する

