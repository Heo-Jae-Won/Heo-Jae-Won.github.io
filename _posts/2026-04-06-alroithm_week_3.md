## <span style="color:#802548">_実装ーバブルソート_</span>

- 最初のロジックは以下だった。
- ただ、iを全然使わずにしていた

```java
public static void executeBubbleSort(int[] arr) {
    for(int i = 0; i < arr.length; i++) {
        for (int j = 0; j < arr.length - 1; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
                break;
            }
        }
    }
}
```

- goに変換して実行しようとしたが、失敗した
- declared and not used: i というエラーが出た
- 私のロジックに何か問題があるのことに気づいた
- javaではできたのは、JavaとGoのCompilerの違いから起因したと思った
    - 実際にGoのコンパイラは上のオプションを非活性化ができない
- 以下はコンパイルエラーが出る関数

```go
func ExecuteBubbleSort(arr []int) {
	for i := range arr {
		for j := 0; j < len(arr)-1; j++ {
			if arr[j] > arr[j+1] {
				arr[j], arr[j+1] = arr[j+1], arr[j]
				break
			}
		}
	}
}
```

- そのため、iも利用するしっかりした関数を作った
- 結局BubbleSortは、最後の値を埋めて見なくてもいいようにするアルゴリズムだ、ということに気づいた
- 最後に差大値を移動させてそこは見なくてもいいので、jの範囲が　- i されてる
- i はそのループカウントを指し示すための変数だった。

```java
public static void executeBubbleSort(int[] arr) {
    for(int i = 0; i < arr.length; i++) {
        for ( int j = 0; j < arr.length - 1 - i; j++) {
            if (arr[j] > arr[j + 1]) {
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}
```

- go langバージョン
- j + 1が動く基準なので、基準が len(arr) - i の範囲内で動くことを明確にした

```go
func ExecuteBubbleSort1(arr []int) {
	for i := range arr {
		for j := 0; j + 1 < len(arr)-i; j++ {
			if arr[j] > arr[j+1] {
				arr[j], arr[j+1] = arr[j+1], arr[j]
			}
		}
	}
}
```

- 以下のようにもできる
- 基準になるインデックスを変えただけ
    - j が動く基準なので、基準が len(arr) - i の範囲内で動くこと

```go
func ExecuteBubblesort2(arr []int) {
	for i := range arr {
		for j := 1; j < len(arr)-i; j++ {
			if arr[j] < arr[j-1] {
				arr[j], arr[j-1] = arr[j-1], arr[j]
			}
		}
	}
}
```

- そこで少し改善をすると、スワップがない場合は、もう整列が終わったということになるので、ループさせないようにする

```go
func ExecuteBubblesort2(arr []int) {
	for i := range arr {
		isSwapped := false
		for j := 1; j < len(arr)-i; j++ {
			if arr[j] < arr[j-1] {
				arr[j], arr[j-1] = arr[j-1], arr[j]
				isSwapped = true
			}
		}

		if !isSwapped {
			break
		}
	}
}
```

- 上は昇順で、以下は降順
- if文の矢印の方向が逆になっただけ

```go
func ExecuteBubblesortDESC(arr []int) {
	for i := range arr {
		isSwapped := false
		for j := 1; j < len(arr)-i; j++ {
			if arr[j] > arr[j-1] {
				arr[j], arr[j-1] = arr[j-1], arr[j]
				isSwapped = true
			}
		}

		if !isSwapped {
			break
		}
	}
}
```

- C#は以下のよう
- C#は、JAVA字下げのしかたが違うので、それだけ注意すればほぼJAVAと同じ

```C#
static void ExecuteBubblesortDESC(int[] arr)
{
    for (int i = 0; i < arr.Length; i++)
    {
        bool isSwapped = false;

        for (int j = 1; j < arr.Length - i; j++)
        {
            if (arr[j] > arr[j - 1])
            {
                int temp = arr[j];
                arr[j] = arr[j - 1];
                arr[j - 1] = temp;

                isSwapped = true;
            }
        }

        if (!isSwapped)
        {
            break;
        }
    }
}
```


