# 2025-04-11-teenager-singup-nh-japan

## <span style="color:#802548">_ジックナンバー脱皮_</span>

- if文で値を受け取って条件に合わせて分岐処理する場合、下記のような形になります。

```jsx
if (payload.responseCode =='0') {
    //logic
    openFailLayer();
} else if(payload.responseCode == '1') {
    //logic
    location.href = '/success';
} else {
    //logic
    openFailLayer();
}
```

- マジックナンバーはメンテナンス性を損なうので、明示的にします。

```jsx
const IDENTITY_NOT_AUTHENTICATED = '0';
const IDENTITY_AUTHENTICATED = '1';
const IDENTITY_NOT_OWNED = '2';
if (payload.responseCode === IDENTITY_NOT_AUTHENTICATED) {
    //logic
    openFailLayer();
} else if (payload.responseCode === IDENTITY_AUTHENTICATED) {
    //logic
    location.href = '/succeess';
} else if (payload.responseCode === IDENTITY_NOT_OWNED) {
    //logic
    openFailLayer();
}
```

- if文も良いですが、以下のようにオブジェクトに含めるとメンテナンス性が向上します。

```jsx
const IDENTITY_NOT_AUTHENTICATED = '0';
const IDENTITY_AUTHENTICATED = '1';
const IDENTITY_NOT_OWNED = '2';
const authenticationMap = {
    IDENTITY_NOT_AUTHENTICATED: function() {
        //logic
        return false;
    },
    IDENTITY_AUTHENTICATED: function() {
        //logic
        return true;
    },
    IDENTITY_NOT_OWNED: function() {
        //logic
        return false;
    }
}

function executePostAuthentication(resultCode) {
    authenticationMap[resultCode]();
}

function authAjax() {
    $.ajax({

    }),
    success: function(res,status,xhr) {
        var payload = JSON.parse(res.payload);

        if (executePostAuthentication(payload.resultCode) == false) {
            openFailLayer();
        }
    }
}
```

## <span style="color:#802548">_重複クリック防止の悩み_</span>

- 過去に私が作った多重クリック防止はグローバル変数、setTimeoutを利用しました。

```jsx
var reqFlag = false;

function checkReq(){
    if(reqFlag == false){
            reqFlag = true;
            return false;
    }

    setTimeout(function(){
            reqFlag = false;
    },3000)

    return true;
}

function handleButtonClick() {
    $('#button').on('click',function() {
        if(reqCheck()) {
            return;
        }
        .
        .
        .
        //ajax 関数
    })
}
```

- しかし、上記の場合、もしapiが3秒を超える場合にクリックが可能です。
- - サーバーからレスポンスを受け取った後、クリック可能な状態に変更するには、新しいUIまたはプログラミング技術を導入する必要がありました。
- ローディングウィンドウのような新しいUIは私が決めることができない領域で、スケジュールに負担が大きかったので、プログラムで実装可能なjs closureを使おうとしました。

## <span style="color:#802548">_closure実行コンテキスト_</span>

- しかし、思うように動かなかったです

```jsx
function sendRequest(fn) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            fn(data)
        }
    }
}

function handleButtonClick() {
    $("#button").on('click',function() {
        .
        .
        .
        var sendData = {};
        sendData.payload = id;

        sendRequest(testAjax)(sendData);
    })
}
```

- 問題はclickする度にsendRequest()()を呼び出すと毎回新しい実行コンテキストが発生するのが問題でした。
- そこで、実行コンテキストを保存しておく変数を作りました。
- そうしたら、これで思い通りに動くようになりました。

```jsx
function sendRequest() {
    var clicked = false;
    return function() {
        if(!clicked) {
            clicked = true;
            $.ajax({
                .
                .
                .
                success: function(res, status, xhr){
                    if(res.statusCode === '200') {
                        location.href = '/success';
                    }
                    clicked = false;
                },
                error: function(error) {
                    alert(error);
                     clicked = false;
                } ,
                complete: function() {
                     clicked = false;
                }
            })

        }
    }
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function() {
       fetchData();
    })
}
```

## <span style="color:#802548">_ajax関数は外側に引いて読みやすいように_</span>

- ajax関数を外に出してリクエスト関数が長くなりすぎないようにしたかったです。
- すると、clicked変数がclosure関数が属するスコープから外れてしまいました。
- 外れたらもう変数の意味が消えるのでパラメーターで変数を渡してみましたが、何の役にも立ちませんでした。

