# codespacesでastroを使うときは開発環境の立ち上げに—hostをつけよう

Created: 2023年2月13日 8:46
Updated: 2023年2月13日 21:58
Published_Time: 2023年2月13日
Slug: astro-host-in-codespaces
Tags: Astro, Codespaces
Published: Yes
URL: https://blog.shaba.dev/posts/astro-host-in-codespaces

## はじめに

Codespaces愛用しています、しゃばです。

ポートフォリオをいよいよ作りたく、どうせなら最近話題のastro+svelteを触ってみたいと思っています。

ただのastro導入段階で少し躓いたのでメモ。

## 何がおきたか

`npm run dev`でastroを起動したかったのですが、いつまで立ってもcodespacesさんがポートフォワードしてくれず。
自分でポートを開いてもつないでくれない状態でした。

## どうしたか

`npm run dev`実行時にもメッセージが出ていましたが、デフォルトだと次のようなメッセージが出ます

```bash
$ npm run dev

> portfolio@0.0.1 dev
> astro dev

  🚀  astro  v2.0.10 started in 64ms
  
  ┃ Local    http://localhost:3000/
  ┃ Network  use --host to expose
```

なるほど外部からのアクセスなので`—host`をつけて実行してやる必要がありそうです。

```bash
$ npm run dev -- --host

> portfolio@0.0.1 dev
> astro dev --host

  🚀  astro  v2.0.10 started in 37ms
  
  ┃ Local    http://localhost:3000/
  ┃ Network  http://172.16.5.4:3000/
  
11:51:59 PM [content] Watching src/content/ for changes
11:52:00 PM [astro] update /.astro/types.d.ts
11:52:00 PM [content] Types generated
```

↓

![スクリーンショット 2023-02-13 8.52.07.png](%25E3%2582%25B9%25E3%2582%25AF%25E3%2583%25AA%25E3%2583%25BC%25E3%2583%25B3%25E3%2582%25B7%25E3%2583%25A7%25E3%2583%2583%25E3%2583%2588_2023-02-13_8.52.07.png)

でました、やったね。

<aside>
💡 `—host` は、`npm run dev`実行で呼ばれる`astro dev`につけてあげる必要があります。`npm run dev —host` だと`npm run dev`コマンドに対してオプションが付くことになるので、ダッシュ2つ`—-`をつけることで、以降のオプションを`npm run dev`の実行先のコマンドへ渡すことが出来ます。

</aside>

## 追加で

毎回`— —host`をつけるのは面倒なので、`packages.json`を書き換えます。

```json
{
	"name": "portfolio",
	"type": "module",
	"version": "0.0.1",
	"scripts": {
		"dev": "astro dev --host", // ←ここ
		"start": "astro dev --host", // ←ここ
		"build": "astro build",
		"preview": "astro preview",
		"astro": "astro"
	},
	"dependencies": {
		"astro": "^2.0.10"
	}
}
```