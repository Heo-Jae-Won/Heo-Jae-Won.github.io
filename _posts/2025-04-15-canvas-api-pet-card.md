---
title: "pet-card maker program"
published: true
---


## <span style="color:#802548">_CSRからSSR環境への移行_</span>

- importの文法は基本的にnode_modulesからインポートする文法です。
    - node_modulesフォルダがあれば、つまり、一度だけnpm installが行われていれば、外部ネットワーク・内部ネットワークを問いません。
    - ただ、node.jsがインストールされていないと最初にinstallをすることができないのですが、閉じたネットワーク環境ではnode.jsをインストールすることができませんでした。
- そのため、script srcに切り替える必要があり、そのためには外部からscript jsをダウンロードして持ち込む必要があります。

```
https://www.jsdelivr.com/package/npm/fabric ---cdn download
https://cdnjs.com/ ---cdn
```

- 元のCSR環境では下記のように各機能が全部分離されている状態でした。
- callする順番は気をつけなければなりません。親関数が呼び出された後、子関数が活性化される子関数が存在します。
    - 従って、canvasが最初で、次にtextを活性化させ、次にfontを活性化させる順番です。

```jsx
import initCanvas from "./function/initCanvas";
import initText from "./function/initText.js";
import initFont from "./function/initFont";
.
.
.
const init = () => {
    initCanvas.call(this);
    initText.call(this);
    initFont.call(this);
    .
    .

};

export default init;
```

- 以下はSSRに切り替えたものです。CSRと同じように最初に実行されるべきものから呼び出します。
- 各機能別に分離されている関数は全部一箇所にまとめた形ですが、これは顧客の要求でした。
- OOPのように各責任ごとにファイルを分離せず、一つのファイルにまとめてほしいと要請されました。
    - OOPの原則に反することですが、要求通りに実装することが優先だったので、そのように実装しました。
    - 各ファイルごとに100行程度を一つのファイルにまとめると、3000行を超えてしまい、開発/メンテナンスには不向きなファイルになってしまいました。

```jsx
//init.js
const init = function() {

    const initCanvas = function() {
        .
        .
    }

    
  
    const initFont = function() {
        .
        .
    }

    const initText = function () {
        const init = () => {
            initFont({
                fontA: getEl('#fontA'),
              
            }, this.canvas);

            //eventListener business logic
        }

        init();
    }

    initCanvas.call(this);
    initText.call(this);
    initFont.call(this);
}

export default init;
```

- PhotoMaker を構成する機能を使うためには機能をまとめたinit関数を呼び出す必要があります。
- 以前のthis bindingされた関数は全てPhotoMaker インスタンスを示すことになります。
    - PhotoMaker で作られたメンバー変数であるcanvasなどを使うことができるということです。

```jsx
//PhotoMaker.js
import init from "init.js";
const PhotoMaker = () => {

    this.canvas = new fabric.Canvas(canvasId, {
        isDrawingMode: false,
        width: ,
        height: ,
    });

    init.call(this);

    return this;
}

export default PhotoMaker ;
```

- init.jsの内のinitText関数のthis.canvasもPhotoMakerインスタンスのメンバー変数です。

```jsx
const initText = function () {
    const init = () => {
        initFont({
            fontA: getEl('#fontA'),
          
        }, this.canvas);

        //eventListener business logic
    }

    init();
}
```

- これでphotoEditorをnewで生成して使うことができます。
- 各ページごとにjsを作るようになってるので、PhotoEditor機能が必要なページは全部その形でnewを呼び出します。

```jsx
//photoMaker.js
import PhotoMaker from "photoEditor.js";

$(function() {
    new PhotoMaker('canvas');

    //page business logic
    $("#button").click(function() {

    })
});
```

- HTMLも一つにまとめてほしいという要望があったので、HTMLも一つにまとめました。
- 一つのphotoMakerオブジェクトだけが存在するので便利ですが、HTMLが長くなり修正が難しくなるデメリットがあります。

```html
<!--photoMaker.html-->
<head>
    <script src="photoMakerPage.js"></script>
</head>

.
.
.
```

## <span style="color:#802548">_戻る機能_</span>

- 最初の戻るロジックはとてもシンプルでした。

```jsx
this.undoStack = [];

const saveState = (e) => {

  if(e.isForeGround == true && e.target) {
      this.undoStack.pop();
      const copiedCanvasAsJson = JSON.stringify(this.foregroundCanvas.toObject(["name"]));
      this.undoStack.push(copiedCanvasAsJson);
  }

}

this.foregroundCanvas.on("object:modified", saveState);
```

- しかし、テキスト関連機能がどんどん追加され、複雑になり始めました。
- 特に、後で追加されたtextPlaceHolderの要件が問題でした。
    - textに値がなくなったらtextPlaceholderを表示するようにしましたが、canvas apiにはhtmlのような自動placeholderがありませんでした。
    - そのため、手動で全部イベントを操作する必要があったのですが、それがバックグラウンドと絡まって問題を起こしました。
    - text関連インスタンスが6つずつ追加され問題が発生しました。
    - textの値の変化やフォントの変化自体は1つずつ認識されて戻ります。
        - しかし、textPlaceHolderの場合、6つのインスタンスが一度に追加されるのですが、この場合、1つだけ戻してはいけません。
        - 実際のユーザにとって、textHolderは戻しとして扱われてはいけないからだ。
        - つまり、そのケースを区別して、ある時は6つを、ある時は1つを戻す必要がありました。