```jsx
function sendRequest() {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            textAjax(data, clicked);

        }
    }
}

function testAjax(data, clicked) {
    $.ajax({
        .
        .
        data:data
        success: function(res, status, xhr){
            if(res.statusCode === '200') {
                location.href = '/success';
            }
            clicked = false;
        },
        error: function(error) {
            alert(error);
            clicked = false;
        } ,
        complete: function() {
            clicked = false;
        }
    })
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function() {
       fetchData(sendData);
    })
}
```

- 結局、過去のjquery ajax活用法では不可能だと判断した。
- Promise 関数のthen()のように続ける方式のJuqery ajaxを使うことにしました。

```jsx
function sendRequest() {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            textAjax(data)
                .done(function(res,status,xhr){
                    if(res.statusCode === '200') {
                        location.href = '/success';
                    }
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    clicked = false;
                })

        }
    }
}

function testAjax(data) {
    return $.ajax({
                .
                .
                data:data
            })
}

var fetchData = sendRequest();

function handleButtonClick() {
    $("#button").on('click',function(
       fetchData(sendData);
    })
}
```

- 今は色んなajax関数を受け取れるように関数をパラメーターで受け取ろうとしました。
- 最初に実行コンテキストを作る時、必要なajax関数を一緒に渡すように変更する必要があります。

```jsx
function sendRequest(ajaxFn) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            ajaxFn(data)
                .done(function(res,status,xhr){
                    if(res.statusCode === '200') {
                        location.href = '/success';
                    }
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    checkMobileDeviceOs();
                    clicked = false;
                })

        }
    }
}

function testAjax(data) {
    return $.ajax({
                .
                .
                data:data
            })
}

var fetchData = sendRequest(testAjax);

function handleButtonClick() {
    $("#button").on('click',function()
       fetchData(sendData);
    })
}
```

## <span style="color:#802548">_clousreを共通化すること_</span>

- コールバック関数も一緒にajaxと一緒に入れておけば、これを共通関数として活用できると思いました。
- - それでsuccessCallbackも一緒にparameterで受け取るように変更しました。

```jsx
function sendRequest(ajaxFn, successCallback) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            
            ajaxFn(data)
                .done(function(res,status,xhr){
                    successCallback(res)
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    clicked = false;
                })

        }
    }
}

function testAjax(data) {
    return $.ajax({
                .
                .
                data:data
            })
}

function successCallback(res) {
    var popupText = "";
    var popup = "";
    if (res.statusCode === '200') {
        location.href = '/success';
    } else if (res.statusCode === '199') {
        popupText = "error1"
    } else if (res.statusCode === '198') {
        popupText = "error2"
    } else {
        location.href = '/failure'
        return;
    }

    popup = $("#errorGuide").text(popupText);
    popup.html(popup.html().replace(/\n/g,'<br>'));

    Layer.open("#errorPopup");
}

var fetchData = sendRequest(testAjax, successCallback);

function handleButtonClick() {
    $("#button").on('click',function() {
       fetchData(sendData);
    })
}
```

- モジュール化しているので、共通関数として活用できます
- 共通関数として活用することを提案したが、残念ながら理解しにくいという理由で拒否された。

```jsx
//common.js

function sendRequestOnlyOnce(ajaxFn, successCallback) {
    var clicked = false;
    return function(data) {
        if(!clicked) {
            clicked = true;
            ajaxFn(data)
                .done(function(res,status,xhr){
                    successCallback(res)
                    clicked = false;
                }).fail(function(error){
                    alert(error);
                    clicked = false;
                }).always(function(){
                    clicked = false;
                })

        }
    }
}
```

- これで、ajaxは全てcommon.jsであるsendRequestOnlyOnce関数を使えます
- 以下のような順序で行うと、多重クリック防止が自動的に入ることになります。

```jsx
//ajax
function testAjax(data) {

    return $.ajax({
                url:'~~',
                method:'post',
                dataType:'json',
                data:data
            })
}

//成功コールバック
function successCallback(res) {
    .
    .
}

// 実行コンテキスト形成
var fetchData = sendRequestOnlyOnce(testAjax, successCallback);

function handleButtonClick() {
    $("#button").on('click',function() {
       fetchData(sendData);
    })

```
