---
title: "communication-batch"
published: true
categories: [project]
---
## <span style="color:#802548">_windowsがシャットダウンする際に行われる後処理_</span>
- ノートパソコンの場合、正常作動すると、ランプがずっと白い。
    - シャットダウンにすると、横のランプが点灯した後、完全に電源が切れる。
    - なのに、点灯してる何秒際に蓋を閉じると、ススリープ状態に入ってしまうことがある。
        - この場合、パソコンはすでにシャットダウンの命令を受けているため、レースコンディションが発生する。
        - 結果として操作ができなくなり、強制終了以外に手段がなくなる。
- 強制終了した場合、シャットダウン時に実行される処理は動作しない。
    - 特に「リモートデスクトップ For AVD」は、設定ファイルをシャットダウン時に Open → Write → Close するプロセスが動かない。
    - Close が行われなかったため、次回 Open も失敗する。
- これに関連するログは、設定ファイルだけでなく、レジストリやキャッシュなどにも残る。
    - 設定ファイルには改ざん検知用のチェックサムコードも入っているため、他人が直接編集できない。
    - そのため、リモートデスクトップ For AVD を使用する場合は、一度アンインストールしてから再インストールする必要がある。
- シャットダウンは、次回の起動を早くするために完全な初期化は行われない。
- そのため起動は速いが、設定の完全な初期化が必要な場合は再起動が必要。
- 今回のケースでは、設定ファイルが再起動でも復元できなかったため、再インストールした。
- この経験を通して、ウェブサーバーだけじゃなく、OSの段階でも後処理が行われることに気づいた。

## <span style="color:#802548">_windodwsのhostsファイル_</span>
- チームメンバーから、突然 SI2 環境への接続ができなくなったと助けを求められた。
- 彼は TerTerm というツールを使っているが、この TerTerm は Windows の HostFile を参照しているため、彼に自身の HostFile を渡すようにお願いした。
- 確認したところ、問題の環境の IP アドレスが HostFile に書いてなかった。
- それを追加すれば、接続できるはずだと伝えた。
- 念のため、私も同じ環境で実験を行った。

```
パスはこちら

C:\Windows\System32\drivers\etc\hosts
```

- 私が持っているその値を修正すると、多分彼と同じくできなくなるのではないかと予測した。
    - それで次のように該当の内容を消してから確認した結果、できなくなった。
    - 管理者ではないと操作が不可能なため、管理者としてテキスト編集をさせる必要がある。


## <span style="color:#802548">_文字列解釈に利用するLinuxコマンドの違い：S-jis環境でのless, catの動作の違い_</span>
- cat ではファイルの内容を文字化けせずに表示できたが、less では正しく表示できなかった。
- その違いを理解するために調べてみたところ、cat はバイト列をそのまま出力するだけで、解釈は Tera Term などのターミナルに任せる仕組みであることが分かった。
- したがって、cat は余計な処理を行わずシンプルに動作する。
- less で文字化けが発生した理由は、文字列をそのまま出力するのではなく、一定の規則に基づいて処理しているためである。

```
encoding detection

locale checks

multibyte handling

heuristics
```

- そのため、基本的にお勧めされるのはLessのほうだが、いろんなチェックをしてくれるからだ。
- しかも、Lessは基本的にエンコーディングがUTF-8のため、S-JISなどに合わせて解釈ができない。
- しかし、LessがCatより性能が高いことは否めない。

```
ページネーションができる
検索ができる
エスケープができる
フレーズされることがない
```

- そのため、Lessを使うことがおすすめされる。
- Lessが活用できるようになる方法は以下の通りのコマンドだ。
- このコマンドはファイルの内容を更新しないが、StdOut に表示される

```shell
iconv -f SHIFT_JIS -t UTF-8 | less
```



## <span style="color:#802548">_OSにおけるセッション：ユーザーセッションと管理者セッションの動作の違い_</span>
- substを使ってDドライブを作る必要がある作業があった。　
- なぜDドライブを作るのかは社内AIに質問投げてみたら、Dドライブにログなどを残す仕組みだ答えてもらった。
- 手順には管理者Cmdを開いてくださいと書いてあったため、管理者コマンドを開いて下記のコマンドを叩いた。

