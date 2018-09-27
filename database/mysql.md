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