- ここで重要なポイントは以下の通り
    - 最後から整列されること
    - 基準となるインデックスが１から始まること
    - wapされる要素がなくなった時点で、整列が完了したと判断して処理を終了する

- 時間計算量は以下のよう
    - あらかじめ整列されている場合（ベストケース）では、計算量は O(N) になる。
    - それ以外の場合は、二重のfor文を使っているため、計算量はどうしても O(N²) になってしまう

## <span style="color:#802548">_実装ー挿入ソート_</span>

- 一週間後で挑戦して解けた
- 最初のロジックは以下だった

```go
func InsertionSort(arr []int) {
	for i := 1; i < len(arr); i++ {
		std := arr[i]
		n := i
		for arr[n] < arr[n-1] {
			arr[n] = arr[n-1]
			n--
		}

		arr[n] = std
	}
}
```

- ただ、nが負数になる問題があって負数になったらWhile文を抜けるようにした

```go
func InsertionSort(arr []int) {
	for i := 1; i < len(arr); i++ {
		std := arr[i]
		n := i
		for n > 0 　/* 追加 */&& arr[n] < arr[n-1] {
			arr[n] = arr[n-1]
			n--
		}

		arr[n] = std
	}
}
```


- そのあともエラーが出た
- 実際に回数ごとの展開を書いたところ、比較の対象がバブルソートとは違って、基準数なのに気付いた

```go
func InsertionSort(arr []int) {
	for i := 1; i < len(arr); i++ {
		std := arr[i]
		n := i
		for n > 0 && std /* 変換 */< arr[n-1] {
			arr[n] = arr[n-1]
			n--
		}

		arr[n] = std
	}
}
```

- 上と同じ結果だけどインデックスへの捉え方が違う
- 厳密にいうと、こちらがシフトにもっと近い

```go
func InsertionSort(arr []int) {
	for i := 1; i < len(arr); i++ {
		std := arr[i]
		n := i-1
		for n >= 0 && std < arr[n] {
			arr[n+1] = arr[n]
			n--
		}

		arr[n+1] = std
	}
}
```

- ただ、それは理解しづらかったので、以下のように変数名を切り替えて理解しやすくした


```go
func InsertionSortDESCShiftBased(arr []int) {
	for i := 1; i < len(arr); i++ {
		curValue := arr[i]
		comparedIndex := i - 1
		for comparedIndex >= 0 && curValue > arr[comparedIndex] {
			arr[comparedIndex+1] = arr[comparedIndex]
			comparedIndex--
		}

		arr[comparedIndex+1] = curValue
	}
}
```

- 以下はできない
- 操作中、インデックスが動くので、インデックスじゃなくて、必ず値を格納する必要がある
- もう当時のインデックスに対する値が失われてるのでエラーになる

```go
func InsertionSortDESCShiftBased(arr []int) {
	for i := 1; i < len(arr); i++ {
		curIndex := i
		comparedIndex := i - 1
		for comparedIndex >= 0 && arr[curIndex] > arr[comparedIndex] {
			arr[comparedIndex+1] = arr[comparedIndex]
			comparedIndex--
		}

		arr[comparedIndex+1] = arr[curIndex] /* もう当時のインデックスに対する値が失われてる*/
	}
}
```

- For文になると、以下のようになる
- While文より理解しづらくなるので、ここではWhile文を使うことをおすすめする

```go
var n int
for n = i - 1; n >= 0 && std < arr[n]; n-- {
    arr[n+1] = arr[n]
}
```

- C#にすると以下となる
- Javaとほぼ変わらない
- importなどのインペラティブが usingなどに変わっただけ

