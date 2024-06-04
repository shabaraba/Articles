# notion API + NEXT.js + netlify でブログ立ち上げ ⑥ SEO関連の設定

Created: 2022年3月11日 0:35
Updated: 2024年5月2日 21:27
Published_Time: 2022年3月28日
Slug: build-blog-using-notion-api-6
Tags: Netlify, Nextjs, Notion, 個人開発
Published: Yes
URL: https://blog.shaba.dev/posts/build-blog-using-notion-api-6

[Next.js で動的にXMLサイトマップ（sitemap.xml）を生成する方法まとめ｜Miningoo](https://miningoo.com/2327/)

```bash
yarn build && yarn postbuild
```

```bash
$ yarn postbuild
yarn run v1.22.17
warning ../../../../package.json: No license field
$ next-sitemap
Loaded env from /Users/xxx/yyy/.env
✅ [next-sitemap] Generated index sitemap and 1 sitemap(s)
○ [https://blog.from-garage.com/sitemap.xml](https://blog.from-garage.com/sitemap.xml) (index)
○ [https://blog.from-garage.com/sitemap-0.xml](https://blog.from-garage.com/sitemap-0.xml)
✨  Done in 0.48s.
```

## rssについて

下記サイトを参考に実装

[Next.js に feed を導入して RSS と Atom のフィードを生成しよう | fwywd（フュード）powered by キカガク](https://fwywd.com/tech/next-feed-rss-atom)

したんだけど... buildした後確認したら500エラーになる
内容確認したらどうやらbuild前後でメソッドが存在しなくなっているよう
調べると下記issueがあった

[show "res.writeHead is not a function" after next build · Issue #837 · isaachinman/next-i18next](https://github.com/isaachinman/next-i18next/issues/837)

今のnextjsでも治ってないかも

[こちら](https://tech-wiki.online/jp/how-to-list-object-methods-javascript.html)を参考にcontext.resのプロパティを確認した

```tsx
const getMethods = (obj) => {
    let properties = new Set()
    let currentObj = obj
    do {
      Object.getOwnPropertyNames(currentObj).map(item => properties.add(item))
    } while ((currentObj = Object.getPrototypeOf(currentObj)))
    return properties
  }
  console.log(getMethods(context.res))
```

結果
`yarn dev`時

```tsx
Set(101) {
  'domain',
  '_events',
  '_eventsCount',
  '_maxListeners',
  'outputData',
  'outputSize',
  'writable',
  'destroyed',
  '_last',
  'chunkedEncoding',
  'shouldKeepAlive',
  'maxRequestsOnConnectionReached',
  '_defaultKeepAlive',
  'useChunkedEncodingByDefault',
  'sendDate',
  '_removedConnection',
  '_removedContLen',
  '_removedTE',
  '_contentLength',
  '_hasBody',
  '_trailer',
  'finished',
  '_headerSent',
  '_closed',
  'socket',
  '_header',
  '_keepAliveTimeout',
  '_onPendingData',
  'req',
  '_sent100',
  '_expect_continue',
  'statusCode',
  'constructor',
  '_finish',
  'statusMessage',
  'assignSocket',
  'detachSocket',
  'writeContinue',
  'writeProcessing',
  '_implicitHeader',
  'writeHead',
  'writeHeader',
  'writableFinished',
  'writableObjectMode',
  'writableLength',
  'writableHighWaterMark',
  'writableCorked',
  '_headers',
  'connection',
  '_headerNames',
  '_renderHeaders',
  'cork',
  'uncork',
  'setTimeout',
  'destroy',
  '_send',
  '_writeRaw',
  '_storeHeader',
  'setHeader',
  'getHeader',
  'getHeaderNames',
  'getRawHeaderNames',
  'getHeaders',
  'hasHeader',
  'removeHeader',
  'headersSent',
  'writableEnded',
  'writableNeedDrain',
  'write',
  'addTrailers',
  'end',
  '_flush',
  '_flushOutput',
  'flushHeaders',
  'pipe',
  'setMaxListeners',
  'getMaxListeners',
  'emit',
  'addListener',
  'on',
  'prependListener',
  'once',
  'prependOnceListener',
  'removeListener',
  'off',
  'removeAllListeners',
  'listeners',
  'rawListeners',
  'listenerCount',
  'eventNames',
  '__defineGetter__',
  '__defineSetter__',
  'hasOwnProperty',
  '__lookupGetter__',
  '__lookupSetter__',
  'isPrototypeOf',
  'propertyIsEnumerable',
  'toString',
  'valueOf',
  '__proto__',
  'toLocaleString'
}
```

`yarn build && yarn start` 時

```tsx
Set(27) {
  'destination',
  '_res',
  'textBody',
  'constructor',
  'originalResponse',
  'sent',
  'statusCode',
  'statusMessage',
  'setHeader',
  'getHeaderValues',
  'hasHeader',
  'getHeader',
  'appendHeader',
  'body',
  'send',
  'redirect',
  '__defineGetter__',
  '__defineSetter__',
  'hasOwnProperty',
  '__lookupGetter__',
  '__lookupSetter__',
  'isPrototypeOf',
  'propertyIsEnumerable',
  'toString',
  'valueOf',
  '__proto__',
  'toLocaleString'
}
```

buildする前まであったメソッド`write`とか`end`がない...
代わりになりそうな`body`とか`send`はtypescript時点ではないので、作れないね...

nextjsを10.0.0に下げると治ったという報告も先のissueにはあったけど、あまり下げたくないし
とりあえず`getStaticProps`で静的に生成するようにして、ブログ自体を定期的にビルドするようにしよう（だったら他のページも静的でいいのでは...要検討）

### 外部から取得できるようにする

rss用のページを用意しても、外部からどのurlがrss用なのか検知させられないと使用してもらえない。
調べると、`link`タグにrss用のurlを記載することで認識してもらえるようになるとのこと。

next-seoを使う想定なので、`next-seo.config.js`で下記内容を追加

```tsx
export default {
	...,
+	additionalLinkTags: [
+	    {
+	      rel: 'alternate',
+	      type: 'application/rss+xml',
+	      title: "RSS2.0",
+	      href: 'https://blog.from-garage.com/rss/feed.xml',
+	    },
+	    {
+	      rel: 'alternate',
+	      type: 'application/atom+xml',
+	      title: "Atom",
+	      href: 'https://blog.from-garage.com/rss/atom.xml',
+	    },
+	  ],
	...
}
```

netlifyにpushして、preview状態でも良いので下記ツールサイトで確認

[RSSフィード取得・検出ツール - BeRSS.com](https://berss.com/feed/)

![スクリーンショット 2022-03-14 9.36.03.png](%E3%82%B9%E3%82%AF%E3%83%AA%E3%83%BC%E3%83%B3%E3%82%B7%E3%83%A7%E3%83%83%E3%83%88_2022-03-14_9.36.03.png)

うまく行った