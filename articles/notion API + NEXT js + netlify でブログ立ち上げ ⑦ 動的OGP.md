# notion API + NEXT.js + netlify でブログ立ち上げ ⑦ 動的OGP

Created: 2022年3月15日 13:55
Updated: 2024年5月2日 21:19
Published_Time: 2022年3月29日
Slug: build-blog-using-notion-api-7
Tags: Netlify, Nextjs, Notion, 個人開発
Published: Yes
URL: https://blog.shaba.dev/posts/build-blog-using-notion-api-7

[【動的OGP】Next.js + TypeScript + Vercelデプロイで動的OGPを実現する - Qiita](https://qiita.com/yuikoito/items/619120c592d99f9d3053#ogp%E7%94%A8%E3%81%AE%E7%94%BB%E5%83%8F%E3%82%92%E4%BD%9C%E6%88%90%E3%81%99%E3%82%8B)

[https://github.com/vercel/og-image](https://github.com/vercel/og-image)

codeが凄いシンプルでカスタマイズしやすい

まずは開発するためにCONTRIBUTING.mdを参照

> Clone this repo with `git clone <https://github.com/vercel/og-image`>
Change directory with `cd og-image`
Run `yarn` or `npm install` to install all dependencies
Run locally with `vercel dev` and visit [localhost:3000](http://localhost:3000/) (if nothing happens, run `npm install -g vercel`)
If necessary, edit the `exePath` in [options.ts](https://github.com/vercel/og-image/blob/main/api/_lib/options.ts) to point to your local Chrome executable
> 

シンプルですね。
この通りに実行してみます。
vercelは今回はローカルインストールにしようと思います。

```bash
yarn install
yarn install --dev vercel
yarn vercel dev
```

`yarn install --dev vercel` このタイミングで、vercelにcliログインするよう案内されるのでそのとおりに。

```bash
$ yarn vercel dev
yarn run v1.22.15
Vercel CLI 24.0.0
> Creating initial build
Running "yarn run build"
warning ../../../../package.json: No license field
$ tsc -p api/tsconfig.json && tsc -p web/tsconfig.json
> Success! Build completed
> Ready! Available at http://localhost:3000
```

http://localhost:3000にアクセス

![スクリーンショット 2022-03-16 22.42.16.png](%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88_2022-03-16_22.42.16.png)

画像生成用urlを扱うだけじゃなくて、そのurlを作るための画面も用意されてる、便利ですね。

## カスタマイズ

ここからカスタマイズしていきます。
おおよそ揃ってるのであまりすることもないのですが、下記対応を行います。

1. 背景に画像を入れれるようにする
2. 日本語入力ができるようにする