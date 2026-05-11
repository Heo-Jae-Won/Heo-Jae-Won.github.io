## <span style="color:#802548">_JDK変化の主要変化_</span>
1.2：コレクションフレームワーク (Collection Framework)
1.4：NIOパッケージ（Pathクラス用ユーティリティのFiles、新機軸のブロックI/OであるPath、ノンブロッキングI/OのChannels）
1.5：ジェネリクス、列挙型 (Enum)、アノテーション、オートボクシング、拡張for文、java.util.concurrent（Future, Executor, ExecutorService, 並行コレクション, ロック, アトミック変数）
1.7：try-with-resources文、switch文での文字列利用、ダイヤモンド演算子 (<>) による型推論、非同期I/O、Fork/Joinフレームワーク
1.8：Date and Time API (LocalDateTime)、ストリーム API、ラムダ式、CompletableFuture
1.9：JDKのモジュール化 (Project Jigsaw)、Reactive Streams用のFlow API
11：HTTP Client API
14：switch式15：テキストブロック
16：レコード型 (record)
17：封印クラス (sealed)
21：仮想スレッド (Virtual Threads)



## <span style="color:#802548">_JDK変化を目的ごとに_</span>
- コレクション
    - コレクションを扱うため：collection framework (1.2)
        - List, Set, Map
    - 並行処理用途のコレクションを扱うため： Concurrent Collections(1.5)
        - CopyOnWriteArrayList, ConcurrentHashMap, ConcurrentLinkedDeque, PriorityBlockingQueue
    - コレクション操作を宣言的に記述するため： Stream(1.8)
- タイプ
    - コレクションのタイプの安全性を確保するため： generics (1.5)
    - 定数の安全性を確保するため： enum (1.5)
- ファイル I/O
    - バッファとチャネルによる高速化するため： nio package (1.4)
        - ByteBuffer, SocketChannel, Selector
    - パス単位の操作や非同期I/Oをサポートするため： nio.2 package  (1.7)
        - Path, Paths, Files, AsynchronousFileChannel
    - ストリームの開閉を自動化するため：try with resources (1.7)
- スレッド
    - スレッドの低レベルな管理を抽象化し、並行処理の実装を容易にするため： java.util.concurrent (1.5)
        - Executor, ExecutorService, Executors, ThreadPoolExecutor, Future, Lock, ReentrantLock, AtomicInteger 
    - 大量のリソースを効率的に分割して並列処理するため： Fork/Join framework (1.7)
        - RecursiveTask, RecursiveAction
    - コールバック地獄を避け、非同期処理を繋げるため：CompletableFuture (1.8)
    - 軽量なスレッドで大量の並行処理を支えるため ：virtual threads (21)
- 不変性 
    - 日付操作の不変性とスレッドセーフを確保するため：LocalDateTime (1.8) 
    - 簡潔に不変なコレクションを生成するため：Immutable Collections Factory Methods (1.9)
    - 不変なデータ保持クラス：record class (16)
- 関数的プログラミング 
    - 処理を簡潔に記述・受け渡しするため：ラムダ式 (1.8)
- Reactive Streamsを実現するため：flow api (1.9)
- 標準でモダンなHTTP通信を行うため：HTTP Client API  (11)

## <span style="color:#802548">_メモリを節約する_</span>
- 事前に確定している値の計算には、静的コンストラクタを使用する
- 以下のメソッドは間違えている

```java
public boolean isBabyBoomer() {
    Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
    gmtCal.set(1946, Calendar.JANURAY, 1, 0, 0, 0);
    .3000
    Date boomStart = gmtCal.getTime();
    .
    .

}
```

- 特定のCalendarとDateのみ必要である場合、staticにおいて生成する

```java
static {
    Calendar gmtCal = Calendar.getInstance(TimeZone.getTimeZone("GMT"));
    gmtCal.set(1946, Calendar.JANURAY, 1, 0, 0, 0);
    Date boomStart = gmtCal.getTime();
    .
    .
}

public boolean isBabyBoomer() {
    return birtDate.compareTo(BOOM_START) >= 0 &&
        birthDate.compareTo(BOOM_END) < 0;
}

```


