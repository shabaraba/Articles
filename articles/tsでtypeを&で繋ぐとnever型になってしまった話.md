# tsでtypeを&で繋ぐとnever型になってしまった話

Created: 2023年1月9日 20:09
Updated: 2023年1月13日 9:54
Published_Time: 2023年1月31日
Slug: ts-type-param-cannot-override
Tags: TypeScript, 個人開発
Published: Yes
URL: https://blog.shaba.dev/posts/ts-type-param-cannot-override

## はじめに

このブログはNext.jsとTypeScriptで開発していますが、当方TypeScriptはまだまだ初学者で、
まだまだ作法や細かい仕様を理解しておらず、ぶつかっては覚えて、を繰り返しています。

今回もその躓きの一つで、メモっておきたいのでこの記事を書くことにします。

## 何をしていたか

Notion APIからのレスポンス用のtypeを用意しています。
Page情報からBlockの情報を見て、各ブロック用のentityに[情報を詰める処理](https://github.com/shabaraba/Notiography/blob/b5bcd9249b073c4dad8c8481b862cc0b7b70b080/src/application/modules/post/objects/factories/BlockFactory.ts#L20)があります。

正しいブロックのtypeにキャストしてからentityに詰めるので、[結構厳格にtypeを作っている](https://github.com/shabaraba/Notiography/blob/b5bcd9249b073c4dad8c8481b862cc0b7b70b080/src/core/types/NotionApiResponses.ts)のですが、

共通して持つ情報も多いので、アンパサンドを用いて型をマージして表現しています。

例えばパラグラフ用のtypeだとこんな感じ。

```tsx
type _IBlock = {
    type: 'block';
    object: 'block';
    id: string;
    created_time: string;
    last_edited_time: string;
    has_children: boolean;
    archived: boolean;
}

export type IText = {
	...
}
type _IParagraph = { text: IText[], children: IRetrieveBlockChildrenResponse }
export type IParagraphBlock = _IBlock & { type: "paragraph", paragraph: _IParagraph }
```

（名称にIが入っているのは、もともとinterfaceで作っていた名残です）

## 何がおきたか

vscodeでコーディングしていると、IParagraph型の変数で赤波線が。
変数をホバーすると

```tsx
type IParagraphBlock = never
```

うん？IParagraphBlockは`_IBlock & { type: "paragraph", paragraph: _IParagraph }`型だが？？？

## 原因と解決策

おそらくこちらの記事の内容が原因。

[TypeScript type を & でいい感じにマージしたい。 - かもメモ](https://chaika.hatenablog.com/entry/2020/02/22/083000)

typeの場合はinterfaceと違って同一プロパティのものは上書きされないのね。
今回の場合`_IBlock.type`と`_IParagraph.type`がコンフリクトを起こした結果、never型と判定されてしまったんだと思います。

今回は_IBlockは直接は使用されないので、`_IBlock.type`を削除することで解決としました。