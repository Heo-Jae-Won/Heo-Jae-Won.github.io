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

- 해당 stream으로 구성해서 값을 분류하여 collection에 넣는다.

```java
Map<Boolean, List<Student>> stuBySex = stuStream.collect(partitioningBy(Student::isMale));
List<Student> maleStudent = stuBySex.get(true);    //Map에서 남성 목록을 얻어온다.
List<Student> femaleStudent = stuBySex.get(false); //Map에서 여성 목록을 얻어온다.

Map<Boolean,Long> stuNumBySex = stuStream.collect(partitioningBy(Student::isMale, counting()));
System.out.println("남학생 수: " + stuNumBySex.get(true)); //8
System.out.println("여학생 수: " + stuNumBySex.get(false)); //10

Map<Boolean,Optional<Student>> topScoreBySex = stuStream.collect(partitioningBy(Student::isMale, maxBy(comparingInt(Student::getScore))));
System.out.println("남학생 1등: " + topScoreBySex.get(true)); //Optional[[나자바,남,1,1,300]]. maxBy()는 Optional<Student>이 반환타입
System.out.println("여학생 1등: " + topScoreBySex.get(false));//Optional[[김지미,여,1,1,250]]

Map<Boolean,Optional<Student>> topScoreBySex = stuStream.collect(partitioningBy(Student::isMale, collectingAndThen(maxBy(comparingInt(Student::getScore), Optional::get))));
System.out.println("남학생 1등: " + topScoreBySex.get(true)); //남학생 1등: [나자바, 남, 1, 1, 300]. collectingAndThen을 추가해주면 Student가 반환타입.
System.out.println("여학생 1등: " + topScoreBySex.get(false));//여학생 1등: [김지미, 여, 1, 1, 250]

Map<Boolean, Map<Boolean,List<Student>>> failedStuBySex = stuStream.collect(partitioningBy(Student::isMale, partitioningBy(s -> s.getScore() < 150))); //partitioningBy안에서 또 partitioningBy를 호출해서 조건을 두개를 &&로 연결
List<Student> failedMaleStu = failedStuBySex.get(true).get(true); // 남성 중 성적이 150점 이하인 사람
List<Student> failedMaleStu = failedStuBySex.get(false).get(true); //여성 중 성적이 150점 이하인 사람

Map<Integer, List<Student>> stuByBan = stuStream.collect(groupingBy(Student::getBan, /*toList()*/)); //toList()는 생략가능
Map<Integer, HashSet<Student>> stuByHak = stuStream.collect(groupingBy(Student::getHak, toCollection(HashSet::new)));
```


- 좀 더 복잡한 예시로 점수에 따라 레벨을 나눠 그룹핑한다.

```java
Map<Student.Level, Long> stuByLevel = stuStream.collect(groupingBy(s->{
    if(s.getScore() >= 200)
        return Student.Level.HIGH;
    else if(s.getScore() >= 100)
        return Student.Level.MID;
    else
        return Student.Level.LOW;
},counting())) //[MID] - 8명, [HIGH] - 8명, [LOW] - 2명
```

- counting 하지 않고, Set으로 바꾼다.

```java
Map<Integer, Map<Integer,Set<Student.Level>>> stuByHakAndBan = stuStream.collect(GroupingBy(Student::getHak,groupingBy(Student::getBan,mapping(s->{
    if(s.getScore() >= 200)
        return Student.Level.HIGH;
    else if(s.getScore() >= 100)
        return Student.Level.MID;
    else
        return Student.Level.LOW;
}, toSet()))))
```


- 좀 더 복잡한 예시로 학년 별 그룹화 -> 반 별 그룹화를 진행한다.

```java
Map<Integer,Map<Integer,List<Student>>> stuByHakAndBan = stuStream.collect(
				Collectors.groupingBy(Student::getHak, 
						Collectors.groupingBy(Student::getBan)));
```

- maxBy의 return typ이 Optional이다.
- Optional(Student)가 아니라 그냥 Student로 바꿔주기 위해 추가해준다.
    - collectingAndThen
    - Optional::get

```java
Map<Integer, Map<Integer,Student>> topStuByHakAndBan = stuStream.collect(
    groupingBy(Student::getHak, groupingBy(
        Student::getBan, collectingAndThen(
            maxBy(comparingInt(Student::getScore),Optional::get)
            )
        )
    )
) //각 반의 1등을 출력
```


