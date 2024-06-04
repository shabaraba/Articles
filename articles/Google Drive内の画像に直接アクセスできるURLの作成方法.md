# Google Drive内の画像に直接アクセスできるURLの作成方法

Created: 2022年5月12日 9:43
Updated: 2022年12月12日 13:28
Published_Time: 2022年5月13日
Slug: google-drive-url-directly
Tags: GoogleDrive
Published: Yes
URL: https://blog.shaba.dev/posts/google-drive-url-directly

## はじめに

在籍している会社で開発しているサービス内で、画像をURLから取得して処理をする機能があります。
APIとして提供していて、誰でも利用できる状態なのですが、ログを見てみるとしばしば画像取得に失敗している様子。

詳細を見ていくと、画像取得に失敗しているときのURLが、Google Driveで発行される共有URLを指定していることがわかりました。

## Google Driveの共有URLについて

Google Driveの共有URL、結構クセモノ感強めな印象を受けていました。
共有URLを発行すると下記のような値が得られるかと思います。

https://drive.google.com/file/d/1zJ■■■■■■■■■■■■■■hBF/view?usp=sharing

このURLをそのままブラウザのオムニバーに張り付けてみると、画像単体ではなくて、Google Driveが開いて、その中で画像がプレビューとして表示されます。

上記の共有URLは、画像の外部URLではなくて、Google Driveのviewを表示するURLなんですね。
だからそのまま使用しても画像として認識されず、エラーになってしまいます。

## 対策

ではGoogle Driveは画像の外部ストレージとしては使えないのか、というと、今のところ回避策が用意されています。
答えはこちらのqiita記事にある通りなのですが、改めて自分の言葉で残します。

[Google Drive上のファイルに直接アクセスする - Qiita](https://qiita.com/rot-z/items/299ac40361690c51ce1d)

やり方だけまず書くと、下記のようなURLを指定すればよいです。

https://drive.google.com/uc?id=<ファイルID>

ここでいうファイルIDとは、共有URLの黄色い背景の箇所のことです。

https://drive.google.com/file/d/1zJ■■■■■■■■■■■■■■hBF/view?usp=sharing

つまり

**https://drive.google.com/uc?id=1zJ■■■■■■■■■■■■■■hBF**

とすればよいわけですね。

## 終わりに

できれば隠しURLではなくて、Google Drive上でも取得できるようにしてほしいな～