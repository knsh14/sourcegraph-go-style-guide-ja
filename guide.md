Go style guide
---

[原文](https://about.sourcegraph.com/handbook/engineering/go_style_guide)

この文章はSourcegraphのコードで使われているスタイルをまとめたものです。コード外のスタイルについては[こちら](https://about.sourcegraph.com/handbook/communication/style_guide)を参照して下さい。
このドキュメントは全てを網羅しているわけではありません。[Go Code Review Comments](https://code.google.com/p/go-wiki/wiki/CodeReviewComments)や[Effective Go](http://golang.org/doc/effective_go.html)の後に参照してください。

# Panics
`panic`は通常たどり着くはずがないところで使うようにしましょう。

# Options
通常の場合、`Get(build BuildSpec, opt *BuildGetOptions) (*Build, Response, error)`のように関数の引数にoption型の構造体のポインタを渡すときは`nil`を渡します。
ポインタが`nil`の場合は関数はデフォルトの挙動をします。
もし構造体のポインタが`nil`では行けない場合、値型を使うかドキュメントにその旨を書いておく必要があります。

# Pull requests
PRで新しい定数や定義を追加した場合、gofmtが自動的にインデントを調整してしまいます。なので最初は改行で分けて別のブロックで定義してレビューを依頼します。
そうすることで、レビュアーが余計な変更に意識を割かれずにすみますし、他の作業とのコンフリクトも起きづらくなります。
マージする直前やマージした直後に改行を消して適切な場所に配置します。

# Group code blocks logically
変数は実際にそれを使う直前に定義しましょう

```go
a, b, c := Vector{1, 2, 3}, Vector{4, 5, 6}, Vector{7, 8, 9}
a, b := swap(a, b)

total := add([]Vector(a, b, c))...)
```

よりも

```go
a, b := Vector{1, 2, 3}, Vector{4, 5, 6}
a, b := swap(a, b)

c := Vector{7, 8, 9}
total := add([]Vector(a, b, c))...)
```

のほうが好ましいコードです。

`Vector`型の変数`c`は`Add()`の直前まで利用されません。なので直前で宣言しませんか？
コンポーネントを論理的にグループ化することにより、実行する際のコンポーネントの周囲のコンテキストが失われないようにできます。
具体的には

* 頭のバッファを少なくするようにします。ScreenReaderを使うのも効果的です。
* 定義や変数を見つけやすくするためにコードベースを小さく保ちましょう。複雑な制御が必要な場合にも効果的です。
* コードを理解するために条件分岐などを最小限にします。

この話はインターフェイスや構造体などにも当てはまります。

# Keep names short
`var vectorA, vectorB Vector`よりも`var a, b Vector`のほうが好ましいコードです。
Goでは[常に短い変数名のほうが好まれます](https://github.com/golang/go/wiki/CodeReviewComments#variable-names)。

更に付け加えると
* 短い名前のほうが聞きやすいし読みやすいです
* 短い名前のほうが移動しやすいです
* 短い名前のほうがタイプしやすいです

上の例の`vectorA`や`vectorB`でも変数名にコンテキストがあるのでよさそうに見えます。
この変数が他の場所で利用される場合には曖昧さがなくなりよいでしょう。
ですが、ブロックでスコープが切られている場合には必要ありません。

# Make names meaningful
`var tVec, sVec Vector`よりも`var total, scaled Vector`のほうがよいでしょう。

意味のある変数名にすると何をしているコードなのか理解しやすくなります。
具体的には
* その変数が何をするかを覚えておく必要が減ります
* 定義や使い方を確認するためにあちこち見に行く必要がありません
* 重要な変数なのかただの一時変数なのかを判別するのに役立ちます

できる限りコメントよりも意味のある変数名を使うようにしましょう。
コメントは追加の説明であり、変数が後で使われたときに元の定義に飛ぶ回数を減らすものではありません。

# Use pronounceable names

```go
var tVec Vector
var addAllVecs(...)
```
よりも
```go
var total Vector
func add(...)
```
のほうがよいでしょう。

発音しやすい名前は
* テキスト読み上げ機能で読むことができます
* 文字を1つづつ読み上げるよりも早くすみます

[@juliaferraioli がコードを読み上げる動画を見ると良いでしょう](https://www.youtube.com/watch?v=xwjvufcJK-Q)

デモを見て気づく事がありましたね？screenreaderが変数名を扱うときにへんじゃありませんでしたか？

[@juliaferraioli](https://twitter.com/juliaferraioli)はscreenreaderが“gee-thub”と読んでいるのを“GitHub”だとわかるまで15分も頭を悩ませていたという話をしてくれました。

なので変数名などには発音できる名前を使ってください。
単語に手を加えないでください。
その単語がどのように発音されるかを考えてください。
できる限り単語の組み合わせの変数名は避けてください。
Screenreaderはそれを1単語として扱ってしまうかもしれません。

# Use new lines intentionally
```go
a, b := Vector{1, 2, 3}, Vector{4, 5, 6}
a, b := swap(a, b)

c := Vector{7, 8, 9}
total := add([]Vector(a, b, c))...)
```

もしグルーピングの章を再度見直すと、改行の使い方についても見ることができます。
改行はなにも考えずに使うスパイスのようなものです。
ですが改行は強力なメッセージを持つこともあります。
改行は段落を区切るような使い方をおすすめします。
全く改行を使わないと読み手が迷子になってしまいます。
ただ、使いすぎるとコードのメッセージがばらばらになってしまいます。
改行をうまく使うことで、ユーザーに論理的なブロックを見せて読みやすくすることができます。

これらを使ってより意図を伝えられるコードを書きましょう！