- これを反映するためにtextFlag変数を導入しました。

```jsx
const initText = function () {
    const init = () => {
        initFont({
            fontA: getEl('#fontA'),
            fontB: getEl('#fontB'),
        }, this.canvas);

        const leftTextPlaceHolder = new fabric.Text(leftDefaultText, {textFlag: "initial"})
        const rightTextPlaceHolder = new fabric.Text(rightDefaultText, {textFlag: "initial"})
        const centerTextPlaceHolder = new fabric.Text(centerDefaultText, {textFlag: "initial"})
        .
        .

    }

    init();
}
```

- しかし、placeholderも最初の6つが一度に生成される場合と、入力した後、再度入力値を消す必要がある場合を区別する必要がありました。
- そのため、それぞれのケースを分けて別々のflag変数を使うことにしました。
- 戻る時は、そのflag変数を確認しながら進めます。

```jsx
this.canvas.on('text:editing:entered', (e) => {
    .
    .
    .

    if (e.target.name.include('text_left')) {

        if (!rightText.text && !rightPlaceholder) {
            this.canvas.add({...rightPlaceholder, textFlag: "center"});
        }
        .
        .
        .
    }

})
```

## <span style="color:#802548">_回転によるカード余白チェックができない問題_</span>

- カード画像に余白がないように画像を全て重ねたかどうかを確認してほしいという要求がありました。
- 該当事項の実装を簡単に考えていましたが、それほど簡単ではなかったです。
- 回転をすると、左、上の値が当然回転したオブジェクトに合わせて決まると思っていました。
- ところが、全くそうではなかったです。元のl左、上の座標が思うように動かなかったです。
- そのため、回転された画像の場合、余白をチェックするロジックも全て壊れてしまいました。
- 左、上の値を一々全部角度に合わせて変えることを試みました。

```jsx
const imageObj = this.canvas.getObjects().find((obj) => obj.name?.includes('image'))

let imgLeft = imageObj.left
let imgTop = imageObj.top
if(imageObj.angle === 0 || imageObj.angle === 360 || imageObj.angle === -360 ) {
    imageObj.left = imageObj.oCoords.tl.x;
    imageObj.top = imageObj.oCoords.tl.y;
} else if(imageObj.angle === -90) {
    imageObj.left = imageObj.oCoords.tr.x;
    imageObj.top = imageObj.oCoords.tr.y;
} else if(imageObj.angle === -180) {
    imageObj.left = imageObj.oCoords.br.x;
    imageObj.top = imageObj.oCoords.br.y;
} else if(imageObj.angle === -270) {
    imageObj.left = imageObj.oCoords.bl.x;
    imageObj.top = imageObj.oCoords.bl.y;
}
```

- しかし、上のようなコードはjsで90度ずつ回した時だけ有効です。
- 基本的にユーザーがマウスで回すイベントには意味がありませんでした。
- それで悩んでみましたが、どうしても答えが出なかったのでググってみました。
- ググってもどうしても見つからず、結局CHATGPTに聞いて解決しました。
    - アルゴリズムがわかったら、そのアルゴリズムの実装方法について質問します。
    - そのアルゴリズムはpoint-in-polygon algorithmであります。

```jsx
const petImage = this.canvas.getObjects().find((obj) => obj.name?.includes('image'))
const cardImage = this.canvas.getObjects().find((obj) => obj.name?.includes('background'))
const doesCardCoveredFullyByPetImage = isCardInsidePet(cardImage.getCoords(),  petImage.getCoords())

if(!doesCardCoveredFullyByPetImage)  {
    alert('空白はNGです')
    return; 
}

function isCardInsidePet(innerCoords, outerCoords) {
    return innerCoords.every(({ x, y }) => {
        let inside = false;
        for (let i = 0, j = outerCoords.length - 1; i < outerCoords.length; j = i++) {
            let xi = outerCoords[i].x, yi = outerCoords[i].y;
            let xj = outerCoords[j].x, yj = outerCoords[j].y;

            let intersect = ((yi > y) !== (yj > y)) &&
                (x < (xj - xi) * (y - yi) / (yj - yi) + xi);

            if (intersect) inside = !inside;
        }
        return inside;
    });
}
```

- every関数なので、すべての点がpolygon(ペット画像)の中に存在すれば、つまり、inside変数がtrueであればtrueを返します。
- for文でペット画像の全ての点に対して巡回を開始します。for文には2つの変数を使うことができます。
- intersectでカード画像との交差を検証します。
    - カード画像から線を描いてペット画像と偶数回交差した場合、カードに余白があることです。
    - 奇数回交差した場合、カードに余白がない。

```jsx
function isCardInsidePet(innerCoords, outerCoords) {
    return innerCoords.every(({ x, y }) => {
        let inside = false;
        for (let i = 0, j = outerCoords.length - 1; i < outerCoords.length; j = i++) {
            let xi = outerCoords[i].x, yi = outerCoords[i].y;
            let xj = outerCoords[j].x, yj = outerCoords[j].y;

            let intersect = ((yi > y) !== (yj > y)) &&
                (x < (xj - xi) * (y - yi) / (yj - yi) + xi);

            if (intersect) inside = !inside;
        }
        return inside;
    });
}
```