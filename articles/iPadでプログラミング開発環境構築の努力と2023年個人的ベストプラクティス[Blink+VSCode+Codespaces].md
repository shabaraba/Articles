# iPadでプログラミング開発環境構築の努力と2023年個人的ベストプラクティス[Blink+VSCode+Codespaces]

Created: 2022年12月19日 8:50
Updated: 2023年2月14日 9:10
Published_Time: 2022年12月19日
Slug: coding-with-ipad
Tags: Blink, Codespaces, VSCode, iPad, 開発環境
Published: Yes
URL: https://blog.shaba.dev/posts/coding-with-ipad

## はじめに

これまで３年くらいかけてiPadでwebアプリを開発する環境を整えるべく色々努力してきたのですが、
これからはPCとほぼ変わらない環境で開発できるかも知れません。

最新の個人的ベストプラクティスはQiitaの方に紹介しました。

[iPadだけでVSCode環境を得られるようになった - Qiita](https://qiita.com/shabaraba/items/b73fb114e19a76dfd87f)

なのでこのエントリではこれに至るまでの道のりもあわせて書こうと思います。

## これまでの努力

本題に入る前にこれまでの努力をここで供養させてください🙏

### JSAnywhereやpythonista3など各言語用のエディタを購入

iPadでプログラミングやるぞ！ってなってはじめにやったのは、[JSAnywhere](https://apps.apple.com/jp/app/javascript-anywhere-jsanywhere/id363452277)や[Pythonista3](https://apps.apple.com/jp/app/pythonista-3/id1085978097)といった特定言語用のエディタを入れての開発でした。

が、正直コーディングをお遊びレベルで触りたい人向けな感じで、webアプリを開発することはできなさそうでした。

ただPythonista3は頑張った結果Djangoは動いたので、めっちゃ頑張ればPythonでだと開発できるのかも知れません。

ちなみにPythonista3はweb開発用としては断念しましたが、スクリプトをアプリ化できるので、Git clientアプリの[Working Copy](https://apps.apple.com/jp/app/working-copy-git-client/id896694807)と使一緒に[今でも使ってたりします](https://qiita.com/shabaraba/items/bbc214df8e8e7a438332)。
使いやすいのすで結構おすすめです。

### Gitpod

次に考えたのが、**「IDEをエミュレートしたwebサービスどっかにないかな」**でした。
色々調べていくうちに、[Gitpod](https://www.gitpod.io/)というサービスに出会います。

[Gitpod: Always ready to code.](https://www.gitpod.io/)

これがめちゃくちゃ便利で、`gitpod://`の後ろにリポジトリのurlを記載すればvscodeライクな画面で開発ができるというwebサービスでした。

拡張機能もvscodeのものがそのまま使えたので、しばらくはこれに居座ります。

### ConohaVPS + Coder

…ごめんなさい、なんでこっちに移ったのかちゃんと思い出せないのですが、当時のGitpodにもなにか不満を覚えて（たしかvscodevimがうまく動かないとかそんなんだったと思います）、次の代替サービスを探した結果、Coderに出会います。

[Coder - Remote development on your infrastructure](https://coder.com/)

こちらはセルフホストができて、その場合は無料で使えるところが魅力的でした。

すでにサービス公開用のVPSをConohaでとっていたので、そこを間借りする形で建てておけば、料金を気にせずいつでも開発できたので、開発体験が上がったのも良かったですね。

その時のお話がこちらの記事です。

[iPadだけでプログラミング環境を整えた話[VPS+Coder]](https://blog.shaba.dev/posts/programming-with-coder)

vpsサービスはいくつか触ってみて検討しましたが、[ConohaVPS](https://www.conoha.jp/vps/)に落ち着きました。
（後述の通り今は開発環境としては使用していませんが、アプリのデプロイ先として今も保持しています。）

操作もわかりやすくて、お値段もまぁ許容かな、といった感じです。

が、（少なくとも当時は）いくつか致命的な問題点がありました。

- websocketをsafariがうまく処理できずbasic認証をかけれない
- browser previewプラグインがうまく動かない
- …

そんな感じで開発が辛くなってきたので、再び代替案を探し始めます。

### ConohaVPS + Blink + vim

**「じゃぁもうvimで開発すればよくね？」**と思って、調べたのが`sshターミナル`アプリです。

ターミナルアプリは[a-shell](https://holzschu.github.io/a-Shell_iOS/)や[iSH](https://ish.app/)、[prompt2](https://panic.com/jp/prompt/)を経由して、[Blink](https://blink.sh/)を利用することに決めました。

a-shellは結構ちゃんとエミュレートしていて、phpのlaravelが動いたというびっくりアプリでした（とはいえlaravel起動中激重なので実用には至りませんでしたが）。

prompt2もしばらく使っていましたが、キー長押しでも連続して通信してくれないので、vimで下にスクロールしたい場合`j`を連打する必要があるのが不満点でした。

Blinkはお値段結構はりますが、mosh対応などかなりの高機能で、大満足しています。

また本物の（？）vimについては後日お話しますが、カスタマイズが結構大変かつ楽しく、結構開発に問題ないレベルまで育てることができました。

クラウド上での開発環境を整えるのは結構大変で、

- エディタの準備（僕の場合はvimで決まっていましたが、プラグインの選定など）
- もろもろのライブラリのインストール
- ローカルを立ち上げたときにホストから確認できるよう開発環境用のサブドメインの整備
    - 最低限のbasic認証とか

１年位上記の構成で個人開発を続けていました。
実際に、その環境でTimLogというアプリだったり、look-into-basketsというアプリを開発してきました。

ただ問題点も感じてきて、具体的には（お金をケチっているため）スペック不足に陥り、dockerのフリーズやvps自体のフリーズ、接続断など、なかなか開発環境として辛いものがありました。

### AWS EC2 Spot Instances + Blink + neovim

そんな折見つけたのがEC2の[Spot Instances](https://aws.amazon.com/jp/ec2/spot/)というサービス。
EC2もVPSと同じくスペックを上げるにつれてお高い金額になっていきますが、Spot Instancesは使用されていないEC2キャパシティを間借りさせてもらうようなサービスで、お安い値段でハイスペックな環境を用意できます。

<aside>
💡 あくまで「間借り」なので、元の持ち主が使用する場合は強制的に接続断になります。Spot Instances単体だとwebサービスを立ち上げるのは厳しいかもですが、ただの開発環境だととても使い勝手が良いです。

</aside>

クラウド環境上で開発するので、Conoha VPS時代でも頑張ったところは解消できませんでしたが、[より使い勝手が良くなるよう工夫](https://qiita.com/shabaraba/items/bbc214df8e8e7a438332)をして、iPadで開発をする際はこちらの構成でこれまで行ってきました。

## そして、今

さて、ここまでが僕のiPad開発環境の遍歴で、ここからが本題です。

冒頭で紹介したQiita記事の構成で開発を進めています。

[iPadだけでVSCode環境を得られるようになった - Qiita](https://qiita.com/shabaraba/items/b73fb114e19a76dfd87f)

### Codespacesとは

Githubが以前から開発を進めていた[Codespaces](https://github.co.jp/features/codespaces)にまさかの無料プランが登場しました。

- 1コアあたり120時間/月まで無料で2coreから選択可能（つまり60時間/月まで無料）
- ストレージは15GB/月まで無料

と、社会人が個人開発する分には十分無料で収まる感じで、すごい衝撃でした。マジでありがとうGithub。
（フリーランスとかだと60時間は厳しいかも。でも事業立ち上げるなら有料でもいいですよね。）

試しにiPadからchromeでCodespacesを立ち上げましたが、問題なく使えそうでした。

### Blinkの大幅アップデート

そんな感じで、「これからはCodespacesで開発していくかなー」と思っていたのですが、しばらく個人開発がご無沙汰だったので久しぶりにBlinkを起動してみると

[Blink Shell: Blink Shell: The top mobile terminal for Apple devices.](https://docs.blink.sh/advanced/code)

え、`code`コマンドが実装されている…？

え、Blink内でVSCodeが起動する…？

え、Blink-fsの拡張機能を入れればiPad内のファイルにもアクセス可能…？

え、しかもCodespacesにも接続できる…？

ということでさらなる神アプリとなったBlinkでCodespacesを起動したのが冒頭の記事内の動画です。

細かい確認はまだですが、Next.jsはするっと起動しました。

あとCodespaces起動欄の`open in VSCode`のリンクでBlinkが起動するようになるの、とても良いはからいですよね。

## 最新版の開発環境

というわけで最新の開発環境は

- Blink
- Codespaces

という構成になりました。

ここまで来たかiPad開発環境。

iPadOSのアップデートで外付けモニタ対応だったりステージマネージャーだったり、より大量の情報を一度に確認可能になった今、本当にPCと同等の開発環境を手に入れつつありますね。