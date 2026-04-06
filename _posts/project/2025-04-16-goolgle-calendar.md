---
title: "meeting-room-reservation"
published: true
---
## <span style="color:#802548">_考慮する場合の数：開始時間が終了時間以降にならないようにする_</span> 

- 会議室の予約に基本的に必要な条件を自分で考えて実装することになりました。
- 最初に考えたのは、開始時間が終了時間以降にならないようにすることです。
- 最初に活用したのは、java.timeパッケージのZonedDateTimeクラス クラスに属するメソッドのisBefore(), isAfter()でした。
- しかし、これだけでは不十分で、isBefore()とisAfter()というメソッドでは、会議が終わる時間に合わせて登録する場合、重複とみなされました。
    - 15:00に終わったら15:00開始で登録することがほとんどの場合だったので、isBeforeやisAfterメソッドは使うことができませんでした。

```java
//inputStartDateTime: ユーザーが選択した会議開始時間
//inputEndDateTime: ユーザーが選択した会議終了時間
boolean previousThaninputEndDateTime = inputStartDateTime.atZone(ZoneId.of("Asia/Seoul"))
															.isBefore(inputEndDateTime.atZone(ZoneId.of("Asia/Seoul")));
```

- compareToに変更し、15:00に終わったら15:00に始まってもエラーに見なさないようにしました。

```java
//inputStartDateTime: ユーザーが選択した会議開始時間
//inputEndDateTime: ユーザーが選択した会議終了時間
public void insertEvent(MeetingReqDto meetingReqDto) throws Exception {
	boolean previousThaninputEndDateTime = inputStartDateTime.atZone(ZoneId.of("Asia/Seoul"))
																.compareTo(inputEndDateTime.atZone(ZoneId.of("Asia/Seoul"))) < 0;
	if (!previousThaninputEndDateTime) {
		throw new BizException(ErrorEnum.CALENDAR_END_BEFORE_START);
	}

}
```

## <span style="color:#802548">_考慮する場合の数：スケジュールが重ならないようにする_</span> 

- 次の課題は、スケジュールが重複しないようにすることでした。
- 考えても考えても明確なアルゴリズムが浮かばなかったです。
- まずは全てのスケジュールが必要なのではなく、開始時間付近の2つのスケジュールがわかればよいという結論に至りました。
- したがって、ユーザが要求した開始時刻の前後に1つずつスケジュールを照会する。

```java
//inputStartDateTime: ユーザーが選択した会議開始時間
Events events = service.events().list("ddddd@gmail.com")
								.setTimeMin(inputStartDateTime)
								.setMaxResults(2)
								.setSingleEvents(true)
								.setOrderBy("starttime")
								.execute();
List<Event> items = events.getItems();
```

- もし前後のスケジュールがなければ、重複を考慮する必要がないので、スケジュールを作ってそこでアーリーリターンを行います。

```java
// 選択した時間帯に後日予定がない場合は、重複検査を行わず、すぐに登録する。
if (items == null) {
	service.events().insert("ddddd@gmail.com", event).execute();
	return;
}
```

- itemは前後のスケジュールで数が2つなので、for文で巡回します。
- 前のスケジュールと重複しても、後のスケジュールと重複してもいけないので、どちらか一つでも引っかかると例外をスローします。

```java
//inputEndDateTime: ユーザーが選択した会議終了時間
//listStartDateTime: 既存会議開始時間
ZonedDateTime listStartDateTime = null;
boolean previousThanListStartDateTime = false;
for (Event item : items) {
	listStartDateTime = ZonedDateTime.from(
											Instant.from(DateTimeFormatter.ISO_DATE_TIME.parse(item.getStart().getDateTime().toStringRfc3339()))
													.atZone(ZoneId.of("Asia/Seoul"))
											);

	// スケジュールの重複を確認します。終了時間と重なってすぐに開始する場合は、重複と見なさないです。
	previousThanListStartDateTime = inputEndDateTime.atZone(ZoneId.of("Asia/Seoul"))
													.compareTo(listStartDateTime) <= 0;
	if (!previousThanListStartDateTime) {
		throw new BusinessException(ErrorEnum.CALENDAR_ALREAY);
	}
}
```