- innerclassは静的クラスで宣言
- 特に外部のクラスの状態を知らなくてもいいときは、必ず性的クラスにすること
    - static内部クラスは隠れた参照がない
    - 外部のクラスがGCに除去される
    - 内部クラスを生成するため、外部クラスのオブジェクトを生成する必要がなくなる

```java
public class MyLinkedList {
    private Node head;

    // STATIC: Each Node is small. It only stores 'data' and 'next'.
    // It has NO reference back to the MyLinkedList instance.
    public static class Node {
        int data;
        Node next;

        Node(int data) {
            this.data = data;
        }
    }

    public void add(int data) {
        // Can be created without an outer instance reference
        Node newNode = new MyLinkedList.Node(data);
        // ... logic to attach node
    }
}
```


- 外部のプロパティにアクセスが必要な時は静的クラスに宣言してはいけない

```java
public class MyLinkedList {
    private Node head;
    private int size;

    public class ListIterator {
        private Node current = head;
        .
        .

    }
}
```

- ただ、ウェブ開発ではDTOとかモデルクラスだとしたら、必ずStaticキーワードを使用したほうがいい


## <span style="color:#802548">_Enumの利用し方_</span>

- 定数はEnumを使うこと
- Enumもメソッドが実装できることを忘れないこと

```java
public enum Operation {
    PLUS { double apply(double x, double y) { return x + y;}},
    MINUS { double apply(double x, double y) { return x - y;}},
    TIMES { double apply(double x, double y) { return x * y;}},
    DIVIDE { double apply(double x, double y) { return x / y;}},

    abstract double apply(double x, double y);
}
```

- ２つのプロパティがあっても１つのプロパティだけを受け取っても問題ない

```java
enum Transportation{
    BUS(100){
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    }

    TRAIN(150){
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    }

    SHIP(100){
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    }

    AIRPLANE(300){
        int fare(int distance){
            return distance * BASIC_FARE;
        }
    }

    abstract int fare(int distance);

    protected final int BASIC_FARE;

    Transportation(int basicFare){
        BASIC_FARE = basicFare;
    }

    public int getBasicFare(){
        return BASIC_FARE;
    }
}

System.out.println("bus fare =  " + Transportation.BUS.fare(100)); //100 * 100
System.out.println("train fare =  " + Transportation.TRAIN.fare(100)); //150 * 100
System.out.println("ship fare =  " + Transportation.SHIP.fare(100)); // 100 * 100
System.out.println("airplane fare =  " + Transportation.AIRPLANE.fare(100)); //300 * 100
```

- enumにも内部Enumを実装することもできる

```java
enum PayrollDay {
    MONDAY(PayType.WEEKDAY), TUESDAY(PayType.WEEKDAY),
    WEDNESADAY(PayType.WEEKDAY), THURSDAY(PayType.WEEKDAY), 
    FRIDAY(PayType.WEEKDAY),
    SATURDAY(PayType.WEEKEND), SUNDAY(PayType.WEEKEND),

    private final PayType payType;
    PayrollDay(PayType payType) {
        this.paytType = payType;
    }

    double pay(double hoursWorked, double payRate) {
        return payType.pay(hoursWorked, payRate)
    }

    private enum PayType {
        WEEKDAY {
            double overtimePay(double hours, double payRate) {
                return hours <= HOURS_PER_SHIFT ? 0 :
                    (hours - HOURS_PER_SHIFT) * payRate / 2;
            }
        },
        WEEKEND {
            double overtimePay(double hours, double payRate) {
                return horus * payRate /2;
            }
        };

        private static final int HOURS_PER_SHIFT = 8;

        abstract double overtimePay(double hrs, double payRate);

        double pay(double hoursWorked, double payRate) {
            double basePay = hoursWorked * payRate;
            return basePay + overtimePay(hoursWorked, payRate);
        }
    }
}
```



