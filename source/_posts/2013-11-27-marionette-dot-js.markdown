---
layout: post
title: "Backbone.Marionette"
date: 2013-11-27 11:20
comments: true
categories: 
---
[Backbone.Marionette](http://marionettejs.com/)は Backbone.js用のライブラリです。
Backboneはその名の通りアプリケーションの背骨にあたる部分しか提供しません。
これによって開発者はBackboneの土台の上に自由にアプリを設計できるのですが、典型的なWebアプリ開発では多くの部分で同じようなコードを使いまわすケースが多く、規模が大きくなるとDRYに保つためにそれぞれをまとめる必要がでてきます。
Marionetteはこうした"Boilerplate"なコードの雛形を提供しますので、開発のスピードアップやコードの共通化に役立てることができます。

** Backbone.Marionette.ItemView **

`ItemView`はBackbone.Viewの拡張です。Backbone.ViewがマリオだとしたらBackbone.Marionette.ItemViewはファイヤーマリオです。いくつかの典型的なプロセスをよしなにやってくれているのでコード量を減らすことができます。そのいくつかのプロセスとは、ざっくり言うと下記の3つです。

* テンプレートエンジンにモデルを埋め込んでDOMを構築する
* 構築したDOMを表示する
* 使われなくなったらゾンビ化しないようにしてくれる

Backboneに限らず、そもそも「ビュー」というものが提供する典型的な役割とは、「モデル」という概念的なデータを現実世界に表現することです。（シャレではありませんが）ファッションでいうといろんなモデルさんに服を着せ、髪型をととのえ、ステージにたたせるスタイリストさんのような感じでしょうか。Backboneでの具体的なプロセスでいうと、「扱うモデルを決めてそのイベントを購読」し、「テンプレートエンジンにモデルのデータを埋め込み」、最後に「それを表示する」ということになります。`ItemView`は、この一連の流れがあらかじめ定義されていますので、開発者がすることはただ`ItemView`を作るだけになります。…これは具体例を見たほうがはやそうですね。つまり、

こんな感じのが

```javascript
var UserView = Backbone.View.extend({
  template: $('#user-template');
  render: function() {
    var compiledTemplate = _.template(this.template.html());
    var user = this.model.toJSON();
    var html = compiledTemplate(user);
    this.$el.html(html);
  }
})
```

こうなります。
```javascript
var UserView = Backbone.Marionette.ItemView.extend({
  template: '#user-template'
});
```

CoffeeScriptだとこんな感じかな
```coffeescript
UserView = Backbone.View.extend(
  template: $('#user-template')
  render: ->
    compiledTemplate = _.template(@template.html())
    user = @model.toJSON()
    html = compiledTemplate(user)
    @$el.html html
)
```
↓
```coffeescript
UserView = Backbone.Marionette.ItemView.extend(template: '#user-template')
```
うーんなかなかスッキリしたんじゃないでしょうか。