## <span style="color:#802548">_考慮する場合の数：修正の際には自分自身を修正する場合は許可する_</span> 

- 入力の場合と全く同じアルゴリズムを使おうとしました。
- しかし、修正の場合、同じ時間帯に置いて修正すると、重複と判断する結果を引き起こしました。
- 実際、間違えて同じ時間帯で修正をしたのに、修正をしたら重複と表示されたら困惑することでしょう。
- この場合、重複でも問題ないため、そのまま修正されるようにアルゴリズムを修正しました。

```java
List<Event> filteredItems = items.stream().filter(element -> {
	return !element.getId().equals(meetingReqDto.getEventId());
}).collect(Collectors.toList());

for (Event item : filteredItems) { //not All item, but filteredItem
	//duplication check logic
}
```

## <span style="color:#802548">_考慮する場合の数：週末にはinsertしないようにする_</span> 

- とりあえず、プロントエンド側で週末の場合、別の色で塗って入力時に警告を出すようにしましたが、それだけでは不十分だと思いました。
- なので、週末にスケジュールを登録できないように週末かどうか確認するようにしました。
- 入力された値が週末であれば、わざわざGoogle cloud dbを活用する必要もないので、最初にチェックするように追加しました。

```java
private boolean isWeekend(String isoDateTimeString) {
	LocalDateTime dateTime = LocalDateTime.parse(isoDateTimeString);
	DayOfWeek dayOfWeek = dateTime.getDayOfWeek();
	return ((dayOfWeek == DayOfWeek.SATURDAY) || (dayOfWeek == DayOfWeek.SUNDAY));
}

public void insertEvent(MeetingReqDto meetingReqDto) throws Exception {
	if(isWeekend(meetingReqDto.getStartDateTime())) {
		throw new BusinessException(ErrorEnum.CALENDAR_WEEKEND);
	}
}
```

## <span style="color:#802548">_考慮する場合の数：休日には登録しないようにする_</span>

- 週末を含めたので、祝日も除外することにしました。
- ただ、週末以外の休日を取得する方法はJAVA APIにはないので、別途に探さなければなりませんでした。
- 方法は国が提供するAPI、あるいは現在使っているGoogle Calendar APIでした。
- 類似性が高いGoogle Calendar APIを使うことにしました。
- 下記のように二つを取得させた結果、反応性が悪くなりました。最初に私たちが作ったスケジュールselectが完了した後、休日のスケジュールをselectする形だったからです。

```java
public void insertEvent(MeetingReqDto meetingReqDto) throws Exception {
	Events events = service.events().list("ddddd@gmail.com")
									.setTimeMin(startDateTime).setMaxResults(2)
									.setSingleEvents(true)
									.setOrderBy("starttime")
									.execute();
	List<Event> items = events.getItems();

	Events holidays = service.events()
							.list("ko.south_korea#holiday@group.v.calendar.google.com")
							.setTimeMax(lastDay)
							.setTimeMin(firstDay)
							.execute();
	List<Event> holidays = events.getItems();

	items.add(holidays);
}
```

- この部分を修正するには、同時にエーピーアイをコールすることが必要でした。
- その中でCompletableFutureを活用しました。
- parallelStream, ExceutorService + Future, Spring Reactiveなどの選択肢がありました。
    - parallelStreamやFuture classはエラー処理が難しかった。
    - Spring Reactiveは私たちのプロジェクトにない技術であり、学ぶのに時間がかかるという評価が多かったです。
    - なので、エラー処理が簡単で、fucntional chainingも可能なCompletableFutureを使うことにしました。
- 最初はOauth サービスを使うCalendarオブジェクトを各リクエストが共有して使う形にしました。

