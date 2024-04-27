---
title: "discordgoでE2Eのテストコードを書く" # 記事のタイトル
emoji: "😸" # アイキャッチとして使われる絵文字（1文字だけ）
type: "tech" # tech: 技術記事 / idea: アイデア記事
topics: [] # タグ。["markdown", "rust", "aws"]のように指定する
published: false # 公開設定（falseにすると下書き）
---
# あいさつ
みなさん、こんにちは！マグロです。
4月になり、新年度が始まりました。新しい環境でのスタートはいかがでしょうか？

僕は新年度になってもdiscordbotを作っています。

今回は、discordgoでテストコードを書く方法について解説します。

# テストコードとは
プログラムの動作を確認するためのコードのことです。
テストコードを書くことで、プログラムの動作が正しいかどうかを確認することができます。

**まあ要するに書けばお得です。**

# Goでのテストコードの書き方

知見がある人はスキップしてください。
:::details Goでのテストコードの書き方
Goでは、テストコードを書くための標準パッケージが用意されています。
## testファイルの作成
テストコードは、テスト対象のコードと同じディレクトリに`_test.go`という名前のファイルを作成します。

例えば、`add.go`というファイルがある場合、テストコードは`add_test.go`という名前のファイルに書きます。

## テスト関数の作成
テストコードは、`Test`から始まる関数を作成します。

```go:add_test.go
package add

import "testing"

func TestAdd(t *testing.T) {
    if add(1, 2) != 3 {
        t.Error("add(1, 2) should be 3")
    }
}

func add(a, b int) int {
    return a + b
}

```

```TestAdd```関数は、```add(1, 2)```が```3```を返すかどうかを確認しています。

## テストの実行

テストコードは、以下のコマンドで実行します。

```bash
$ go test
```

## interfaceを使った結合テスト
Goにはinterfaceというものがあります。

簡単に言うと、中身の関数(メゾット)が同じであれば、異なる構造体でもひとくくりにして扱えます。

例えば、以下のようなコードがあるとします。

```go
package main

import "fmt"

type math struct{}

func (m *math) adder(a, b int) int {
	return a + b
}

func addCmd(m *math) int {
	a := 1
	b := 2
	return m.adder(a, b)
}

func main() {
	m := math{}
	a := addCmd(&m)
	fmt.Println(a)
}
```

上記の処理はただの足し算ですが、データベースや外部APIとの通信など、複雑な処理を行う場合、テストを書くことが難しくなります。

そこで、interfaceを使ってテストを行います。

```go
package main

import (
    "testing"
)

type math struct{}

func (m *math) adder(a, b int) int {
    return a + b
}

type mathMock struct {
    adderFunc func(a, b int) int
}

func (m *mathMock) adder(a, b int) int {
    return m.adderFunc(a, b)
}

type mathFunc interface {
    adder(a, b int) int
}

var (
    _ mathFunc = (*mathMock)(nil)
    _ mathFunc = (*math)(nil)
)

func addCmd(m mathFunc) int {
    a := 1
    b := 2
    return m.adder(a, b)
}

func TestAdd(t *testing.T) {
    m := &mathMock{
        adderFunc: func(a, b int) int {
            return 3
        },
    }
    if addCmd(m) != 3 {
        t.Error("addCmd should be 3")
    }
}

```

上記のコードでは、```math```構造体の代わりに```mathMock```構造体を使っています。

```mathMock```構造体は、```adderFunc```という関数を持っており、コード内のように戻り値を指定することができます。

**データベース関連の操作であれば、データベースを使用せずにテストを行うことができます。**

このように、interfaceを使うことで、結合テストを行いやすくすることができます。

:::

# discordgoでのテストコードの書き方

ディレクトリ構成です。以下のGitHubリポジトリを参考にしています。

https://github.com/maguro-alternative/remake_bot

```
├── bot                         // DiscordBotを動かすためのディレクトリ
│   ├── cogs                    // DiscordBotのコグ
|   ├── commands                // スラッシュコマンド
│   ├── config                  // 環境変数設定ファイル
│   └── main.go
```

## 共通事項
当たり前ですが、単体テストはdiscordgo側が行っています。
そのため、今回書くのは**結合テスト**です。

discordgoでは、```discordgo.Session.AddHandler```でイベントを受け取った際の処理を登録できます。
しかし、引数と戻り値の指定があり、テストコードを書くことが難しいです。

:::details 何で書くのが難しいの？
```go
s.AddHandler(func(s *discordgo.Session, m *discordgo.MessageCreate) {
    // ここに処理を書く
})
```

上記のコードは、```s```と```m```を引数に取りますが、**戻り値と引数をこれ以上追加するとエラーが発生します。**
どのような関数なのかでイベントを判別しているためです。

結合テストでは、正しく処理できたか、想定通りのエラーが発生するかを確認する必要があります。

最低限```error```を返さないと、テストで確認できるものがありません。
:::

## cogsのテストコードの書き方



