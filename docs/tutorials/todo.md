# ToDoリストを作りながら学習しよう！

このチュートリアルでは、書籍の CHAPTER 1～4 までに解説しているいろいろな機能を使って、次のような ToDo リストを作成します。

- ToDo の追加・削除
- 作業中・完了の作業状態の切りかえ
- 作業状態ごとの表示の絞り込み

チュートリアルの流れから、Vue.jsの基本機能が何を行ってくれるのか？を、知ることができるようになっています。
社内勉強会やハンズオン勉強会でも、活用してください。

## 完成形

- [★デモページ](http://chibinowa.net/demo/tutorial-todo/)

## ファイルを準備しよう

使用するファイルは「index.html」「main.js」「main.css」の3つのファイルと、CDNからスタンドアロン版のVue.jsファイルを読み込みます。

- [main.js](http://chibinowa.net/demo/tutorial-todo/main.js)
- [main.css](http://chibinowa.net/demo/tutorial-todo/main.js)

CSS の説明はしていないため、コピペして使用してください。

ページレイアウトは次のとおりです。

```html
<!DOCTYPE html>
<html>
<head>
  <meta charset="utf-8">
  <title>Vue.js App</title>
  <link rel="stylesheet" href="main.css">
</head>
<body>
  <div id="app">
    <!-- 絞り込みラジオボタン -->
    <!-- ToDo テーブル -->
    <!-- 新規登録フォーム -->
  </div>
  <script src="https://cdn.jsdelivr.net/npm/vue@2.5.16/dist/vue.js"></script>
  <script src="main.js"></script>
</body>
</html>
```

## STEP1 インスタンスの作成

まずは、アプリケーションを紐付ける要素 `#app` を作成します。

<code-caption>index.html</code-caption>
```html
<body>
  <div id="app">
    <!-- ここにテンプレートを書いていく -->
  </div>
</body>
```

コンストラクタ関数 `Vue` を使ってルートインスタンスを作成します。
アプリケーションで使用したいデータは `data` オプションへ登録していきます。

<code-caption>main.js</code-caption>
```js
const app = new Vue({
  el: '#app',
  data: {
    // 使用するデータ
  },
  methods: {
    // 使用するメソッド
  }
})
```

`data` オプションへ登録したデータはすべてリアクティブデータに変換されます。

## STEP2 ローカルストレージ API の使用

データはサーバーではなく「ローカルストレージ」へ保存することにします。
ストレージ周りの実装は Vue.js 公式サンプル「[TodoMVC の例](https://jp.vuejs.org/v2/examples/todomvc.html)」のコードを使用させていただきます。

```js
// https://jp.vuejs.org/v2/examples/todomvc.html
var STORAGE_KEY = 'todos-vuejs-demo'
var todoStorage = {
  fetch: function() {
    var todos = JSON.parse(
      localStorage.getItem(STORAGE_KEY) || '[]'
    )
    todos.forEach(function(todo, index) {
      todo.id = index
    })
    todoStorage.uid = todos.length
    return todos
  },
  save: function(todos) {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(todos))
  }
}
```

この公式のコードの内容については詳しく説明しませんが、これは `Storage API` を使ったデータの取得・保存の処理だけを抜き出したものです。
小さなライブラリだと思ってください。
なぜ、こういった処理の抜き出しをするかは、書籍 SHAPTER 7 のコラム（XXXページ）で説明しています。
特に手を加える必要はないので、このコードは `main.js` ファイルの一番上の方に加えておきましょう。

実際にストレージに保存されるデータのフォーマットは、次のような JSON です。

```json
[
  { "id": 1, "comment": "新しいToDo1", "state": 0 },
  { "id": 2, "comment": "新しいToDo2", "state": 0 }
]
```

## STEP3 データの構想

さあ、ここから実際に作るコードです！
アプリケーションに付けたい機能から考えると、必要になるデータは次のようなものでしょうか。

- ToDo のリストデータ
  - 要素の固有ID
  - コメント
  - 今の状態
- 作業中・完了・すべて などオプションラベルで使用する名称リスト
- 現在絞り込みしている作業状態

## STEP4 リスト用テーブル

次に、ToDo リストデータを表示するテーブルの枠組みを作成します。

```html
<div id="app">
  <table>
    <!-- テーブルヘッダー -->
    <thead>
      <tr>
        <th class="id">ID</th>
        <th class="comment">コメント</th>
        <th class="state">状態</th>
        <th class="button">-</th>
      </tr>
    </thead>
    <tbody>
      <!-- ① ここに <tr> で ToDo の要素を1行づつ繰り返し表示 -->
    </tbody>
  </table>
</div>
```

## STEP5 リストレンダリング

ToDo リストデータ用の空の配列を `data` オプションへ登録します。

これは、データが何もない時でも配列として認識されるようにするためと、もともと `data` オプション直下のデータは後から追加ができないので初期値で宣言しておく必要があるためです。

```js
var app = new Vue({
  el: '#app',
  data: {
    todos: []
  }
})
```

テーブルタグの ① の部分ので要素の数繰り返し表示させるには、対象となるタグ（ ここでは `<tr>` タグ ）に `v-for` ディレクティブを使用します。

```html
<!-- ここに <tr> で ToDo の要素を1行づつ繰り返し表示 -->
<tr v-for="item in todos" v-bind:key="item.id">
  <!-- 要素の情報 -->
</tr>
```

ディレクティブの値は JavaScript の式になっており次のように書きます。

```
v-for="各要素の一時的な名前 in 繰り返したい配列やオブジェクト"
```

`v-for` を記述したタグとその内側で `todos` データの各要素のプロパティが使用できるようになります。
`<tr>` タグの内側に「ID」「コメント」「状態変更ボタン」「削除ボタン」のカラムを追加していきましょう。

```html
<tbody>
  <!-- ここに <tr> で ToDo の要素を1行づつ繰り返し表示 -->
  <tr v-for="item in todos" v-bind:key="item.id">
    <th>{{ item.id }}</th>
    <td>{{ item.comment }}</td>
    <td class="state">
      <!-- 状態変更ボタンのモック -->
      <button>{{ item.state }}</button>
    </td>
    <td class="button">
      <!-- 削除ボタンのモック -->
      <button>削除</button>
    </td>
  </tr>
</tbody>
```

このボタンはまだなにも機能しないモックのため、機能はこれから実装していきます。

<page-info page="">リストレンダリング</page-info>

## STEP6 フォーム入力値の取得

新しい ToDo をリストへ追加するための入力フォームを作成します。

`ref` 属性を使って参照するための名前をタグに付けておくと、その DOM に直接アクセスできます。
注意点として、`$refs` は仮想 DOM ではないため、更新のためのアクセスには向きません。

```html
<input ref="comment">
```

これは、メソッド内から次のように参照できます。
実際は、次のセクションで使用します。

```js
this.$refs.comment.value
```

`v-model` ディレクティブを使えばデータとフォーム入力を同期することもできますが、今回は、入力したデータを画面に表示させないのと常にデータとして持っている必要がないので、この `$refs` を使って入力値を取得することにします。

```html
  <!-- ToDoリストテーブル -->
  </tbody>
</table>

<h2>新しい作業の追加</h2>
<form class="add-form" v-on:submit.prevent="doAdd">
  <!-- コメント入力フォーム -->
  コメント <input ref="comment">
  <!-- 追加ボタンのモック -->
  <button type="submit">追加</button>
</form>
```

テーブルの下あたりに追加しておきます。

```html
v-on:submit.prevent="doAdd"
```

この `v-on` ディレクティブによって、ボタンをクリックしたり入力フォームでエンターを押してフォームのサブミットが行われると、それをハンドリングして `doAdd` メソッドが呼び出されるようになります。

<page-info page="">$refs</page-info>
<page-info page="">v-model</page-info>


## STEP7 リストへの追加

つづいて `doAdd` メソッドを定義しましょう。

このメソッドは、フォームの入力値を取得して新しい ToDo の追加処理をおこないます。
ルートコンストラクタの `methods` オプションに、メソッドを登録します。

```js
new Vue({
  // 省略...
  methods: {
    // ToDo 追加の処理
    doAdd: function(event, value) {
      // ref で名前を付けておいた要素を参照
      var comment = this.$refs.comment
      // 入力がなければ何もしないで return
      if (!comment.value.length) {
        return
      }
      // { 新しいID, コメント, 作業状態 }
      // というオブジェクトを現在の todos リストへ push
      // 作業状態「state」はデフォルト「作業中=0」で作成
      this.todos.push({
        id: todoStorage.uid++,
        comment: comment.value,
        state: 0
      })
      // フォーム要素を空にする
      comment.value = ''
    }
  }
})
```

ちょっと長くなりましたが、`$refs` を使っている以外は通常の JavaScript の式です。
コメントと一緒に1行づつ読んでみてください。

通常の配列メソッド `push` を使うだけで、リストデータへ追加できます。

`ref` 属性で参照名を付けたタグは、メソッド内から次のように参照できます。
DOM API が使用可能なので `value` プロパティから値を取得します。

ちなみに、テンプレートでは「名前」だけでデータを使用できましたが、メソッド内でデータやメソッドを使用するときは `this` を付ける必要があります。

```
this.$refs.参照名
```

## STEP8 ストレージへの保存の自動化

さて、JavaScript 内ではデータは追加されましたが、これではまだローカルストレージに保存されていません。
ブラウザをリロードしたら消えてしまいます。
`doAdd` メソッドの最後に `todoStorage.save` メソッドを使って保存してもいいのですが、追加・削除・作業状態の変更すべて同じ処理をしなければいけません。

`todos` データの内容が変わると、自動的にストレージへ保存してくれたら素敵ですね。
これは `watch` オプションの「ウォッチャ」機能を使うことで可能です。
ウォッチャはデータの変化に反応して、あらかじめ登録しておいた処理を自動的に行います。

```js
watch: {
  監視するデータ: function(newVal, oldVal) {
    // 変化した時に行いたい処理      
  }
}
```

単純に楽になるだけでなく、同じコードを何度も書かないというのは抽象化と同じで保守性にも繋がります。
無理やり感のない程度に、抽象化や共通化をしていきましょう。

```js
new Vue({
  // 省略...
  watch: {
    // オプションを使う場合はオブジェクト形式にする
    todos: {
      // 引数はウォッチしているプロパティの変更後の値
      handler: function(todos) {
        todoStorage.save(todos)
      },
      // deep オプションでネストしているデータも監視できる
      deep: true
    }
  }
})
```

これで、`todos` データに何か変化があれば、自動的にストレージへ保存されるようになりました。

## STEP9 保存されたリストを取得しよう

ストレージへの保存ができたので、次はストレージからの取得です。
このアプリケーションの「インスタンス作成時」に、ローカルストレージに保存されているデータを「**自動的**」に取得して、Vue.js のデータとして読み込みましょう。
**特定のタイミングに何か処理をはさみたい**ときは「**ライフサイクルフック**」のメソッドを使用します。

タイミングがいくつか用意されていますが、今回の「インスタンス作成時」には `created` メソッドを使うといいでしょう。

```js
new Vue({
  // 省略...
  created() {
    // インスタンス作成時に自動的に fetch() する
    this.todos = todoStorage.fetch()
  }
})
```

データの取得には、先に作っておいた `todoStorage` オブジェクトの `fetch` メソッドを使用します。

ライフサイクルメソッドの定義は「`methods` の中ではない」ことに注意しましょう。

ローカルストレージは Ajax と違い同期的に結果を取得できるため、返り値を代入すればいいだけなので簡単です！

## STEP10 状態の変更と削除の処理

つづいて「状態の変更」と「削除」機能を実装しましょう。
`methods` オプションにそれぞれのメソッドを作成します。

### doChangeState メソッド（状態変更）

`item.state` の値を反転します。

### doRemove メソッド（削除）

インデックスを取得して配列メソッドの `splice` を使って削除します。

どちらも引数として、要素の参照を渡しています。

```js
new Vue({
  // 省略...
  methods: {
    // 省略...
    // 状態変更の処理
    doChangeState: function(item) {
      item.state = item.state ? 0 : 1
    },
    // 削除の処理
    doRemove: function(item) {
      var index = this.todos.indexOf(item)
      this.todos.splice(index, 1)
    }
  }
})
```

まだモックの状態だった、状態変更ボタンのイベントをハンドルします。

```html
<button v-on:click="doChangeState(item)">
  {{ item.state }}
</button>
```

つづいて削除ボタンもハンドルします。
削除操作には少し気を付ける必要があるため、キー修飾子 `.ctrl` を使って「**コントロールキーを押しながらクリック**」しなければ呼び出されないようにします。

```html
<button v-on:click.ctrl="doRemove(item)">
  削除
</button>
```

## STEP11 選択用フォームの作成

特定の作業状態のリストのみを表示させる「絞り込み機能」を追加しましょう。

スローガンテキストの下にラジオボタンをリストで表示します。
ToDo リストと同じように動的に作成するため、選択肢の `labels` リストを作成しました。

```js
data: {
  // 省略...
  labels: [
    { value: -1, label: 'すべて' },
    { value: 0,  label: '作業中' },
    { value: 1,  label: '完了' }
  ],
  // 選択している labels の value を記憶するためのデータ
  // 初期値を「-1」つまり「すべて」にする
  current: -1
}
```

`labels` リストを `<label>` タグで繰り返し描画して、内側の `<input>` タグの `value` 属性には、データ側の `label.value` データをバインドします。

```html
<label v-for="label in labels">
  <input type="radio"
    v-model="current"
    v-bind:value="label.value">{{ label.label }}
</label>
```

`v-model` ディレクティブを使って、ラジオボタンの選択値と `current` データを同期させます。
ラジオボタンが変更されると、その要素の `label.value` が `current` プロパティへ代入される仕組みです。

## STEP12 条件によるリストの絞り込み機能

`current` データの選択値によって、表示させる ToDo リストの内容を振り分けるため「算出プロパティ」という機能を使用します。
算出プロパティはデータから別の新しいデータを作成する関数型のデータです。

定義方法は、`computed` オプションに、加工したデータを返すメソッドを登録します。
いわゆる、JavaScript の `getter` と同じような働きをします。
算出プロパティは、元になったデータに変更があるまで、結果を**キャッシュする**という機能を持っています。

```js
new Vue({
  // ...
  computed: {
    computedTodos: function() {
      // データ current が -1 ならすべて
      // それ以外なら current と state が一致するものだけに絞り込む
      return this.todos.filter(function(el) {
        return this.current < 0 ? true : this.current === el.state
      }, this)
    }
  }
})
```

定義方法が違うだけで、使い方はデータと一緒です。
一覧表示テーブルの `v-for` ディレクティブで使用している `todos` の部分を `computedTodos` に置き換えましょう。

**▼ 変更前**
```html
<tr v-for="item in todos" v-bind:key="item.id">
```

**▼ 変更後**
```html
<tr v-for="item in computedTodos" v-bind:key="item.id">
```

たとえば「◯件見つかりました」という結果の要素数を表示したいとき、単純にその配列の `computedTodos.length` を見れば欲しい数字が得られます。

```
{{ computedTodos.length }} 件を表示中
```

キャッシュ機能があるおかげで、メソッドと違い何度使用しても処理は 1 度しか行われません。

## STEP13 文字列の変換処理

最後の仕上げとして「状態変更ボタン」のラベルが数字になっているのを直しましょう。

状態変更ボタンで使っている状態の `item.state` データは、文字列そのものではなく「キー」になる数字を保存しています。
一般的にもカテゴリーなどのデータでは、こういった数字や短い英数字のキーの状態で保存されます。
しかし、このままでは、作業中なら「0」完了なら「1」と表示され、まったく意味がわかりません。

絞り込みのセレクトボックス用の `labels` データを使い、`value` から `label` へ変換する `toLabel` メソッドを作成しましょう。

引数の `state` と `value` プロパティが一致した要素の `label` プロパティを返します。

```js
methods: {
  // 省略...
  // 作業中・完了のラベルを表示する
  toLabel: function(state) {
    // ここでは ES6 を使わないので filter で代用
    var el = this.labels.filter(function(el) {
      return el.value === state
    })[0]
    // 一致する要素が見つかったら label を返す
    return el ? el.label : state
  }
}
```

Mustache でメソッドを通すように変更します。

```html
<button v-on:click="doChangeState(item)">
  {{ toLabel(item.state) }}
</button>
```

これで人が理解できる文字で表示されるようになりました。