```shell
subst D: C:\mywork\Ddrive
```　　　　　　　　　　　　　　　

- しかし、Dドライブがsubstのコマンドでも作られなかった。　　　　　　　　　　
- まず成功したのかどうかから確認しようとした。
- そして確認ができる方法は下記のコマンドだった。

```shell
subst

# 成功したら、
# X:\: => C:\D$
```

- つまり、成功ということだ。それでもFileExplorerには表示されなかった。
- Cドライブの直下にしなければならないのかとか名前に制限があるのかなどいろいろ試してみてもできなかった。
- 結局自分のパソコンでもやってみたが、自分のパソコンではちゃんとできていた。
- 私に与えられたユーザーの権限の問題かと思いつつ、権限を確認してみた。

```bat
whoami /groups | findstr /i "admin"
```

- 「BUILTIN\Administrators Group used for deny only」が出てきたので、管理者権限は持っている状態だった。

```
BUILTIN\Administrators        Alias            S-1-5-32-544 Group used for deny only 
```


- 管理CmdとしてCMDを開いたら、下記のメッセージが表示される。

```
BUILTIN\Administrators       Alias            S-1-5-32-544 Mandatory group, Enabled by default, Enabled group, Group owner
```

- 管理権限は保持していると考えられるため、本件は権限の問題ではないと結論付けた。
- それで調べたところ、AdminTokenとUserTokenの違いについて説明が始まった。
- Substは基本的に今開いているSessionだけに影響を与えるコマンドだ
    - File ExplorerはいつもUserSessionなので、AdminTokenを利用してるAdminSessionでは開けない。
- FileExplorerもAdminSessionで開けば通じるのかと結論に行き着いて、試してみようとしたんだが、意図通りに動かなかった。
- AdminSessionで開いてもまだDドライブが見れない。

```bat
explorer.exe
```

- 実質的にAdminSessionのFileExplorerを開けないそうだ。
- とても危ないから、基本的にWindowの操作でそれを塞いでいるらしい。
- 代わりにUserSessionのFileExplorerを引っ張ってきて利用している。

```
Admin CMD
 └─ explorer.exe
     └─ connects to existing USER shell process
         └─ runs in USER logon session
```

- 結局UserTokenを使っているUserSessionを持ってる、普通のCmdでsubstを実行するすることになる。
- そうしないとFileExplorerにはDドライブが分割されてない。
- プロセスは下記の通りだ。

```
subst
  ↓
Creates DOS device (in memory)
  ↓
Windows name resolver can translate D:
  ↓
Explorer asks Windows “what drives exist?”
  ↓
Windows says “D exists”
  ↓
Explorer shows D:
```

## <span style="color:#802548">_OSにおけるFileSystem：どの流れでソースコードが解釈するのか_</span>

- ふと、こういうことを考えてしまった。ソースコードでのAPIが直接FileSystemに接近するのであれば、なぜできないのか？
- 結論から言うと、ソースコード、つまり各言語のAPIはFileSystemに直接接近することができない。
- 全てのApiはwindows32APIを呼び出すだけ。

```text
new File("D:\\").exists()

ソースコード
 → 言語API (Java / .NET / C runtime)
   → Win32　API
     → NT kernel (Object Manager)
       → File system driver (NTFS)
       　ー＞BIOS
```

- Win32はDNSみたいにDOSサービスを行っている。
- DOSサービスはSessionとTokenごとなので、SessionとTokenごとにファイルの呼び方が違くなる場合もある。

```
\Session\0\DosDevices\   (services)
\Session\1\DosDevices\   (User A)
\Session\2\DosDevices\   (User B)
```

- もし下記のようにUserたちがMappingしたと仮定してみよう。

```
| User        | `D:` maps to      |
| ----------- | ----------------- |
| User A      | `C:\Users\A\Work` |
| User B      | `C:\Data\Shared`  |
| Admin token | not mapped        |
```

