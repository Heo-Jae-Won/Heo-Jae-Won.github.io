---
title: スタックとキュー
published: true
categories: [algorithm]
---

## <span style="color:#802548">_実装ースタック_</span>
- Stackを実現するためには、LIFO仕組みを実装する必要がある
- 最初を配列には型パラメータに指定することができないことをわからなかったため、その部分だけ修正してもらった
- Javaの場合は、最後の要素をクリアする前にサイズを調節するのだが、私はGoLangと同じくしといた
- サイズを０にしたので、インデックスみたいに見えるかもしれないけど、これは容量として実装されてる
    - サイズが０から始まるのは、最初の要素の数は０だからだ

```java
import java.util.Arrays;

public class Stack<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_SIZE = 16;

    public Stack() {
        this.elements = (E[]) new Object[DEFAULT_SIZE]; // new E[DEFAULT_SIZE] impossible
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    private void ensureCapacity() {
        if (size < elements.length) {
            return;
        }
        elements = Arrays.copyOf(elements, 2 * size + 1);
    }

    public E pop() {
        if (size == 0) {
            throw new IllegalStateException("Stack is empty");
        }

        int lastIndex = size - 1;
        E element = elements[lastIndex];
        elements[lastIndex] = null; 
        size--;

        /*java version.
        int lastIndex = --size;
        E element = elements[lastIndex];
        elements[lastIndex] = null; // prevent memory leak
        */

        return element;
    }
}
```


- Golangでは配列ではなく、動的な配列であるSliceを使う
- なのでJavaと同じく実装する必要はない
    - サイズみたいな変数は消した

```go
type Stack[T any] struct {
    elements []T
}

func (stack *Stack[T]) Push(e T) {
    stack.elements = append(stack.elements, e)
}

func (stack *Stack[T]) Pop() (T, bool) {
    var zero T

    if len(stack.elements) == 0 {
        return zero, false
    }

    lastIndex := len(stack.elements) - 1
    e := stack.elements[lastIndex]
    stack.elements[lastIndex] = zero
    stack.elements = stack.elements[:lastIndex]

    return e, true
}


func (stack *Stack[T]) Peek() (T, bool) {
    var zero T

    if len(stack.elements) == 0 {
        return zero, false
    }

    return stack.elements[len(stack.elements)-1], true
}
```

- mainでの活用は以下のようになる
- Javaとは完全に違う文法を使っているため、慣れるに時間が非常にかかった

```go
func main() {
    var s Stack[int]

    s.Push(10)
    s.Push(20)

    // Peek
    if val, ok := s.Peek(); ok {
        fmt.Println("Top:", val)
    } else {
        fmt.Println("Stack is empty")
    }

    // Pop
    if val, ok := s.Pop(); ok {
        fmt.Println("Popped:", val)
    }

    if val, ok := s.Pop(); ok {
        fmt.Println("Popped:", val)
    }

    // Empty case
    if val, ok := s.Pop(); !ok {
        fmt.Println("Nothing to pop:", val) // val is zero value (0 for int)
    }
}
```

## <span style="color:#802548">_実装ースタックで文字列を反転させる_</span>
- Stackを利用した例

```java
Stack<Character> stack = new Stack<>();

for (char c : "string".toCharArray()) {
    stack.push(c);
}

StringBuilder strBuilder = new StringBuilder();

while (stack.getSize() > 0) {
    strBuilder.append(stack.pop());
}

String reverse = strBuilder.toString();
System.out.println(reverse);
```

## <span style="color:#802548">_実装ーキュー_</span>

- 最初に私が実現したQueueはStackとほぼ同じ仕組みだった
- 当時はサーキュラーバッファという概念を知らなかった
- ただ、ensureCapacity()をスタックと同じくしたから、poll() が O(n) になってしまい、データが大量になると、性能が悪化する

