# notion API + NEXT.js + netlify でブログ立ち上げ ①

Created: 2022年1月18日 19:09
Updated: 2022年12月12日 13:29
Published_Time: 2022年3月1日
Slug: build-blog-using-notion-api-1
Tags: Netlify, Nextjs, Notion, 個人開発
Published: Yes
URL: https://blog.shaba.dev/posts/build-blog-using-notion-api-1

## はじめに

これまで何度もブログを立ち上げては、10記事くらい書いたあたりで頓挫させてきました。
三日坊主気質なのも理由にありますが、一番の理由は**記事が書きにくい**ことだったかなと。

例えばWordPressだと記事を書くためにわざわざシステムにログインして、ブラウザの中で書かなければならず。
環境によっては通信が重くてもっさりしたり、独特のショートカットを使用する必要があったり、よくiPadで記事を書いていましたが、スクロールしたいだけなのに変なところタップして記事が消えてしまったり...

*(:3 」∠)*

そんななかnotionを見つけ、（ほぼ）markdownで書けて、環境を選ばない。

しかもDB不要で、常に同期されていて、APIまであるときた。

素晴らしい

先駆者たちがnotion API を利用してブログやいろんなwebアプリを作っているのを見つけ、

自分もnotion APIを使ってブログを作ってみよう、と思ったので、制作工程を晒していこうと思います。

## デプロイ先選定

今回デプロイ先は**netlify**にします。

理由: 

- 将来的に広告とかマネタイズできる仕組みを入れたいな、と思ったので（nextjsだけど）vercelは除外
- github pagesは動的レンダリングのやり方が分からず...除外

netlifyでも動的レンダリングできるしAPIも作れるので問題なしでした。

（NEXTjsだとAPI routeなるもので自身でサーバーサイドAPIが立てれるとのことで、結局使用していませんが笑）

ていうかNEXTjs凄い...frontendのフレームワークってこんななのか

## サンプルプログラムをnetlifyにデプロイ

### サンプルプログラム

プロジェクトを作成してサンプルアプリを公式から落としてきます。

```bash
npx create-next-app blog --example "https://github.com/vercel/next-learn/tree/master/basics/typescript-final"
```

ビルドして立ち上げ。

```bash
cd blog && yarn build && yarn start
```

[http://localhost:3000/](http://localhost:3000/) にアクセス

![初期画面](%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88_2022-01-27_17.19.04.png)

初期画面

すでにブログ感ある...

### netlifyにデプロイ

一旦このままでnetlifyにデプロイします。

`netlify.toml`をproject rootに作成

```bash
[build]
  base = ""
  publish = ".next"
  command = "yarn build"
[[plugins]]
  package = "@netlify/plugin-nextjs"
[[plugins]]
  package = "netlify-plugin-cache-nextjs"
```

<aside>
💡 いろんな記事を読みましたが明記されていないことに、`publish = “.next”`を指定する必要がある。デプロイ時にnext on netlifyが吐くエラーに書いてある。

</aside>

### git hubにデプロイ

```bash
git push
```

### netlifyで設定

netlify管理画面から、
`Team orverview → Sites → Add new site → import an existing project`
の順に遷移していき、画面に沿ってGitHubの該当リポジトリを選択

`Site settings → Build&Deploy → Environment`
で環境設定

- NOTION_BLOG_DATABASE
- NOTION_TOKEN
- NOTION_VERSION

`Site settings → Domain Management → Custom domains`
で自身のドメインを登録
時間がかかるので、しばらくした後に改めて確認。

当記事ではここまで。