# URIライクな文字列で改行してくれないときのcssの充て方

Created: 2022年3月29日 1:45
Updated: 2023年3月7日 13:11
Published_Time: 2022年3月30日
Slug: new-line-css-for-uri-like-string
Tags: CSS, HTML, 個人開発
Published: Yes
URL: https://blog.shaba.dev/posts/new-line-css-for-uri-like-string

## はじめに

先日下記記事を投稿しました。

[dyld: Library not loaded: /usr/local/opt/icu4c/lib/libicudata.69.dylib で動かなくなったmacのPHPを救出した](https://blog.shaba.dev/posts/library-icu4c-not-found)

その後、ブログを確認すると...

![css崩れ](%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88_2022-03-29_1.48.30.png)

css崩れ

おやおや、モバイルくらいのサイズにするとレイアウトが崩れてしまってます...。

今回はこのレイアウト崩れの対処方法です。

## ファイルパスの文字列で改行ができていないことが原因

しばらく原因がわからなかったのですが、出力する要素を一つずつ確認していくと、どうやら

`/usr/local/opt/icu4c/lib/libicudata.69.dylib`

の文字幅以上に画面幅を縮めると、レイアウトが崩れることがわかりました。
もう少し見ていくと、この文字内で改行が行われないことがわかりました。

なるほど、なぜかこの英字で改行出来ていないから幅が飛び出してしまってるのね。

## overflow-wrap: 'anywhere’を充てる

 更に調べていくと下記cssを発見。

[overflow-wrap - CSS: カスケーディングスタイルシート | MDN](https://developer.mozilla.org/ja/docs/Web/CSS/overflow-wrap)

どうやらスペースを挟んでいない長い英字列はデフォルトでは改行されないらしい。
uriとかパスとかの文字列は基本スペースなんか入らないから、何も指定しないと画面からはみ出る恐れがあるのね。

一応技術ブログだし、よく出てくるはずなので対応。

cssにoverflow-wrap: 'anywhere’を充てることで解決するぽい。

## 改修後

![スクリーンショット 2022-03-29 1.59.22.png](%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88_2022-03-29_1.59.22.png)

治った✨

## 終わりに

当方cssに明るくなく、基本的なものかもしれませんが少しずつぶち当たって行こうと思います。
（これの解決に3時間くらいかかった...）