```java
import java.util.Arrays;

public class Queue<E> {
    private E[] elements;
    private int size = 0;
    private static final int DEFAULT_SIZE = 16;

    public Queue() {
        this.elements = (E[]) new Object[DEFAULT_SIZE]; 
    }

    public void push(E e) {
        ensureCapacity();
        elements[size++] = e;
    }

    private void ensureCapacity() {
        if (size < elements.length) {
            return;
        }
        elements = Arrays.copyOf(elements, 2 * size + 1);
    }

    public E poll() {
        if (size == 0) {
            throw new IllegalStateException("Stack is empty");
        }

        E element = elements[0];

        //drag all value in front of one element
        for (int i = 1; i < elements.length; i ++) {
            elements[i-1] = elements[i];
        }
        elements[--size] = null; 

        return element;
    }
}
```


- そのため、以下のようにロジックを直した
- 空間を確保するために、新しい配列を作る ensureCapacity()を呼び出す
- 以前にヌルとかになった空間は全部前にずれて、０からインデックスが始まらせる

```java
public class Queue<E> {
    private E[] elements;
    private int head = 0;
    private int tail = 0;
    private int size = 0;
    private static final int DEFAULT_SIZE = 16;

    public Queue() {
        elements = (E[]) new Object[DEFAULT_SIZE];
    }

    public void push(E e) {
        ensureCapacity();
        elements[tail] = e;
        tail = (tail + 1) % elements.length;
        size++;
    }

    public E poll() {
        if (size == 0) {
            throw new IllegalStateException("Queue is empty");
        }

        E e = elements[head];
        elements[head] = null;
        head = (head + 1) % elements.length;
        size--;

        return e;
    }

    private void ensureCapacity() {
        if (size < elements.length) return;

        E[] newArr = (E[]) new Object[elements.length * 2];

        for (int i = 0; i < size; i++) {
            newArr[i] = elements[(head + i) % elements.length];
        }

        elements = newArr;
        head = 0;
        tail = size;
    }
}
```

- Go言語に聞いたら、以下の通りだった
- append()ではなく q.elements[q.tail] = e のように直接代入している理由は、あらかじめ capacity（容量）を確保しているためである
    - append()を使うと、確保した初期容量の後ろに要素が追加されてしまい、head と tail の意味がなくなってしまう
    - また、q.tail = (q.tail + 1) % len(q.elements) とすることで、与えられた容量内で循環する（サーキュラーバッファとして動作する）ようにしている
- head と tail は単なる現在の位置を示すポインターではない
    - head は「読み出し位置」を指すインデックス
    - tail は「書き込み位置」を指すインデックス
- ただ、これは容量が調節できない

```go
type Queue[T any] struct {
    elements []T
    head     int
    tail     int
    size     int
}

func NewQueue[T any](capacity int) *Queue[T] {
    return &Queue[T]{
        elements: make([]T, capacity),
    }
}

func (q *Queue[T]) Push(e T) bool {
    if q.size == len(q.elements) {
        return false // full
    }

    q.elements[q.tail] = e
    q.tail = (q.tail + 1) % len(q.elements)
    q.size++
    return true
}

func (q *Queue[T]) Poll() (T, bool) {
    var zero T
    if q.size == 0 {
        return zero, false
    }

    result := q.elements[q.head]
    q.elements[q.head] = zero // important for GC
    q.head = (q.head + 1) % len(q.elements)
    q.size--

    return result, true
}
```


- 容量調節ができないのは、実戦で使えないので、調節も可能にする

