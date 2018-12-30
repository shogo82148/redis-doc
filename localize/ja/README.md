# ドキュメント

## クライアント

すべてのクライアントは`clients.json`ファイルに一覧されています。 JSONオブジェクトの各キーがクライアントライブラリを表します。以下はその例です：

```
"Rediska": {

  # プログラミング言語を指定します。
  "language": "PHP",

  # プロジェクト自体のウェブサイトがあれば、ここに書いてください。
  # 無い場合は省略してください。
  "url": "http://rediska.geometria-lab.net",

  # コードを確認できるレポジトリのURL。
  "repository": "http://github.com/Shumkov/Rediska",

  # クライアントの短い説明。
  # 説明は客観的であるべきです。ユーザーにとって必要な正しいクライアントを選ぶ手助けをすることが目的です。
  "description": "A PHP client",

  # 作者およびメンテナーのTwitter ユーザー名の配列。
  "authors": ["shumkov"]

}
```

## コマンド

Redisのコマンドは `commands.json` ファイルに説明が書かれています。

それぞれのコマンドには、人間が読める形式の完全な説明が書かれたMarkdownファイルがあります。このMarkdownは、より良い経験を提供するためにあるため、以下の点を考慮してください。

- テキストの中では、すべてのコマンドをすべて大文字で書き、バッククォートで囲う必要があります。
    例:  `INCR`

- マジックキーワードを使ってRedisの共通の要素に名前を付けることができます。例: `@multi-bulk-reply`
    これらのキーワードは展開され、ドキュメントの関連部分に自動リンクされます。

少なくとも説明と戻り値の2つの定義済みセクションを含むべきです。戻り値セクションは、@returnキーワードを使用してマークされます。

```
与えられたパターンにマッチするすべてのキーを返します。

@return

@multi-bulk-reply: 与えられたパターンにマッチするすべてのキー
```

## スタイルガイドライン

Please use the following formatting rules:

- 行を80文字で折り返します。
- Start every sentence on a new line.

Luckily, this repository comes with an automated Markdown formatter.
To only reformat the files you have modified, first stage them using `git add`
(this makes sure that your changes won't be lost in case of an error), then run
the formatter:

```
$ rake format:cached
```

The formatter has the following dependencies:

- Redcarpet
- Nokogiri
- `par` ツール

Installation of the Ruby gems:

```
gem install redcarpet nokogiri
```

par (macOS)のインストール

```
brew install par
```

par (Ubuntu)のインストール

```
sudo apt-get install par
```

## Checking your work

You should check your changes using Make:

```
$ make
```

これにより、JSONファイルとMarkdownファイルがコンパイルされ、すべてのテキストファイルにタイポがなくなります。

これらのチェックを実行するには、いくつかのRuby gemと[Aspell](http://aspell.net/)をインストールする必要があります。gemの一覧は`.gems`ファイルにあります。次のコマンドでインストールします。

```
$ gem install $(sed -e 's/ -v /:/' .gems)
```

The spell checking exceptions should be added to `./wordlist`.