```C#
using System;

class Program
{
    static void InsertionSort(int[] arr)
    {
        for (int i = 1; i < arr.Length; i++)
        {
            int std = arr[i];
            int n = i - 1;

            while (n >= 0 && std < arr[n])
            {
                arr[n + 1] = arr[n];
                n--;
            }

            arr[n + 1] = std;
        }
    }

    static void Main()
    {
        int[] arr = { 5, 2, 4, 6, 1, 3 };

        InsertionSort(arr);

        Console.WriteLine(string.Join(", ", arr));
    }
}
```


- ここで重要なポイントは以下のよう
    - Whileを考えること
    - 基準となる数という概念を思いつくこ
    - 基準となるインデックスが１から始まること
    - スワップじゃなくて、シフトということ
    - シフトをコンピューター科学てきに実装する方法

- 時間計算量は以下のよう
    - あらかじめ整列されている場合（ベストケース）では、計算量は O(N) になる。
    - それ以外の場合は、二重のfor文を使っているため、計算量はどうしても O(N²) になってしまう

## <span style="color:#802548">_実装ー整列以降特定の値を数える_</span>

- 私が最初にこの問題を解いた方法は、以下の通りだ
    - カウントは nという変数を導入して処理するが、重複でなくなったら、カウントを元に戻す
    - 重なってる最初の要素を見つけたら、その値を入れる
- 相当多くの変数を使っていたため、それを何とか減らせないか工夫した

```java
public static DupCount[] countDup(int[] arr) {
    int n = 0;
    boolean doubledFlag = false;
    DupCount[] temp = new DupCount[arr.length];
    for (int i = 0; i < arr.length - 1; i++) {
        if (arr[i] == arr[i + 1]) {
            n++;
            doubledFlag = true;
        } else {
            if (doubledFlag) {
                temp[i - n] = new DupCount(arr[i - n], n);
                doubledFlag = false;
            }
            n = 0;
        }

    }

    return temp;
}

private static class DupCount {
    int duplicatedNumber;
    int count;

    public DupCount(int duplicatedNumber, int count) {
        this.duplicatedNumber = duplicatedNumber;
        this.count = count;
    }

    @Override
    public String toString() {
        return "DupCount [duplicatedNumber=" + duplicatedNumber + ", count=" + count + "]";
    }
}
```

- ただ、最後の要素が取れなかったので、最後は以下のようにIf文で手動で処理するようにした

```java
 public static DupCount[] countDup(int[] arr) {
    int n = 0;
    boolean doubledFlag = false;
    DupCount[] temp = new DupCount[arr.length];

    int i = 0;
    for (i = 0; i < arr.length - 1; i++) {
        if (arr[i] == arr[i + 1]) {
            n++;
            doubledFlag = true;
        } else {
            if (doubledFlag) {
                temp[i - n] = new DupCount(arr[i - n], n);
                doubledFlag = false;
            }
            n = 0;
        }
    }

    if (doubledFlag) {
        temp[i] = new DupCount(arr[i - n], n);
        doubledFlag = false;
    }

    return temp;
}
```

- しかし、配列に合うサイズではなく、全てのサイズを取っているため、冗長なNull値などが見えた
- なので、必要ない要素を取り除くために diff という変数を付け加えた
- カウントは初期化されるので、初期化に自由な変数として diffを導入したわけだ
- カウントも１から始まるので、＋１にした

```java
public static DupCount[] countDup(int[] arr) {
    int n = 0;
    boolean doubledFlag = false;
    DupCount[] temp = new DupCount[arr.length];

    int i = 0;
    int diff = 0;
    int length = 1;
    for (i = 0; i < arr.length - 1; i++) {
        if (arr[i] == arr[i + 1]) {
            n++;
            diff++;
            doubledFlag = true;
        } else {
            if (doubledFlag) {
                temp[i - diff] = new DupCount(arr[i - n], n + 1);
                doubledFlag = false;
                length++;
            }
            n = 0;
        }
    }

    if (doubledFlag) {
        temp[i - diff] = new DupCount(arr[i - n], n + 1);
        doubledFlag = false;
    }

    return Arrays.copyOf(temp, length);
}
```