- AのUserは下記のように解釈されてNFTSに到達。

```
D:\logs\app.log
↓
\DosDevices\D:
↓
\Device\HarddiskVolume3\Users\A\Work
↓
\Device\HarddiskVolume3\Users\A\Work\logs\app.log (NTFS objects live in the kernel object namespace: これは誰でも同じ)
```

- BのUserは下記のように解釈されてNFTSに到達。

```
D:\logs\app.log
↓
\DosDevices\D:
↓
\Device\HarddiskVolume3\Data\Shared
↓
\Device\HarddiskVolume3\Data\Shared\logs\app.log (NTFS objects live in the kernel object namespace: これは誰でも同じ)
```

- 基本的にNTFSやBIOSなどの段階では C、Dなどの名前はわからない。
- そういった名前はWindow Kernelで付けてUserが便利に使用させるためだけ。





## <span style="color:#802548">_ファイルの文字コード解釈：数字はなぜ文字コードの区分が付かないのか_</span>
- 数字がある場合は文字のエンコーディングが判別がつかない。
- 全角じゃなくて半角の場合だけど、数字はどのエンコーディングでも同じ値を持っているからだ。

```shell
nkf -g 

#doesn't works
```

- しかし、文字の場合は文字のエンコーディングごとに完全に違う値を持っている。
- 例えば、'あ' を解釈すると下のようになる。

```
| Encoding  | Bytes (hex) |
| --------- | ----------- |
| UTF-8     | `E3 81 82`  |
| Shift-JIS | `82 A0`     |
| EUC-JP    | `A4 A2`     |
| UTF-16LE  | `42 30`     |
```

- どのエンコーディングでもできる場合もある。
- そういった場合は、heuristicsで判断する。
  - 数字と文字の場合を分けて考えることで気づいた人もいると思うが、文字のエンコーディングの判断は何物か判断することではない。
  - むしろできないことを取り除いて、その結果残っているエンコーディングを特定する過程に近い。


## <span style="color:#802548">_ファイルのUTF８保存：UTF８とUTF８(NO BOM)_</span>
- データをインポートするとき、UTF-8とUTF-8（NON-BOM）があって、なぜこれは違うのかと思って調べてみた。
- BOMという仕組みは基本的にUTF-16のために発明されたことであり、UTF-8では必要ではないらしい。
  - ファイルの最初の部分にこう書いてあることが Byte Order Mark（BOM）だが、ほとんどこれが書いてあるファイルがない。

```
EF BB BF
```

- ただ、もし書いてある場合、それをそのまま解釈したら、データが壊れるから、UTF-8（NON-BOM）を推薦してるとのことだ。
- 実際にDB以外にもほとんどのファイルに NON-BOMが推薦される。
- 不要なものを事前に防ぐことが重要だから、Non－BOMを選ぶ。

```
databases
JSON
XML
CSV
source code
config files
```


## <span style="color:#802548">_DBString：UTF-8　vs  UTF-8MB4_</span>
- 上のファイルの文字コードはUTF-8はUTF-8(no BOM)との比較だった。
- 現代の絵文字とかを表現するためには、UTF-8MB4に文字コードをする必要がある。
- 下記の表をみると、英語は１Byte、日本語は 3Byte、絵文字は４Byteということがわかる。
- Utf8では文字ごとに３バイトが限界で絵文字は入力できない。


```text
| Character | Code point | UTF-8 bytes |
| --------- | ---------- | ----------- |
| A         | U+0041     | 41          |
| あ         | U+3042     | E3 81 82    |
| 😀        | U+1F600    | F0 9F 98 80 |
```

- Oracleも同じ状況。
- 絵文字などもできるようにするためにはAL32UTF8が必要。

```
| Oracle charset | Status                         |
| -------------- | ------------------------------ |
| `AL32UTF8`     | ✅ **Real UTF-8 (use this)**    |
| `UTF8`         | ❌ **Legacy / broken (CESU-8)** |
```



