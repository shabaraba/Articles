# Cannot set LC_XXX to default locale: No such file or directoryを解決する

Created: 2023年10月6日 14:59
Updated: 2024年5月2日 17:28
Published_Time: 2024年5月1日
Slug: fix-vim-lc
Tags: vim
Published: Yes
URL: https://blog.shaba.dev/posts/fix-vim-lc

vimを開いたら言語設定？が壊れていた。

前にも直した記憶があるが、しょっちゅう壊れる系なのか？

何度も調べたくないので今回対応した方法を記録する。

```bash

$ locale -a
locale: Cannot set LC_CTYPE to default locale: No such file or directory
locale: Cannot set LC_MESSAGES to default locale: No such file or directory
locale: Cannot set LC_COLLATE to default locale: No such file or directory
C
C.utf8
POSIX
ja_JP.utf8

$ localt
zsh: correct 'localt' to 'local' [nyae]? n
zsh: command not found: localt

$ localectl list-locales
C.UTF-8
ja_JP.UTF-8

$ sudo localedef -f UTF-8 -i en_US en_US.UTF-8

$ localectl list-locales
C.UTF-8
en_US.UTF-8
ja_JP.UTF-8

$ locale
LANG=en_US.UTF-8
LANGUAGE=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=en_US.UTF-8
```