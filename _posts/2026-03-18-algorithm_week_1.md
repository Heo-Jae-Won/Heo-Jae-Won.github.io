## <span style="color:#802548">_時間計算量と選択ソート_</span>
- 時間計算量
    - コンピュータが問題を解く際、入力データ量の増加に対して、必要な処理のステップ数（計算や操作回数）がどう変化するかを指し示す。
    - O(1), O(n), O(nlogn), O(ｎ２)などがあり、Ｏ(1)が一番効率高い。例は以下のよう。
        - 配列のアクセス
        - for文
        - Quickソート
        - 選択ソート、ダブルFor文
- 選択ソート
    - 未整列のデータ列から最小値（または最大値）を毎回選択し、整列済みの列の末尾に移動させる単純な並べ替えアルゴリズム

## <span style="color:#802548">_最大の値を探索_</span>
- 最大の値を探索
    - 最大値を想定すること。
    - スターポイントのための１個目のFor文
    - 比較のための２個目のFor文
- ダブルFor文なので、O（ｎ２）になる

```java
public static void main(String[] args) throws Exception {
    int[] abc = { 0, 1, 7, 2, 13, 15, 6 };
    int max = abc[0];
    for (int i = 0; i < abc.length; i++) {
        for (int j = 1 + i; j < abc.length; j++) {
            if (max <= abc[j]) {
                max = abc[j];
            }
        }
    }
}
```

- go langではJavaと宣言とか、データ型を決める方法が違う
    - 関数も大文字にする必要がある。
    - パラメータも Arrayじゃなくて、Sliceタイプを使うようになる

```go
func FindMaxValue(arr []int) int {
	max := arr[0]
	for i := range arr {
		for j := i + 1; j < len(arr); j++ {
			if max < arr[j] {
				max = arr[j]
			}
		}
	}
	return max
}
```

## <span style="color:#802548">_最小値を探索_</span>

- 最小限の値を探索
- 上と同じ。ただ以上の方向が変わっただけ

```java
public class App {
    public static void main(String[] args) throws Exception {
        int[] abc = { 3, 1, 7, 2, 13, 15, 6 };
        int min = abc[0];
        for (int i = 0; i < abc.length; i++) {
            for (int j = 1 + i; j < abc.length; j++) {
                if (min >= abc[j]) {
                    min = abc[j];
                }
            }
        }
    }
}
```

- go lang

```go
func FindMinValue(arr []int) int {
	min := arr[0]
	for i := range arr {
		for j := 1 + i; j < len(arr); j++ {
			if min > arr[j] {
				min = arr[j]
			}
		}
	}

	return min
}
```


## <span style="color:#802548">_配列を反転させる_</span>

- 配列を反転させることではSwappingの技術が必要
    - SwappingはJavaではTemp変数が必要
    - 要素にアクセスするためには、要素 - 1が必要。
    - そこで i を引く理由は後半から逆に行きながらSwappingするからだ
- 時間計算量は n/2なので O(n)になる
    - n / 2になる理由は前半だけを見て後半と比較すると後半の要素はループが不要

```java
public static void main(String[] args) throws Exception {
    int[] abc = {0,1,7,2,4,13,6};
    for(int i = 0; i < abc.length / 2; i++) {
        int temp = abc[i];
        abc[i] = abc[abc.length - 1 -i];
        abc[abc.length - 1 -i] = temp;
    }
}    
```

```go
func ReverseArray(arr []int) {
	for i := 0; i < len(arr)/2; i++ {
		arr[i], arr[len(arr)-1-i] = arr[len(arr)-1-i], arr[i]
	}
}
```

- ただ、不変性が重要なので、不変性を担保するために、新しいオブジェクトを返すことにする
- 今回はプリミティブ型なので、Cloneでも問題はないが、参照型としたら、シャローコピーになってしまう。
- ディープコピーにするためにはFor文でループしながら１つづつコピーするしかない

```java
public int[] returnReversedArray(int[] arr) {
    int newArr = arr.clone();
    for(int i = 0; i < newArr.length / 2; i++) {
        int temp = newArr[i];
        newArr[i] = newArr[newArr.length - 1 -i];
        newArr[newArr.length - 1 -i] = temp;
    }
}   

Integer[] newArr = new Integer[arr.length];
for (int i = 0; i < arr.length; i++) {
    newArr[i] = arr[i].clone(); //オブジェクトの時
}
```

- プリミティブ型はただのシャロ―コピーもディープコピーと同じ効果が現れる
- makeはJavaに例えれば、newと同じ
- Copyは以下のFor文と同じ効果