## <span style="color:#802548">_DBString：Collation vs 文字コード_</span>
- Charset (文字コード)と Collation (コーレーション)は違う。
  - Charsetは与えられた文字をいくらのByteを付与するかの問題。
    - 例えば、utf8mb4、latin1、sjisは漢字についてエンコーディングバイトが違う。
  - しかし、Collactionは　文字のOrderについて関与する。



- oracleはCollationをUserに表示しない。権限もない。
- mysqlは表示する。だからCollcationの問題はMysqlで発生する。
- MysqlでのCollationの例は下記のことになる。

```
| Suffix | Meaning               |
| ------ | --------------------- |
| `_ci`  | Case-insensitive      |
| `_cs`  | Case-sensitive        |
| `_bin` | Binary (byte-by-byte) |
```


- Collationは下記の3つに影響を及ぼす。

```
Pagination
Unique indexes
GROUP BY results
```

- Collationごとに文字比較の結果が異なる分、Bugにつながる可能性もあり。
- 言語としてはunicode_ciが推薦される。
- 大文字小文字の区分がつかなければならないときはByteで区分をつける_binが必要。

```
VARCHAR(20) COLLATE utf8mb4_general_ci
INSERT 'abc'
INSERT 'ABC' → ❌ DUPLICATE KEY
utf8mb4_bin
Now both allowed.
```

- 要すると、下記の表がおすすめ。

```
| Use case             | Recommendation       |
| -------------------- | -------------------- |
| User names, emails   | `utf8mb4_unicode_ci` |
| Login ID / token     | `utf8mb4_bin`        |
| Case-sensitive codes | `*_bin`              |
| Japanese text        | `utf8mb4_unicode_ci` |
| Exact byte match     | `BINARY`             |
```

- ただ、多言語対応が必要の場合は、utf8mb4_0900_ai_ciがおすすめ。
- 多様な状況に対応ができるからだ。


```
| Feature            | Supported |
| ------------------ | --------- |
| Unicode standard   | ✅ UCA 9.0 |
| European languages | ✅         |
| Accents ignored    | ✅ (`ai`)  |
| Case ignored       | ✅ (`ci`)  |
| Emoji              | ✅         |
| Correct sorting    | ✅         |
```


- 文字コードを場合は、照合順序と違って、Oracleでも見える
- ただ、見れる範囲がDBごとに違う

```
| Feature                          | Oracle           | MySQL          |
| -------------------------------- | ---------------- | -------------- |
| DB-level charset                 | ✔                | ✔              |
| Table-level charset              | ❌                | ✔              |
| Column-level charset             | ❌ (except NCHAR) | ✔              |
| Recommended modern design        | AL32UTF8 DB      | utf8mb4 DB     |
```

- Oracleでは下記のSQLで確認できる。

```sql
SELECT parameter, value
FROM nls_database_parameters
WHERE parameter = 'NLS_CHARACTERSET';
```

- Mysqlでは下記のSQLで確認できる。


```sql
--サーバー
SHOW VARIABLES LIKE 'character_set%';
SHOW VARIABLES LIKE 'collation%';

-- schema
SELECT
  schema_name,
  default_character_set_name,
  default_collation_name
FROM information_schema.SCHEMATA
WHERE schema_name = 'your_db_name';

-- table
SELECT
  table_name,
  table_collation
FROM information_schema.TABLES
WHERE table_schema = 'your_db_name';

-- column
SELECT
  column_name,
  character_set_name,
  collation_name,
  data_type
FROM information_schema.COLUMNS
WHERE table_schema = 'your_db_name'
  AND table_name = 'your_table_name';
```

- MYSQLはColumnが一番優先順位を持ってる。次はTABLE、DB、Server。
- 一度に全部見られる方法はこちら。

```sql
SELECT
  c.table_name,
  c.column_name,
  c.character_set_name,
  c.collation_name
FROM information_schema.COLUMNS c
WHERE c.table_schema = 'your_db_name'
ORDER BY c.table_name, c.ordinal_position;
```


