ScalaMatsuri Day1
===

# Akkaで分散システムの障害に備える

- Slide
  - https://www.slideshare.net/yugolf/preparing-for-distributed-system-failures-using-akka-scala-matsuri2017-72575226

- Microservices = Distributed Sysytems
- CPUのパフォーマンスは頭打ちになるので分散システムによるスケーリングを考える.
- serverの数が増える = 障害点が増える
- networkは制御できない
- 機能横断要件の定義が分散システムでは必要
  - availability
  - response time and latency
- Antifragile orgs
- systems based failures
  - timeout
  - bulkhead
  - circuit breaker (Hystrix etc)
- Akka deal with DS failures

- Akka
  - 逐次処理ではなく非同期ベース
  - messageを送信するだけ.その後は自分の仕事をするだけ.
  - 1 actor 1 thread model
  - 位置透過性 同一JVMであったりべつべつのJVMであったり
  - hierarchy / let it crash

- timeout
  - tell (fire & forget)
  - ask 応答が欲しい Future
- circuit breaker
  - 問い合わせ先のサーバが死んでいる場合は,retryのしきい値を超えると処理を切り離す
  - state closed/open/half open
  - decrease latency -> 無駄な問い合わせが減らせる
  - CAP trade off
    - read
      - return old information
      - don't return anything
    - write
      - just do my work
      - need synchronize with others
    - システム要件に適した設計を行う
- bulkhead
  - Akkaではblockingが必要になりそうなActorはActorごと分ける
  - dispatcherのなかでblockingがあるとSleepやWaitingがふえるのでdefaultのdispatcherとは隔離する
- Akka cluster
  - node間でheart beatを投げ合って,nodeが死んでるか否かを確かめ合って死んでるnodeは切り離す
  - akka persistenceでeventsを永続化して他のnodeでもevent情報を復元できる
  - BackOff supervisor replay strategyを実装できる
    - Capped exponential Backoff的なrandom性も設定できるので散らすこともできる
  - split brain resolver
    - ネットワークの影響でnodeが死んでるわけではないのにnodeが切り離されることがある
    - このとき複数のclusterに分断されてしまうようなケースが発生する
    - これを解決するのがsplit brain resolver
      - 特定のnodeが含まれるclusterを残すとか4つの戦略がある
      - 注意はこれはOSSではないのでLightbend Reactive Platformを利用する必要がある
      - OSSでchatwork製のものもある
- networkの影響による異常
  - networkの影響でackが返ってこない時
  - 分散システムでは複数回メッセージを送信しても問題ないようなべき等性を担保した設計が重要

---

# ストリーミング・ワールドのためのサバイバル・ガイド

- Slide
  - https://www.slideshare.net/ktoso/akkachans-survival-guide-for-the-streaming-world

- memo取る余裕なかった.


---

# DMMのAPI GatewayをAkka StreamsとAkka HTTPで作り込んでみた

- Slide
  - https://speakerdeck.com/mananan/implementing-dmm-api-gateway-in-akka-streams-and-http

- DMM EC site
  - 2.5 billion PV/day
- 認可パフォーマンスの向上がリプレースの主な目的
- Architecture
  - front Akka HTTP Server
  - BackendのAPIをAkka HTTP Clientで叩いてfrontに返す
  - StorageはすべてFuture対応
- Akka HTTP
  - low level APIを採用
  - routingを自前で書けて今回の要件を満たすためにhigh level APIでは足りなかった
- Routing flow
  - Not Found Flow/Abort Flowを設ける
  - Akka Streams's GraphDSL
    - Partitionからn分割分岐させるjunctionを書ける
  - Incoming/Outgoing Filter
    - APIを叩く間に挟むFilter
    - 社内事情をここで吸収する.(独自HeaderやREST/非REST APIの互換など)
    - Akka HTTPのUnmashallerでHTTP Entityの解析を行う.便利だった.