## <span style="color:#802548">_ジェネリクスのワイルドカードの再使用性_</span>
- sortというメソッドがジェネリクスを調べるためのいい例だ

```java
static void sort(List<T> list, Comparator<? super T> comp)
```

- ワイルドカードは特に再使用性に役に立つ
- sortが以下と同じく、ワイルドカードでない場合を仮定してみよう

```java
static void sort(List<T> list, Comparator<T> comp)
```

- そうなった場合、List<Apple>を整列するためには、Comparator<Apple>が必要になる

```java
class AppleComp implements Comparator<Apple>{
    public int compare(Apple t1, Apple t2){
        return t2.weight - t1.weight;
    }
}

Collections.sort(appleBox.getList(), new AppleComp());
```

- Grapeというクラスもできたら、同じCompartorを実装しなければならない

```java
class GrapeComp implements Comparator<Grape>{
    public int compare(Grape t1, Grape t2){
        return t2.weight - t1.weight;
    }
}

Collections.sort(grapeBox.getList(), new GrapeComp());
```

- そのため、 <? super T>というワイルドカードを使って上位クラスを作る

```java
class FruitComp implements Comparator<Fruit>{
    public int compare(Fruit t1, Fruit t2){
        return t1.weight - t2.weight;
    }
}

Collections.sort(appleBox.getList(), new FruitComp());
Collections.sort(grapeBox.getList(), new FruitComp());
```

- Fruitクラスで宣言してFruitCompを作っても sortはできる
- ただその場合、型パラメータがないから、Appleだけを指定することができない

```java
List<Fruit> list = new0 ArrayList<>();
list.add(new Grape("파란", 0));　　　　　　
list.add(new Apple("빨간",1));　　　　　　// Grapeだけの型を指定することができない
Collections.sort(list, new FruitComp());
for(Fruit f : list) {
    System.out.println(f.toString());
}
```

## <span style="color:#802548">_ジェネリクスメソッド_</span>

- メソッドのジェネリクスについては、クラスとは別で使える
- 以下のような書き方がある

```java
static Juice makeJuice(FruitBox<? extends Fruit> box)
static <T extends Fruit> Juice makeJuice(FruitBox<T> box)
```

- 呼び出すときの書き方はこちら

```java
FruitBox<Fruit> fruitBox = new FruitBox<Fruit>();
FruitBox<Apple> appleBox = new FruitBox<Apple>();

sysout(Juicer./* <Fruit> 省略可能*/ makeJuice(fruitBox));
sysout(Juicer./* <Apple> 省略可能*/ makeJuice(appleBox));
```

## <span style="color:#802548">_comparator_</span>






## <span style="color:#802548">_lambda表現_</span>
- ラムダ式で必要な例外処理はラムダ式内で行われるべき
- 以下のようがtry catch外で囲まれたら、エラーキャッチがうまく行かない

```java
try(BufferedWriter writer = Files.newBufferedWriter(Paths.get(W_FILENMAE))) {
    lines.forEach(s -> writer.write(s + '\n'));
} catch (IOException ioex) {
    ioex.printStackTrace();
    throw new UncheckedIOException(ioex);
}
```

- ラムダ式を本当に使いたかったなら、以下のようにするべき

```java
try(BufferedWriter writer = Files.newBufferedWriter(Paths.get(W_FILENMAE))) {
    lines.forEach(s -> {
    try {
        writer.write(s + '\n');
    } catch (IOException e) {
        throw new UncheckedIOException(e); // Convert to unchecked to bypass Consumer signature
    }
});
} catch (IOException ioex) {
    ioex.printStackTrace();
    throw new UncheckedIOException(ioex);
}

```

- ただ、読みづらいところがあるので、以下のようにForeach文に変換したほうがいい

