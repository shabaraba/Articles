# 便利なGitコマンド覚書

Created: 2022年11月12日 20:22
Updated: 2023年2月28日 9:40
Published_Time: 2022年11月14日
Slug: useful-git-command
Tags: git
Published: Yes
URL: https://blog.shaba.dev/posts/useful-git-command

## はじめに

ご無沙汰しておりました。
私事ですが少し前まで転職活動をしておりまして、新しい環境にも慣れてきたのでまた少しずつ記事を書いていこうと思います！

## 新しい職場に触発

前職ではgitコマンドを直接打つ方はあまりなく、専ら`SourceTree`だったり、vscodeの`GitGraph``だったりGUIに頼ってました。
僕もしばらくは`GitGraph`を使っていましたが、neovimに開発の場を移してからはCLIに移りました。が、あまり知識のないまま使用していたので、オプション無しのadd, commit, rebase, mergeくらいでどうにかまわしていました。

一方で、現職はGUIを使ってgitを操作されている方は殆どおらず（驚き）、diffの確認もターミナル上でされている方がほとんどです。
画面共有を頻繁に行うので、他のエンジニアがコマンドを打つ場面も度々目にするので、その際に「便利だなぁ」と感じたGitコマンドについて書いていきます。

## 前のコミットに追加で変更を入れたいとき

前職や個人開発でもしばしば遭遇するこの場面。
一度commitしたものの、編集漏れがあった場合とかですね。

これまでは

1. 一度commitをreset
2. 追加編集したファイルを含めて改めてadd, commit

という手順をとっていました。

```bash
git reset HEAD^
git add .
git commit -m “xxx”
```

今は`—amend`オプションを使用して対応しています。

```bash
git add .
git commit --amend
```

`—amend`オプションはstagingされた変更点を直前のcommitに追加するもので、一応commitメッセージの入力フェーズには移行しますが、デフォルトで前のメッセージが入っているので、変更したければ入力しましょう。
特に変更なければそのまま閉じるだけでも良いですが、`—no-edit`をつけるとメッセージ入力をスキップできます。

なお、remoteにpushするときは`—force-with-lease`を使用する必要があります。

## あるcommitで変更が加わったファイルの一覧を確認したいとき

これまでこれが必要になった場面はあまりなかったのですが、リファクタの際に便利だったので。

```bash
git show <commit id> --name-only
```

## リモートブランチでローカルブランチを上書きしたいとき

リモートブランチに誰かがpushしていてconfrictは起こさないのでただただ取り込みたい時。
今まで、自身のローカルブランチを削除して改めてcheckoutし直していました。

```bash
git checkout develop
git branch -D <branch name>
git checkout <branch name>
```

これを、`—rebase`オプションをつけて下記のように1コマンドで実施できました。

```bash
git pull --rebase
```

とても便利。

## git diff をいい感じに見やすくする

gitコマンドではないのですが、git diffって個人的に結構見にくいんですよね。
vscodeのgit diffが至高だと思っていて、あれっぽい見た目にできないか探してみたところ、deltaというものがとてもいい感じに見せてくれることがわかりました。

[git diff をgithubみたいに見やすくしたい人へ　(左右画面分割diff , 差分ハイライト - Qiita](https://qiita.com/momoka0122y/items/9e5ca351791485aa85ff)

```bash
brew install git-delta
```

↓

最近は`git-split-diffs`を使っています。
こちらのほうが個人的に好みです。

[“git diff”コマンドの実行結果を Github 風に表示できる「git-split-diffs」 | ゲンゾウ用ポストイット](https://genzouw.com/entry/2021/05/24/104932/2640/)

ちなみに中のちなみにですが、同じ感じで標準出力にハイライトを付けたい場合は
catではなくbatがおすすめです笑

[【bat】catコマンドの結果にシンタックスハイライトしたい](https://amateur-engineer-blog.com/bat/)

## 要らなくなったブランチを一括削除したい

次のコマンドでローカルに有るmainとdevelop以外のブランチを削除します

```bash
git branch | grep -v "main\|develop" | xargs git branch -D
```

## Git重いんですけど！

次のコマンドで不要なgitのデータを削除します

```bash
git prune
```

## 今居るブランチの親を確認したい

結構いろんなサイト様で紹介されているようなので、コマンドだけ。

```bash
git show-branch | grep '*' | grep -v "$(git rev-parse --abbrev-ref HEAD)" | head -1 | awk -F'[]~^[]' '{print $2}'
```

↓でも多分こっちのほうが良い

```bash
git show-branch | grep -E "\*\s*\[" -A 1 | tail -1 | awk -F'[]~^[]' '{print $2}'
```

## 今いるブランチの変更のみをログに出力したい！

普通に`git log`コマンドを打つと、自身のブランチで変更したコミット以前のものも一緒に引っ付いてきます。
ちょっと見づらいので、GitHubのPRのように、マージされていないコミットのみ表示したいことの方が多いと思います（僕はそうです）。
そんなときは、次のように比較対象のブランチを指定して、`..`を続けて書きます。

```bash
git log develop..
```

これで、現在居るブランチで、developブランチにマージされていないコミットのログだけを表示できます。

### より便利に！

**「いちいち比較ブランチなんか打ってられねぇ！」**って人もいると思います（僕がそうです）。
なので、親ブランチを確認するコマンドと一緒に使っています。

```bash
git show-branch | grep -E "\*\s*\[" -A 1 | tail -1 | awk -F'[]~^[]' '{print $2}' | awk -F'\''[]~^[]' '{print $2}' | xargs -Iparent git log parent..
```

親ブランチを`xargs -I`コマンドでparentという変数（？）に入れて、`git log`で使うようにしています。
これを.zshrcにaliasを書いて、

```bash
alias gl='git show-branch | grep -E "\*\s*\[" -A 1 | tail -1 | awk -F'\''[]~^[]'\'' '\''{print $2}'\'' | xargs -Iparent git log parent..'
```

として、`gl`で呼び出しています。とても便利。

## あいまい検索でブランチを指定してcheckoutしたい！

`peco`を使ってあいまい検索を実現します

[GitHub - peco/peco: Simplistic interactive filtering tool](https://github.com/peco/peco)

```bash
brew install peco
```

`git branch`で取得したブランチ一覧をpecoに渡して、その選択結果をxargsでgit checkoutに渡します。

```bash
git branch | peco | xargs git checkout
```

これを.zshrcにaliasを書いて、

```bash
alias gc='git branch | peco | xargs git checkout'
```

として、`gc`で呼び出しています。

### リモートブランチも対象にしたい！

上記だとローカルにあるブランチのみ対象なのですが、リモートもあいまい検索対象にしたいですよね。
`remotes/origin`があたまにつくので、それを削除してからcheckoutするよう修正します。

```bash
git branch --all | peco | sed -e "s/remotes\/origin\///g" | xargs git checkout
```