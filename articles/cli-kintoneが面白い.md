# cli-kintoneが面白い

Created: 2023年2月16日 17:27
Updated: 2023年2月21日 15:49
Slug: introduce-cli-kintone
Tags: Shell, cli-kintone
Published: No
URL: https://blog.shaba.dev/posts/introduce-cli-kintone

## はじめに

kintoneをcliベースで操作することのできる、cli-kintoneがver1.0.0として正式にリリースされました。

[kintone コマンドラインツール（cli-kintone）](https://cybozu.dev/ja/kintone/sdk/backup/cli-kintone/)

触ってみたら面白かったので少しだけ共有

## cli-kintoneとは

上述の通り、cliベースでkintoneのアプリにレコードを追加したり参照したりできるツールで、**栗きんとん**と読むそうです。
名前考えた人のセンス凄い。

このツールの何が面白いって、IOがjson形式じゃなくcsv形式なんですよね。
業務系のツールってcsv形式でよくダウンロードできると思いますが、それを加工することなくそのまま渡すことができるので、他サービスとの連携を前提とした設計が見て取れます。

他サービスとの連携ではなく、単にレコードを追加したいだけという場合でも、一時的なcsvファイルを作って後で消せばいいだけなので、ひと手間増えますが問題なく使える印象です。

インストール方法などは公式サイトがわかりやすいのでそちらを参照ください。

[kintone コマンドラインツール（cli-kintone）](https://cybozu.dev/ja/kintone/sdk/backup/cli-kintone/)

## 試しに

```bash
#!/bin/zsh

function sbl_usage {
    cat <<EOM
Usage: $(basename "$0") [COMMAND][OPTION]...
    [COMMAND]:
        create  create sbl record
    [OPTION]
    --summary=  required: sbl summary
    --pbl=     required: related pbl record id.
EOM
    exit 2
}

_pbl=""
_summary=""
_status="TODO"
_priority="-1"

function create() {
    _priority=`/etc/cli-kintone record export --base-url $BASE_URL --app $APP_ID_SBL --api-token $API_TOKEN_SBL --condition "ルックアップ_1 = 5498" --order-by "priority desc" --fields "priority" | head -2 | tail -1 | sed -s 's/\"//g'`
    _priority=`expr $((_priority+1))`

    _csv="$HOME/ksbl_create_record.csv"
    echo "ルックアップ_1,summary,priority,sprint,status" > ${_csv}
    echo "${_pbl},\"${_summary}\",${_priority},${_sprint},${_status}" >> ${_csv}

    eval "/etc/cli-kintone record import --base-url $BASE_URL --app $APP_ID_SBL --api-token $API_TOKEN_SBL,$API_TOKEN_PBL --file-path=\"${_csv}\""
    eval "rm ${_csv}"
}

if [[ $1 != "create" ]]; then
    sbl_usage
fi

for var in $@
do
    # long options
    if [[ ${var} = --* ]]; then
        opt=(${var%%\=*})
        val=(${var#*\=})
        [[ $opt = "--pbl" ]] && _pbl=$val
        [[ $opt = "--summary" ]] && _summary=$val
    fi
done

[[ $1 = "create" ]] && create
```

## ちょっとハマったこと

### 表示件数を絞りたい

多分今はない？

### ルックアップフィールドがある場合

ルックアップフィールドを使っている場合、参照先のアプリのAPI TOKEN（参照権限）も必要です。

このことに気づかず時間を使ってしまいました。

API TOKENはカンマ区切りで複数指定できるので、