## <span style="color:#802548">_DBString：マイグレーションする場合の注意_</span>
- DBでの文字コードを S-jis からUTFー８に移行するためには、以下のような注意が必要だと思った
    - 取るバイトが違くなるから、バイト基盤のバリデーションは全部直す
    - UTF8でなく、utf8mb4に変換する



## <span style="color:#802548">_C:データ型とポインター_</span> 

- 数字は以下のよう。

```C
int         
short      
long       
long long  
float
double
```

- 文字に関しての変数は以下のよう。

```C
char a = 'a';
char *r = "abc";        // const. 修正ができない
char r[] = "abc";       // writable。修正できる
//char r[4] = "abc"; --> Compilerが　char r[] = "abc";を　char r[4] = "abc";のように解釈してくれる
```

- 文字列はポインタと一緒に使われることが多いので、ここでポインタについて簡単に押さえておこう。
- 以下の関数があったとしたら、どう渡すべきか？
- もうポインターが設定されてる変数なので、ポインターを表す「＆」は必要ではない。

```C
static int paramCheck (
    char *GetParam
) {

}

char *r = "abc"; 
paramCheck(r);
```

- もし関数がポインターを要求することになると、変数の値ではなくて住所を渡すべきなので、＆をつけて住所値を渡す。

```C
static int paramCheck (
    int *D1Code
) {

}

int a = 1;
paramCheck(&a);
```

- しかし、配列の場合、*がついてなくてもポインターそのもの。
- そのため、＆がついてなくてもそのまま渡す。
- 配列に値を与えるために以下の通りする

```C
int Main () {
    char logName[50];

    memset(logName, 0x00, sizeof( logName )); 
}

char logName[50];
strcpy(logName, "system.log");
```



- もし、Stringを保管したいと思うなら、以下の配列を利用する。

```C
char *logName[50];
logName[0] = "system.log";
logName[1] = "error.log";
printf("%s %s\n", logName[0], logName[1]); // system.log error.log. stringを保存する。
```

- データ型をstructに広げていこう。
- 例えば、このようなstructがあったとしたら、どう渡すべきか？
- 前提を簡単にするために、Fieldは全部ポインターなしでいこう。
- 渡す変数に＊がついているため、prmには＆が必要ではない。


```C
typedof struct {
    char dbFlg;
    char d1[d1_len + 1];
    char FLG[1 + 1];
    int returnCode;
} D1_ARG;

static int paramCheck (
    D1_ARG *prm
    char *GetParam
) {

}

D1＿ARG *prm;
paramCheck(prm, getParam);
```

- ポインターはタイプの後と変数の前についてくるが、変数の前のほうが一般的

```java
D1_ARG* prm; // not common
D1_ARG *prm; // common
```


## <span style="color:#802548">_C:*はコンテクストによって意味が違う_</span> 

| Syntax         | Meaning                            |
| -------------- | ---------------------------------- |
| `int *readCnt` | type declaration (pointer to int)  |
| `*readCnt`     | dereference (get value at address) |

```C
int x = 10;
int *p = &x; // これはアドレス

printf("%d", *p); //これは 値


int *p //タイプ宣言＝＞アドレス
*p     //間接参照
```

## <span style="color:#802548">_C： コンテクストによる * 意味の違い_</span> 

- 以下の風に数字の値を変換して渡す。
- JAVAで例えると、intじゃなくて、intを Wrappingした classを使うような感じ。

```C
main() {
    int readCnt = 0;
    int returnCode = 0;

    returnCode = dealWith333(&readCnt);
    

}

int dealWith333(int *readCnt) {
    int returnCode;
    returnCode = openFile(loopCnt, readCnt)

    return (returnCode);
}

int openFile(int loopcCnt, int *readCnt) {
    for (loopCnt= *readCnt; loopCnt < 10; loopCnt++) {
        .
        .
        .
        (*readCnt)++;
    }
}
```

- *readCntを受け取るため、readCntは住所値を持ってる。
- 逆に *readCntを 渡すと、 *がついてるとdereferencedになってしまう。
- だから、その値を渡すことになってしまい、エラーに落ちる。

