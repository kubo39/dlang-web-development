# D Web開発

## 構成(例)

- リバースプロキシ,アプリケーションサーバ(vibe.d),データベース(MySQL)の3層構成

### リバースプロキシ

- ここではnginxを想定
  - アプリケーションサーバの前段でバランシング
  - 静的ファイルの配信
  - SSLの終端
  - gzip圧縮
  - リダイレクト
  - レスポンスヘッダの細工

### アプリケーションサーバ

- vibe.d
  - 1台もしくは複数台のサーバを並べる
  - 非同期I/Oベースなのでブロックしないような工夫が必要
    - ディスクI/Oは別スレッドで処理を行うなど

### データベース

- ここではMySQL or Postgresを想定
  - 接続にはコネクションプールを使う
  - 基本的にクエリのチューニングやレプリケーションなど勘所は他の言語と一緒

## コード

たとえばこういうことを考えてコードを書く

### テスト

- テストしやすいデザインを目指すには
  - コントローラーのテストを薄めに
    - テストダブル(Mock,Spy,Fake)を使ってDBを使わない軽量な方法にできないか検討する
  - モデル層は厚くテストする
    - fixtureを使ってDBを使ったテストにしたい

### Adapter Pattern

- vibe.dはORMのような抽象化層を提供しない
- hibernatedのようなORMはあるが使いやすいとはいえない
- 自前で薄い抽象化層をDatabaseAdapterとして作るのがよさそう
  - 実際に使う操作のみ提供するように(全部の機能に対して最初から用意しない)
  - 抽象化層があれば後でDBの変更がしやすい

## プロファイリング,モニタリング

- プロファイリングは自分たちで頑張る部分が多い
  - GoのpprofやRubyのrack-lineprofのような使いごこちのものは期待できない
  - CPUプロファイルはperfやoprofile,google-perftoolsなどから選定
  - メモリプロファイリングはcore.memoryやmallinfo(3)などが使える
- モニタリングに関しては他の言語とほぼ同じだが使える外部サービスは限定されるかも