```go
func (q *Queue[T]) resize(newCapacity int) {
    newElements := make([]T, newCapacity)

    // Copy in correct order starting from head
    for i := 0; i < q.size; i++ {
        newElements[i] = q.elements[(q.head+i)%len(q.elements)]
    }

    q.elements = newElements
    q.head = 0
    q.tail = q.size
}

func (q *Queue[T]) Push(e T) bool {
    // Resize when full
    if q.size == len(q.elements) {
        newCap := len(q.elements) * 2
        if newCap == 0 {
            newCap = 1
        }
        q.resize(newCap)
    }

    q.elements[q.tail] = e
    q.tail = (q.tail + 1) % len(q.elements)
    q.size++
    return true
}

func (q *Queue[T]) shrinkIfNeeded() {
    if len(q.elements) <= 1 {
        return
    }

    if q.size <= len(q.elements)/4 {
        q.resize(len(q.elements) / 2)
    }
}

func (q *Queue[T]) Poll() (T, bool) {
    var zero T
    if q.size == 0 {
        return zero, false
    }

    result := q.elements[q.head]
    q.elements[q.head] = zero
    q.head = (q.head + 1) % len(q.elements)
    q.size--

    q.shrinkIfNeeded()

    return result, true
}
```


## <span style="color:#802548">_深く探究ーデキューで文字列を反転させる_</span>

- ただのキューだけじゃ、時間計算量が非効率になる
- そのため、Dequeを作成して文字列を反転させる

```go
type Deque[T any] struct {
	buf      []T
	head     int
	tail     int
	size     int
	capacity int
}

func NewDeque[T any](cap int) *Deque[T] {
	if cap < 1 {
		cap = 1
	}
	return &Deque[T]{
		buf:      make([]T, cap),
		capacity: cap,
	}
}

func (d *Deque[T]) resize() {
	newCap := d.capacity * 2
	newBuf := make([]T, newCap)

	for i := 0; i < d.size; i++ {
		newBuf[i] = d.buf[(d.head+i)%d.capacity]
	}

	d.buf = newBuf
	d.head = 0
	d.tail = d.size
	d.capacity = newCap
}

func (d *Deque[T]) PushBack(v T) {
	if d.size == d.capacity {
		d.resize()
	}

	d.buf[d.tail] = v
	d.tail = (d.tail + 1) % d.capacity
	d.size++
}

func (d *Deque[T]) PushFront(v T) {
	if d.size == d.capacity {
		d.resize()
	}

	d.head = (d.head - 1 + d.capacity) % d.capacity
	d.buf[d.head] = v
	d.size++
}

func (d *Deque[T]) PopFront() T {
	if d.size == 0 {
		panic("empty deque")
	}

	v := d.buf[d.head]

	var zero T
	d.buf[d.head] = zero // avoid memory leak

	d.head = (d.head + 1) % d.capacity
	d.size--

	return v
}

func (d *Deque[T]) PopBack() T {
	if d.size == 0 {
		panic("empty deque")
	}

	d.tail = (d.tail - 1 + d.capacity) % d.capacity
	v := d.buf[d.tail]

	var zero T
	d.buf[d.tail] = zero

	d.size--

	return v
}

func (d *Deque[T]) Size() int {
	return d.size
}

func ReverseString(s string) string {
	d := NewDeque[rune](len(s))

	for _, ch := range s {
		d.PushBack(ch)
	}

	result := make([]rune, 0, len(s))

	for d.Size() > 0 {
		result = append(result, d.PopBack())
	}

	return string(result)
}
```


## <span style="color:#802548">_深く探究ーかっこバリデーション_</span>
- かっこが一個入ってくると、ペアになるかっこも入ってくるはずだと思ったが、どうすればペアで管理ができるのかが全然わからなかった
- 結局これは解決できず、ChatGPTに聞いた
- 以下はJava版だが、ペアで管理する発想には至らなかった
- Mapというデータ型をこう利用する方法があると勉強になった

