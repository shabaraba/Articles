# dyld: Library not loaded: /usr/local/opt/icu4c/lib/libicudata.69.dylib で動かなくなったmacのPHPを救出した

Created: 2022年3月28日 21:39
Updated: 2022年12月12日 13:28
Published_Time: 2022年3月29日
Slug: library-icu4c-not-found
Tags: Mac, PHP
Published: Yes
URL: https://blog.shaba.dev/posts/library-icu4c-not-found

## Qiitaに移行しました

[](https://qiita.com/shabaraba/items/293818a70be54dcda6c8)

一問一答系なのでQiitaのほうがふさわしそう。

下記アーカイブ...

---

## PHPがバージョンすら返してくれないんだが

普段の業務や個人開発ではもっぱらdockerを使っていて、PCに直に入っているPHPとかはあまり触ることがありませんでした。
一方で、当方vimを使っていて、たまに単純なコードを確認するのに[quickrun](https://github.com/thinca/vim-quickrun)を利用しています。
vimはdocker内で起動しているのではなくてホスト側から起動しているので、quickrunをするとホストのPHPが使用されます。

今回もなんの気なしに新規バッファにコードを貼り付けて`:Quickrun`を実行しました。
すると

```bash
dyld[77982]: Library not loaded: /usr/local/opt/icu4c/lib/libicuio.69.dylib
  Referenced from: /usr/local/Cellar/php/8.1.3/bin/php
  Reason: tried: '/usr/local/opt/icu4c/lib/libicuio.69.dylib' (no such file), '/usr/local/lib/libicuio.69.dylib' (no such file), '/usr/lib/libicuio.69.dylib' (no such file), '/usr/local/Cellar/icu4c/70.1/lib/libicuio.69.dylib' (no such file), '/usr/local/lib/libicuio.69.dylib' (no such file), '/usr/lib/libicuio.69.dylib' (no such file)
```

うん？
前まで動いていたのにライブラリがない？

となり、

```bash
php -v
```

としても、上記エラーが返るようになってしまっていました。

dockerにうつつを抜かしている間に、なんてこった。

## Homebrewのupdateをかけたのが原因らしい

nodeを最新にアップデートしたら治ったよ、とか
brewを最新にアップデートしたら治ったよ、とかありましたが、いずれでも解決せず...。

調べていくと下記ブログにたどり着きました。
どうやらHomebrewさんは新しいパッケージしかサポートしないようになったんだとか。

[Homebrewで過去のバージョンを使いたい【tap版】 - Carpe Diem](https://christina04.hatenablog.com/entry/install-old-version-with-homebrew)

下記ブログと合わせて確認して、直していきます。

[dyld: Library not loaded: /usr/local/opt/icu4c/lib/libicudata.67.dylibのエラーでPHP7.4が突然使えなくなった話](https://kin29.info/dyld-library-not-loaded-usrlocalopticu4cliblibicudata-67-dylib%E3%81%AE%E3%82%A8%E3%83%A9%E3%83%BC%E3%81%A7php7-4%E3%81%8C%E7%AA%81%E7%84%B6%E4%BD%BF%E3%81%88%E3%81%AA%E3%81%8F%E3%81%AA/)

## 復旧作業

### tap用リポジトリの作成

パッケージの過去のバージョンを入れるには`tap`という機能を使用するらしいです。
まずはtap用のリポジトリを作成します。

```bash
$ brew tap-new shaba/taps #<-shabaは何でも良いらしいです
Initialized empty Git repository in /usr/local/Homebrew/Library/Taps/shaba/homebrew-taps/.git/
[main (root-commit) 5b33ac7] Create shaba/taps tap
 3 files changed, 90 insertions(+)
 create mode 100644 .github/workflows/publish.yml
 create mode 100644 .github/workflows/tests.yml
 create mode 100644 README.md
==> Created shaba/taps
/usr/local/Homebrew/Library/Taps/shaba/homebrew-taps

When a pull request making changes to a formula (or formulae) becomes green
(all checks passed), then you can publish the built bottles.
To do so, label your PR as `pr-pull` and the workflow will be triggered.

```

作れたっぽい。

### 旧バージョンのicu4cをインストール

上記2ブログと違い今回遭遇したのはバージョン69が存在しないよエラーでした。

なのでバージョン情報を変えて実施。

```bash
# インストール前のフォルダの状態確認
$ ls /usr/local/Cellar/ | grep icu
icu4c/
$ ls /usr/local/opt/ | grep icu4c
icu4c@

# インストールしたいパッケージをhomebrewのリポジトリから抽出して、自分のリポジトリに登録する。
$ brew extract icu4c shaba/taps --version 69
==> Searching repository history
==> Writing formula for icu4c from revision c278d3d to:
/usr/local/Homebrew/Library/Taps/shaba/homebrew-taps/Formula/icu4c@69.rb

# 旧バージョンのインストール
$ brew install shaba/taps/icu4c@69
fatal: 'origin' does not appear to be a git repository
fatal: Could not read from remote repository.

Please make sure you have the correct access rights
and the repository exists.
fatal: 'origin/master' is not a commit and a branch 'master' cannot be created from it
fatal: invalid upstream 'origin/master'
==> Downloading https://github.com/unicode-org/icu/releases/download/release-69-1/icu4c-69_1-src.tgz
==> Downloading from https://objects.githubusercontent.com/github-production-release-asset-2e65be/49244766/f8594300-97e0-11eb-93b6-3cc5dfa61ec0?X-Amz-Algorithm=AWS4-HM
######################################################################## 100.0%
==> Installing icu4c@69 from shaba/taps
Warning: Your Xcode (13.2.1) is outdated.
Please update to Xcode 13.3 (or delete it).
Xcode can be updated from the App Store.

Warning: A newer Command Line Tools release is available.
Update them from Software Update in System Preferences or run:
  softwareupdate --all --install --force

If that doesn't show you any updates, run:
  sudo rm -rf /Library/Developer/CommandLineTools
  sudo xcode-select --install

Alternatively, manually download them from:
  https://developer.apple.com/download/all/.
You should download the Command Line Tools for Xcode 13.3.

==> ./configure --prefix=/usr/local/Cellar/icu4c@69/69.1 --disable-samples --disable-tests --enable-static --with-library-bits=64
==> make
==> make install
==> Caveats
icu4c@69 is keg-only, which means it was not symlinked into /usr/local,
because macOS provides libicucore.dylib (but nothing else).

If you need to have icu4c@69 first in your PATH, run:
  echo 'export PATH="/usr/local/opt/icu4c@69/bin:$PATH"' >> ~/.zshrc
  echo 'export PATH="/usr/local/opt/icu4c@69/sbin:$PATH"' >> ~/.zshrc

For compilers to find icu4c@69 you may need to set:
  export LDFLAGS="-L/usr/local/opt/icu4c@69/lib"
  export CPPFLAGS="-I/usr/local/opt/icu4c@69/include"

For pkg-config to find icu4c@69 you may need to set:
  export PKG_CONFIG_PATH="/usr/local/opt/icu4c@69/lib/pkgconfig"

==> Summary
🍺  /usr/local/Cellar/icu4c@69/69.1: 259 files, 72.7MB, built in 4 minutes 10 seconds
==> Running `brew cleanup icu4c@69`...
Disable this behaviour by setting HOMEBREW_NO_INSTALL_CLEANUP.
Hide these hints with HOMEBREW_NO_ENV_HINTS (see `man brew`).

# インストール確認
$ ls -l /usr/local/Cellar/ | grep icu
drwxr-xr-x 3 macbookpro  96 Mar 25 21:20 icu4c/
drwxr-xr-x 3 macbookpro  96 Mar 25 21:45 icu4c@69/ # <- 追加

$ ls -l /usr/local/opt/ | grep icu4c
lrwxr-xr-x 1 macbookpro 20 Mar 24 01:24 icu4c -> ../Cellar/icu4c/70.1/
lrwxr-xr-x 1 macbookpro 23 Mar 25 21:49 icu4c@69 -> ../Cellar/icu4c@69/69.1/ # <- 追加
```

なんかだいぶ怪しかったけど、どうやらインストールできたらしい。
あとは先のブログ様でも記載されている通り、シンボリックリンクを`/usr/local/opt/icu4c/lib`から貼ってあげれば良いそうです。

```bash
 > {/usr/local/opt/icu4c/lib}
$ ln -s /usr/local/opt/icu4c@69/lib/libicudata.69.1.dylib libicudata.69.dylib
 > {/usr/local/opt/icu4c/lib}
$ ln -s /usr/local/opt/icu4c@69/lib/libicutest.69.1.dylib libicutest.69.dylib
 > {/usr/local/opt/icu4c/lib}
$ ln -s /usr/local/opt/icu4c@69/lib/libicutu.69.1.dylib libicutu.69.dylib
 > {/usr/local/opt/icu4c/lib}
$ ln -s /usr/local/opt/icu4c@69/lib/libicuuc.69.1.dylib libicuuc.69.dylib
 > {/usr/local/opt/icu4c/lib}
$ ln -s /usr/local/opt/icu4c@69/lib/libicuio.69.1.dylib libicuio.69.dylib
 > {/usr/local/opt/icu4c/lib}
$ ln -s /usr/local/opt/icu4c@69/lib/libicui18n.69.1.dylib libicui18n.69.dylib
```

先のブログ様ではこれでいけてた！
Go！

```bash
$ php -v
dyld[81763]: Library not loaded: @loader_path/libicuuc.69.dylib
  Referenced from: /usr/local/Cellar/icu4c@69/69.1/lib/libicuio.69.1.dylib
  Reason: tried: '/usr/local/Cellar/icu4c@69/69.1/lib/libicuuc.69.dylib' (no such file), '/usr/local/lib/libicuuc.69.dylib' (no such file), '/usr/lib/libicuuc.69.dylib' (no such file)Library not loaded: @loader_path/libicuuc.69.dylib
  Referenced from: /usr/local/Cellar/icu4c@69/69.1/lib/libicui18n.69.1.dylib
  Reason: tried: '/usr/local/Cellar/icu4c@69/69.1/lib/libicuuc.69.dylib' (no such file), '/usr/local/lib/libicuuc.69.dylib' (no such file), '/usr/lib/libicuuc.69.dylib' (no such file)Library not loaded: @loader_path/libicudata.69.dylib
  Referenced from: /usr/local/Cellar/icu4c@69/69.1/lib/libicuuc.69.1.dylib
  Reason: tried: '/usr/local/Cellar/icu4c@69/69.1/lib/libicudata.69.dylib' (no such file), '/usr/local/lib/libicudata.69.dylib' (no such file), '/usr/lib/libicudata.69.dylib' (no such file)
zsh: abort      php -v
```

なぜだ。

tapでインストールしたライブラリ内で同じく呼び出しが失敗しているっぽい。

インストールしたフォルダ配下を覗いてみると、各ライブラリで呼び出す予定のファイル名が存在せず...なぜだ。

```bash
 > {/usr/local/opt/icu4c@69/lib}
$ ls
icu/                   libicudata.dylib@      libicui18n.dylib@    libicuio.dylib@        libicutest.dylib@    libicutu.dylib@      libicuuc.dylib@
libicudata.69.1.dylib  libicui18n.69.1.dylib  libicuio.69.1.dylib  libicutest.69.1.dylib  libicutu.69.1.dylib  libicuuc.69.1.dylib  pkgconfig/
libicudata.a           libicui18n.a           libicuio.a           libicutest.a           libicutu.a           libicuuc.a
```

場当たり的な気もしますが、`/usr/local/opt/icu4c@69/lib`配下でも同様にシンボリックリンクを貼ります。

```bash
> {/usr/local/opt/icu4c@69/lib}[]
$ ln -s /usr/local/opt/icu4c@69/lib/libicui18n.69.1.dylib libicui18n.69.dylib
> {/usr/local/opt/icu4c@69/lib}[]
$ ln -s /usr/local/opt/icu4c@69/lib/libicuio.69.1.dylib libicuio.69.dylib
> {/usr/local/opt/icu4c@69/lib}[]
$ ln -s /usr/local/opt/icu4c@69/lib/libicuuc.69.1.dylib libicuuc.69.dylib
> {/usr/local/opt/icu4c@69/lib}[]
$ ln -s /usr/local/opt/icu4c@69/lib/libicutu.69.1.dylib libicutu.69.dylib
> {/usr/local/opt/icu4c@69/lib}[]
$ ln -s /usr/local/opt/icu4c@69/lib/libicutest.69.1.dylib libicutest.69.dylib
> {/usr/local/opt/icu4c@69/lib}[]
$ ln -s /usr/local/opt/icu4c@69/lib/libicudata.69.1.dylib libicudata.69.dylib
```

改めて。

```bash
$ php -v
PHP 8.1.3 (cli) (built: Feb 18 2022 09:32:50) (NTS)
Copyright (c) The PHP Group
Zend Engine v4.1.3, Copyright (c) Zend Technologies
    with Zend OPcache v8.1.3, Copyright (c), by Zend Technologies
```

まぁ返ってきたので良しとしましょう。
お疲れさまでした！

でもこれbrewをupdateしたらまたどこかのタイミングで発生するのかな...？
結構面倒くさいなぁ