```C
int dealWith333(int *readCnt) {
    int returnCode;
    returnCode = openFile(loopCnt, *readCnt /* これはミス */)

    return (returnCode);
}
```

- ポインター変数にとって丸かっこを間違えて囲むと、ポインターを移動させることになる。
- 以下は値を++することではなく、ポインターが++になってしまい、エラーになる。

```C
int openFile(int loopcCnt, int *readCnt) {
    for (loopCnt= *readCnt; loopCnt < 10; loopCnt++) {
        .
        .
        .
        *readCnt++　/* これはミス */;
    }
}
```

- C言語では、++がもっと優先順位がた * より高いから以下の通りになってしまう
- それを防ぐために、○を使って優先すべきの間接参照の値の両端にくっつける
```C
*readCnt++　ーーーー＞  *(readCnt++)　// エラー
(*readCnt)++;                       // OK
```

- 以下も不可能。
- *readCnt を 0にすることは、値を０にすることではない。
    - ポインターを０にする行為、つまり、NULLになってしまう。

```C
main() {
    int *readCnt = 0;
    int returnCode = 0;

    returnCode = dealWith333(readCnt);
    

    returnCode = finishBatchJob(readCnt);
}
```

## <span style="color:#802548">_C： 文字列における**の活用_</span> 

- 数字じゃなく、Stringも押さえていこう。
- 基本的にCの関数のParameterはcopyを渡す。
- pに他の値を代入してもpは原本ではなく、Copyなので原本は変わらない。

```C
void change(char *p) {
    p = "world";   // only changes local copy
}

char *str = "hello";
change(str); // hello
```

- しかし、その原則を利用して値の変換ができる。
- もう一度 * で囲んだら原本のポインターを渡すことになる。
- なので、Stringに他の値を代入できる。

```C
void change(char **p) {
    *p = "world";   // change caller's ポインター
}

char *str = "hello";
change(&str);　// world
```


- 関数のパラメータにおいて、char ** は文字列のポインターだ
- szRecBuffは 間接参照された szFileBuffなので、文字列のポインターでなく、文字列だ
- その文字列を間接参照すると、文字になる
- あの文字の値を比較して、if文に当てはまると、次のポインターを移動する
    - szRecBuffは文字列なので、次のポインターは次の文字を指す


```C
static int chopData(
    char *szCol,
    char **szFileBuff
) {
    char *szRecBuff;
    char szColmBuff[13 + 1];

    szRecBuff = *szFileBuff; // string
    if( (*szRecBuff == '"') ) {
        isQuoted = false;
        szRecBuff++; // next char
    }
    .
    .
    .
} 
```

- pszCheckRecは文字列なので、そこで＆をつけたら、文字列のポインターになる
- つまり、char **pszCheckRecみたいなパラメータに持たせることができる

```C
char *pszCheckRec;
char szFileBuff[512];
pszCheckRec = szFileBuff;
nRet = chopData( szCol, &pszCheckRec);
```

- pszBufがchar**であるため、Stringのポインターだ
- それを間接参照した*pszBufはStringになる
- それを二重間接参照した**pszBufはCharになる

```C
int inspectCSV(
    char *pszCol,
    char **pszBuf,
    short nMode
) {
    char *pszBufTmp;
    int i;
    pszBufTmp = *pszBuf;
    .
    .

    if(**pszBuf == ',' )  {
        (*psBuf)++;
    } else {
        return (SU_CSV_END);
    }

    return (0);
}
```


## <span style="color:#802548">_C： 構造体における**の活用_</span> 

- 最後にStructも押さえていこう。
- Structでも**を使える。特にSetterのように使える。

```C
typedef struct {
    int age;
    char code;
    char *name;
} Person;

void setAge(Person **p) {
    (*p)->age = 30;
}
void setCode(Person **p) {
    (*p)->code = 'A';
}
void setName(Person **p) {
    (*p)->name = "brick";
}
```