```java
try (BufferedWriter writer = Files.newBufferedWriter(Paths.get(W_FILENMAE))) {
    for (String s : lines) {
        writer.write(s + '\n'); // The outer catch will catch this!
    }
} catch (IOException ioex) {
    ioex.printStackTrace();
    throw new UncheckedIOException(ioex);
}
```


## <span style="color:#802548">_streamの中間操作と最終端操作_</span>
- streamの最終端操作を行ったときは、該当するstreamはもう使えない

```java
Stream<Event> itemStream=items.stream();
List<Event> items = itemStream.filter(element -> {
			return !element.getId().equals(calendarEventReq.getEventId());
	    }).collect(Collectors.toList());
int count = itemStream.count(); // error
```

- １つのstreamオブジェクトをいつくかのオブジェクトに分けて処理することもできない
- そのため、１つはコメントにする

```java
 Stream<String[]> strArrStream = Stream.of(
		            new String[]{"abc","def","jkl"},
		            new String[]{"ABC","DEF","JKL"}
		        );

//Stream<Stream<String>> strStrmStrm = strArrStream.map(Arrays::stream); 
Stream<String> strStrm = strArrStream.flatMap(Arrays::stream); 
```

- 中間操作として使われるのは以下のメソッドがある

```java
map()
sorted()
filter()
```

- 最終操作として使われるのは以下のメソッドがある

```java
forEach()
reduce()
collect()
```

- 頻繫に使われるmapの用例を見てみよう

```java
Collection<? extends GrantedAuthority> authorities = Arrays.stream(claims.get("auth").toString().split(","))
                .map(SimpleGrantedAuthority::new)
                .collect(Collectors.toList());
```

- mapの中ではflatmapというメソッドもある

```java
Stream<String[]> arrayStream = Stream.of(
    new String[]{"abc","def","jkl"},
    new String[]{"ABC","DEF","JKL"}
);

Stream<Stream<String>> doubleStream = arrayStream.map(Arrays::stream);  // ただのmapではForeachとかが上手くいかない
Stream<String> singleStream = arrayStream.flatMap(Arrays::stream);      // 上手くいく

strStrm.map(String::toLowerCase) 
        .distinct()
        .sorted()
        .forEach(System.out::println);
```

- 頻繫に使われるfilterの用例を見てみよう

```java
Stream<Event> itemStream=items.stream();
List<Event> items = itemStream.filter(element -> {
			return !element.getId().equals(calendarEventReq.getEventId());
	    }).collect(Collectors.toList());
```

- 最終操作のキーワードとしてgroupingByとpartioningByが存在する
- 以下の単純な学生クラスを仮定してみよう

```java
public class Student{
    String name;
    boolean isMale;
    int hak;
    int ban;
    int score;

    getter()
    setter()

    enum Level{HIGH, MID, LOW}
}
```


## <span style="color:#802548">_streamのcollectのpartitiongByとgroupingBy_</span>

- partitioningByはPredicateを受け取って、取り出すときはgetと true, falseを指定してその条件に当てはまるListだけを返す
- 基本的な活用は以下となる

```java
Map<Boolean, List<Student>> stuBySex = stuStream.collect(partitioningBy(Student::isMale));
List<Student> maleStudent = stuBySex.get(true); 
List<Student> femaleStudent = stuBySex.get(false);
```

- そこでカウントを返したいと思うならば、counting()というメソッドも追加

```java
Map<Boolean,Long> stuNumBySex = stuStream.collect(partitioningBy(Student::isMale, counting()));
long maleStudentCount = stuNumBySex.get(true); 
long femaleStudentCount = stuNumBySex.get(false);
```

- 1番成績がいい学生を取るためには、以下となる

```java
Map<Boolean,Optional<Student>> topScoreBySex = stuStream.collect(partitioningBy(Student::isMale, maxBy(Comparator.comparingInt(Student::getScore))));
Optional<Student> maleStudentCount = topScoreBySex.get(true); 
Optional<Student> femaleStudentCount = topScoreBySex.get(false);
```

- Optionalを取り出すことが億劫であれば、直接中身を取り出しても問題ない

