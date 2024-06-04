# notion API + NEXT.js + netlify でブログ立ち上げ ⑤ 画面作成

Created: 2022年2月13日 7:23
Updated: 2024年5月2日 21:19
Published_Time: 2022年3月27日
Slug: build-blog-using-notion-api-5
Tags: Netlify, Nextjs, Notion, 個人開発
Published: Yes
URL: https://blog.shaba.dev/posts/build-blog-using-notion-api-5

内容としては

1.  NotionSDKを使ってdatabases queryを叩けるメソッドを作成
2. 記事一覧に必要なデータを持つ型を作成
3. interface作成

こんな感じで記載していきたいと思います。

## おtypographyページを作成して確認。

ブロックごとにcomponent作成

Blocksコンポーネントとしてループ

## お勉強

### フォルダ内一括import

`/components/posts/blocks/`フォルダに、ブロックのコンポーネントを作成することにしました。
ブロックの種類は結構あるので、利用する箇所で毎回全てを`import`するのは結構大変ですし無駄です。
pythonの`__init__.py`みたく、フォルダ内のモジュールを一括で`import`できないか探していましたが、やりかたありました。

フォルダの中に`index.ts`を用意

index.ts

```tsx
export * from './Paragraph'
export * from './Heading1'
export * from './Heading2'
export * from './Heading3'
export * from './Callout'
export * from './Code'
export * from './Image'
```

そして使用したい箇所で

```tsx
import * as NotionBlock from '../entities/notion/blocks'
```

とすると、`index.ts`で`export`したコンポーネントが下記の要領で利用できるようになります。

```tsx
NotionBlock.Paragraph
```

### 戻り値ムズい問題

Notion APIのレスポンスは結構詳細な情報まで返してくれるのですが、その分ネストがめちゃくちゃ深いです。
また記事ごとに使用ブロックの種類が違うので、レスポンスの型が一意に決まりません。
結構複雑なので、レスポンスの中身は一旦anyで受け付けて、`type`を見て扱いたいentityに詰め直すようにしました。

```tsx
import { Client } from "@notionhq/client"
import type {
  QueryDatabaseResponse,
  ListBlockChildrenResponse,
} from '@notionhq/client/build/src/api-endpoints.d'

import type {
  NotionPostHead,
  NotionTag,
  NotionIcon,
} from '../entities/notion_entities';

import * as NotionBlock from '../entities/notion/blocks';
import * as NotionBlockInterfaces from '../interfaces/NotionApiResponses';

export default class Notion {
	...
  static createBlockList(response: ListBlockChildrenResponse) {
    let blocks: Array<NotionBlock.Block> = [];
    response.results.map((item: any) => {
      let itemType = item.type;
      switch (itemType) {
        case 'paragraph':
          let paragraph: NotionBlockInterfaces.IParagraphBlock = item;
          blocks.push(new NotionBlock.Paragraph(paragraph));
          break;
        case 'heading_1':
          let heading1: NotionBlockInterfaces.IHeading1Block = item;
          blocks.push(new NotionBlock.Heading1(heading1));
          break;
        case 'heading_2':
          let heading2: NotionBlockInterfaces.IHeading2Block = item;
          blocks.push(new NotionBlock.Heading2(heading2));
          break;
        case 'heading_3':
          let heading3: NotionBlockInterfaces.IHeading3Block = item;
          blocks.push(new NotionBlock.Heading3(heading3));
          break;
        case 'callout':
          let callout: NotionBlockInterfaces.ICalloutBlock = item;
          blocks.push(new NotionBlock.Callout(callout));
          break;
        case 'code':
          let code: NotionBlockInterfaces.ICodeBlock = item;
          blocks.push(new NotionBlock.Code(code));
          break;
        case 'image':
          let image: NotionBlockInterfaces.IImageBlock = item;
          blocks.push(new NotionBlock.Image(image));
          break;
        default:
          break;
      }
    })
    return blocks;
  }
}
```

こんな感じでメソッドを作って

```tsx
import Layout from '../../components/layout';
import Block from '../../components/posts/Block';
import { useRouter } from 'next/router';
import Head from 'next/head';
import {v4 as uuidv4} from 'uuid';

import * as NotionBlock from '../../entities/notion/blocks';

import Notion from '../../lib/notions'
import { InferGetStaticPropsType, GetStaticPaths } from 'next'

type Props = InferGetStaticPropsType<typeof getStaticProps>;

export default function Post({postBlockJson}: Props) {
  const postBlockList = Notion.createBlockList(postBlockJson);

  return (
		// ...
  )
}
```

