Workflow Engines Night
===

- [ATND/Workflow Engines Night](https://atnd.org/events/88470)
- [hashtag #workflowenginesnight](https://twitter.com/search?f=tweets&vertical=default&q=%23workflowenginesnight&src=typd)

# StackStorm ワークフローエンジンによる運用コスト・リスク低減の取り組み by 大山 裕泰 氏

## システム運用者を悩ませる課題

- タスク要求処理
- 報告
- 部内調整
  - 社内DNS運用者などがアクター
  - いろんなチャネルから依頼が来る
- 個別・具体的なシステムへのオペレーション
  - システムが変わった時に教育コスト等あらたにかかる
- 定形運用のAPI化
  - フリーフォーマットからカタログ化したオペレーションのAPIを提供
- システムの抽象化
  - 例えば、依頼者はVMが欲しいわけではなく、IP reachableなマシンが欲しいはず

- StackStorm
  - Netflix製
  - IFTTT × WorkFlow
    - e.g. If GitHub Trigger then Jenkins action something.
  - An external world -> Trigger -> Sensor -> Rule -> Workflow -> (Multi) Action -> Another external world
  - Packというmoduleが提供されている(Rubyでいうgemのようなイメージ)
    - e.g. GitHub
    - Provided nearly 10,000,000 packs
- Q&A
  - exactly onceはどう実現する?
  - on failureの柔軟性は?

# Digdagへ日次バッチを移行して幸せになる話 by @i_szyn

- [speakerdeck/JenkinsからDigdagへ日次バッチを移行して幸せになるお話](https://speakerdeck.com/dmmlabo/jenkinskaradigdagheri-ci-batutiwoyi-xing-sitexing-seninaruohua)

## なぜDigdag?

- 他事業部のデータ取り込み処理
- データ加工処理
- BIレポートむけデータ集計処理
- Extract(Python) -> Transform(Hive query) -> Load
- 脱Jenkins日次バッチ
- 要件
  - 可用性 -> Posgresqlを落とさなければ○
  - スケーラビリティ-> 並列処理
  - 処理フローがコードで管理できる
  - リトライ可能 - 失敗処理
  - 進捗確認ができる -> Rest API

## システム構成

- Postgresql(Primary standby構成 Streaming replication)
- Pgpool-Ⅱ(Active standby構成 Watchdog)

- Digdag server(兼client)を2台でlsyncd & rync

## Workflow設計

- 事業単位で１つのワークフローを構成
  - e.g. ゲーム事業

- Web UIで実行時間とかわかる

## Pluginの話

- slackオペレータ
- myui/digdag-plugin-exampleが参考になる

## チーム間連携

- Polling -> Success -> (another task) Start
- mogという自社CLIでタスクのstatusを取得可能/startもできる
  - mogによりタスクが処理時間依存だったのを解決できた

## ハマリドコロ

- 処理時間が長いWorkflowだと、DigdagのDatabase接続のSocket Timeout

## Q&A

- dev/prodの切り替えは?
  - digdagの起動時パラメータで解決
- task実行中にdigdag serverが死んだときどんな動き
  - taskのStateをDBに保存して、それをPollingでやりとりしている
  - ShellやRuby Scriptは例外
- Taskを実行しないdigdagとTaskを実行するdigdagという構成が古橋さんのおすすめ
  - `--disable-local-agent` optionを利用する

# Digdagによる大規模データ処理の自動化とエラー処理 by @frsyuki

## Slide

- [Digdagによる大規模データ処理の自動化とエラー処理](https://www.slideshare.net/frsyuki/digdag-76749443)

## What's workload automation?

- あらゆる手作業の自動化

## Challenge: Modern complex data analysis

- Ingest -> Enrich -> Model -> Load -> Utilize
- このdata analysisのフローで複数クラウドが当たり前なのだが、これをクラウドに依存させない形で解決したいが大元のモチベーション

## Run anythingのために

- myui/dockernized-digdag-server

## 有向グラフではloopが書けない

- digdagは動的にタスクを増やせるのでloopが書ける

## Digdag at TD

- 850 active workflows
- 3600 workflows run every day

## Q&A

- server mode: clientからpushされたTaskを実行される
- scheduler mode: localにあるWorkflowをリロードして実行する
  - 手元で動かしたいときに便利
- REST APIの仕様は今後かわらない。なぜならTD社でもう動いているから。

# digdag導入でよかったこと&ハマったこと by 塩崎 健弘 氏

- [speakerdeck/Digdagを仕事で使ってみて良かったこと、ハマったこと](https://speakerdeck.com/shiozaki/using-digdag-in-production-environment)

- UbuntuだとPythonオペレータでpython2が実行される
- embulk gem manager的なやつが欲しい
- src読んで発見する機能がある

# とあるマーケティング部隊でのdigdagの活用事例 by grimrose

- [gist/とあるマーケティング部隊でのdigdagの活用事例](https://gist.github.com/grimrose/5bea98db4c82056dad1ab84d6653308e)

- operatorでエラーになっても原因がわかりにくい