```java
Map<Boolean,Optional<Student>> topScoreBySex = stuStream.collect(partitioningBy(Student::isMale, collectingAndThen(maxBy(Comparator.comparingInt(Student::getScore), Optional::get))));
Student maleStudentCount = topScoreBySex.get(true); 
Student femaleStudentCount = topScoreBySex.get(false);
```

- 連続してpartitioningByをすることもできる

```java
Map<Boolean, Map<Boolean,List<Student>>> failedStuBySex = stuStream.collect(partitioningBy(Student::isMale, partitioningBy(s -> s.getScore() < 150)));
List<Student> failedMaleStu = failedStuBySex.get(true).get(true); 
List<Student> failedMaleStu = failedStuBySex.get(false).get(true); 
```

- groupingByの基本的な活用方法は以下となる
- 条件に当てはまるかどうかを判断するPredicatteは受け取ってない

```java
Map<Integer, List<Student>> stuByBan = stuStream.collect(groupingBy(Student::getBan, toList()));
Map<Integer, HashSet<Student>> stuByHak = stuStream.collect(groupingBy(Student::getHak, toCollection(HashSet::new)));
```

- groupingByでも条件に応じて分類することができる
- 以下はEnumキーを指定する流れのソースコード

```java
Map<Student.Level, Long> stuByLevel = stuStream.collect(groupingBy(s->{
    if(s.getScore() >= 200)
        return Student.Level.HIGH;
    else if(s.getScore() >= 100)
        return Student.Level.MID;
    else
        return Student.Level.LOW;
},counting())) //[MID] - 8, [HIGH] - 8, [LOW] - 2
long highCount = stuByLevel.getOrDefault(Student.Level.HIGH, 0L); 
long lowCount = stuByLevel.getOrDefault(Student.Level.LOW, 0L); 
```

- 学年ごとに分けてから、もう1度クラスの成績ごとに分ける

```java
Map<Integer, Map<Integer, Map<Student.Level, List<Student>>>> stuByHakAndBanMap = 
    stuStream.collect(
        groupingBy(Student::getHak, 
            groupingBy(Student::getBan, 
                groupingBy(s -> {
                    if(s.getScore() >= 200) return Student.Level.HIGH;
                    else if(s.getScore() >= 100) return Student.Level.MID;
                    else return Student.Level.LOW;
                })
            )
        )
    );

List<Student> highScoreStudents = stuByHakAndBanMap.get(1).get(2).get(Student.Level.HIGH);
```

- maxByの戻り値がOptionalのため、Optional：：getを追加する

```java
Map<Integer, Map<Integer,Student>> topStuByHakAndBan = stuStream.collect(
    groupingBy(Student::getHak, groupingBy(
        Student::getBan, collectingAndThen(
            maxBy(Comparator.comparingInt(Student::getScore),Optional::get)
            )
        )
    )
)
Student highestScoreStudent = topStuByHakAndBan.get(1).get(2);
```

- 必要な要素はvalueにあるため、Mapをループするためには、values()を使う

```java
for(Map<Integer,Student> ban: topStuByHakAndBan.values()) {
    for(Student s : ban.values())
        System.out.println(s);
}
```

- groupingByのキーの場合は、操作して作っても問題ない


```java
Map<String,Set<Student.Level>> stuByScoreGroup = stuStream.collect(
                                                                Collectors.groupingBy(
                                                                    s->s.getHak() + "-" + s.getBan(), // 1-1, 1-2, 1-3..
                                                                    Collectors.mapping(s -> {
                                                                        if(s.getScore() >= 200)
                                                                            return Student.Level.HIGH;
                                                                        else if(s.getScore() >= 100)
                                                                            return Student.Level.MID;
                                                                        else
                                                                            return Student.Level.LOW;
                                                                    }, Collectors.toSet())
                                                                )
                                                        );
```

- ループは以下の通りになる
- 学生リストは取得できないが、各学年とクラスごとのレベルは一発で把握できる

