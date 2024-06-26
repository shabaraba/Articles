# USキーボードでかな入力時の配列をmacとwinで揃える方法

Created: 2023年4月13日 8:46
Updated: 2023年4月17日 10:50
Published_Time: 2023年4月17日
Slug: share-us-keyboard-using-kana
Tags: keyboard, 論理配列
Published: Yes
URL: https://blog.shaba.dev/posts/share-us-keyboard-using-kana

## はじめに

下記の記事で、普段使いのキーボードをmoonlanderに乗り換えました。

[HHKBからMoonlanderに乗り換えた](https://blog.shaba.dev/posts/introduce-moonlander)

キー配置を自由にイジれるのが想像以上に面白く、色んな配列にチャレンジしています。

その一環として、飛鳥かな配列というかな入力の配列を現在練習中なのですが、かな入力の設定で躓いたので、記録しておきます。

なお、この記事は練習も兼ねて、moonlanderで飛鳥かな配列を使って書いています。

## 想定読者

- US配列の自作キーボードを使用したい
- かな入力を使いたい
- winとmac両方でキーボードを共有したい
- winで他の人とPCを共有したく、各々使用するキー配列が異なる

…誰だこんな奇特なやつ。

## 配列ごと、OSごとにかな入力の配置が異なる

今までかな入力なんぞ見向きもしてこなかったから知らなかったのですが、US配列とJIS配列、US配列の場合は更にwinとmacでかな入力の配置が異なることがわかりました。

例えば「ろ」は、JIS配列だと「＼」ですが、US配列だと「`」に配置されていますし、
winだと『「』を入力するには「[」をタイプしますが、macだと「]」になります。

上記の組み合わせで、「半濁点゜」の位置もぜんぜん違うところにあったりします。

そのため、想定読者（=僕）の場合、これらの違いを吸収する仕組みが必要になります。

## google日本語入力を使う

その設定はgoogle日本語入力で解決出来ました。

mac側で、下図のように「日本語入力では常に日本語キー配列を使う」を有効にすることで、
かな入力のときだけJIS配列として認識してくれるようになります。

![スクリーンショット 2023-04-13 8.46.23.png](%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588_2023-04-13_8.46.23.png)

JISだとOS間で配置に違いはないので、上記問題を解決することが出来ました。

## 終わりに

ということで、現在僕は、プログラムをするときはqwery配列のレイヤー、資料や連絡を書く際は飛鳥のレイヤーを使い分けています。
まだまだ飛鳥のタイビングは激おそなので、いっぱい練習したいと思います！