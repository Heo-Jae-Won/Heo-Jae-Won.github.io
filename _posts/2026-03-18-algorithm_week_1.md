## <span style="color:#802548">_時間計算量_</span>
- 時間計算量
    - コンピュータが問題を解く際、入力データ量の増加に対して、必要な処理のステップ数（計算や操作回数）がどう変化するかを指し示す。
    - O(1), O(n), O(nlogn), O(ｎ２)などがあり、Ｏ(1)が一番効率高い。例は以下のよう。
        - 配列のアクセス
        - for文
        - Quickソート
        - 選択ソート、ダブルFor文

## <span style="color:#802548">_選択ソート_</span>
- 選択ソート
    - 未整列のデータ列から最小値（または最大値）を毎回選択し、整列済みの列の末尾に移動させる単純な並べ替えアルゴリズム

## <span style="color:#802548">_実装ー最大の値を探索_</span>
- 最大の値を探索
    - 最大値を想定すること。
    - スターポイントのための１個目のFor文
    - 比較のための２個目のFor文
- ダブルFor文なので、O（ｎ２）になる

```java
int[] abc = { 0, 1, 7, 2, 13, 15, 6 };
int max = abc[0];
for (int i = 0; i < abc.length; i++) {
    for (int j = 1 + i; j < abc.length; j++) {
        if (max <= abc[j]) {
            max = abc[j];
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

## <span style="color:#802548">_実装ー最小値を探索_</span>

- 最小限の値を探索
- 上と同じ。ただ以上の方向が変わっただけ

```java
int[] abc = { 3, 1, 7, 2, 13, 15, 6 };
int min = abc[0];
for (int i = 0; i < abc.length; i++) {
    for (int j = 1 + i; j < abc.length; j++) {
        if (min >= abc[j]) {
            min = abc[j];
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


## <span style="color:#802548">_実装ー配列を反転させる_</span>

- 配列を反転させることではSwappingの技術が必要
    - SwappingはJavaではTemp変数が必要
    - 要素にアクセスするためには、要素 - 1が必要。
    - そこで i を引く理由は後半から逆に行きながらSwappingするからだ
- 時間計算量は n/2なので O(n)になる
    - n / 2になる理由は前半だけを見て後半と比較すると後半の要素はループが不要

```java
int[] abc = {0,1,7,2,4,13,6};
for(int i = 0; i < abc.length / 2; i++) {
    int temp = abc[i];
    abc[i] = abc[abc.length - 1 -i];
    abc[abc.length - 1 -i] = temp;
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

## <span style="color:#802548">_実装ー選択ソート昇順_</span>

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

## <span style="color:#802548">_実装ー選択ソート降順_</span>

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




## <span style="color:#802548">_振り返り_</span>
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

## <span style="color:#802548">_深く探求ー文字列を反転させる_</span>

- １週目は数字を反転させた
- 今回は文字列だが、回帰と違いを理解するために、For文で書く
- 文字列　ー＞ 配列列の場合

```java
public static StringBuilder reverseStr(String str) {
    StringBuilder reverse = new StringBuilder();
    for (int i = str.length() - 1; i >= 0; i--) {
        reverse.append(str.charAt(i));
    } 
    return reverse;
}
```

- 数字を反転させる方法と同じく、半分だけ見ればいいので、StringBuilderを利用したほうよりはパフォーマンスが高い
    - method callがない
    - リサイジングがない
    - インデックスが有効なのかチェックが走らない
- 時間計算量は以下になる
    - toCharArray()にコピー　->  O(n)
    - swap ループ -> O(n/2)
    - new String(arr) -> 再びコピー O(n)

```java
public static String reverse(String s) {
    char[] arr = s.toCharArray();
    for (int i = 0; i < arr.length / 2; i++) {
        char temp = arr[i];
        arr[i] = arr[arr.length - 1 - i];
        arr[arr.length - 1 - i] = temp;
    }
    return new String(arr);
}
```

- JAVA apiは以下のようだ
- 実は以下が一番パフォーマンスが高いと認められる
- 時間計算量は以下になる
    - StringBuilder()にコピー ->  O(n)
    - swap ループ -> O(n/2)
    - new String(arr) -> 再びコピー O(n)
- 配列を手動でいじる時と時間計算量は変わらない
    - しかし、JVMのレベルでの最適化を踏まえると、JAVAのAPIを使ったほうがいい

```java
public static void main(String[] args) {
    char[] newArray = { 'a', 'b', 'c', 'd', 'e', 'f' };

    String reversed = new StringBuilder()
            .append(newArray)
            .reverse()
            .toString();
}
```

- 以下がJAVAのAPIであるStringBuilderのreverseメソッドの実装部だ
- 文字列に関して、文字列は必ず負数であるため、(n-1) >> 1 は (n-1) / 2 と同じだ
- ただ、CPU単位で踏まえると、効率が違う
    - 　>> 1 はCPUの観点ですごく簡潔な１つの命令
        - ただの移動だけで済む
    -  / 2 はすごく難しい１つの命令
        - 商を計算する
        - 余りを計算する
        - 負の数を正しく処理する
        - オーバーフローを回避する
- 現代JVMはそういった部分で最適化をしてくれる可能性が高いが、昔はそうでなかったので、まだああいう書き方が残っている

```java
public AbstractStringBuilder reverse() {
        byte[] val = this.value;
        int count = this.count;
        int n = count - 1;
        if (isLatin1()) {
            for (int j = (n-1) >> 1; j >= 0; j--) {
                int k = n - j;
                byte cj = val[j];
                val[j] = val[k];
                val[k] = cj;
            }
        } else {
            StringUTF16.reverse(val, count);
        }
        return this;
    }
```

- go lang
- JavaはUTFー16に文字列を保存しているので、charAt()を使ったら、自動的にByte単位で計算してそのインデックスの文字列を引っ張てくる
- go langにはそんなのがないので、Byte単位で全部変換する
- 0 ~ 255からが１Byteの範囲であるので、その中で文字ことに決まった数字があるので、１Byteの中で97が入ってきたら、それは 'a'に変換される
- 同じく ２Byteは １６Bitなので、２の16乗ー１まで数字を格納する

```go
func ReverseString1(s string) string {
	strbuilder := strings.Builder{}
	for i := len(s) - 1; i >= 0; i-- {
		strbuilder.WriteByte(s[i])
	}
	return strbuilder.String()
}
```