```java
Set<String> keySet = stuByScoreGroup.keySet();
for(String key: keySet){
    System.out.println("[" + key + "]" + stuByScoreGroup.get(key));
}
// [1-1][HIGH]
// [2-1][HIGH]
// [1-2][MID, LOW]
// [2-2][MID, LOW]
// [1-3][MID, HIGH]
// [2-3][MID, HIGH]
```





## <span style="color:#802548">_Optionalオブジェクト_</span>

- OptionalオブジェクトがはNull処理をしなくても済むから利便性が高い
- ofはNot-nullで、OfNullableはNullable
- Nullである可能性があるオブジェクトはOfNullableが必要


```java
String str = "abc";
Optional<String> optVal = Optional.of(str);  
Optional<String> optVal = Optional.ofNullable(str);
```

- Optionalオブジェクトを初期化するとき、Nullでなく、empty()を使う

```java
Optional<String> optVal = null; //ダメ
Optional<Integer> optVal = Optional.empty(); 
```

- Optionalオブジェクトの値を取り出すメソッドは主に２つある

```java
Optional<String> optVal = Optional.of(str);  
Optional<String> optVal = Optional.ofNullable(str);
String str2 = optVal.orElseGet(String::new); 
String str3 = optVal.orElseThrow(NullPointerException); 
```

- orElseGetとorElseThrowは全部ラムダ式を使うべき
- ちなみに、ラムダ式で生成されるストリングオブジェクトは定数プールでなく、Heapで生成されるので、new String("bbb")と同じ
    - メモリのアドレスが違くなる

```java
Optional<String> abc = Optional.ofNullable(str);
String str1 = abc.orElseGet("bbb");                         //ダメ
String optionalStr = abc.orElseGet(()->"bbb");              
String optionalStr = abc.orElseGet(()->new String("bbb")); 
System.out.println(str1 == optionalStr); // false
```

- Optionaオブジェクトの中身を取り出すためには、isPresent()を呼び出す

```java
if(Optional.ofNullalbe(str).isPresent()){
    System.out.println(str);
}
```

## <span style="color:#802548">_nio2 package_</span>
- fileじゃなく、Path+Files+Pathsを使う

```java
// One method call that uses OS-level optimization
Path source = Paths.get("old.txt");
Path dest = Paths.get("new.txt");

try {
    Files.copy(source, dest, StandardCopyOption.REPLACE_EXISTING);
} catch (IOException e) { e.printStackTrace(); }
```

- Path クラスの用例は以下となる
- パスがなければ、作成する

```java
Path newPath = Path.of("logs/app.log"); 
// Joining parts (handles slashes for you automatically)
Path docPath = Path.of("C:", "Users", "Docs", "notes.txt"); 
if (Files.notExists(path)) {
    // 2. Create the directory along with any missing parent folders
    Files.createDirectories(path);
    System.out.println("Directory structure created: " + path.toAbsolutePath());
} 
```

- 生成してファイルがあるとして、属性を見たいとき

```java
if (Files.exists(filePath)) {
    try {
        BasicFileAttributes attr = Files.readAttributes(filePath, BasicFileAttributes.class);

        System.out.println("Is Regular File: " + attr.isRegularFile());
        System.out.println("Is Directory:    " + attr.isDirectory());
        System.out.println("File Size:       " + attr.size() + " bytes");
        System.out.println("Created On:      " + attr.creationTime());
        System.out.println("Last Modified:   " + attr.lastModifiedTime());

    } catch (IOException e) {
        System.err.println("Failed to read file attributes: " + e.getMessage());
    }
} else {
    System.out.println("File does not exist at target path.");
}
```

- ofじゃなく、getでもファイルのパスを指定できる
- ファイルの名前とそのパスについてわかることができる

```java
Path baseDir = Paths.get("/home/user");
Path fullPath = baseDir.resolve("documents/resume.pdf");
Path fileName = fullPath.getFileName(); // resume.pdf
Path fileParent = fullPath.getParent(); // /home/user/documents
```


