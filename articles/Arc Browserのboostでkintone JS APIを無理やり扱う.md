# Arc Browserのboostでkintone JS APIを無理やり扱う

Created: 2023年8月4日 13:17
Updated: 2023年8月16日 10:28
Published_Time: 2023年8月16日
Slug: how-to-use-object-in-arc-boost
Tags: Arc
Published: Yes
URL: https://blog.shaba.dev/posts/how-to-use-object-in-arc-boost

## はじめに

最近なのかわかりませんが、前から個人的に目をつけていたArc Browserが一般公開になりました。
（waitlistに登録していたのですがとうとう来ませんでした…）

早速インストールして常用してみて1週間くらい経ちましたが、今のところ凄い良いです！

- 通常タブがデフォルト12時間で勝手に消える仕様とか
- ピン留めしたタブ内で別のドメインを開いた時は勝手にモーダル形式で開かれるとか
- ダウンロードしたファイルはプレビュ付きですぐ確認できてファインダ介さずアクセスできるとか
- スクショは要素ごとに選択して撮れるとか

挙げるとキリがないのですが、とにかく痒いところに手が届くよう作られていて、とてもUXが良いので、とてもオススメです！

ダウンロードはこちらから↓

[A friend is sharing Arc with you!](https://arc.net/gift/d77c1bd2)

## ブックマークレットが(そのままだと)使えない辛み

なんですけど、唯一辛いところがありまして、ブックマークレットが使えないんですよね。

結構業務で便利ブックマークレットを用意していたので、こればっかりは辛い。

代わりにBoostという、ページを自由にカスタマイズできる機能（js込）があるので、そちらを使ってねという感じかと思っているのですが、なかなか上手くいかない。

## Boost機能

詳しくは公式HPを参照いただきたいのですが、簡単に言えばドメイン専用のcssやjsを書くことができる機能です。
また、指定したdomを表示しないようにすることもできるので、よくアクセスするサイトで必要な要素のみ残したり、簡単なカスタマイズをchrome拡張を通さずに実装できるスグレモノです。

作ったBoostは全世界に公開することもできて、いろんな便利機能をコードを書くことなしに享受することもできます！

[Arc Browser Boosts - Arc Boosts](https://arcboosts.com/boosts)

…ただ使った感じ、chrome拡張だとそのページが持つjsの変数にアクセスできたけれど、Boostではできないらしい（詳しい内容はわかっていません）。

つまり、個人的に業務でよく使っているkintone JS APIが使えない。

## kintone JS APIとは

kintoneのサービス画面内で、サービス情報（例えばレコードの情報など）を、dom解析を介さなくても取得、更新できるように用意されているjsメソッドです。
kintoneオブジェクトが各サービスページで用意されていて、そのオブジェクトにCRUD用のメソッドが生えている感じですかね。

Arcに来る前は、ブックマークレットでkintone JS APIを利用して、簡単な集計を行っていました。

…が、前述の通り、Arcは現在ブックマークレットは使えないのと、Boostではページ元のjs変数にアクセスできないみたいなので、そのままBoostに書いただけではkintone JS APIが使えません。

<aside>
💡 ブックマークレットを起動するchrome拡張もあり、そちらを経由すればArcでもブックマークレットは使用できます。が、kintone JS APIの性質上、メンバー変数が日本語になることがままあります。chrome拡張を挟むと日本語はurlエンコードされてしまい、うまく変数を読み込めずエラーになってしまいます。要は相性が悪いのですね。

</aside>

## 対処案

ちょっといい案が思いついていないのですが、`location`や`addEventListener`は使えるので、

- 指定urlの場合のみ
- 画面が読み込まれたタイミングで、やりたい操作が書かれたscriptタグ（元ブックマークレットだったコード）を埋め込み
- ショートカットキーで呼び出す

ようにすることで暫定対処することにしました。

```jsx
if (location.href.includes("/path/params")) {
  // document.addEventListenerではない
  addEventListener("load", () => {
    const script = document.createElement('script');
    script.id = 'js-test';
    script.innerHTML = `
    document.addEventListener('keydown', keydownEvent,false);
    function keydownEvent(event){
      if(event.key === "a" && event.ctrlKey) {
          exec();
      }
    }
    const exec = () => {
      const dateTable = kintone.app.record.get().record.日付テーブル.value;
	    ...// ここならkintoneオブジェクトが見える
    }
    `;

    document.body.appendChild(script);
  })
}
```

## おわりに

これで使えないことはないので、一旦しばらくはこの運用で様子みしてみます。
もし他に良い方法があれば教えていただきたいです！

Arcはとても使いやすいブラウザだと思うので、ブックマークレットかそれに代わるなにかの実装を心待ちにしています！