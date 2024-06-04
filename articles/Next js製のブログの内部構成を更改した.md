# Next.js製のブログの内部構成を更改した

Created: 2023年1月11日 17:20
Updated: 2023年1月31日 0:01
Published_Time: 2023年1月12日
Slug: upgrade-directory-structure
Tags: Nextjs, Notion, 個人開発
Published: Yes
URL: https://blog.shaba.dev/posts/upgrade-directory-structure

## はじめに

このブログは[こちら](https://github.com/shabaraba/Notiography)のコードで作成しています。

[GitHub - shabaraba/Notiography: a blog framework using notion api](https://github.com/shabaraba/Notiography)

作成した当時はNext.jsもTypeScriptも（ほぼ）初めて触ったので、かなり適当な設計になっていたと思います。

運用を始めてしばらくは当リポジトリはノータッチだったのですが、Next.jsをはじめとするライブラリが古くなってきていたので、最新に更新するがてら、ディレクトリ構成や設計を再考することしてみました。

まだやっつけなところも残っていますが、一旦区切って記録したいと思います。

## 更改結果

### root直下

rootの構成はこのようにしました。

```bash
.next/
public/
src/
.files…
```

今まで全てのディレクトリがroot直下にいて、流石にわかりにくすぎたので、`src/`に切り出しました。
`public/`は、ビルド時に生成される`.next/`で使用する（はず）なので、`src/`からは外しました。

### src内

全部は書き切りませんが、ディレクトリだけでも。

```bash
src/
	application/
		modules/
			post/
				logics/
					PostLogic
					PostLogicNotionImpl
				objects/
					dtos/
					dxos/
					entities/
					factories/
				repositories/
				services/
			tag/
				…
		usecases/
	core/
		dxo/
		entities/
		types/
		lib/
	pages/
	styles/
	components/
		units/
		modules/
		patterns/
```

バックエンドは`application/`に閉じ込めて、フロントエンドは`components/`に閉じ込めました。
両者で共通して使用するものは`core/`に入れています。

`styles/`は…何を入れているのか忘れました←

フロントはChakra-UIを使用しているので、スタイルを当てる必要はほぼないはずなので、特別必要な`tsx`に直書きして`styles/`ディレクトリは将来的に消そうと思っています。

Next.jsはapiも作れるので不要かもですが、Netlify functionsを作るときは`application/`直下に入れようと思っています。

## 工夫点

### バックエンド側はクリーンアーキテクチャぽく

下記のような構成にしてみました。

- usecases: 外部からバックエンドを叩く際に呼ばれる。複数のserviceを使ってレスポンスを作成して返す。logicクラス以下は直接呼ばない。
- services: 機能単位でまとめたクラス。usecaseもしくはserviceからのみ呼ばれるが、極力他のserviceからは呼ばれないようにしたい。logicから渡ってきたentityを組み合わせて、dtoにして呼び出し元に返す。
- logics: ビジネスロジックを記載するクラス。repositoryを呼べる唯一のクラス。service以上のクラスは呼ばない。
- repositories: DBや外部サービスからデータをやり取りすることだけに関心を持つクラス。entityを造ってlogicに渡す。

### pagesからimportできるバックエンドのコードは usecases/ のみ

Next.jsを触っていて煩わしく思っていたのが、`pages/`内の特殊メソッド（`getStaticProps`とか）のせいで、フロントで使うためのimportとバックで使うためのimportが混在してしまうことでした。

気をつけていれば問題ないのでしょうが、たまーに更新するようなプロジェクト（=コレ）だと、バック用のmoduleを間違えてフロントで使ってしまい、

`fsをフロントで使うな`

みたいなエラーが出て

え、使ってないけど？（怒

みたいなやり取りが発生してロスが多かったです。

なので、今回バックエンド側をクリーンアーキテクチャもどきな構成に作り変えたので、`getStaticProps`も専用のusecaseに作ることにしました。

```tsx
export default () => {
	...
}

export const getStaticProps = async () => ArticleListPageUsecase.getStaticProps();

```

```tsx
export class IndexUsecase {
	public static getStaticProps() {
		// use some service class...
	}
}
```

本当は、未だpagesが`application/`に依存してしまうので、interfaceを`core/`に用意して、`application/`内でimplementを作成することで完全に分離させるのが良いと思っています。
今後の課題ですかね。

ただこうすることで

- 上述のミスは防げる
- `pages/`がページを作ることに集中できる
- `usecases/`も本来の用途っぽく使える

ので、こんな運用して良いのかわかっていませんが個人的には気に入っています。

### logicクラスはいつでも分離できるように

今はNotionをCMS代わりに使っていますが、サービスの流行り廃りはとても激しいので、いつより良い代替品が来ても良いように、logicクラスだけはいつでも切り離せるようにinterfaceを使用して作成しています。
（今は直にnewしているだけなので、DIコンテナ作りたい感じです）

### componentsの切り方

フロントの知識が乏しいので良い設計なのかわかっていないのですが、更改前は本当に散らかっていて何がなんやらだったので、以下の3つに分類することにしました。

- `units/`：単独では使えない、もしくはあらゆる箇所で使い回されることを想定したコンポーネント
- `modules/`：単独もしくはunitsを組み合わせて作られた、明確に意味を持ったコンポーネント
- `patterns/`：modulesを組み合わせて作られた、ページ上で一塊だと認識できるくらいのスケールを持ったコンポーネント

今後の機能追加の中でどう感じるか実験的な意味合いもあります。

### クラス間のオブジェクトのやり取り

Next.jsを使っているので冗長だとは思っているのですが、なるべくデータのやり取りを厳格にできるよう

- repositoryが返す値は必ずentity
    - なのでentityを作成するfactoryクラスが居る
- logicが返す値は必ずentity
- serviceが返す値は必ずdto
    - なのでentity→dtoに変換するdxoクラスが居る
- usecaseが返す値は必ずjson
    - なのでdto→レスポンスに必要な情報のみを抽出するdxoクラスが居る
- フロントではただのobjectは扱わず、typeを使って型安全を心がける

ようにしました。

正直かなり冗長だった…と思うけど、大規模システムだとこんな設計のほうが安全のはずなので、良い練習にはなりました。
まだ道半ばでpushしているので、時間があったら引き続きリファクタしていきたいです。

## 今後

### テストコード書きたい

実はまだないんです、テストコード。
tsで書いたことがないので、経験も意味も込めて実装したいです。

### 機能追加した

- ページャ
- タグでの絞り込み
- 関連する記事の提示

は最低でも必要だと思っているので、早い段階で造っていきたいと思っています！

## 終わりに

というわけで、自身の備忘録も込めた投稿でした。

コレで良いのかどうか意見も聞きたかったのですが、コメント機能もないので、もし突っ込んで頂ける方が居たらtwitterかgithubのissueでお願いします！！！