```java
public List<CalendarEventRes> getEventList(String firstDayOfMonth, String lastDayOfMonth)
		throws FileNotFoundException, IOException, GeneralSecurityException {
	Calendar service = getServiceAuth();

	CompletableFuture<Events> companyEventsCf = CompletableFuture.supplyAsync(() -> { 
		return service.events().list("ddddd@gmail.com")
								.setTimeMax(lastDay)
								.setTimeMin(firstDay)
								.execute(); 
	});
	CompletableFuture<Events> holidayEventsCf = CompletableFuture.supplyAsync(() -> { 
		return service.events().list("ko.south_korea#holiday@group.v.calendar.google.com")
								.setTimeMax(lastDay)
								.setTimeMin(firstDay)
								.execute(); 
	});

	List<Event> items =  companyEventsCf.thenCombine(holidayEventsCf, (prev, cur) -> {
		cur.putAll(prev);
		return cur;
	}).join().getItems();	

	return items;							
}
```

- しかし、Googleで作ったCalendarクラスがスレッドセーフなのか確信がありませんでした。
- スレッドセーフなクラスでないとマルチスレッド環境で問題を起こす可能性がありました。
- スレッドセーフでない共有オブジェクトなのでレースコンディションに陥る可能性があると判断しました。
- したがって、共有オブジェクトではなく、それぞれのスレッドで別々にオブジェクトを作る形に変更しました。

```java
public List<CalendarEventRes> getEventList(String firstDayOfMonth, String lastDayOfMonth)
		throws FileNotFoundException, IOException, GeneralSecurityException {
	Calendar service = getServiceAuth();

	CompletableFuture<Events> companyEventsCf = CompletableFuture.supplyAsync(() -> { 
		Calendar service = getServiceAuth();
		return service.events().list("ddddd@gmail.com")
								.setTimeMax(lastDay)
								.setTimeMin(firstDay)
								.execute(); 
	});
	CompletableFuture<Events> holidayEventsCf = CompletableFuture.supplyAsync(() -> { 
		Calendar service = getServiceAuth();
		return service.events().list("ko.south_korea#holiday@group.v.calendar.google.com")
								.setTimeMax(lastDay)
								.setTimeMin(firstDay)
								.execute(); 
	});

	List<Event> items =  companyEventsCf.thenCombine(holidayEventsCf, (prev, cur) -> {
		cur.putAll(prev);
		return cur;
	}).join().getItems();	

	return items;							
}
```

- リカバリーが必要の場合はexceptionally()を使います。
- ラムダ式なのでチェック例外は対応できませんのでラムダ式内にtry~catch文が必要です。
- チェック例外を CompletableFutureクラスで求められるCompletionExceptionに ラップします.

```java
public List<CalendarEventRes> getEventList(String firstDayOfMonth, String lastDayOfMonth)
		throws FileNotFoundException, IOException, GeneralSecurityException {
	Calendar service = getServiceAuth();

	CompletableFuture<Events> companyEventsCf = CompletableFuture.supplyAsync(() -> { 
		try {
			Calendar service = getServiceAuth();
			return service.events().list("ddddd@gmail.com")
									.setTimeMax(lastDay)
									.setTimeMin(firstDay)
									.execute(); 
		} catch (Exception ex) {
				throw new CompletionException(ex);
		}
	}).exceptionally(ex -> {
		return new Events();
	});

	CompletableFuture<Events> holidayEventsCf = CompletableFuture.supplyAsync(() -> { 
		try {
			Calendar service = getServiceAuth();
			return service.events().list("ko.south_korea#holiday@group.v.calendar.google.com")
									.setTimeMax(lastDay)
									.setTimeMin(firstDay)
									.execute(); 
		} catch (Exception ex) {
				throw new CompletionException(ex);
		}
	}).exceptionally(ex -> {
		return new Events();
	});

	List<Event> items =  companyEventsCf.thenCombine(holidayEventsCf, (prev, cur) -> {
		cur.putAll(prev);
		return cur;
	}).join().getItems();	

	return items;						
}
```

- 無限待機になってはいけないのでタイムアウトを設定します。
- orTimeoutメソッドはJDK1.9以上のバージョンでしか使用できません。