- ChatGPTが最適化してくれたソースコードだ
- Countはあるとしたら、最初から１から始まるべきなのに、そこまで考えが及ばなかった
- 重複された値の最後のやつだけ引っ張ってきて入れればいい
    - そのため、入れる値を探すためのインデックスは i - n が要らない
    - ただ i - 1でもいい
    - n 自体の必要性が要らなくなったので、消す
- doubledFlagもカウントが １超過になったら、重複なのでその変数もいらないから消す

```java
public static DupCount[] countDup(int[] arr) {

    DupCount[] temp = new DupCount[arr.length];
    int size = 0;
    int count = 1;

    for (int i = 1; i < arr.length; i++) {

        if (arr[i] == arr[i - 1]) {
            count++;
        } else {
            if (count > 1) {
                temp[size++] = new DupCount(arr[i - 1], count);
            }
            count = 1;
        }
    }

    // last group
    if (count > 1) {
        temp[size++] = new DupCount(arr[arr.length - 1], count);
    }

    return Arrays.copyOf(temp, size);
}
```

- GOに変換した

```go
type DupCount struct {
    dupNum int
    count  int
}

func NewDupCount(dupNum int, count int) *DupCount {
    return &DupCount{
        dupNum: dupNum,
        count:  count,
    }
}

func (d DupCount) String() string {
    return fmt.Sprintf("DupCount [dupNum=%d, count=%d]", d.dupNum, d.count)
}

func CountDup(arr []int) []DupCount {
    temp := []newDupCount
    size := 0
    count := 1

    for i = 1; i < len(arr); i++ {
        if arr[i] == arr[i - 1] {
            count++
        } else {
            if (count > 1) {
                temp[size++] = NewDupCount(arr[i-1], count)
            }

            count = 1;
        }
    }

    if(count > 1) {
        temp[size++] = NewDupCount(arr[i-1], count)
    }
}
```

- goについての変数宣言などが間違えていた
- Sliceを利用するので、サイズという変数も要らなくなる
- sliceに要素を追加するときは、Appendを利用する
- 宣言された構造体を共有して使うためには、ポインタとして渡す必要がある

```go
func CountDup(arr []int) []DupCount {
    var temp []DupCount
    if len(arr) == 0 {
        return temp
    }

    count := 1

    for i := 1; i < len(arr); i++ {
        if arr[i] == arr[i-1] {
            count++
        } else {
            if count > 1 {
                temp = append(temp, *NewDupCount(arr[i-1], count))
            }
            count = 1
        }
    }

    // Handle last group
    if count > 1 {
        temp = append(temp, *NewDupCount(arr[len(arr)-1], count))
    }

    return temp
}
```


- C#に変換すると、Javaとほぼ同じ
- ただ、Array.copyOfじゃなくて、Copyに名前が変わる

```C#
Array.Copy(temp, result, size);
```


- ここで重要なポイントは以下の通り
    - 最後の要素を手動で操る必要がある場合もある
    - カウンタを数えて重複有無が判別がつくので、ブリアン変数はいらない
    - 重複状態を初期化するロジックをどこに置けばいいか


## <span style="color:#802548">_実装ー整列された配列で１番近いいペアを探索_</span>	
- 最初に組んだプログラムだ
- 数字動詞に引き算をした結果を格納して、格納した値の中で1番小さい数字を選んだら、それが正解だと思った
- しかし、一番最後の数字に更新されるエラーが出た
    - １番近いペアが複数になると思えなかった私の不注意だった