- Implementing Router with Graph API
  - 最初の設計実装
   - PartitionとMergeの間について
     - Not Found Route * m
     - Found Route * n
     - Not Found/FoundがすべてMergeにたどり着く
   - Partition sizeが増えるとHeapも増える
   - GCもそれなりに起きていた
  - 見直し後の設計実装
    - Routeごとに１つのPartitionとMergeを割り当てる
    - firstと比べて100-200M heapが減った
    - GCの回数も減少
  - FilterをまとめるのにflatMapConcatを利用
- Testing the Performance Gateway API
  - gatlingを利用
  - GatewayのBackendにFast Endpoint/Slow Endpointの２つをエミュレートしたサーバを配置した負荷テストも実施
    - Gateway全体のスループットが落ちないかを検証するため
- Summary
  - back-pressure controlを利用するとgateway実装としてAkkaは有効
  - Akka HTTPによるProxyはGatewayの特性に適合している
- Q&A
  - Graph構造の中で例えばBlockingが入るような重たい処理が入るときはどう制御している?
    - Blockingが入るような重たい処理はdispatcherを分けている
  - Graphの中で制御できない予期できない例外が発生した場合はどうなる?
    - Akka HTTPで503等のResponseを返すことが保証できる

---

# Akka Streams による Kafka の Reactive 化