- もちろん、以下のVersionがもっと性能はいい。
- 不要なポインターの移動がなくなるからだ

```C
void setAge(Person *p) {
    p->age = 30;
}

void setCode(Person *p) {
    p->code = 'A';
}

void setName(Person *p) {
    p->name = "brick";
}
```


- 以下は意図した目的の構造体の値の変更ができない
- ポインターでない変数は原本じゃんくて、コピーだけを渡されて使うので、以下の関数が終わる時点で変化も失われる

```C
void setCode(Person p) { 
    p->code = 'A'; 
} 

void setName(Person p) { 
    p->name = "brick"; 
}
```

## <span style="color:#802548">_C：なぜ配列の初期容量の宣言にはなN ＋ １が多いのか_</span> 

- CのChar配列は必ず \0 で終わる。
- そのため、3個の文字が必要だとはいえ、4個が必要になる。
- それを明確にするためには、N＋１の形で記載する。

```C
[ 'A' ][ 'B' ][ 'C' ][ '\0' ]
char szBaceDateId[3 + 1];
```

- 次はより明確になる方法

```C
#define BACE_DATE_ID_LEN 3
char szBaceDateId[BACE_DATE_ID_LEN + 1];
```

- 配列の場合、最後に \0 を与えるのは手動でしなければならない。
- 以以下はどれも同じ目的に見える

```C
char szLogName[50] = '\0';
char szLogName[50] = {0};
char szLogName[50] = "";
```

- それともsnprintf Methodを使う。
- snprintfはいつも末に \0があるように保障する。
- このMethodが文字列を扱う際は、常に %s を通してコピーするようにする。

```C
snprintf(szLogName, sizeof(szLogName), "%s", source);
```
- NullをGuardするLogicは以下のよう。

```C
if (source)
    snprintf(szLogName, sizeof(szLogName), "%s", source);
else
    szLogName[0] = '\0';

//簡潔に
snprintf(szLogName, sizeof(szLogName), "%s", source ? source : "");
```


## <span style="color:#802548">_C：Charダブル配列の読み方_</span> 

- ダブル配列の場合、最初のカッコに入っている数字が要素数。
- 後ろはStringの文字数。5だったら、４が文字数。

```C
char array[3][4 + 1] = {
    "0123",
    "1234",
    "2345"
};
```


## <span style="color:#802548">_C：動的にStringに値を与える_</span> 

- 1番では初期化する時のStringの代入仕方を説明した。

```C
char *a = "abc"; //  readable
char a[] = "abc"; // writable
```

- 1番目　

```C
strncpy(dest, src, sizeof(dest));
dest[sizeof(dest) - 1] = '\0'; 
```

- 2番目 

```C
char buf[50];
sprintf(buf, "parameter error");
```

- 3番目 
- 一番おすすめ

```C
char buf[50];
snprintf(buf, sizeof(buf), "parameter error");
```

- 4番目　
- 組み込みOSでのやり方

```C
char buf[10];
int i = 0;

while (src[i] != '\0' && i < 9) {
    buf[i] = src[i];
    i++;
}
buf[i] = '\0';
```

- Stringの値をただ接近して修正するのはできない

```C
typedef struct {
    char name[32];
} USER;

USER u;
u.name = src;   // ❌ 配列には直接接近できない
```

- できるが、危険なやり方。
- ただ、レガシーのシステムはこのMethodだらけ。

```C
char dest[5];
strcpy(dest, "Hello"); // needs 6 bytes including '\0' → overflow
```

- 5番目。これは置換じゃなくて追加

```C
int remaining = sizeof(moji) - strlen(moji) - 1;
strncat(moji, added, remaining);
```

- クリアする時には、3つの中で何でも呼び出す。
- String自体をクリアするためには、memsetが必要。

```C
memset(buf, 0, sizeof(buf));
memset(buf, '\0', sizeof(buf));
memset(buf, 0x00, sizeof(buf));
```

- 以下は方法はポインターを無効にするためのやつなので、コンテンツとは関係がない。

```C
char *p = NULL; 
```