```java
public static int[] executeFindingNearestOne(int[] arr) {
    int[] temp = new int[arr.length - 1];
    for(int i = 0; i < arr.length - 1; i++) {
        temp[i] = arr[i + 1] - arr[i];
    }

    int minIndex = 0;;
    for (int i = 0; i < temp.length; i++) {
        if(temp[0] >= temp[i]) {
            minIndex = i;
        }
    }

    int[] retArr = {arr[minIndex], arr[minIndex + 1]};

    return retArr;
}
```


- 全ての一番近い数字を記録する必要があったので、クラスを別途に作成した
- そのため、最小値が複数になるケースを想定して、最小値に当たる値を見つけたら、そのインデックスを格納する
- 最後に、インデックスを利用して元の配列の値を探す

```java
public static NearestNumbers[] executeFindingNearestOneNoOverLapping(int[] arr) {
    int[] temp = new int[arr.length - 1];
    for(int i = 0; i < arr.length - 1; i++) {
        temp[i] = arr[i + 1] - arr[i];
    }

    int min = 0;
    for (int i = 0; i < temp.length; i++) {
        if(temp[0] >= temp[i]) {
            min = temp[i];
        }
    }

    int size = 0;
    int[] prevArr = new int[temp.length];
    for (int i = 0; i < temp.length; i++) {
        if(temp[i] == min) {
            prevArr[size++] = i;
        }
    }

    NearestNumbers[] retArr = new NearestNumbers[size];
    for(int i = 0 ; i < size; i++) {
        retArr[i] = new NearestNumbers(arr[prevArr[i]], arr[prevArr[i] + 1]);
    }

    return retArr;
}

private static class NearestNumbers {
    int num1;
    int num2;
    public NearestNumbers(int num1, int num2) {
        this.num1 = num1;
        this.num2 = num2;
    }
    @Override
    public String toString() {
        return "NearestNumbers [num1=" + num1 + ", num2=" + num2 + "]";
    }
}
```

- ここで重要なポイントは以下の通り
    - 各数値間の最も小さい差を計算する
    - 最も小さい差を示す数値のインデックスを求める

## <span style="color:#802548">_実装ー整列後重複除外_</span>
- Javaの配列が固定サイズだということを忘れていた
- だから以下のような馬鹿みたいにソースコードを書いてしまった

```java
public static int[] executeRemoveDuplication(int[] arr) {
    int[] indexs = {};
    int[] newArray = {};
    
    for (int i = 0; i < arr.length - indexs.length; i++) {
        if(arr[i] == arr[i + 1]) {
            indexs[i] = i + 1;
        }
    }

    int doubledCount = 0;
    for (int j = 0; j < arr.length; j++) {
        boolean doubled = false;
        for(int k = 0; k <indexs.length; k++) {
            if(j == indexs[k]) {
                doubled = true;
                doubledCount++;
            }
        }

        if(!doubled) {
            newArray[j - doubledCount] = arr[j];
        }
    }

    return newArray;
}
```


- 配列を全部検査して重複のものを消す方法で、私がやってた方法。

```java
public static int[] executeRemoveDuplication(int[] arr) {
    int n = 0;
    Integer[] temp = new Integer[arr.length];
    int i = 0;
    for (i = 0; i < arr.length - 1; i++) {
        if(arr[i] == arr[i + 1] ) {
            n++;
            continue;
        }


        temp[i - n] = arr[i];
    }
    temp[i - n] = arr[i];

    int j = 0;
    for(int k = 0; k < temp.length; k++) {
        if(temp[k] != null) {
            j++;
        }
    }

    int[] newArr = new int[j];
    for(int k = 0; k < newArr.length; k++) {
        newArr[k] = temp[k];
    }

    return newArr;
}
```

- 実際のサイズだけを集中していくもっと簡単な方法。
- 全部検査ではなくて、必要な論理てきに有効なサイズだけを見ていく

