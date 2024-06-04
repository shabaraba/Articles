# Vivaldiの救出方法

Created: 2023年3月6日 12:23
Updated: 2023年4月16日 1:37
Published_Time: 2023年4月16日
Slug: fix-vivaldi-brawser
Tags: Vivaldi
Published: Yes
URL: https://blog.shaba.dev/posts/fix-vivaldi-brawser

## はじめに

ある日突然Vivaldiが起動しなくなった。
正しくは、一瞬起動してすぐ落ちるようになった。

## どうしたか

ログを探したがクラッシュログというものしか見当たらず、dmgファイルなので開けない。

ネットで調べていくと、`Library/Application Support/Vivaldi`という箇所に設定が保存されているらしい。

その中の`Default`フォルダを消せば初期化できるんだとか。

PC再起動したりVivaldiのダウングレードも試したがだめだったので、なにか設定かキャッシュが悪さをしていると考え、Default→Default2として改めてVivaldiを開いてみると初期画面が表示された。

その後Default2フォルダから設定ぽそうなファイルやフォルダをDefaultに移動させていき、

- ブックマーク
- スピードダイヤル
- 開いていたタブ

は復活することが出来た。

- ブラウザ全体の設定
- 拡張機能

についてはどうしても戻すことが出来ず断念。

（ただし拡張機能については、再DLさえすれば設定は引き継げていた。DLは別管理なのかな？）