- ファイルの編集権限なども確認できる

```java
boolean isWritable = Files.isWritable(myPath);
boolean isHidden = Files.isHidden(myPath);
long size = Files.size(myPath); // Size in bytes
```

- ファイルの直接的操作

```java
// Copy (REPLACE_EXISTING allows overwriting the target)
Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);

// Move/Rename
Files.move(source, target);

// Delete (Throws exception if file doesn't exist)
Files.delete(target);
```

- ファイルをコピーするときのプロセス

```java
import java.io.IOException;
import java.nio.file.Files;
import java.nio.file.Path;
import java.nio.file.StandardCopyOption;
import java.nio.file.attribute.PosixFilePermission;
import java.nio.file.attribute.PosixFilePermissions;
import java.util.Set;

public class SecurityCopyDemo {

    public static void checkPrivilegeAndCopy(Path source, Path target) {
        try {
            // 1. Check if the file physically exists first
            if (Files.notExists(source)) {
                System.out.println("Error: Source file does not exist.");
                return;
            }

            // 2. Detect if the file lacks write permissions
            if (!Files.isWritable(source)) {
                System.out.println("File is read-only. Attempting to grant write privileges...");
                grantWritePrivilege(source);
            }

            // 3. Ensure the destination's parent folder exists
            if (target.getParent() != null && Files.notExists(target.getParent())) {
                Files.createDirectories(target.getParent());
            }

            // 4. Perform the copy operation (REPLACE_EXISTING handles overwrites)
            Files.copy(source, target, StandardCopyOption.REPLACE_EXISTING);
            System.out.println("File successfully copied to: " + target);

        } catch (IOException e) {
            System.err.println("Operation failed: " + e.getMessage());
        }
    }

    private static void grantWritePrivilege(Path path) throws IOException {
        String os = System.getProperty("os.name").toLowerCase();

        if (os.contains("win")) {
            // Windows Approach: Use legacy File wrapper mapping fallback
            path.toFile().setWritable(true, false); // true = writable, false = for all users
            System.out.println("Windows write attribute applied.");
        } else {
            // Linux / Unix / macOS Approach: Use explicit POSIX permissions
            try {
                // Read current permissions
                Set<PosixFilePermission> perms = Files.getPosixFilePermissions(path);
                
                // Append Owner and Group write bits
                perms.add(PosixFilePermission.OWNER_WRITE);
                perms.add(PosixFilePermission.GROUP_WRITE);
                
                // Update file configuration on disk
                Files.setPosixFilePermissions(path, perms);
                System.out.println("POSIX write permissions (Owner/Group) applied.");
            } catch (UnsupportedOperationException e) {
                // Safe fallback if filesystem does not support POSIX views
                path.toFile().setWritable(true, false);
            }
        }
    }

    public static void main(String[] args) {
        Path src = Path.of("restricted.txt");
        Path dest = Path.of("backup", "restored.txt");
        
        checkPrivilegeAndCopy(src, dest);
    }
}
```


- ファイルを読み込むときのプロセス
- newBufferedReaderはエンコーディングを指定できる

```java
import java.nio.file.Files;
import java.nio.file.Path;
import java.io.BufferedReader;
import java.io.IOException;

public class LargeFileProcessor {
    public static void main(String[] args) {
        Path largeFile = Path.of("huge_data.log");

        // Try-with-resources ensures the reader is closed automatically
        try (BufferedReader reader = Files.newBufferedReader(largeFile, StandardCharsets.ISO_8859_1)) {
            String line;
            // Reads only one line into memory at a time
            while ((line = reader.readLine()) != null) {
                // Process your data here
                if (line.contains("ERROR")) {
                    System.out.println("Error found: " + line);
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

- nio2のPathクラスたちの長所は以下となる
    - エラーハンドルがちゃんとできてる
    - シンボリックリンクを扱える
    - 一括操作や全般的なパフォーマンスに優れる
