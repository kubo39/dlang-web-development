# My thought of testing

## 基本的な考え

- テストもなるべく共通化を意識
  - Parameterized Testが目指す形
    - この形になっていれば非常に保守しやすいテストにになっている証
  - xUnit Test Patterns
    - Creation Method: Setup
    - Custom Assetion: Asserion Utilitiy functions/objects
    - 4 phase test
      - Setup,Exercise,Verify,TearDown
      - この形になってはじめて Parameterized Test を検討できる
      - いきなり持ち込めるものではない
  - 共通部分をユーティリティに
    - パラメータのValidationとか
    - copy&pasteを減らせる
      - copy&pasteだとここだけ変更し忘れが起きやすい
- Controller層のテストは薄く
  - Controller層でやっているのは繋ぎこみの部分が主
  - Test Double を使ってモデルへの依存を小さく
    - Test Double: Mock,Spy,Fake,Dummy
    - 依存があると連鎖的にテストが落ちる
      - なにが問題だったのか一見わかりにくい
- Model層のテストは厚く
  - だいたいビジネスロジックで一番大切なところなので手厚く
  - fixtureを使ってdbを使ったテストにする
- 小さなテストユーティリティやコンポーネントを作る
  - 単一責任の原則
  - 責務を小さく、明確に
- Test Coverageに騙されない
  - TestCaseの数とCoverageは直交する
  - TestCaseの数が減るような設計を重視すべき
- テストをいつ書くか
  - 設計が固まってないときにこそ先にテストを書くべき
    - テストしやすい設計に自然となる
    - テストが書きづらいコードはプロダクションコードからみても使いづらい