```java
import java.util.*;

public class BracketValidator {

    public static boolean isValid(String s) {
        Deque<Character> stack = new ArrayDeque<>();

        Map<Character, Character> pairs = Map.of(
            ')', '(',
            '}', '{',
            ']', '['
        );

        for (char ch : s.toCharArray()) {

            if (ch == '(' || ch == '{' || ch == '[') {
                stack.push(ch);

            } else if (ch == ')' || ch == '}' || ch == ']') {

                if (stack.isEmpty()) {
                    return false;
                }

                char top = stack.pop();

                if (top != pairs.get(ch)) {
                    return false;
                }
            }
        }

        return stack.isEmpty();
    }

    public static void main(String[] args) {
        System.out.println(isValid("()[]{}")); // true
        System.out.println(isValid("(]"));     // false
        System.out.println(isValid("([)]"));   // false
        System.out.println(isValid("{[]}"));   // true
    }
}
```


## <span style="color:#802548">_深く探究ー言語のAPIを使うのが推薦される理由_</span>
- 以下のようにQueueのPollメソッドを実現した

```java
public E poll() {
    if (size == 0) {
        throw new IllegalStateException("Stack is empty");
    }

    E element = elements[0];

    //drag all value in front of one element
    for (int i = 1; i < elements.length; i ++) {
        elements[i-1] = elements[i];
    }
    elements[--size] = null; 

    return element;
}
```

- elements.length は容量（capacity）なので、実際には使用されていない領域までループしてしまう
- そのため、実際に使用されている要素数を表す size を使うようにする

```java
for (int i = 1; i < size; i++) {
    elements[i - 1] = elements[i];
}
```

- 上とやることは全く同じだが、以下がNativeMethodでLow-levelに近いからより速い

```java
System.arraycopy(elements, 1, elements, 0, size - 1);
```


## <span style="color:#802548">_深く探究ーvar s Stack[int]とvar s *Stack[int]の違い_</span>


- 下記の場合を仮定してみると、

```go
type Stack[T any] struct {
    data []T
}
```

- var s Stack[int]は、s.data = nil になる
- コンストラクタが不要
- newやmakeなども不要
- cuz “Zero value should be usable without initialization” is go's goal
- ただし、var s *Stack[int]は、s = nil になる
    - つまり、構造体が存在しないということだ
    - 従って、s.Push(10) を実行しても、s が nil なので、panicに陥る
        - あれはJavaで例えば、null.Pushと同じ

```go
var s Stack[int]
s.data = append(s.data, 10)

x := s.data[0]
x = 20

fmt.Println(s.data[0]) // still 10
```

- そのため、実質的に下記の組み合わせで使うことになる
- これは、Tの容量が小さいとき、状態を共有する必要がないとき、すなわち、一般的によく使われる

```go
type Stack[T any] struct {
    data []T
}

var s Stack[int]
s.data = append(s.data, 10)

x := s.data[0]
x = 20

fmt.Println(s.data[0]) 
```



- 一方、下記の場合を仮定してみると、

```go
type Stack[T any] struct {
    data []*T
}
```

- 値を格納するためには、全てをポインター化して入れることになる
- 値を取り出すときは、間接参照して値を取り出すことになる

```go
s := Stack[int]{}
val := 10
s.data = append(s.data, &val)

*s.data[0] = 20　//間接参照(dereferencing)

fmt.Println(*s.data[0]) 
```


## <span style="color:#802548">_振り返り_</span>

- 効率に優れるキューを実装するために、サーキュラーバッファーの形で実装する必要があるという考えに及ばなかった
- そのため、headとtailというポインターの役割を担当する架空の概念が思いつかなかった

```go
head int
tail int
```

- head と　tailの意味をちゃんと知らなかった
- そのため、q.elements[q.elements.head + 1] = e のように式を作成した
- headが読み出すところ、tailが書き込むところだとわかってからは、以下を理解した

```go
、q.elements[q.elements.head] = e 
```

- サーキュラーバッファーという意味がよくわからなくて、append()で要素を追加してしまった
- サーキュラーバッファーという意味がよくわからなくて、ただの tail++ でtail値を計算してしまった
    - 確保した容量の中で循環させるためには、配列の長さで割り算の余りを取る必要がある。

```go
q.elements[q.tail] = e
tail = (tail + 1) % len(q.elements);
```



