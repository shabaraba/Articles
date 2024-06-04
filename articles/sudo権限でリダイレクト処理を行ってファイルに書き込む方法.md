# sudo権限でリダイレクト処理を行ってファイルに書き込む方法

Created: 2022年12月13日 9:35
Updated: 2022年12月13日 9:52
Published_Time: 2022年12月13日
Slug: redirect-with-sudo
Tags: Shell
Published: Yes
URL: https://blog.shaba.dev/posts/redirect-with-sudo

## やりたかったこと

使用するシェルを変更しようと、`/etc/shells`末尾にコマンドラインからパスを追記したかったです。

```bash
which zsh >> /etc/shells
```

この時、`/etc/shells`はrootの所有物なので、permission error が発生したため、sudo をつけて再度実施しました。

```bash
sudo which zsh >> /etc/shells
```

```bash
permission denied: /etc/shells
```

**は？（強め）**

## 原因

ファイルに書き込む際の`>`や`>>`を**リダイレクト**と呼びますが、このリダイレクト処理はsudoを頭につけようがログインユーザーの権限で行われるそうです。

## 解決方法: tee コマンドにパイプする

`tee`コマンドを使うことが解決方法の一つに挙げられることが多いようです。

teeとは**入力された値を標準出力とファイル出力の両方処理するコマンド**です。

コマンドなのでsudoを頭につけることができて、

```bash
which zsh | sudo tee -a /etc/shells
```

とすることで解決できました！

<aside>
💡 `-a` オプションはファイル末尾に追記するときに使用します。
先頭に追加する際は外してください。

</aside>

<aside>
💡 標準出力が不要なら `> /dev/null`を後ろにつけることで回避できます。

</aside>

## まとめ

最近職場に触発されて色々とコマンドライン上でやろうとする気持ちが芽生えてきているので、少しずつ覚えていこうと思います！