## <span style="color:#802548">_再帰_</span>
- 再帰
    - ベースケース
        - 再帰が止まる条件
    - 再帰呼び出し
        - ベースケースに当てはまるまでずっと繰り返して自信を呼び出す


## <span style="color:#802548">_再帰を用いたデータ仕組み_</span>
- スタックフレーム
        - CallStackにはいくつかのスタックフレームがある
        - そのスタックフレームも再帰を用いた仕組み


## <span style="color:#802548">_実装ーfactorial_</span>

- 最初に実装したFactorial
- グローバル変数が必要

```java
public static int factorial(int n) {
    a = a * n;
    if (n > 1) {
        factorial(n - 1);
    }

    return a;
} 
```

- AIに上のコードを投げて得られたものだけど、かなり違いがあってどこでこのようなロジックの違いが起こったのか考えてみた
    - グローバル変数が必要ではない関数に書き換える
    - 数学的には F(n) = n * F(n-1)なので、戻り値も左のような形になる
        - そこでベースケースまで考えが及ばなかった
        - そもそもベースケースの定義を間違っていて、停止じゃなくて、持続の条件として使っていた
    - factorial(0)が０ではないことを認識していなかった
        - 例えば、factorial(6)は 6 x 5 x 4 x 3 x 2 x 1 x factorial(0)になる
        - factorial(0)で停止するので、 0 x fatorial(-1)じゃなくて、１をリターンする

```java
public static int factorial(int n) {
    if (n  < 1) {
        return 1;
    }
    return n * factorial(n - 1);
}
```

- ただ、 n < 1 よりは n - 1 < 0がより理解しやすくてその部分だけ変更した

```java
public static int factorial(int n) {
    if (n - 1  < 0) {
        return 1;
    }
    return n * factorial(n - 1);
}
```

- go lang version
- nが1未満よりは０になった時がもっと理解しやすいと思い、書き換えた

```go
func Factorial(n int) int {
	if n-1 < 0 {
		return 1
	}

	return n * Factorial(n-1)
}
```


## <span style="color:#802548">_実装ーfibonacchi数列再帰_</span>

- ベースケースの重要性を感じてからは、ちゃんとベースケースを定義することになった
- 最初にひねり出して書いたFibonacci数列関数

```java
public static int fibonacci(int a, int b) {
    if (a + b >= 5) {
        return a + b;
    }
    return fibonacci(b, a + b);
}
```

- わかりしやすくするため、変数名を書き換える
- 問題は、ベースケースがFibonacciのN番目の値をわかることが前提なので、相当活用しにくい

```java
public static int fibonacci(int small, int big, int limit) {
    if (small + big >= limit) {
        return small + big;
    }
    return fibonacci(big, small + big);
}
```

- で、ChatGptに質問投げて受け取ったコードは全然違ってて分析した
- ChatGptはFibonacci数列を何番進めるかを着目してソースコードを書いてくれた
- ベースケースが分かりやすいと思った
- 考えてみれば、どんな戻り値の形が必要なのかをまず決めてからベースケースを決めたらいいかもと思った

```java
public static int fib(int n) {
    // base case
    if (n <= 1) {
        return n;
    }

    // recursive case
    return fib(n - 1) + fib(n - 2);
}
```

- ただ、私としてはベースケースが理解しにくかった
- 結局全てのfibonacciメソッドはfibonacci(0)とfibonacci(1)に分解されるので、こうやって変換したら、ロジックが理解できた
    - Fibonacci(1)の分を足したら、それがFionbacci数列になる
    - リターンの再帰関数が(n-1)と(n-2)というパラメータになってるので、0とか1をケースしたら、無限繰り返しになるのではないかと最間違えた
    - しかし、結局EarlyReturnになるので、全然そんな心配は無用だった

```java
public int fibonacci(int n) {
	switch(n) {
        case 0:
            return 0
        case 1:
            return 1
	}

	return fibonacci(n-1) + fibonacci(n-2)
}
```

- go lang

```go
func Fibonacci(n int) int {
	switch n {
	case 0:
		return 0
	case 1:
		return 1
	}

	return Fibonacci(n-1) + Fibonacci(n-2)
}
```

- 上はFibonnaciの演算のカウント分をパラメータとして受け取ってる
- パラメータの数字に当たる位置を探すための関数はどうすればいいか思った結果、あっけなく答えが出た
- その位置の結果値は演算を１回少なくしただけなので、以下のようになる

```go
func FibonacciLocation(n int) int {
    Fibonacci(n - 1);
}
```

## <span style="color:#802548">_実装ー文字列反転させる_</span>
- これを再帰にする