```java
public static int[] executeRemoveDulication(int[] arr) {
    int[] temp = new int[arr.length];
    int size = 0;

    temp[size++] = arr[0];

    for (int i = 1; i < arr.length; i++) {
        if (arr[i] != arr[i - 1]) {
            temp[size++] = arr[i];
        }
    }

    return Arrays.copyOf(temp, size);
}
```

- golangeでsliceを使うと、より簡単にできる

```go
func ExecuteRemoveDuplication1(arr []int) []int {
	result := []int{arr[0]}

	for i := 1; i < len(arr); i++ {
		if arr[i] != arr[i-1] {
			result = append(result, arr[i])
		}
	}

	return result
}
```

## <span style="color:#802548">_実装ー整列後重複の要素だけを集める_</span>

- 最初は以下のようにソースコードを作成した
- 問題は以下は変数が曖昧なところと、最後の重複が処理できないところが問題になった

```java
public static int[] extractDuplicatedNumber(int[] arr) {
    int[] temp = new int[arr.length];
    int size = 0;
    int count = 0;

    for (int i = 0; i + 1 < arr.length; i++) {
        if (arr[i+1] == arr[i]) {
            count++;
        } else {
            if(count > 0) {
                temp[size++] = new DupCount(arr[i])
            }
            count = 0;
        }
    }

    return Array.copyOf(temp, size);
}
```

- 変数名は重複が起こった数字がいくつなのかを明確にした
- 以前のソースコードは、「1, 2, 2」のような配列では、末尾で重複が発生するが、else文に入らないため、正しく動作しなかった
- そのため、末尾の要素も処理対象に含めた


```java
public static int[] extractDuplicatedNumber(int[] arr) {
    int[] temp = new int[arr.length];
    int size = 0;
    int duplicatedNumberCount = 1;

    for (int i = 1; i < arr.length; i++) {
        if (arr[i] == arr[i - 1]) {
            duplicatedNumberCount++;
        } else {
            if (duplicatedNumberCount > 1) {
                temp[size++] = arr[i - 1];
            }
            duplicatedNumberCount = 1;
        }
    }

    if (duplicatedNumberCount > 1) {
        temp[size++] = arr[arr.length - 1];
    }

    return Arrays.copyOf(temp, size);
}
```

- 以下のようにすると、冗長なif文は削除できる
- ただし、このような書き方に慣れていない人にとっては読みやすさが損なわれる

```java
public static int[] extractDuplicatedNumber(int[] arr) {
    int[] temp = new int[arr.length];
    int size = 0;
    int count = 1;

    for (int i = 1; i <= arr.length; i++) {

        if (i < arr.length && arr[i] == arr[i - 1]) {
            count++;
        } else {
            if (count > 1) {
                temp[size++] = arr[i - 1];
            }
            count = 1;
        }
    }

    return Arrays.copyOf(temp, size);
}
```

- ここで重要なポイントは以下の通り
    - 最後の要素を処理する必要がある


## <span style="color:#802548">_深く探求ーロジックの改善_</span>	

- ChatGPTに聞いた結果、findNearest()の改善が見られた
    - 最終的に隣接した２つの数値の差が１番小さいものしか使わないので、最初からそれを探す
    - なので、不要な臨時配列は要らなくなる
    - それに応じて最後も隣接した２つの数値の差が最小値になった場合だけ格納することにする

```java
public static NearestNumbers[] findNearest(int[] arr) {

    int min = Integer.MAX_VALUE;

    for (int i = 0; i < arr.length - 1; i++) {
        int diff = arr[i + 1] - arr[i];
        if (diff < min) {
            min = diff;
        }
    }

    int count = 0;
    for (int i = 0; i < arr.length - 1; i++) {
        if (arr[i + 1] - arr[i] == min) {
            count++;
        }
    }

    NearestNumbers[] result = new NearestNumbers[count];

    int index = 0;
    for (int i = 0; i < arr.length - 1; i++) {
        if (arr[i + 1] - arr[i] == min) {
            result[index++] = new NearestNumbers(arr[i], arr[i + 1]);
        }
    }

    return result;
}
```