- ただStringを空白にするためには、以下で十分。

```C
char str[100];

str[0] = '\0';
strcpy(str, "");
snprintf(str, sizeof(str), "");
```

- Bufferとかには下記のようにsscanfも使える

```C
int sscanf(const char *str, const char *format, ...);
 nRet = sscanf(szBuff,
            "%s %s %d "
            "%s %c %d %c %s %c %d %c %s %c %d %c %s %c %d %c %s %c %d %c",
            &param.jpName,
            szBuff2,
            &param.nParamNum,
            &param.whereClause[0].date,
            &param.whereClause[0].cFlag,
            &param.whereClause[0].pastDate,
            &param.whereClause[0].host,
            &param.whereClause[0].date,
            &param.whereClause[0].cFlag,
            &param.whereClause[0].pastDate,
            &param.whereClause[0].host,
            &param.whereClause[0].date,
            &param.whereClause[0].cFlag,
            &param.whereClause[0].pastDate,
            &param.whereClause[0].host,
            &param.whereClause[0].date,
            &param.whereClause[0].cFlag,
            &param.whereClause[0].pastDate,
            &param.whereClause[0].host,
            &param.whereClause[0].date,
            &param.whereClause[0].cFlag,
            &param.whereClause[0].pastDate,
            &param.whereClause[0].host,
        );
```



## <span style="color:#802548">_C：tringを比較する_</span> 

- Stringを比較するときには、＝＝ではできない。
- strcmpで比較して０になると、それが同じコンテンツだと示唆する。

```C
if (s1 == s2)  // wrong (compares addresses)
if (strcmp(s1, s2) == 0)
```

- strcmpは case-sensitiveなので、CaseInsensitiveのためには下記のMethodを利用する。
- ただ、Ｃ標準ではない。

```C
POSIX: strcasecmp
```

- よく使用される比較はStringが実質的に空白なのか判断すること。

```C
param.gender[0] == '0' || 
param.gender[0] == ' ' ||
param.gender[0] == '\0' 
```

- 複数のスペースがある時は、下記のように確認する。

```C
char *p = param.gender;

if (*p != '\0') {
   bool blank = true;
    for (; *p; p++) { // for (; *p != '\0'; p++)と同じ。\0になるときは、読み込まる文字がなくなったとき
        if (*p != ' ') {
            blank = false;
            break;
        }
    }
} 
```



## <span style="color:#802548">_C：関数のParameter_</span> 

- 関数を定義だけするときは、変数名がなくてもいい
```C
static int ParamCheck021(char *, char *,char *,char *,char *,char *,char *);
```

- ただ、実装するときには絶対になければならない。

```C
static int ParamCheck021 (
    char *GetParam,
    char *type,
    char *memberNo,
    char *withDrawlDate,
    char *memberCode,
    char *staffId,
    char *staffName
) {
   
}
```

- Stringを渡すときには、必ずポインターが必須。
- どちらにせよ、同じ意味であり、完全に互換性がある。
- 関数のParameterは住所値だけ見ているためだ。
- 配列になり、ポインター変数になり、どちらでも互換性があるので渡したら読み込める。

```C
static int ParamCheck021Astrick ( char *GetParam, char *type, char *memberNo, char *withDrawlDate, char *memberCode, char *staffId, char *staffName )
static int ParamCheck021Bracket ( char GetParam[], char type[], char memberNo[], char withDrawlDate[], char memberCode[], char staffId[], char staffName[] )

char abc[] = "abc";
ParamCheck021Astrick(abc) //できる

char *abc = "abc";
ParamCheck021Bracket(abc) //できる
```

- ただ、不要な誤解を晴らすために、以下の仕組みがおすすめ。
- ArrayをParameterTypeとして規定するのはよくない。
- Constは修正を欲しくないとき、Constが付いてないのは修正も許容する時。

```C
int ParamCheck021(
    const char *GetParam,
    const char *type,
    ...
);


int ParamCheck021(
    char *GetParam,
    char *type,
    ...
);
```