```java
public List<CalendarEventRes> getEventList(String firstDayOfMonth, String lastDayOfMonth)
		throws FileNotFoundException, IOException, GeneralSecurityException {
	Calendar service = getServiceAuth();

	CompletableFuture<Events> companyEventsCf = CompletableFuture.supplyAsync(() -> { 
		try {
			Calendar service = getServiceAuth();
			return service.events().list("ddddd@gmail.com")
									.setTimeMax(lastDay)
									.setTimeMin(firstDay)
									.execute(); 
		} catch (Exception ex) {
				throw new CompletionException(ex);
		}
	}).orTimeout(5, TimeUnit.SECONDS) 
	.exceptionally(ex -> {
		return new Events();
	});
	CompletableFuture<Events> holidayEventsCf = CompletableFuture.supplyAsync(() -> { 
		try {
			Calendar service = getServiceAuth();
			return service.events().list("ko.south_korea#holiday@group.v.calendar.google.com")
									.setTimeMax(lastDay)
									.setTimeMin(firstDay)
									.execute(); 
		} catch (Exception ex) {
				throw new CompletionException(ex);
		}
	}).orTimeout(5, TimeUnit.SECONDS) 
	.exceptionally(ex -> {
		return new Events();
	});

	List<Event> items =  companyEventsCf.thenCombine(holidayEventsCf, (prev, cur) -> {
		cur.putAll(prev);
		return cur;
	}).join().getItems();	

	return items;						
}
```

- 例外をスローする必要がある場合はexceptionally()ではなく、handle()を使います。
- 一つでもラムダ式で例外をスローしたら、thenCombine()も例外をスローします。
	- その例外はCompletionExceptionなので、プロントエンド側が処理できるようにBusinessExceptionにラップします。


```java
public List<CalendarEventRes> getEventList(String firstDayOfMonth, String lastDayOfMonth)
        throws FileNotFoundException, IOException, GeneralSecurityException {
    Calendar service = getServiceAuth();

    CompletableFuture<Events> companyEventsCf = CompletableFuture.supplyAsync(() -> {
        try {
            Calendar service = getServiceAuth();
            return service.events().list("ddddd@gmail.com")
                    .setTimeMax(lastDay)
                    .setTimeMin(firstDay)
                    .execute();
        } catch (Exception ex) {
            throw new CompletionException(ex); 
        }
    }).orTimeout(5, TimeUnit.SECONDS)  
      .handle((result, ex) -> {
          if (ex != null) {
              throw new CompletionException(ex);
          }
          return result;  
      });

    CompletableFuture<Events> holidayEventsCf = CompletableFuture.supplyAsync(() -> {
        try {
            Calendar service = getServiceAuth();
            return service.events().list("ko.south_korea#holiday@group.v.calendar.google.com")
                    .setTimeMax(lastDay)
                    .setTimeMin(firstDay)
                    .execute();
        } catch (Exception ex) {
            throw new CompletionException(ex);  
        }
    }).orTimeout(5, TimeUnit.SECONDS)  
      .handle((result, ex) -> {
          if (ex != null) {
              throw new CompletionException(ex);
          }
          return result;  
      });

    try {
		List<Event> items = companyEventsCf.thenCombine(holidayEventsCf, (prev, cur) -> {
			cur.putAll(prev);
			return cur;
		}).join();

		return items;	
	} catch (CompletionException ex) {
		throw new BusinessException(ErrorEnum.CALENDAR_GOOGLE_FAILED); 
	}
}
```


- ラムダ式の中でtry-catch文が繰り返されると、可読性が下がります。
- チェック例外を扱えるようにするためのラッパークラスを作成します。

```java
public class LambdaExceptionWrapper {
    @FunctionalInterface
    public interface SupplierWithException<T, E extends Exception> {
        T get() throws E;
    }

    public static <T, E extends Exception> Supplier<T> checkedExceptionWrapper(SupplierWithException<T, E> supplier) {
        return () -> {
            try {
                return supplier.get(); //変わった部分
            } catch (Exception e) {
                throw new RuntimeException(e);　//チェック例外をラップしてruntimeExceptionに変えます。
            }
        };
    }
}
```


- ラムダ式の中にチェック例外を扱えるラッパークラスを通じてtry~catch文を削除して可読性を上げました。