- golangに変換したら、以下になる
- 初期容量を気にしなくてもいいので、ロジックがより簡潔になる

```go
type NearestNumbers struct {
    formerNum int
    latterNum int
}

func NewNearestNumbers(formerNum int, latterNum int) *NearestNumbers {
    return &NearestNumbers{
        formerNum : formerNum,
        latterNum : latterNum
    }
}

func SeekNearesetNumbers(arr []int) ([]*NearestNumbers, bool) {
    if len(arr) < 2 {
        return nil, false
    }

    var nearestNumbers []*NearestNumbers
    minDiff := math.MaxInt64;
    for i := 1; i < len(arr); i++ {
        if arr[i] - arr[i-1] < minDiff {
            minDiff = arr[i] - arr[i - 1]
        }
    }

    for i := 1; i < len(arr); i++ {
        if arr[i] - arr[i-1] == minDiff {
            nearestNumbers = append(nearestNumbers, NewNearestNumber(arr[i-1], arr[i]))
        }
    }

    return nearestNumbers, true
}

arr := []int{1, 3, 4, 8, 9, 10}
pairs, ok := SeekNearestNumbers(arr)

if ok {
    for _, p := range pairs {
        fmt.Printf("(%d, %d)\n", p.formerNum, p.latterNum)
    }
}
```

- Golangのsliceと同じく、Javaも最初から配列じゃなくてArrayListを使う方法もある
- ただ、ArrayListを使うと、clear() などの操作のコストが比較的高い。

```java
List<NearestNumbers> result = new ArrayList<>();
int min = Integer.MAX_VALUE;

for (int i = 0; i < arr.length - 1; i++) {
    int diff = arr[i + 1] - arr[i];

    if (diff < min) {
        result.clear();
        result.add(new NearestNumbers(arr[i], arr[i + 1]));
        min = diff;
    } 
    else if (diff == min) {
        result.add(new NearestNumbers(arr[i], arr[i + 1]));
    }
}
```

- countDup()も改善ができる


```java
public static DupCount[] countDup(int[] arr) {
    DupCount[] temp = new DupCount[arr.length];
    int size = 0;

    int count = 1;

    for (int i = 1; i <= arr.length; i++) {
        if (i < arr.length && arr[i] == arr[i - 1]) {
            count++;
        } else {
            if (count > 1) {
                temp[size++] = new DupCount(arr[i - 1], count);
            }
            count = 1;
        }
    }

    return Arrays.copyOf(temp, size);
}
```

- 最初から配列じゃなくてArrayListを使う方法もある

```java
public static DupCount[] countDup(int[] arr) {
    List<DupCount> list = new ArrayList<>();

    int count = 1;

    for (int i = 1; i <= arr.length; i++) {
        if (i < arr.length && arr[i] == arr[i - 1]) {
            count++;
        } else {
            if (count > 1) {
                list.add(new DupCount(arr[i - 1], count));
            }
            count = 1;
        }
    }

    return list.toArray(new DupCount[0]);
}
```

## <span style="color:#802548">_深く探求ーなぜバブルソートより挿入ソートが効率に優れてるのか_</span>
- 時間計算量は同じであるにもかかわらず、選択ソートがより適している理由は、スワップよりシフトがCPUの効率に優れているためである
    - スワップは3回の代入が必要だが、シフトは1回で済む
    - 演算回数は同じように見えても、CPUの観点ではシフトが圧倒的に有利
    - CPUのcacheの動作にも起因する

- 以下はspatial localityとTemporal localityが保てる
    - CPUは１つの要素だけを読み込んまない
    - CPUは配列で１つの要素だけをアクセスするソースコードがあっても、６４BYTEの単位でデータを読み込む
    - そのため、データの塊（CHUNK）を一度読み込むと、それがキャッシュに保存される
    - そのキャッシュを複数回再利用できる
