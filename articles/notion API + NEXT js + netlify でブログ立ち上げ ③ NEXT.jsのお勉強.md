# notion API + NEXT.js + netlify でブログ立ち上げ ③  NEXT.jsのお勉強

Created: 2022年3月20日 0:52
Updated: 2022年12月12日 13:29
Published_Time: 2022年3月19日 0:00 (JST)
Slug: build-blog-using-notion-api-3
Tags: Netlify, Nextjs, Notion, 個人開発
Published: Yes
URL: https://blog.shaba.dev/posts/build-blog-using-notion-api-3

## 初めに

今回はブログを開発するにあたり、今回初めて触る技術、Next.jsとChakraUIについて学んだことを簡単にまとめようと思います。

## Next.jsのお勉強

基本的な内容をお勉強します。

### SSGとSSRとISR

Nextjsに限らずですが、世の中には3種類のwebアプリの作り方があるようです。
詳しい内容は大量の他サイト様が既に記載されているので割愛しますが、簡単に言うと

1. SSG: StaticSiteGeneration
    1. 予め全ページをビルドしておいて、デプロイ後バックエンドと通信しない（jsでの非同期通信は除く）
    LPなど静的ページの場合に選択する
2. SSR: ServerSideRendering
    1. URLを叩かれて初めてバックエンドに問い合わせて、HTMLを返してもらう
    DBに接続して画面のデータ数が動的に変わる場合に選択する。
3. ISR: IncrementalStaticRegeneration
    1. 基本的にはSSRだが、指定した時間ごとにキャッシュ作成し、キャッシュのデータを表示して裏でバックエンドに問い合わせ、キャッシュを更新する

パフォーマンス面ではSSGが一番だけど、データを取り扱うならSSRになる。
ただ、データ更新頻度が低い（リアルタイム性が重要視されない）場合は、ISRが選択肢に入る。

今回はブログなので、ISRで作れないか試してみました。

### NextjsでのISRのやり方

Nextjsでは、`pages`配下に置かれた`tsx`ファイルがページとして認識されます。
その際、pages配下の相対パスがそのままページのuriになります。
とてもわかりやすい。

pages配下のコンポーネントでは、reactの通常のコンポーネントの他に

- getStaticProps
- getStaticPaths
- getServerSideProps

の3つの特殊メソッドを使用できます。
それぞれ下記の用途で使用します。

- getStaticProps
    - SSG / ISRのページで、ビルド時にバックエンド側で行う処理を記述できる。同時に、コンポーネントに渡す引数を戻り値として設定できる。ISRにする場合は、`revalidate`を戻り値のキーに加える。
- getStaticPaths
    - `[id].tsx`といった動的にファイル名が決定するコンポーネントで使用でき、自身が取りうるパスの配列を戻り値として返すことで、そのuriにアクセスした際に読み込まれるようになる。
- getServerSideProps
    - SSRのページで使用できる。問い合わせがあった際にバックエンド側で行う処理を記載できる。コンポーネントに渡す引数を戻り値として設定できる。
    

また忘れてはならない特徴が、これらのメソッドはサーバーでのみ実行され、**フロントにはメソッド自体が渡されません。**
つまり、DBアクセスキーといったセキュアなものも記載できるといった特徴があります。
めちゃ便利...なんだけど、１ファイルでバックのみ対象の記述とフロントのみ対象の記述が混在するのは慣れるまで混乱しそう...。

今回はSSRは利用しないので、SSG / ISRで利用するgetStaticProps、getStaticPathsを使用します。
下記は記事一覧を取得するためのgetStaticPropsと、記事詳細で記載するgetStaticPathsです。

```tsx
// server側で呼ばれる
export const getStaticProps = async () => {
  const token: string = process.env.NOTION_TOKEN;
  const databaseId: string = process.env.NOTION_BLOG_DATABASE;
  const notion = new Notion(token, databaseId);
  const allPostsData = await notion.getPostList();

  return {
    props: {
      allPostsData: allPostsData,
    },
    revalidate: 1 * 60
  }
}
```

`revalidate: x` と記載しておくと、

「最終レンダリングからx秒後以降にアクセスが有った場合に、キャッシュを返した後裏側でgetStaticPropsを呼び、キャッシュを更新する」
という動きになります。
上記の例では1分で設定しています。

`getStaticProps`の戻り値はそのままコンポーネントの引数になります。

```tsx
import { InferGetStaticPropsType } from 'next'

type Props = InferGetStaticPropsType<typeof getStaticProps>;

export default function Home({allPostsData}: Props){
  return (
    <Layout home>
      <PostList
        data = {allPostsData}
      />
    </Layout>
  )
}
```

`InferGetStaticPropsType`は`getStaticProps`の戻り値の型を明示的に記載しなくても予測してくれる便利ものです。

`[id].tsx`のようにパスも動的に決まる場合は、`getStaticProps`に加えて`getStaticPaths`も同時に利用するようにします。

```tsx
// server側で呼ばれる
export const getStaticPaths: GetStaticPaths = async () => {
  const token: string = process.env.NOTION_TOKEN;
  const databaseId: string = process.env.NOTION_BLOG_DATABASE;
  const notion = new Notion(token, databaseId);
  const allPostsData = await notion.getPostList();
  const pathList = allPostsData.map(post => {
    return {
      params: {id: post.slug}
    }
  });
  return {
    paths: pathList,
    fallback: false
  }
}
```

これらはサーバー側で呼ばれる処理なので、**DBに直接アクセスするよう記載しても外に漏れません。**
APIに渡さなくても良いのは楽ですね。

### Next.jsの環境変数の扱いについて

Next.jsでは環境変数名のprefixによって、サーバ専用の環境変数とフロントも読み込める環境変数の2種類を使い分けれるようになっています。

- `NEXT_PUBLIC_`のprefixがついた環境変数名: フロントでもサーバでも読み込み可能
    - google analyticsなど、外部に漏れても問題ないもので使用する
- 無印の環境変数名: サーバのみ読み込み可能
    - DBアクセスキーなど、セキュアなもので利用する

実際、当ブログの開発では下記のような環境変数をセットしています。

```php
NOTION_TOKEN=secret_xxx
NOTION_VERSION=20xx-xx-xx
NOTION_BLOG_DATABASE=abcdefg

NEXT_PUBLIC_GA_ID=G-XXX
```

上３つはNotionAPIを利用するのに必要なセキュア情報なので、バックでのみ利用するよう無印、google analyticsはフロントで利用するので`NEXT_PUBLIC`prefixつきです。

### API

Next.jsは`pages/`配下に`api/`フォルダを用意することが出来て、その中にバックエンドのAPIを用意することができます。
パスは通常の`pages/`と同様に、フォルダ構成がそのままuriとなります。

今回はここらへんで。
次回はフロントで利用するフレームワークを選定してお勉強します。