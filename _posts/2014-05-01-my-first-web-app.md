---
layout: post
unique_identifier:
 - 'http:/example.jp/bookid_in_url'
 - 'BookID'
 - 'URL'
title: 'Webアプリをまずはつくってみよう'
creator: '五十嵐邦明'
date: '2014-05-01'
categories:
---

# Webアプリをまずはつくってみよう

それでは、早速Webアプリをつくってみましょう。作るのは、記事と写真が投稿できる簡易なブログアプリです。まずはWebアプリでどんなことができるのか、その作り方はどのようなものかを体験してもらうのがこの章の目的です。それぞれのコマンドやコードの意味は、本書の中で追って説明していきます。

## Webアプリをつくる

### アプリの作成とWelcome画面

まずはターミナルを起動して以下のコマンドを打ってみましょう。

```bash
mkdir my_web_apps
cd my_web_apps
```

mkdir はフォルダを作成するコマンド、cd はターミナル内で現在のフォルダを移動するコマンドです。(Winで普段使いなれている「コンピューター(エクスプローラー)」を起動するには、ターミナルで explorer . と打つと現在のフォルダを開くことができます。MacでFinderを開くにはターミナルから open . と打ちます。これらからフォルダを作るのと同じ働きをするコマンドが mkdir です。)

続けて以下のコマンドを実行します。

```bash
rails new blog_app
```

以下のような結果が表示されるでしょうか。(実行には少し時間がかかります)

```console
$ rails new blog_app
      create
      create  README.rdoc
      create  Rakefile
... (略)
Your bundle is complete!
Use `bundle show [gemname]` to see where a bundled gem is installed.
         run  bundle exec spring binstub --all
* bin/rake: spring inserted
* bin/rails: spring inserted
```

ターミナルの画面にこのように"Your bundle is complete!"と表示されれば成功です。railsコマンドはたくさんのフォルダとファイルを自動で作ります。

次は以下のコマンドを実行してみてください。

```bash
cd blog_app
rails server
```

うまく動作している場合は、以下のような表示になります。