`[id].tsx`側で呼んでいます。

### paragraphの改行とinlineの両立問題

paragraphについて

めっちゃ頑張った

改行と横並びの両立

```tsx
import React from "react"
import { Text, Code } from '@chakra-ui/react'
import type { Paragraph as ParagraphEntity, } from '../../../entities/notion/blocks';

import _ from 'lodash'

type Props = {
  enable: boolean,
  color: string,
  backgroundColor: string,
  children?: React.ReactNode
}

export function Paragraph({entity}: {entity: ParagraphEntity}) {
  if (entity.texts.length === 0) return <br />

  const Bold: React.VFC<Props> = ({enable, color, backgroundColor, children}: Props) => {
    if (enable) return <Text as='strong' fontWeight='bold' color={color} backgroundColor={backgroundColor} >{children}</Text>
    return <>{children}</>
  }

  const Italic: React.VFC<Props> = ({enable, color, backgroundColor, children}: Props) => {
    if (enable) return <Text as='i' color={color} backgroundColor={backgroundColor} >{children}</Text>
    return <>{children}</>
  }

  const Underline: React.VFC<Props> = ({enable, color, backgroundColor, children}: Props) => {
    if (enable) return <Text as='u' color={color} backgroundColor={backgroundColor} >{children}</Text>
    return <>{children}</>
  }

  const Strikethrough: React.VFC<Props> = ({enable, color, backgroundColor, children}: Props) => {
    if (enable) return <Text as='s' color={color} backgroundColor={backgroundColor} >{children}</Text>
    return <>{children}</>
  }

  const CodeText: React.VFC<Props> = ({enable, color, backgroundColor, children}: Props) => {
    if (enable) return <Code color={color} backgroundColor={backgroundColor} >{children}</Code>
    return <>{children}</>
  }

  const RichText = ({text}: any) => {
    if (text.content == null) return <br />

    let content = text.content ?? ''
    console.log(content)
    const color = text.annotations.color + '.500'
    const backgroundColor = (text.annotations.color + '.500').split('_background').length > 1 ? (text.annotations.color + '.500').split('_background')[0] : null
    return (
      <Bold
        enable={text.annotations.bold}
        color={color}
        backgroundColor={backgroundColor}
      >
        <Italic 
          enable={text.annotations.italic}
          color={color}
          backgroundColor={backgroundColor}
        >
          <Underline 
            enable={text.annotations.underline}
            color={color}
            backgroundColor={backgroundColor}
          >
            <Strikethrough 
              enable={text.annotations.strikethrough}
              color={color}
              backgroundColor={backgroundColor}
            >
              <CodeText 
                enable={text.annotations.code}
                color={color}
                backgroundColor={backgroundColor}
              >
                {content}
              </CodeText>
            </Strikethrough>
          </Underline>
        </Italic>
      </Bold>
    )
  }
  
  const getLineEntities = (entity: ParagraphEntity) => {
    let lineEntities = []
    entity.texts.forEach(text => {
      let eachLine = text.content.split('\n')
      eachLine.forEach(line => {
        let lineEntity = _.cloneDeep(text)
        lineEntity.content = line
        lineEntities.push(lineEntity)
        let endLineEntity = _.cloneDeep(text)
        endLineEntity.content = null
        lineEntities.push(endLineEntity)
      })
      lineEntities.pop()
    })
    return lineEntities
  }

  return (
    <Text>
      {getLineEntities(entity).map(text => <RichText text={text} /> )}
    </Text>
  )
}
```

childrenはとくべつなprops

jsdomをつかってbuild出来ない問題

canvasが無いから

めちゃくちゃ重かったので入れたくない

```jsx
module.exports = {
  target: 'serverless',
  webpack: (config, { buildId, dev, isServer, defaultLoaders, webpack }) => {
    if (isServer) {
      config.plugins.push(new webpack.IgnorePlugin(
        {
          resourceRegExp: /canvas/,
          contextRegExp: /jsdom$/,
        }))
    }
    return config
  }
}
```