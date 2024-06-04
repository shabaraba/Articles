# NotionWebClipperで保存した記事をブログに公開した

Created: 2023年2月8日 14:42
Updated: 2023年2月12日 14:17
Published_Time: 2023年2月12日
Slug: publish-my-links
Tags: Notion, 個人開発
Published: Yes
URL: https://blog.shaba.dev/posts/publish-my-links

## はじめに

こんなページを作りました。

![スクリーンショット 2023-02-08 15.05.25.png](%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588_2023-02-08_15.05.25.png)

## なんのページ？

Notion関連のchrome拡張機能で、Notion Web Clipperがあります。

[Notion Web Clipper](https://chrome.google.com/webstore/detail/notion-web-clipper/knheggckgoiihginacbkhaalnibhilkk?hl=ja)

気に入ったページをNotionに保存できるというもので、ブックマークより使い勝手が良いのでよく使っています。
特に、興味のある技術系だったり個人開発に関わることだったり、勉強用として保存することが多いです。

**「…これらを公開したら僕のこと（こんな事考えてる人なんだとか）なんとなくわかってもらえるんじゃね？」**

と思ったので、公開する機能をブログに付けました。

## 実装方法

ほとんどは記事一覧ページの機能で満足していました。

ただ

- Notionに保存されるマイリンクとブログの記事で持つプロパティが異なること
- いくら似ているといってもブログ記事と外部リンクとで性質が違うこと

が考えられたので、serviceなどは共有化せず、mylink用のmoduleを改めて作ることにしました。

細かい内容はこちらをご覧ください。

[Issue39 mylink page by shabaraba · Pull Request #42 · shabaraba/Notiography](https://github.com/shabaraba/Notiography/pull/42)

（上記PL内でlayoutファイルの分割も変更してるんよな…本当はPL分けるべきだったんだけど入れちゃった…）

## 今後の予定は？

NotionWebClipperのすごいところが、リンクだけじゃなくて外部ページ様の内容がNotionブロックに置き換わって保存されるところです（=魚拓が取れる）。

外部ページ様が何らかの理由で閉じられたとしても、手元では内容を確認できる状態ですね。

なので、今はこのmylinkページはただの外部リンクですが、その先が404だったら内部で生成してそちらに遷移させるようにしたいと思っています。