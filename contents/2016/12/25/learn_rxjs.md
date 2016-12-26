Learn RxJS
===

### Preconditions

- RxJS ver5.x

### materials

- http://reactivex.io/rxjs/manual/overview.html

---

# Introduction

RxJSはobservable sequencesを用いて非同期イベント駆動プログラミングを実現するためのライブラリ.
以下を提供する.

- Observable typeとそれに伴うObserver, Schedulers, Subjects type.
- `map`, `filter`, `reduce`などの非同期イベントをコレクションのように操作するためのoperators.

イベントシーケンスの理想的な管理を目指す.

essential concept

- `Observable`: まだ処理されていない値やイベントの呼び出し可能なコレクションの概念を表す.
- `Observer`: `Observable`から受け取った値のどう処理するかの責務を担うコールバックのコレクション.
- `Subscription`: `Observable`の実行を表す.キャンセル処理も行える.
- `Subject`: EventEmitterと等価.値を複数のObserverにマルチキャストする唯一の手段.
- `Schedulers`: 並列処理を管理するdispatcherの集合.`setTimeout`や`requestAnimationFrame`等が起こった時の計算処理をコーディネートすることができる.


# Observable

## Pull versus Push

PullとPushはdata Producerがdata Consumerとどのようにコミュニケーションできるかということに関して異なるプロトコルである.

Pull型のシステムでは,ConsumerはいつProducerからdataを受け取るかを決めている.Producer自身はいつConsumerにdataが届くについては関心がない.

すべてのJavaScript functionはPull system.functionはdata Producerである.

ES2015で提供されるgenerator functionとiteratorは別のタイプのPull systemと言える.`iterator.next()`はConsumerで,`iterator`(Producer)から複数の値をpullする.

| |Producer|Consumer|
|:---:|:---:|:---:|
|Pull|Passive|Active|
|Push|Active|Passive|

Push型のシステムでは,ProviderがいつdataをConsumerに送信するかを決める.Consumerはいつdataが届くについては関心がない.

今日のJavaScriptではPromiseがPush型システムの代表的な型と言える.Promiseは解決された値を登録されているコールバック関数に送信する.

RxJSはJavaScriptに新たなPush型システムとしてObservableを導入する.*Observable
は複数の値のProducerでそれらの値をConsumer(Observer)にpushする.*

- `function`は呼び出し時に一つの値を同期的に返す遅延評価演算.
- `generator`は呼び出し時にzeroから無限個の値を返す遅延評価演算.
- `Promise`はいつか一つの値を返すか失敗するかの演算.
- `Observable`は呼び出し時からzeroから無限個の値を同期的もしくは非同期的に返すことができる演算.

## Observables as generalizations of functions

一般的な主張とは反対に,ObservablesはEventEmitterでも複数の値のためのPromiseでもない.
RxJS Subjectsを用いてEventEmitterのように振る舞うときもあれば,EventEmitterのように振る舞わないときもある.

see also observable-as-a-generator.js

Observableは処理と実行を分けている.EventEmitterはSubscriberの存在有無に関わらず実行と処理内の副作用を共有してしまう.この点でObservableはEventEmitterとは異なる.

Observableは同期的に一つの値を送信できるという点ではfunctionと同じことができる。しかしながら,functionにはできないことができる.

see also

- observable-can-deliver-multi-values.js
- observable-can-deliver-multi-values-async.js

### Conclusion

- `func.call()` は同期的にひとつの値を与える.
- `observable.subscribe()` はいくつかの値を同期的にも非同期的にも与える.

---

## Anatomy of an Observable

`Observable`は

- `Rx.Observable.create`やcreating operatorによって生成される.
- Observerによってsubscribeされ,ObservableからObserverにはObserverの`next`,`error`,`complete`メソッドを介して通知が伝達される.
- Observableの実行はdispose(廃棄)される.

### Creating Observables

`Rx.Observable.create`は`Observable`のconstructorのalias.引数に`subscribe` functionを取る.

```js
const observable = Rx.Observable.create(function subscribe(observer) {
  const id = setInterval(() => {
    observer.next('hi');
  }, 1000);
});
```

`create`によって`Observable`を生成できるが,通常は`from`や`interval`のようなcreation operatorを使うことが多い.

### Subscribing to Observables

生成されたObservableインスタンスは以下のようにして`subscribe`することができる.

```js
observable.subscribe(x => console.log(x));
```

`Observable.subscribe`と`Observable.create()`の引数の関数名は同じである必要はないが,実用上概念的には同じとみなすことができる.

`Observable.subscribe`は"Observable execution"を開始し,実行における**一つのObserver**に対し値やイベントを送信するシンプルな方法の一つ.

`Observable.subscribe`は`addEventListener`や`removeEventListener`のようなevent handler APIとは根本的に異なる.

### Executing Observables

Observable Executionによって送信することができる値のタイプが3種類ある.

- "Next" notification: `Number`,`String`,`Object`のような値を送信する.
- "Error" notification: JavaScript errorやexceptionを送信する.
- "Complete" notification: 値を送信することはできない.

"Error"と"Complete"はObservable Exection中に一度しか呼び出しは起こらない.そしてどちらかが呼び出された場合は,もう片方が呼び出されることはない.

> 一度のObservable Exectionにおいて,zeroから無限のNext notificationが送信される.もし,ErrorかComplete notificationが送信されたら,それ以降は何も送信されない.

see also

- observable-contract.js

### Disposing Observable Executions

Observable Executionは無限になりうるし,有限時間内にObserverがExecutionを破棄したいようなケースもある.これを満たすために,cancellation APIが必要になる.

一つのExecutionにつきただひとつのObserverに対して排他的なので,いったんObserverが値を受け取り終えたら,無駄な計算やメモリリソースを避けるためにExectionを止める手段を持っている必要がある.

`Observable.subscribe`はObservable executionインスタンスが新たに生成される.これをSubscriptionと呼ぶ.

```js
const subscription = observable.subscribe(x => console.log(x));
```

Subscriptionは実行中のExecutionを表し,Executionをキャンセルするための最小限のAPIを提供する.

`Subscription.unsubscribe()`で実行中のExecutionをキャンセルすることができる.

```js
const observable = Rx.Observable.from([10, 20, 30]);
const subscription = observable.subscribe(x => console.log(x));
subscription.unsubscribe();
```

`Observable.create()`を使ってObservableインスタンスを生成するときは,すべてのObservableは実行中のリソースをどう破棄するかを定義する必要がある.(must)

see also

- define-dispose.js

---
