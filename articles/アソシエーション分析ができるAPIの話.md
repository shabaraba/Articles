# アソシエーション分析ができるAPIの話

Created: 2023年1月27日 16:16
Updated: 2023年8月29日 14:51
Published_Time: 2023年1月29日
Slug: create-association-analysis-api
Tags: Aws, Python, 個人開発
Published: Yes
URL: https://blog.shaba.dev/posts/create-association-analysis-api

詳細はZennに移行しました。

[アソシエーション分析ができるAPIをlambdaとAPI-Gatewayで作った](https://zenn.dev/shabaraba/articles/c581b6126b1a23)

todo: なにか裏話

---

- 以下アーカイブ
    
    ## はじめに
    
    今は訳合ってサービスを閉じてしまっているのですが、スマレジプラットフォームアプリでlook-into-basketsというアプリを公開していました。
    
    [GitHub - shabaraba/look-into-baskets](https://github.com/shabaraba/look-into-baskets)
    
    取引履歴から期間を指定してアソシエーション分析結果を表示するアプリだったのですが、
    
    - フロントとバックが密結合だったので、分割したかった
    - 「アソシエーション分析」というもの自体が割と使い勝手がよく、いろんなことに応用が効きそう
        - このブログのタグを分析すれば、僕の興味とかスキルセットとか見えるんじゃないか、とか
    
    という理由から、**「じゃAPIにするか」**ということで、こんなリポジトリを作成しました。
    
    [GitHub - shabaraba/AssociationAnalysisApi](https://github.com/shabaraba/AssociationAnalysisApi)
    
    ## 実現方法
    
    分析系のライブラリが揃っていることから、もともとPythonを使ってresponderというFWで実装していたので、
    コードを流用すべく、今回もPythonで実装したかったです。
    
    とすると現状あるサービスだとlambdaくらいしか選択肢がなさそうだったので、AWS上に構築することにしました。
    
    ### 使用技術
    
    - aws-cli
    - aws-gateway
    - lambda (container image)
        - Python3.7
        - mlxtendライブラリ
    
    ### 分析機能の実装
    
    分析周りはもともとlook-into-basketsで使っていたものと、下記のqiita記事を参考に作っています。
    
    [Pythonでアソシエーション分析 - Qiita](https://qiita.com/makaishi2/items/c5f06f844cdb8454b6c3)
    
    内容は上記記事がすべて説明してくれているので、上記記事と[リポジトリ](https://github.com/shabaraba/AssociationAnalysisApi)を見てください。
    
    唯一、分析結果は「AならばB」という感じで得られるのですが、AもBも本来はリスト形式（厳密にはtuple）です。
    
    が、今考えている用途では扱いにくいので、AもBも単数のもののみ抽出するようにしています。
    
    ### AWS-CLI周りの実装
    
    はじめは至って普通のlambdaを使う予定だったので、DeveloperIO様などを参考に作っていました。
    
    が、分析系なのでライブラリが重量級で、layerを使ってもlambdaの容量上限（256Mb）に収めることができませんでした。
    
    ダメかと諦めつつある時に、docker imageでlambdaをあげると10Gbまで容量が増えるとのことだったので、そちらで作るようapp_stackを修正しています。
    
    他に詰まった点として、lambdaをAPIで使う場合、レスポンスの型は指定されていて、それに背くとエラーになります。
    
    pythonの場合、下記のような感じにする必要があります。
    
    ```python
    return {
    	"statusCode": 200,
    	"body": json.dumps({...})
    }
    ```
    
    ## 挙動確認
    
    試しにqiita APiで、ストックが10以上の最新記事300件（2023年1月時点）でつけられているタグをアソシエーション分析してみます。
    
    ```bash
    curl --location --request GET 'https://qiita.com/api/v2/items?page=3&per_page=100&query=stocks%3A%3E10'
    ```
    
    ↓ レスポンスのフィールドは指定できないので、下記のようにmapでタグのみを抽出
    
    ↓ (postmanを使って確認しているのでpmオブジェクトが見えています)
    
    ```jsx
    console.log(JSON.stringify(pm.response.json().map(v => v.tags.map(t => t.name))))
    ```
    
    ↓ 抽出結果
    
    ```json
    ["プロジェクト","エンジニア","エンジニアリング","エンジニアの心得"],
    ["ShellScript","Bash","国際化","Gettext","POSIX"],
    ["プロジェクト管理","エンジニアリング"],
    ["JavaScript","チーム開発","ゲーム制作","Phaser3"],
    ["Windows","コマンドプロンプト"],
    ["フロントエンド","Next.js","t3-stack"],
    ["ポエム","アプリ開発","Flutter","個人開発","flutter大学"],
    ["Svelte","prisma","Supabase","SvelteKit"],
    ...,
    ["Xcode","Swift","SwiftUI"]
    ```
    
    ↓　出力結果をpostmanでリクエストに乗せてAPI実行
    
    ↓　リフト値とかは結構適当だけど、それなりに絞って出力
    
    ```bash
    curl --location --request POST 'https://xxxxxx.amazonaws.com/prod/yyy/' \
    --header 'Content-Type: application/json' \
    --data-raw '{
        "data": [
    			["プロジェクト","エンジニア","エンジニアリング","エンジニアの心得"],
    			["ShellScript","Bash","国際化","Gettext","POSIX"],
    			...,
    			["Qiita","Markdown","エディタ"],
    			["Xcode","Swift","SwiftUI"]
        ],
        "min_support": 0.01,
        "rule": {
            "metric": "confidence",
            "min_threshold": 0.2
        },
        "condition": {
            "confidence": 0.5,
            "lift": 1
        }
    }'
    ```
    
    ↓　レスポンスにはリフト値などが含まれているけれど、ここでは一旦タグの組み合わせのみ確認するため、
    
    ```jsx
    console.log(JSON.stringify(pm.response.json().map(v => [v.antecedents[0], v.consequents[0]])))
    ```
    
    を挟んで出力
    
    ↓
    
    ```json
    [
        [ "TypeScript", "JavaScript" ],
        [ "新人プログラマ応援", "初心者" ],
        [ "AI", "ChatGPT" ],
        [ "ShellScript", "POSIX" ],
        [ "POSIX", "ShellScript" ],
        [ "Rails", "Ruby" ],
        [ "Next.js", "React" ],
        [ "ShellScript", "shell" ],
        [ "shell", "ShellScript" ],
        [ "C#", "Unity" ],
        [ "Bash", "ShellScript" ],
        [ "ShellScript", "Bash" ],
        [ "UNIX", "shell" ],
        [ "shell", "UNIX" ],
        [ "UNIX", "ShellScript" ],
        [ "ShellScript", "UNIX" ],
        [ "Design", "デザイン" ],
        [ "Web", "フロントエンド" ],
        [ "POSIX", "shell" ],
        [ "shell", "POSIX" ],
        [ "UNIX", "POSIX" ],
        [ "POSIX", "UNIX" ],
        [ "深層学習", "Python" ],
        [ "転職", "初心者" ],
        [ "深層学習", "初心者" ],
        [ "駆け出しエンジニア", "初心者" ],
        [ "転職", "未経験エンジニア" ],
        [ "linebot", "Line" ],
        [ "Line", "linebot" ],
        [ "Bash", "POSIX" ],
        [ "POSIX", "Bash" ],
        [ "Jupyter", "Python" ],
        [ "UNIX", "Bash" ],
        [ "Bash", "UNIX" ],
        [ "Bash", "shell" ],
        [ "shell", "Bash" ],
        [ "C", "Elixir" ],
        [ "CSS", "HTML" ],
        [ "VSCode", "ChatGPT" ],
        [ "Git", "GitHub" ],
        [ "Node.js", "JavaScript" ],
        [ "日本語訳", "JavaScript" ]
    ]
    ```
    
    これらの組み合わせが関連度高いらしい。
    
    大体言語とFWのセットっぽいけど、unix系のタグが人気？なのかな。
    
     `[ "VSCode", "ChatGPT" ]`とかは流行りですよね。
    
    ## 終わりに
    
    それっぽい出力ができたので、細かい調整はありますが一旦は使えそうかな。
    
    [https://twitter.com/shaba_raba/status/1616746209942855681](https://twitter.com/shaba_raba/status/1616746209942855681)