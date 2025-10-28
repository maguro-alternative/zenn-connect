# Zenn CLI

* [📘 How to use](https://zenn.dev/zenn/articles/zenn-cli-guide)

## 記事の作成

まだarticlesディレクトリが存在しない場合は、事前にnpx zenn initを実行してください（参考：zenn-cli のセットアップ）

以下のコマンドによりmarkdownファイルを簡単に作成できます。
```
$ npx zenn new:article
```

コマンド実行時に記事の Front Matter をオプションで指定することもできます。
```
$ npx zenn new:article --slug 記事のスラッグ --title タイトル --type idea --emoji ✨
```

## プレビューする
本文の執筆は、ブラウザでプレビューしながら確認できます。ブラウザでプレビューするためには次のコマンドを実行します。
```
$ npx zenn preview # プレビュー開始
```