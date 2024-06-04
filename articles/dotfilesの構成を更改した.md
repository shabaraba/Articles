# dotfilesの構成を更改した

Created: 2024年5月29日 17:12
Updated: 2024年5月29日 22:10
Slug: update-dotfiles
Tags: 個人開発
Published: No
URL: https://blog.shaba.dev/posts/update-dotfiles

## カオスだったdotfilesのディレクトリ構成

dotfilesをなんとかせねばとずっと思っていたのでちょっとずつ手を加えていったが、ある程度形になってきたので一度メモを取る。

[GitHub - shabaraba/dotfiles: dotfiles](https://github.com/shabaraba/dotfiles)

## makefile内で処理するのを辞めた

dotfilesプロジェクトからホームディレクトリにリンクを渡すのを、今まではmakefileに記載していて、

```bash
make install
```

とか

```bash
make deploy
```

とかのコマンドで実行していた。

..が、当方別にmakefileに明るいわけでもなく、どう記載したら良いのかよくわからないところが多く、ストレスだった。
し、最近はあまりmakefileを作らないとも聞いた。

…ので、処理は基本`shellscript`に書き出して、`makefile`からはそれを呼ぶだけにとどめた。

## dotfiles内の.configを消した

dotfilesに手を加えているうちに鬱陶しく感じていたのが、`.config`の存在。
`.`がついているので、エディタが検索から排除してしまい、定義元とかtelescopeのファイル検索に引っかからなかった。

設定で引っかかるようにもできると思うが、他のプロジェクトではdotfile系は検索に引っかかってほしくないので、設定は維持したい。

考えた結果、`.config`ディレクトリは当プロジェクトから削除して、代わりに設定したいツールごとにディレクトリを切って、
`deployer.sh`がよしなにホームディレクトリの`.config`内にリンクを作成するように変更した。

## Neovimディレクトリ内を整理した

vim→neovim（vimscriptで記述）→luaで記述→[packer.nvim](https://github.com/wbthomason/packer.nvim)を使用→[NvChad](https://nvchad.com/)を使用→[Lazy.nvim](https://github.com/folke/lazy.nvim)を使用

というよくある遷移をたどってきたが、

- それぞれの設定がバックアップとして残っていたり
- NvChadを辞めたのにNvChadの構成のままだったり
- ツギハギした結果インデントすら揃っていなかったり

とカオスだったので、ここをなんとかしたかった

結果、こんな感じになりました。

```bash
nvim
├── init.lua
├── lazy-lock.json
├── lua
│   ├── core
│   │   ├── autocmds.lua
│   │   ├── init.lua
│   │   └── utils.lua
│   ├── mappings.lua
│   ├── options.lua
│   ├── plugins
│   │   ├── _disabled
│   │   │   ├── ...
│   │   │   └── ...
│   │   ├── action
│   │   │   ├── ...
│   │   │   └── ...
│   │   ├── coding
│   │   │   ├── ...
│   │   │   └── ...
│   │   ├── core
│   │   │   ├── lsp
│   │   │   │   ├── ...
│   │   │   │   └── ...
│   │   │   └── treesitter
│   │   │       ├── ...
│   │   │       └── ...
│   │   ├── init.lua
│   │   ├── ui
│   │   │   ├── ...
│   │   │   └── ...
│   │   └── view
│   │       ├── ...
│   │       └── ...
│   └── root.lua
└── startup.log
```