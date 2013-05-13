# Gitの使い方
## はじめに

Gitとはバージョン管理システム（VCS）の一つです。ファイルの作成日時、変更日時、変更点などの履歴を保管して、何度も変更を加えたファイルであっても過去の状態に戻れたり、変更内容を確認できるようにするソフトウェアです。

今回修正するアプリケーションでは、バージョン管理システム（VCS）としてGitをお使いでしたので、今回はこれを使って作業した内容を残しましょう。

## 修正したものの確認

`git status`と入力すると、変更したファイル・追加したファイル・削除したファイルなどが表示されます。

## コミット

コミットとは、現在の状態をバージョン管理システム（VCS）に登録する作業のことウィいます。Gitにおけるコミットは2段階の作業が必要です。

`git add ファイル名`でファイルをコミット対象として登録します。

その後、`git commit -m コミットメッセージ`でファイルをコミットします。

## NextStep

`git status`でswpなどのファイルが表示されていたかと思います。当日は、「これらは無視して結構です。」とお話しましたが、いちいち目で確認するのは大変です。

そこで、余計なファイルはGitの対象外とする機能があります。`.gitignore`（前にピリオドあり）で検索してみてください。

# アプリケーションの改造について
## 目標

すでに作られている小説ダウンロードプログラムは短編小説のみの対応でした。複数の章から成る長編に対応するのが今回の目標です。小説サイトは
[小説家になろう](http://syosetu.com/)
です。

## 現状できていること、できていないこと

現状で次のことができていました。

* 指定したURLからデータを取得する
* 取得したデータをNokogiriを使って解析して、タイトル・著者名・内容を取得する
* 取得したタイトル・著者名・内容を使ってテキストファイルを作成する

現状で次のことが出来ていませんでした。

* 短編小説と長編小説をデータとして区別できない
* 長編データをまとめたテキストファイルとして生成できない

## 短編小説と長編小説との区別

まずは、指定したURLの出たから、短編小説か長編小説かを区別する方法を見つけることからはじめました。

試行錯誤の結果、HTML内に`period_subtile`という`class`が設定してある`td`タグがあるかないかで判断できそうです。Nokogiriを使った場合、

```ruby
doc = Nokogiri::HTML(text)
doc.xpath('//td[@class="period_subtitle"]')
```

で対象の`td`タグを取得出来ます。得られる結果は配列で、`td`タグがない場合（つまり、短編小説である場合）は`[]`、`td`タグがある場合（つまり、長編小説である場合）は`[要素#1, 要素#2, 要素#3, ...]`のように1つ以上の要素が入った配列となります。

## URLの取得

長編小説の中にあるそれぞれの章は

```ruby
要素#1.children.attribute('href').value
```

で取得できます。

配列の各要素ごとに`children.attribute('href').value`を実行し、その結果を配列として受け取れると、あとの処理もeachなどを使って繰り返し処理ができそうです。

配列の各要素ごとに何らかの処理を行い、結果を配列として受け取るには[map](http://doc.ruby-lang.org/ja/1.9.3/method/Enumerable/i/collect.html)を使います。プログラムとしてまとめると、

```ruby
urls = doc.xpath('//td[@class="period_subtitle"]').map do |node|
  node.children.attribute('href').value
end
```

と書くことができます。

## 小説の取得

得られたURLの一覧を使って小説の取得処理を行います。`urls`の大きさによって短編小説と長編小説を分けることができます。プログラムとしては、次のように書くことができます。

```ruby
if urls.size == 0
  # 短編小説
else
  # 長編小説
end
```

短編小説の部分はこれまでの記述で問題ないでしょう。自作のクラスである`NarouNovel`を使って

```ruby
NarouNovel.new(text)
```

を`urls.size == 0`が`true`のときの処理として追記。

長編小説の部分も`NarouNovel.new(text)`を`urls`の要素分繰り返すとできそうです。

## 小説のマージ

`NarouNovel.new(text)`が返すのは、タイトル・著者名・本文をそれぞれ`title`・`author`・`document`に格納したインスタンスです。

複数の小説のマージはとりあえず`document`を連結することにしましょう。本当のところは`title`を各`document`ごとに付与すると素敵ですが、それは後の課題ということで。

## NextStep

今回のMoreで長編小説を取得することが出来ましたが、次の点を後回しにしました。

* 長編小説の各章のタイトルを本文内に反映できていない

    これについては、「どのように出力させたいか」が決まれば、実現は比較的容易にできるかと思います。

* 処理が冗長

    短編小説にて、これまでよりも処理が長くなっているように感じたかもしれません。実際、そのとおりでサイトからデータを2回取得してしまっています。これを修正してみてください。

* グローバル変数をなくしたい

    コメントの中に、「グローバル変数を無くしたい」と書かれていました。素晴らしい目標です。これについては結構ハードルが高いかなと思います。ハードルが高いと考える理由は2点あります。

    1. controller内で2つのメソッド感でどのようにデータを持ちまわるかを考える必要が有るため。
    2. 現状で着ていることを壊さないでプログラムを修正する必要が有るため。

    このために、「自動テスト」が必要になってきます。自動テストについては「RSpec」で検索してみると良いと思います。

## Tips

1. Railsのプログラムを修正するたびに`rails server`を止める必要はありません。
    プログラムを修正するたびに、`rails server`を`Ctrl + C`で終了させてました。確実なのですが、ちょっと面倒臭いです。実は`app`フォルダ以下のファイルは修正しても、`rails server`を止める必要はなく、自動的に更新されます。

2. Rubyプログラムをお手軽に試すには、`irb`や`pry`を使ってみるとよいでしょう。
    今回、Nokogiriの使い方や短編小説・長編小説の使い方で色々悪戦苦闘しました。こういう場合、Railsアプリを修正してはブラウザの画面を叩くのはちょっと面倒臭いです。そこで、`irb`というコマンドがあります。

    `irb`はRubyのプログラムをその場で実行することができます。`irb`と端末に入力するとすぐに使えます。

    ```ruby
    irb(main):001:0> puts 1 + 2
    3
    => nil
    irb(main):002:0>
    ```

    `irb`の機能強化版として`pry`というGemもあります。`gem install pry`でインストール後、`pry`と端末に入力するとすぐに使えます。

3. デバッグについて
    今回、デバッグには`logger.debug 変数名`をプログラム内に入れ、変数値を出力させました。いわゆるprintfデバッグです。

    ちょうどQA@ITというサイトに[Ruby on Railsのデバッグ方法。何かいい方法ありませんか？」(http://qa.atmarkit.co.jp/q/2914/)という質問がありましたのでご紹介しておきます。