[@kpciesielski](https://twitter.com/kpciesielski)

- akka-stream-kafkaのコミッタ
- kafka
  - producerがtopicを書き込み,consumerが消費する
  - topicはpartitionで分割できる
  - partitionはコピーではない
  - consumerに複数のpartitionからtopicを送信することをbalancingと呼ぶ
    - consumerはpartitionが複数あることは意識しない
  - consumerはclusterを組める
    - groupのような識別子でグルーピング
    - kafkaは同じclusterに所属するconsumerがある場合は並列にそれぞれに送信できる
  - consumerとproducerは粗結合
    - この特徴がmicroservicesで重要
  - consumerはあるタイミングでACKを返すことができる
    - kafkaはACKをうけとるとコミットする?
- Akka Streams
  - アクターモデルを使用したデータ変換パイプラインDSL
  - backpressure
  - powerful testkits for async
  - 拡張可能(様々な技術のコネクターを書ける)
  - Stage
    - Input: Source
    - In/Out: Flow
    - Output Sink
- Akka Streams + Kafka
  - Consumer: Source
  - Producer: Sink
  - Producer: Flow writeに対してもbackpressure
  - Akka ecosystem内で開発中
  - examples
    - plain consumer: commitなしのSource?
      - play Java Kafkaを利用するとReactive性が落ちる.ループを書いてポーリングするような処理が必要.
      - Akka plain consumerを使った実装のほうがplain consumerの実装とほぼ同じパフォーマンスを出せる.
    - commitable source
      - writeではBlockingが必要
      - 100件とかになるとbufferingして一度にcommitとかを検討する(Batch)
      - Akka Streams Kafkaでは`mapAsync`, `grouping`等を利用してAt-least-onceを保証する.
      - Akka batched consumerのメッセージ処理パフォーマンスはまだbatched consumerに届かない.
    - producer
      - plain JavaのAPIを利用すると処理をCallbackを用いて書く必要が出てくる.Callbackは管理が難しくなる.
      - Akka Streams KafkaではMessageを作成して次のStageに流していくような記述ができる.
  - partitionごとのbackpressure: `Consumer.commitablePartitionSource(consumerSettings, Subscriptions.topic("topic1"))`
    - `map`メソッドでパターンマッチで`partition`情報などが取れる
  - Error handling
    - `toMat`でマテリアライズする
    - `streamFuture.onFailure`で例外を処理できる
    - `control.shutDown()`でシャットダウンもできる
  - Kafka Streams
    - Apache Kafka製の別物のライブラリがある
    - Java, stateful processors, windowing, joining, aggregation
    - Akka Streams Kafkaはasyncにフォーカスしている

---

# Scala ＆ Sparkによるデータ・エンジニアリング 7大レシピ

[@ahoy_jon](https://twitter.com/ahoy_jon)

- 1. organization
  - データエンジニアリングではデータを移動したり取得したりするツールに問題があると思われがち.
  - しかし、組織が問題になることにも目を向ける必要がある.
  - ビジョンを共有しないそれぞれのチームにデータを公開すると,その依存を考えながら仕事をすることになる.
    - ビジョンを明文化することが重要.
    - チームとしてチームを超えて協力することを明確にする.
- 2. Work optimization 
  - リードタイム: 企画から完成までの期間
  - インパクト: 今の文脈を超えた良い効果
  - 失敗の管理: 
  - 何かを処理している間(MapReduce)に待つ(Wait)ことを減らすには?
    - あるコンポーネントで何が失敗しそうか考え,その頻度や予防策,復旧プランを考える
- 3. Staging Data
  - ステージングは永続化構造として見えるようにすべき
 
- 5. Cogroup
  - IOモナドに似ている
  - data連結にとても有用
  ```scala
  from(left: RDD[(K, A)], right: RDD[(K, B)])
    .join()
    .outerJoin(Option[A], Option[B])
    .cogroup(Seq[A], Seq[B])
  ```

- 6. Inline data quality
  - データ品質を高めるとバッドデータへのレジリエンスを高める?
  - Annotationsはaggr
- 7. Real programs
  - データパイプラインの各パートはstatelessにデザインすべき

---

# ChatWorkのScala採用プロダクト “Falcon” リリースまでの失敗と成功歴史

- ユーザが生成するデータのスケール遷移
  - 1年前10億メッセージ、直近だと18億メッセージ
  - サービスの成長速度を優先し技術的負債は他社事例にあるように例外なく貯まる
  - サービス成長とともにメッセージを捌ききれなくなりサービスダウン等を引き起こす
- Phase1: Live migration
  - 既存の安定しているシステムに影響を与えない
  - 新システムではダウンタイムなしにmigrateしたい
  - 既存データのmigrationはしない
  - プロジェクトの終盤で様々な問題が発生して一旦プロジェクトを停止
  - 3日使って振り返りを行う
- Rebooting the project
  - projectの体制を見直しマネジメント面を強化
  - Chatworkのインフラとなるのでrobustのものを目指す
  - PoC
  - システムが満たすべき要件
    - Scalabiity
    - Resiliency
    - concurrent connectionsとthroughput2倍を目指す
    - Low cost
    - DDDをベースとした機能性
- PoC
  - chat roomとmemberを含むメッセージ機能にスコープを決める
  - 圧倒的にReadが多いのでCQRS
  - Writeはcasandra.Akka cluster/Akka HTTP/Akka Stream/Akka Persistence
  - ReadはAWS Aurora. Akka HTTP/Akka Stream/Skinny ORM
  - ReadとWriteの同期は非同期にイベント駆動で
  - パフォーマンス的にはなかなかの結果が得られた
  - Akka clusterは運用にのせるためには回避出来な問題がわかり見送りに.(split brain)
  - Casandraでは障害が発生したnodeを復旧するために24時間という見積もりに
  - Auroraはsingle masterでは書き込みがスケールしない.shardingで改善もできるが開発の難易度が高く属人化しやすい.
- Phase2: Re-Architecture from PoC
  - akka clusterは運用コストから見送りに
  - Write DBとしてCasandraではなくKafkaに置き換える
  - Read DBはAuroraからHBaseに変更
  - chatworkではmessaingがコアドメインなのでここにフォーカス
  - DDD Context MapがPhase1よりシンプルになったためコミュニケーションコストが下げれた
  - AuroraからHBaseへのdata migration
    - Basic migration: 最終メンテから4日前までのすべてのデータを以降
    - Diff migration: Basic migrationの対象期間以降のデータ
- DevOps
  - kubernetes
  - kube-aws AWS上にコンテナクラスター組める
  - Jenkins -> Concourse CI (developed by pivotal)
- Finally Release
  - 低遅延等、高スループット等はいい結果が得られた.
  - 今後も改善
  - Scala community応援ありがとう!

---
## あとがき

- セプテーニ・オリジナルの技術読本の「Akka HTTPでLINE bot作ってみました」はサンプルとして面白かった.
- スタッフのみなさんお疲れ様でした!
