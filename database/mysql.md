# MySQL

## ドライバ

### mysql-native

調査している時点での最新版は 2.2.2

- vibe.dのソケットとPhobosのソケットの両方に対応している、デフォルトでvibe.dを使っているプロジェクトを判別してvibe.dのソケットを使うようになっている
  - `"versions": ["Have_vibe_d_core"]` で強制的に指定することもできる
- prepared statementを使える
  - SQLインジェクション対策に有効
  - prepared statementはクエリのパースを毎回行う必要がないのでパフォーマンス向上が期待できる
  - コネクションプールとの併用はmysql-native側でうまいことやってくれてる
    - コネクション側でハンドルを管理してよしなにregister/releaseしてる
- 自前でコネクションプールをもっている
  - といってもコネクションプールの部分はvibe.dのジェネリックな実装の上になってる
  - 基本的にはここから `lockConnection` でDBサーバと通信を行う
    - 再接続などもよしなにやってくれるので基本これを使うべき
  - 最大同時接続数は `maxCurrency` で変更可能
- selectクエリの結果は `ResultRange` というrangeが返り、 `array` で `Row` の配列に変換できる
  - ResultRange / Row ともにデフォルトだとVariantとして扱う必要がある
  - Row は `toStruct` で構造体に変換できる

## SQLインジェクション

任意のSQLを実行される

mysql-nativeでできる対策として

1. prepared statementを使う
2. mysql_escapeを使う

という方法がある。
基本的にはprepared statementで対策し補助的にescape用関数を使用する。

## N+1クエリ

- 例
  - この例だと100回ループが走る

```d
auto conn = client.lockConnection();
auto memos = conn.query("select * from `memos` where `is_private` = 0" ~
    "order by `created_at` desc, `id` desc limit 100");
foreach (memo; memos)
{
    memo.username = conn.queryRow("select `username` from `users` where `id` = ?",
                                  memo.user);
}
```

- inner joinを使ってクエリを書き換える
  - 結合したテーブルを作る
  - outer joinというのもある

```d
auto memos = conn.query("select `memos.*`, `user.username`" ~
    "from `memos` join `users` on `memos.user` = `user.id`" ~
    "where `memos.is_private_id` = 0" ~
    "order by `memos.created_at` desc," ~
             "`memos.id` desc limit 100");
```

## インデックス

- 以下のクエリは `is_private`, `created_at` にインデックスをはっていないので遅い
  - is_private: memosテーブルをフルスキャンして抽出してしまう
  - created_at: テーブルに対してMySQL鯖内でソートが走る(CPU負荷が高い)

```d
auto memos = conn.query("select * from `memos` where `is_private` = 0 order by" ~
    "created_at desc limit 100");
```

- インデックスは `alter table` 構文を使うことで生成できる

```sql
ATLER TABLE memos ADD INDEX (is_private,created_at);
```

## オフセットと破棄

- MySQLのオフセットはソートと組わせるとつらい
  - オフセットが大きいと大量のデータ破棄が発生

```d
conn.query("select * from `memos` order by `created_at` limit 100 offset 10000");
```

- 可能なら取得するデータを制限

```d
conn.query("select `id` from `memos` order by `created_at` limit 100 offset 10000");
```