```java
public static String recursiveReverseString(String abc, StringBuilder reverse, int n) {
    reverse.append(abc.charAt(abc.length() - 1 - n));
    if (n == abc.length() - 1) {
        return reverse.toString();
    }
    return recursiveReverseString(abc, reverse, n + 1);
}
```

- golangにはポインターがあるので、Structはポインターを使って渡す
- そうしないと、共有されない全部各自のSbが生成されるので、ロジックが壊れる

```go
var sb strings.Builder
sb.Grow(len("abcd"))
fmt.Println(recall.ReverseStringNoCreate("abcd", &sb, len("abcd")))

func ReverseStringNoCreate(s string, sb *strings.Builder, n int) string {
	if n-1 < 0 {
		return sb.String()
	}

	sb.WriteByte(s[n-1])

	return ReverseStringNoCreate(s, sb, n-1)
}
```


## <span style="color:#802548">_実装ー列配列の合計_</span>						
- 再帰でもFor文と同じく総合の計算ができる

```java
public static int sumArray(int[] arr, int n) {
    if ( n < 0) {
        return 0;
    }
    
    return arr[n] + sumArray(arr, n - 1);
}
```

- golangではベースケースの条件に変化を与えた
- 実装方法によってカウントとかは違くなる


```go
func SumArray(arr []int, n int) int {
	if n > len(arr)-1 {
		return 0
	}

	return arr[n] + SumArray(arr, n+1)
}
```


## <span style="color:#802548">_実装ーJsonParserクラス作成_</span>
- 最大の深さを理解するための一番いい例として、JSON structures parsingがある
- 以下のJsonデータをparsingするとき、再帰が必要になる。

```json
{
  "a": {
    "b": {
      "c": [1,2,3,4],
      "d":"2",
      "e":false
    }
  }
}
```

- どうしてもJSONのパースは自分の頭ではうまく理解できなかった
    - 私は全てのJSON形式が必ずキーを持っていると思い込んでいた
    - で、キーとバリューのどう分けるのか、どうしてもピンとこなかった
    - 空白や特殊記号をどう処理すればいいのか、どうしてもピンとこなかった
    - 多様なデータ型をどうすればいいのか、どうしてもピンとこなかった
- なので、ChatGPTに聞いて以下のように進めた
    - 1番目は分析
    - 2番目はChatGPTのコードをそのまま手でなぞる
    - 3番目は見ずに自分で打ち込む
- メインポイントは以下だった
    - データ型に分ける
    - 空白や特殊記号はスキップする


- まずは以下のようにNodeタイプ仕組みを作成する
- JsonはObject,Value,Array３つがあるので、その３つのタイプを統一させるための抽象クラスが必要

```java
public abstract class Node {

}

class ObjectNode extends Node {
    Map<String, Node> map = new HashMap<>();
}

class ArrayNode extends Node {
    List<Node> list = new ArrayList<>();
}

class ValueNode extends Node {
    Object value;
    ValueNode(Object value) { this.value = value; }
}
```

- 次は実際のパースを実装する
- 以下で直接再帰が起こるわけではなくて、parseValue()で起こる
- 空白を取り除くためparseValue()じゃなくて、parse()にして一緒に呼んでる形だ
- 実は parse()のところのskipWhitespace()は不要だけど、防御的プログラミングのために残す
    - 誰かparseValue()のロジックがskipWhitespace()が削られたとき正常な動作を確保できる
    - パースの前はいつも空白をトリムすることを明示する仕組み

```java
class MiniJsonParser {

    private final String json;
    private int index = 0;

    public MiniJsonParser(String json) {
        this.json = json;
    }

    public Node parse() {
        skipWhitespace();
        return parseValue();
    }
}
```

- parseValue()は読み込んだ文字列に応じて切り分けて処理される
- peek()は現在のインデックスの文字を持ってくるメソッドだ
- parseKey()が別にない理由はキーはオブジェクトにしかないためだ

```java
private Node parseValue() {
    skipWhitespace();
    char c = peek();

    if (c == '{')
        return parseObject();
    if (c == '[')
        return parseArray();
    if (c == '"')
        return new ValueNode(parseString());
    if (c == '-' || Character.isDigit(c))
        return new ValueNode(parseNumber());
    if (match("true"))
        return new ValueNode(true);
    if (match("false"))
        return new ValueNode(false);
    if (match("null"))
        return new ValueNode(null);

    throw new RuntimeException("Unexpected char: " + c);
}
```

- consume()は該当の文字はスキップするメソッドだ
    - javaオブジェクトとしては｛、：、, などの特殊記号は必要でないので素通りする
        - Jsonオブジェクトの場合、mapは put()が結果値を追加する行動
        - Json配列の場合、listは add()が結果値を追加する行動
        - Json文字列は append()結果値を追加する行動
        - Json数値は負数と小数に切り分ける