```java
import static foo.bar.LambdaExceptionWrapper.checkedExceptionWrapper;

public List<CalendarEventRes> getEventList(String firstDayOfMonth, String lastDayOfMonth)
        throws FileNotFoundException, IOException, GeneralSecurityException {

    Calendar service = getServiceAuth();

    // Wrapping the supplier with the exception handler
    CompletableFuture<Events> companyEventsCf = CompletableFuture.supplyAsync(checkedExceptionWrapper(() -> {
        Calendar service = getServiceAuth();
        return service.events().list("ddddd@gmail.com")
                .setTimeMax(lastDay)
                .setTimeMin(firstDay)
                .execute();
    })).orTimeout(5, TimeUnit.SECONDS)
      .handle((result, ex) -> {
          if (ex != null) {
              throw new CompletionException(ex);
          }
          return result;
      });

    CompletableFuture<Events> holidayEventsCf = CompletableFuture.supplyAsync(checkedExceptionWrapper(() -> {
        Calendar service = getServiceAuth();
        return service.events().list("ko.south_korea#holiday@group.v.calendar.google.com")
                .setTimeMax(lastDay)
                .setTimeMin(firstDay)
                .execute();
    })).orTimeout(5, TimeUnit.SECONDS)
      .handle((result, ex) -> {
          if (ex != null) {
              throw new CompletionException(ex);
          }
          return result;
      });

    try {
        List<Event> items = companyEventsCf.thenCombine(holidayEventsCf, (prev, cur) -> {
            cur.putAll(prev);
            return cur;
        }).join();

        return items;
    } catch (CompletionException ex) {
        throw new BusinessException(ErrorEnum.CALENDAR_GOOGLE_FAILED);
    }
}
```

## <span style="color:#802548">_必要な情報だけジェイソンデータで送る_</span>

- 最終的にAPIを完成して私がメンタリングをする時貰ったサーバーにデプロイをした後、プロント側と協業を始めました。
- フロント側で必要ないデータは受け取っても目障りなだけで、ネットワークトラフィックが増えるので、必要な値だけ送ってほしいと言われました。
- ハッシュマップを使ってその要件を実装しました。

```java
public HashMap<String, Object> getEvent(String eventId)
		throws FileNotFoundException, IOException, GeneralSecurityException {
	Calendar service = serviceAuth();

	Event event = service.events().get("ddddd@gmail.com", eventId).execute();
	HashMap<String, Object> map = new HashMap<String, Object>();
	map.put("summary", event.getSummary());
	map.put("description", event.getDescription());
	map.put("start", event.getStart());
	map.put("end", event.getEnd());
	map.put("id", event.getId());
	
	return map;
}
```

## <span style="color:#802548">_マップの代わりにDTOを使う_</span> 

- そして、*マップクラス*でプロント側にreturnするのはメンテナンスにとても良くないので、それをDtoやVOに変えるようにアドバイスをもらいました。
- 特に、プロント側とのコミュニケーションにSwaggerを使う場合、マップクラスはサポートされないので、DTOに変えてほしいと言われました。
- どのように入れるか悩みましたが、最初はリクエストDTOとレスポンスDTOを区別せずにそのまま入れました。
    - 何がリクエストで何がレスポンスなのか、変数名による区別が難しかったです。

```java
public class MeetingDto {
	
	@NotBlank
	private String startDateTime; 
	@NotBlank
	private String endDateTime;
	@NotBlank
	private EventDateTime startEventDateTime; 
	@NotBlank
	private EventDateTime endEventDateTime;
}
```

- さらに、リクエストDTOとレスポンスDTOに受け取るデータのタイプが違う必要がありました。
- レスポンスで根のデータとプロント側で受け取ったリクエストデータでGoogleに送る時、違うデータタイプが要求されたからです。
- 従って、レスポンスとリクエストDTOを分離して作りました。

```java
public class MeetingReqDto {
	
	@NotBlank
	private String startDateTime; 
	@NotBlank
	private String endDateTime;
}

public class MeetingResDto {

	@NotBlank
	private EventDateTime startDateTime; 
	@NotBlank
	private EventDateTime endDateTime;

}
```