{% highlight console %}
$ rails server
=> Booting Puma
=> Rails 5.0.0.beta1 application starting in development on http://localhost:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
I, [2015-12-27T15:39:46.449421 #79231]  INFO -- : Celluloid 0.17.2 is running in BACKPORTED mode. [ http://git.io/vJf3J ]
Puma 2.15.3 starting...
* Min threads: 0, max threads: 16
* Environment: development
* Listening on tcp://localhost:3000
{% endhighlight %}

では、ブラウザを起動して以下のURLを入力してアクセスしてみましょう。

* localhost:3000

![welcome rails](assets/my-first-web-app/welcome_rails.png)

これは、Railsが起動し、あなたのブラウザからのリクエストを受け付けて、表示している画面です。ここまでのわずかの手順で、Webアプリをつくり、画面を表示しているのです。

### ページの作成

ひきつづき、以下のコマンドを入力してください。rails server が起動している場合は、Ctrl-c (controlキーを押しながらcキー)で終了してからコマンドを打ちます。

(※Rails4.2以前では `rails db:migrate` の替わりに `bin/rake db:migrate` と実行してください。)

```bash
rails generate scaffold entry title description:text picture
rails db:migrate
rails server
```

```console
$ rails generate scaffold entry title description:text picture
Running via Spring preloader in process 79311
      invoke  active_record
      create    db/migrate/20151227064132_create_entries.rb
...(略)

$ rails db:migrate
== 20151227064132 CreateEntries: migrating ====================================
-- create_table(:entries)
   -> 0.0012s
== 20151227064132 CreateEntries: migrated (0.0012s) ===========================

$ rails server
=> Booting Puma
=> Rails 5.0.0.beta1 application starting in development on http://localhost:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
I, [2015-12-27T15:42:43.062862 #79399]  INFO -- : Celluloid 0.17.2 is running in BACKPORTED mode. [ http://git.io/vJf3J ]
Puma 2.15.3 starting...
* Min threads: 0, max threads: 16
* Environment: development
* Listening on tcp://localhost:3000
```

ここでまたブラウザを使い、以下のURLを表示させます。

* http://localhost:3000/entries

![entries](assets/my-first-web-app/scaffold_index.png)


画面が表示されたら、New Entry のリンクをたどってみましょう。Title, Description などを入力し、Create Entry ボタンで保存してみてください。また、保存したデータを編集や削除をしてみてください。ここまでの作業で簡易なブログの基本機能ができました。

### 写真アップロード機能の追加

次は、写真をアップロードできるようにしてみましょう。Rails にファイルをアップロードする機能を追加するには、ライブラリをインストールする必要があります。Railsルートフォルダ(```my_web_apps/blog_app```)内の Gemfile というファイルを開いてください。

```ruby
gem 'sqlite3'
```

という記述を探して、その下あたりに次の一行を追加して保存します。

```ruby
gem 'carrierwave'
```

追加できたら、Terminal で、次のコマンドを実行してください。 (rails server は Ctrl-c で終了させてください。)

```bash
bundle
rails generate uploader Picture
```

```console
$ bundle
Fetching gem metadata from https://rubygems.org/...........
Fetching version metadata from https://rubygems.org/...
Fetching dependency metadata from https://rubygems.org/..
Resolving dependencies...
Using rake 10.4.2
...
Installing carrierwave 0.10.0
Bundle complete! 13 Gemfile dependencies, 67 gems now installed.
Use `bundle show [gemname]` to see where a bundled gem is installed.
$ rails generate uploader Picture
Running via Spring preloader in process 80021
      create  app/uploaders/picture_uploader.rb
```

次にファイルを編集します。```app/models/entry.rb``` を開いて、次の行

```ruby
class Entry < ApplicationRecord
```

の直後に、以下を追加します。

```ruby
mount_uploader :picture, PictureUploader
```

さらに、```app/views/entries/_form.html.erb``` を以下のように編集します。(- 記号の行を削除して、かわりに + 記号の行を追加してください。)

```diff
- <%= f.text_field :picture %>
+ <%= f.file_field :picture %>
```

あと少しです。最後に ```app/views/entries/show.html.erb``` を開いて編集します。

```diff
- <%= @entry.picture %>
+ <%= image_tag(@entry.picture_url) if @entry.picture.present? %>
```

rails server を起動して、ブラウザから http://localhost:3000/entries へアクセスしてみましょう。

```bash
rails server
```

```console
$ rails server
=> Booting Puma
=> Rails 5.0.0.beta1 application starting in development on http://localhost:3000
=> Run `rails server -h` for more startup options
=> Ctrl-C to shutdown server
I, [2015-12-27T15:50:47.054387 #80069]  INFO -- : Celluloid 0.17.2 is running in BACKPORTED mode. [ http://git.io/vJf3J ]
Puma 2.15.3 starting...
* Min threads: 0, max threads: 16
* Environment: development
* Listening on tcp://localhost:3000
```

New Entry リンクをクリックすると、「ファイルを選択」ボタンが増えているかと思います。ボタンを押して画像ファイルを選び、アップロードしてみましょう。

![new](assets/my-first-web-app/entries_new.png)

![show](assets/my-first-web-app/entries_show.png)

画像をアップロードできる簡易ブログアプリができあがりました。初めてのWebアプリづくりはいかがでしたか？とてもシンプルなアプリですが、Webアプリの基本となるデータ作成、表示、更新、削除の機能や、写真投稿機能を付加するためのライブラリの使い方などがこの中につまっています。本書を読み終わる頃には、ここで書いたコードやコマンドの意味が分かるようになりますので、今はまだ分からなくて大丈夫です。（まだ何も説明していないので当たり前ですね。）次の章から、Webアプリがどうような仕組みで動作するのかを説明していきます。