```java
private Node parseObject() {
    ObjectNode obj = new ObjectNode();
    consume('{');
    skipWhitespace();

    //ベースケース
    if (peek() == '}') {
        consume('}');
        return obj;
    }

    while (true) {
        // キーをパース
        String key = parseString();
        skipWhitespace();
        consume(':');

        // 再帰
        Node value = parseValue();

        obj.map.put(key, value);

        skipWhitespace();
        if (peek() == '}') {
            consume('}');
            break;
        }
        consume(',');
    }

    return obj;
}

private Node parseArray() {
    ArrayNode arr = new ArrayNode();
    consume('[');
    skipWhitespace();

    //ベースケース
    if (peek() == ']') {
        consume(']');
        return arr;
    }

    while (true) {
        // 再帰
        Node value = parseValue();
        arr.list.add(value);

        skipWhitespace();
        if (peek() == ']') {
            consume(']');
            break;
        }
        consume(',');
    }

    return arr;
}

private String parseString() {
    consume('"');
    StringBuilder sb = new StringBuilder();

    while (peek() != '"') {
        sb.append(next());
    }

    consume('"');
    return sb.toString();
}

private Double parseNumber() {
    int start = index;

    if (peek() == '-')
        next();

    while (Character.isDigit(peek()))
        next();

    if (peek() == '.') {
        next();
        while (Character.isDigit(peek()))
            next();
    }

    return Double.parseDouble(json.substring(start, index));
}
```

- 便宜機能メソッド
- 主に文字列を読み込んでどう処理するかについてのメソッド
    - インデックスを軸に処理してる姿が見える


```java
private char peek() {
    return json.charAt(index);
}

private char next() {
    return json.charAt(index++);
}

private void consume(char expected) {
    if (peek() != expected) {
        throw new RuntimeException("Expected " + expected);
    }
    index++;
}

private void skipWhitespace() {
    while (index < json.length() && Character.isWhitespace(peek())) {
        index++;
    }
}

private boolean match(String keyword) {
    if (json.startsWith(keyword, index)) {
        index += keyword.length();
        return true;
    }
    return false;
}
```

- これをJavaオブジェクトとして変換するためにも再帰が必要

```java
public class ToJavaObject {

    public static Object toJavaObject(Node node) {
        if (node instanceof ValueNode) {
            return ((ValueNode) node).value;
        }

        if (node instanceof ObjectNode) {
            Map<String, Object> map = new HashMap<>();
            ObjectNode obj = (ObjectNode) node;

            for (Map.Entry<String, Node> entry : obj.map.entrySet()) {
                // 再帰
                map.put(entry.getKey(), toJavaObject(entry.getValue()));
            }

            return map;
        }

        if (node instanceof ArrayNode) {
            List<Object> list = new ArrayList<>();
            ArrayNode arr = (ArrayNode) node;

            for (Node n : arr.list) {
                // 再帰
                list.add(toJavaObject(n));
            }

            return list;
        }

        throw new RuntimeException("Unknown node type");
    }

}
```

## <span style="color:#802548">_深く探求ーStringとStringBuilder_</span>

- Stringの ＋ 演算は新しいStringBufferを作ることになる
- 内部的には以下のようにソースコードが変換される

```java
reverse += str.charAt(i);

//
reverse = new StringBuilder(reverse)
              .append(str.charAt(i))
              .toString();
```

- そのため、新しいStringBufferは以前のStringBufferをコピーすること＋追加する演算が必要
- 1回づつ演算回数が増えるので、最終的に１＋２＋...＋ｎになる

```text
"abcd"


Iteration 1
    copy: 0 chars  X
    append: 1 char --->a
    → total work ≈ 1
Iteration 2
    copy: 1 char   --->a
    append: 1 char --->+b
    → total work ≈ 2
Iteration 3
    copy: 2 chars  --->ab
    append: 1 char --->+c
    → total work ≈ 3
Iteration 4
    copy: n-1 chars --->abc
    append: 1 char  --->+d
    → total work ≈ 4
    .
    .
    .
Iteration n
    copy: n-1 chars --->n-1 count char
    append: 1 char  ---> + nth char
    → total work ≈ n
```

- できれば、Stringには＋の演算はしないほうがいい

```java
public static String recursiveReverseString(String abc, StringBuilder reverse) {
    reverse.append(abc.charAt(abc.length() - 1 - n));
    if (n == abc.length()) {
        return reverse.toString();
    }
    return recursiveReverseString(abc, reverse);
}
```

- ただ、上にもまだ問題は残っている
- StringBuilderはサイズを超えるとオブジェクトのコピペのプロセスが走る
- Resizingが走ってもまだ時間計算量は O(n)ではあるが、不要な動作が入ってくしまうので、効率が悪くなる
- そのため、サイズをわかったら指定したほうがいい

