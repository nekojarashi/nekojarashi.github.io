---
layout: post
title: "acts_as_paranoid"
date: 2013-08-20 11:19
comments: true
categories:
---

CoreDrive開発チームの原です。

今回、論理削除されたレコードをデフォルトで取得しない `acts_as_paranoid` を導入しました。まずは一般的な使い方から。

** 1.Gem を追加 **

```ruby
gem "acts_as_paranoid", "~>0.4.0" # Rails 3.2 の場合
```

** 2.Gem をインストール **

```ruby
bundle install
```

** 3.論理削除するテーブルに `deleted_at` カラム（datetime型）を追加し、マイグレーションを実行する。 **

例：Postモデルに追加する場合

```ruby
rails generate migration AddDeletedAtToPosts deleted_at:datetime
```
```ruby
rake db:migrate
```

** 4.対応するモデルに acts_as_paranoid を宣言する **

```ruby
class Post < ActiveRecord::Base
  acts_as_paranoid

  ...
end
```

** 5.この時点で、クエリのSQLにデフォルトで `deleted_at is NULL`が自動的に挿入される。コンソールを立ち上げて、Post.all を実行してSQLを確認してみる。 **

```ruby
  Post.all
  ⇒  SELECT "posts".* FROM "posts" WHERE ("posts"."deleted_at" IS NULL)
```

** 6.destroy を呼ぶと、レコードは削除されずに `deleted_at` に値が設定される。 **

```ruby
  post = Post.find(1)
  post.destroy
  ⇒  UPDATE "posts" SET deleted_at = '2013-08-20 03:47:01.506771' WHERE "posts"."id" = 1 AND ("posts"."deleted_at" IS NULL)
```

## 各種メソッド

* only_deleted<br>
  deleted_at が入っているレコードを検索する。

* with_deleted / unscoped<br>
  すべてのレコードを検索する。

* destroy! / delete_all!(conditions)<br>
  レコードを削除する。

## オプション

* :column <br>
  deleted_at 以外のカラムを論理削除の対象とする場合に使用する。

```ruby
  acts_as_paranoid :column => 'カラム名'
```

## 関連付けされたモデルの場合
今回はこれにちょっとはまりました。GitHubに書いてある通りに記述しても正しく動いてくれなくて。とりあえず、以下の場合を想定します。

・PostモデルにCommentモデルが関連づけられている。

```ruby
class Post < ActiveRecord::Base
  has_many :comments
  acts_as_paranoid
  ...

end
```
```ruby
class Comment < ActiveRecord::Base
  belongs_to :post
  acts_as_paranoid
  ...

end
```

このままPostのインスタンスに紐づくCommentを取得しようとすると、deleted_at が入っていないCommentを取得しようとします。

```ruby
  post = Post.find(2)
  post.comments
  ⇒  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = 2 AND ("comments"."deleted_at" IS NULL)
```

Post に紐づく Comment で、`deleted_at` が付いているレコードを取得するには、上記の各種メソッドで紹介した `with_deleted` または `unscoped` を使用します。

```ruby
  post = Post.find(2)
  post.comments.with_deleted
  ⇒  SELECT "comments".* FROM "comments" WHERE "comments"."post_id" = 3
```

さて、上記の各種メソッドは、`post.comments` のような配列のクラスには使えますが、`comment.post`のような場合に使うことができません。`comment.post` で `deleted_at` が入った Post を取得したい場合は、CommentモデルにPostモデルとの関連を違う名前で追加します。

```ruby
class Comment < ActiveRecord::Base
  belongs_to :post
  belongs_to :post_including_deleted, :class_name => "Post", :with_deleted => true, :foreign_key => :post_id

  acts_as_paranoid
  ...
end

```

※ 関連名に *_with_deleted は使えないようです。

:foreign_key を指定しないと結果が全部 nil になって返ってきます。この部分が GitHub のサンプルコードになかったので原因を探すのに手間取りました。

それでは、追加したメソッドでCommentを取得してみます。

```ruby
  comment = Comment.find(1)
  comment.post_including_deleted
  ⇒  SELECT "posts".* FROM "posts" WHERE "posts"."id" = 1 LIMIT 1
```

これで `deleted_ at` が入ったPostを取得することができました。