```go
func ReturnNewReverseArray(arr []int) []int {
	newArray := arr //ーーー＞シャロ―コピー

    newArray := make([]int, len(arr)) //ーーー＞ディープコピー
    copy(newArray, arr)
    /*for i := range arr {
        newArray[i] = arr[i]
    }*/

	for i := 0; i < len(newArray)/2; i++ {
		newArray[i], newArray[len(arr)-1-i] = newArray[len(newArray)-1-i], newArray[i]
	}

	return newArray
}
```

## <span style="color:#802548">_選択ソート昇順_</span>

- 選択ソート昇順
- If文で比較の対象が abc[i] じゃなくて、abc[minValueIdx]になる理由は、実際に最小値を見つけたとき、minValueIdxが変わるからだ

```java
public static void main(String[] args) throws Exception {
    int[] abc = { 3, 1, 7, 2, 13, 15, 6 };
    int minValueIdx = abc[0];
    for (int i = 0; i < abc.length; i++) {
        for (int j = 1 + i; j < abc.length; j++) {
            if (abc[minValueIdx] > abc[j]) {
                minValueIdx = j;
            }
        }

        int temp = abc[i];
        abc[i] = abc[minValueIdx];
        abc[minValueIdx] = temp;
    }
}
```

## <span style="color:#802548">_選択ソート降順_</span>

- 選択ソート降順
- if文の方向だけ変えればいい

```java
public static void main(String[] args) throws Exception {
    int[] abc = { 3, 1, 7, 2, 13, 15, 6 };
    int maxValueIdx = abc[0];
    for (int i = 0; i < abc.length; i++) {
        for (int j = 1 + i; j < abc.length; j++) {
            if (abc[maxValueIdx] <= abc[j]) {
                maxValueIdx = j;
            }
        }

        int temp = abc[i];
        abc[i] = abc[maxValueIdx];
        abc[maxValueIdx] = temp;
    }
}
```

- 不変性を担保するため、新しい配列を返す

```java
public static void main(String[] args) throws Exception {
    int[] abc = { 3, 1, 7, 2, 13, 15, 6 };
    int[] sortedArray = abc.clone();

    int maxValueIdx = sortedArray[0];
    for (int i = 0; i < sortedArray.length; i++) {
        for (int j = 1 + i; j < sortedArray.length; j++) {
            if (sortedArray[maxValueIdx] <= sortedArray[j]) {
                maxValueIdx = j;
            }
        }

        int temp = sortedArray[i];
        sortedArray[i] = sortedArray[maxValueIdx];
        sortedArray[maxValueIdx] = temp;
    }
    
    System.out.println(Arrays.toString(sortedArray));
}
```

- go lang

```go
func SelectionSort(arr []int) []int {
	maxIndex := 0
	newArr := make([]int, len(arr))
	copy(newArr, arr)

	for i := range newArr {
		for j := i + 1; j < len(newArr); j++ {
			if newArr[maxIndex] < newArr[j] {
				maxIndex = j
			}
		}

		newArr[i], newArr[maxIndex] = newArr[maxIndex], newArr[i]
	}

	return newArr
}
```




## <span style="color:#802548">_選択ソートを実装する際に迷ったポイント_</span>
- ダブルFor文でｊの値をどう設定していけばいいのかわかなかった

```java
//最初
for (int i = 0; i < abc.length; i++) {
    for (int j = 1/*ここどうすればいい？ */; j < abc.length; j++) {

    }
}

//AI参考
for (int i = 0; i < abc.length; i++) {
    for (int j = 1 + i; j < abc.length; j++) {

    }
}
```

- 位置を入れ替えるためには、Indexが必要だったのにそれに気づかなかった

```java
//最初
int maxValue = sortedArray[0];
for (int i = 0; i < sortedArray.length; i++) {
    for (int j = 1 + i; j < sortedArray.length; j++) {
        if (sortedArray[maxValueIdx] <= sortedArray[j]) {
            maxValue = sortedArray[j];
        }
    }
}

//AI参考
int maxValueIdx = sortedArray[0];
for (int i = 0; i < sortedArray.length; i++) {
    for (int j = 1 + i; j < sortedArray.length; j++) {
        if (sortedArray[maxValueIdx] <= sortedArray[j]) {
            maxValueIdx = j;
        }
    }
```

## <span style="color:#802548">_深く探求_</span>

- Arrays.sort()はどのアルゴリズムを使っているのか

- go langはどんなsortアルゴリズムを使用しているのか