```java
StringBuilder str = new StringBuilder("abcdefg".length());
```

- それはGolangも同じなので、初期容量を指定する

```go
var sb strings.Builder
sb.Grow(len("abcd"))
```



## <span style="color:#802548">_深く探求ー絵文字を含めた文字列_</span>

- 文字列反転させるで紹介したメソッドは絵文字などがあるとロジックが壊れる
- javaも絵文字を鑑みてappendCodePointメソッドを使うことにする

```java
String s = "😊a";

int[] codePoints = s.codePoints().toArray();

StringBuilder sb = new StringBuilder();
for (int i = codePoints.length - 1; i >= 0; i--) {
    sb.appendCodePoint(codePoints[i]);
}

System.out.println(sb.toString());
```

- ここは、Unicode code pointという概念をしっかり押さえていく必要がある

| Character | Code point |
| --------- | ---------- |
| `A`       | U+0041     |
| `a`       | U+0061     |
| `あ`       | U+3042     |
| `😊`      | U+1F60A    |


- Javaでも Unicode code pointを使ったように、Goでも使う
- runeがそのデータ型で、内部は int32である

```go
func ReverseString(s string) string {
	runes := []rune(s)
	for i, j := 0, len(runes)-1; i < j; i, j = i+1, j-1 {
		runes[i], runes[j] = runes[j], runes[i]
	}
	return string(runes)
}
```





## <span style="color:#802548">_深く探求ーSQLの再帰_</span>

- 典型的な例　ー＞ SQL 深さによる探索
- 最初のアドレスと最新のアドレスを探すことにしたら、再帰が必要

```
name            |pcode          |new_pcode(次のアドレス)
A               |20184          |20143
A               |20143          |20155
A               |20155          |
```

- 以下はMYSQLで使えるSQL

```sql
WITH RECURSIVE Explosion(name, pcode, new_pcode, depth)
AS
(
    SELECT name,pcode,new_pcode,1　                                /* depthは再帰カウンタ*/
    FROM AddressHistory
    Where name = 'A'
    AND new_code IS NULL　　　　　　                                /* 最初の検索スタート条件*/

    UNION ALL                                                   　 /* 結果を積み重ねる*/

    SELECT Child.name, Child.pcode, Child.new_pcode, depth + 1     /* 次の関数を呼び出す再帰処理 */
    FROM Explosion AS Parent, AddressHistory AS Child
    WHERE Parent.pcode = Child.new_pcode
    AND Parent.name = Child.name
)

SELECT name, pcode, new_pcode
FROM Explosion
WHERE depth = (SELECT MAX(depth) FROM Explosion);
```

- 結果値は以下

```
name        |pcode       |new_pcode
A           |20184       |20143
```

- oracle version.

```sql
WITH Explosion(name, pcode, new_pcode, depth)
AS
(
    SELECT name,pcode,new_pcode,1 /* depthは再帰カウンタ*/
    FROM AddressHistory
    Where name = 'A'
    AND new_code IS NULL /* 最初の検索スタート条件*/

    UNION ALL   /* 結果を積み重ねる*/

    SELECT Child.name, Child.pcode, Child.new_pcode, depth + 1   /* 次の関数を呼び出す再帰処理 */
    FROM Explosion AS Parent
    Inner JOIN AddressHistory AS Child
    on Parent.pcode = Child.new_pcode
    AND Parent.name = Child.name 
)

SELECT name, pcode, new_pcode
FROM Explosion
WHERE depth = (SELECT MAX(depth) FROM Explosion);
```

## <span style="color:#802548">_振り返り_</span>

- ベースケース以外のところであるリターン文はベースケースでは気にしなくていい
- ベースケースに達すると、処理が打ち切られて値が返るからだ

```go
func Factorial(n int) int {
	if n-1 < 0 {
		return 1
	}

	return n * Factorial(n-1)
}
```

- 再帰による結果値を追加する形が必ず必要

```go
func SumArray(arr []int, n int) int {
	if n > len(arr)-1 {
		return 0
	}

	return arr[n] + SumArray(arr, n+1)
}
```


- しかし、それがかならずしもリターン文の以内である必要はない

```java
public static String recursiveReverseString(String abc, StringBuilder reverse) {
    reverse.append(abc.charAt(abc.length() - 1 - n));
    if (n == abc.length()) {
        return reverse.toString();
    }
    return recursiveReverseString(abc, reverse);
}
```


- ベースケースがよく思いつかないとしたら、数字を左に移動させてからもう一度考える

```java
public static int factorial(int n) {
    if (n - 1  < 0) { // n < 0 より理解しやすい
        return 1;
    }
    return n * factorial(n - 1);
}
```