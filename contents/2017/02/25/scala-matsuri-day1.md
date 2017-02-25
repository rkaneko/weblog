ScalaMatsuri Day1
===

# Akkaで分散システムの障害に備える

- Microservices = DS
- CPUのパフォーマンスは頭打ちになるので分散システムによるスケーリングを考える.
- serverの数が増える = 障害点が増える
- networkは制御できない
- 機能横断要件の定義が分散システムでは必要
  - availability
  - 
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


---

# DMMのAPI GatewayをAkka StreamsとAkka HTTPで作り込んでみた

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