- 挿入ソートがキャッシュに非常に優しい理由は、Bubble sortと違って、０に戻らないからだ
- i が変わってもほぼ隣接したインデックスなので、キャッシュの活用に有利

```go
for n >= 0 && std < arr[n] {
    arr[n+1] = arr[n]
    n--
}
```

- 平均を考えたら、以下のようにロードしたCPUキャッシュを生かせることができる
- Worstケースだったら、Cache Loadが[95-80], [79-64], [63-48], [47-32],[31-16],[15-0]を全部ロードする必要がある
- そういった場合になると、iが大きくなってしまうと、L1キャッシュの限界容量を超えてしまい、全部キャッシュが失われてしまう
- ただし、実際は部分的に整列されたデータが多いため、CPUキャッシュで効率が優れる

```text
Step i = 100:

Cache load:
[96 ~ 111]　

Use:
100 → 99 → 98 → 97 → (reuse same cache line)

→ minimal cache misses
```

- 実際のソースコード

```go
func InsertionSort(arr []int) {
	for i := 1; i < len(arr); i++ {
		std := arr[i]
		n := i
		for n > 0 　/* 追加 */&& arr[n] < arr[n-1] {
			arr[n] = arr[n-1]
			n--
		}

		arr[n] = std
	}
}
```


- その反面、BubbleSortは次の照会になったら、いつも jの値が０に戻るため、キャッシュが初期化される場合が多い
- L1とL2Cacheはかなり小さいからだ

```text
| Level | Size    |
| ----- | ------- |
| L1    | ~32 KB  |
| L2    | ~256 KB |
```

- そのため、要素が多ければ多いほど、CPUの局所性が損なわれる
- こういった仕組みはキャッシュに優しくないし、頻繫なロードが必要とされるため、遅くなる

```text
Pass 1:
[0 ~ 15] → [16 ~ 31] → ... → [n]

Pass 2:
[0 ~ 15]  ← must reload (evicted)
[16 ~ 31] ← must reload
...
```

- 実際のソースコード

```go
func ExecuteBubbleSort1(arr []int) {
	for i := range arr {
		for j := 0; j + 1 < len(arr)-i; j++ {
			if arr[j] > arr[j+1] {
				arr[j], arr[j+1] = arr[j+1], arr[j]
			}
		}
	}
}
```


## <span style="color:#802548">_振り返り_</span>
- 以下のコードはただスターポイントが異なるだけではない。
- なぜこんなスターポイントをセットすることがいいのかに気づいた

```java
for (int j = 0; j < arr.length - 1; j++) {
    if (arr[j] > arr[j + 1])
}

for (int j = 1; j < arr.length; j++) {
    if (arr[j - 1] > arr[j])
}
```


- アルゴリズムのやり方によって For文での活用し方が異なる
    - 選択ソートは最初の値から整列するからスタートポイントを移動させる
    - Bubbleソートは最後の値から整列するからエンドポイントを移動させる

```java
for (int j = 1 + i; j < abc.length; j++)        // SelectionSort
for (int j = 0; j < arr.length - 1 - i; j++)    // BubbleSort
```


- 隣接してる数値を比較するとき、詩以後のやつが残るときは、手動で対応するしかない

```java
int i = 0;
for (i = 0; i < arr.length - 1; i++) {
    if (arr[i] == arr[i + 1]) {
        n++;
        doubledFlag = true;
    } else {
        if (doubledFlag) {
            temp[i - n] = new DupCount(arr[i - n], n);
            doubledFlag = false;
        }
        n = 0;
    }
}

/* 手動で*/
if (doubledFlag) {
    temp[i] = new DupCount(arr[i - n], n);
    doubledFlag = false;
}
```

- ArrayListとかのデータ型にすると、ロジックが大いに簡潔になるところを見て、やはりAPIを知ることの大事さを改めて思い知らされた