```java
Map<Integer,Map<Integer,List<Student>>> stuByHakAndBan = stuStream.collect(
    Collectors.groupingBy(Student::getHak, 
        Collectors.groupingBy(Student::getBan)
    )
); //학년별로 그룹화 후 반별로 다시 그룹화

for(Map<Integer,List<Student>> hak : stuByHakAndBan.values()){
for(List<Student> ban : hak.values()){
    System.out.println();
    for(Student s : ban){
        System.out.println(s);
    }
} 
} 
// [나자바, 남, 1학년 1반, 300점]
// [김지미, 여, 1학년 1반, 250점]
// [김자바, 남, 1학년 1반, 200점]

// [이지미, 여, 1학년 2반, 150점]
// [남자바, 남, 1학년 2반, 100점]
// [안지미, 여, 1학년 2반,  50점]

// [황지미, 여, 1학년 3반, 100점]
// [강지미, 여, 1학년 3반, 150점]
// [이자바, 남, 1학년 3반, 200점]

// [나자바, 남, 2학년 1반, 300점]
// [김지미, 여, 2학년 1반, 250점]
// [김자바, 남, 2학년 1반, 200점]

// [이지미, 여, 2학년 2반, 150점]
// [남자바, 남, 2학년 2반, 100점]
// [안지미, 여, 2학년 2반,  50점]

// [황지미, 여, 2학년 3반, 100점]
// [강지미, 여, 2학년 3반, 150점]
// [이자바, 남, 2학년 3반, 200점]


Map<Integer, Map<Integer,Student>> topStuByHakAndBan = stuStream.collect(
    Collectors.groupingBy(Student::getHak, 
            Collectors.groupingBy(Student::getBan, 
                    Collectors.collectingAndThen(
                            Collectors.maxBy(Comparator.comparingInt(Student::getScore)),
                                                                        Optional::get
                            )
                    )
            )
    ); //각 반의 1등을 출력


for(Map<Integer,Student> ban: topStuByHakAndBan.values())
for(Student s : ban.values())
    System.out.println(s);
    // [나자바, 남, 1학년 1반, 300점]
    // [이지미, 여, 1학년 2반, 150점]
    // [이자바, 남, 1학년 3반, 200점]
    // [나자바, 남, 2학년 1반, 300점]
    // [이지미, 여, 2학년 2반, 150점]
    // [이자바, 남, 2학년 3반, 200점]


Map<Integer, Map<Integer,Set<Student.Level>>> stuByHakAndBan = stuStream.collect(
    Collectors.groupingBy(Student::getHak,
        Collectors.groupingBy(Student::getBan,
            Collectors.mapping(s->{
                if(s.getScore() >= 200)
                    return Student.Level.HIGH;
                else if(s.getScore() >= 100)
                    return Student.Level.MID;
                else
                    return Student.Level.LOW;
                    }, Collectors.toSet()
            )
        )
    )
);

Set<Integer> keySet2 = stuByHakAndBan.keySet();

for(Integer key : keySet2){
    System.out.println("[" + key + "]" + stuByHakAndBan.get(key)); //key안에 담긴 요소는 map
}
// [1]{1=[HIGH], 2=[MID, LOW], 3=[MID, HIGH]}. 1학년 1반은 HIGH, 2반은 MID,LOW, 3반은 MID,HIGH
// [2]{1=[HIGH], 2=[MID, LOW], 3=[MID, HIGH]}
```


- 마지막 예시의 경우 아래와 같이 String을 key로 사용하는 Set으로 고칠 수 있다.


```java
Map<String,Set<Student.Level>> stuByScoreGroup = stuStream.collect(
                                                                Collectors.groupingBy(
                                                                    s->s.getHak() + "-" + s.getBan(), // 1-1, 1-2, 1-3... 등을 key로 아래와 같이 mapping됨.
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

Set<String> keySet2 = stuByScoreGroup.keySet();
for(String key: keySet2){
    System.out.println("[" + key + "]" + stuByScoreGroup.get(key)); //key안에 담긴 요소는 Set collection
}
// [1-1][HIGH]. 1학년 1반은 HIGH
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

## <span style="color:#802548">_net vs net.http_</span>

- net 

```java
// Verbose, stream-based, and synchronous only
URL url = new URL("https://example.com");
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");

try (BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream()))) {
    StringBuilder response = new StringBuilder();
    String line;
    while ((line = reader.readLine()) != null) {
        response.append(line);
    }
    System.out.println(response.toString());
}
```

- net.http

```java
// Readable, Builder pattern, and supports Async/HTTP2
HttpClient client = HttpClient.newHttpClient();
HttpRequest request = HttpRequest.newBuilder()
    .uri(URI.create("https://example.com"))
    .GET()
    .build();

// Simple one-liner to get the body as a String
HttpResponse<String> response = client.send(request, HttpResponse.BodyHandlers.ofString());
System.out.println(response.body());
```


- io

```java
// Manual buffer management is required
File source = new File("old.txt");
File dest = new File("new.txt");

try (InputStream in = new FileInputStream(source);
     OutputStream out = new FileOutputStream(dest)) {
    byte[] buffer = new byte[1024];
    int length;
    while ((length = in.read(buffer)) > 0) {
        out.write(buffer, 0, length);
    }
} catch (IOException e) { e.printStackTrace(); }
```

- nio

```java
// One method call that uses OS-level optimization
Path source = Paths.get("old.txt");
Path dest = Paths.get("new.txt");

try {
    Files.copy(source, dest, StandardCopyOption.REPLACE_EXISTING);
} catch (IOException e) { e.printStackTrace(); }
```

