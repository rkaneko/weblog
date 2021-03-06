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
# Observer

`Observer`は`Observable`から送信された値に対するConsumer.

Observerは

- `next`
- `error`
- `complete`

Observableから送信された各typeの値のcallbackを持つObject.部分的な定義も可能.

```js
const observer = {
  next: x => console.log(`Observer got a next value: ${x}`),
  error: err => console.error(`Observer got an error: ${err}`),
  complete: () => console.log(`Observer got a complete notification`),
};
```

仮に,`Observable.subscribe()`の引数にObserverとしてのObjectではなく,functionを渡した場合は,subscribe内で引数で受け取ったfunctionをnext notificationを受け取るcallbackとしてobserver Objectを生成する.また,3つのtypeに対するcallbackをすべて関数として渡すこともできる.

---
# Subscription

`Subscription`は

- 破棄可能なリソースで通常は実行中のObservableを表現するObject.
- Subscriptionが保持するリソースの解放や,実行中のObservableをキャンセルするための`unsubscribe` function(引数を取らない)を持つ.

```js
const observable = Rx.Observable.interval(1000);
const subscription = observable.subscribe(x => console.log(x));
// later
subscription.unsubscribe();
```

リソースを明示的に解放する必要がある場合は,`Observable.create`に渡すsubscribe functionの戻り値にリソース解放のための関数を返すようにする.

see also

- define-dispose.js

subscriptionは親子関係を定義することもできる.

```js
const subscription = observable1.subscribe(x => console.log(x));
const childSubscription = observable2.subscribe(x => console.log(x));

subscription.add(childSubscription);

setTimeout(() => {
    // unsubscribe parent and child subscription
    subscription.unsubscribe();
}, 1000);
```

see also

- unsubscribe-multi-subscription.js

`Subscription.add(otherSubscription)`で定義した関係をundoするために`Subscription.remove(otherSubscription)`がある.

---
# Subject

`Subject`は特別なtypeの`Observable`.

- 複数のObserverに対し値を送信できる.
- EventEmitterのようなもので,複数のlistenerを管理する.
- `Subject.subscribe`は`Observable.subscribe`とは異なり,値を送信する新しい実行呼び出しではなく,Observer listに新たなObserverを登録するのみ.(`addListener`のようなもの.)
- すべてのSubjectはObservableでもあり,Observerでもある.
  - `next(v)`,`error(e)`,`complete()`メソッドを持つ.
  - Subjectに対し新しい値を与えるには,`next(theValue)`メソッドを呼ぶ.これにより,SubjectをlistenしているすべてのObserverに対しマルチキャストされる.

マルチキャストの例

see also

- multicast-to-observers.js

SubjectはObserverでもあるので`Observable.subscribe`の引数として渡すこともできる.
これにより,ひとつのObserverへのキャストから複数のObserverに対してSubjectを介してマルチキャストすることへの変更も可能となる.

Subject typeには,より特化した`BehaviorSubject`,`ReplaySubject`,`AsyncSubject`がある.

## Multicasted Observables

Observable, Subject, 複数のObserversによるマルチキャストの仕組みは以下の通り.

- 複数のObserverはSubjectに対しsubscribeする.
- SubjectはsourceとなるObservableをsubscribeする.

マルチキャストを実現する方法は2つある.

- `Observable.subscribe(subject)`を使う.
- `Observable.subscribe(subject)`を内部的に実行する`ConnectableObservable.connect()`を使う.

`multicast()`はsubscribeされたときにSubjectのように複数のObserverにキャストする振舞いを行う`ConnectableObservable`を返す.

`ConnectableObservable.connect()`は`Subscription`を返すのでshared Observable executionをキャンセルするために`Subscription.unsubscribe`を呼び出す.

see also

- multicasted-observable.js

### Reference counting

`ConnectableObservable.connect()`を明示的に呼び出して,Subscriptionをハンドリングするのは扱いづらいケースもある.最初のObserverが到達したら自動的にconnectされ,最後のObserverがunsubscribeしたらshared executionを自動的にキャンセルしたいというのが一般的.

`Observable.refCount()`を使うと,上記の要件を満たす実装が可能.

see also

- refcount.js

---

## BehaviorSubject

"the current value"という概念をもつSubject.Consumerにemitした値を"the current value"として保持し,新しいObserverが増えた時は即座に"the current value"をObserverは受け取ることができる.

see also

- behavior-subject.js

## ReplaySubject

old valuesとしてObservable executionの一部を保持しておくことができる.

- buffer sizeで固定長の値を保持.
- window timeをミリ秒単位で保持時間を指定する.

see also

- window-time-replay-subject.js

## AsyncSubject

`AsyncSubject`はObservable executionの最後の値のみが,`AysncSubject.complete()`が呼ばれた時に送信される.

AsyncSubjectは`last()` operatorに似ている.

see also

- async-subject.js

---

# Operators

RxJSで核となるのはObservableだが,operatorsのおかげで複雑な非同期処理もコンポーザブルで宣言的な書き方が可能になる.

## What are opeartors?

operatorsはObservableのメソッド(map, filter, merge, etc).呼び出されるとObservableインスタンスに副作用を与えるのではなく新しいObservableインスタンスが返される(pure function).

基本的な考え方は,inputとしてObservableを受け取り,inputをSubscribeする新たなObservableを生成してoutputとして返す.

see also

- operators-example.js

## Instance operators versus static operators

instance methodはわかりやすい.

```js
Rx.Observable.prototype.multiplyByTen = function multiplyByTen() {
  const input = this;
  return Rx.Observable.create(function subscribe(observer) {
    input.subscribe({
      next: v => observer.next(10 * v),
      error: err => observer.error(err),
      complete: () => observer.complete()
    });
  });
}

const observable = Rx.Observable.from([1, 2, 3, 4]).multiplyByTen();
observable.subscribe(x => console.log(x));
```

static methodはObservable classにアタッチされるpure functionのこと.例としては,`interval`のようにミリ秒ごとに値をemitするようなものが挙げられる.

```js
var observable = Rx.Observable.interval(1000);
```

`interval`や`create`のようなObservableを生成するようなメソッドが異なる性質を持つstatic methodもある.`merge`や`combineLatest`,`concat`のように引数として複数のObservableをとるようなメソッドもある.

## Marble diagrams

Observableの実行系列やoperatorsによるinput/outputの変化等を見るためにはmarble diagramsが表現しやすい.

see also:

http://reactivex.io/rxjs/manual/asset/marble-diagram-anatomy.svg

## Choose an operator

どのoperatorsを使えばよいかを探す方法として以下を使うと良い.

- http://reactivex.io/rxjs/manual/overview.html#choose-an-operator

---
# Scheduler

Schedulerはsubscriptionがいつ開始するか,通知がいつ送信されるかを管理するもの.

次の3のコンポーネントから成る.

- data structure: 優先度や他の基準を基にしてどのようにtaskを保持したりキューイングしたりするかを責務とする.
- execution context: いつどこでtaskが実行されるかを示す.(e.g.即時実行,setTimeoutやprocess.nextTickやanimation frameといったコールバックにおける実行)
- (virtual) clock: `now()`によって"time"の概念を提供する.特定のSchedulerによってスケジューリングされたtaskはclockで示されるtimeにのみ従って実行される.

SchedulerはObservableがObserverに対しどのような実行コンテキストで通知を送信するかを定義するためのもの.

サンプルでは,1,2,3という値を同期的にひとつのObservableがemitし,emitする値を送信するための`async` schedulerを指定している.

see also

- how-do-you-define-execution-context.js

`Rx.Scheduler.async.schedule()`メソッドを使用することでSchedulerが内部にもつclockで管理される時間によってdelayを実現することもできる.

## Scheduler Types

|Scheduler | Purpose |
|:---|:---|
|null|scheduleｒを何も指定しなければ,通知は同期的かつ再帰的に送信される.固定時間の操作や再帰的な操作に使用する.|
|Rx.Scheduler.queue|現在のイベントフレームにおけるqueueをスケジューリングする.繰り返し操作のために使用する.|
|Rx.Scheduler.asap|とても小さなtask queueをスケジューリングする.Node.jsのprocess.nextTick()やWeb Worker MessageChannelやsetTimeout等の利用可能な最速の送信メカニズムを使用する.非同期での対話に使用する.|
|Rx.Scheduler.async|setIntervalを使ってスケジュールが動作する.時間を指定したいような操作で使用する.|

## Using Schedulers

Schedulerとしてどのタイプのもの使うかを明示的に指定することなくSchedulerを使用することもできる.

並列処理を扱えるすべてのObservable operatorsはoptionalなSchedulerを持っている.もしSchedulerを指定しなければRxJSは最低並列処理の原則に従ってデフォルトのSchedulerを選択する.これにより使用されるoperatorの要件を満たす最小の並列度をSchedulerは選択するということを意味する.例えば大量で無限のメッセージを返すようなoperatorに対してはqueueタイプのSchedulerが使用され,時間を使用するようなoperatorの場合にはasyncタイプのSchedulerが選択されたりする.

パフォーマンスを最適化したい場合に,デフォルトとは異なるタイプのSchedulerを選択することができる.

static creation operatorは`from(array, scheduler)`のように引数にSchedulerを取れることが一般的な設計になっている.

`Observable.subscribe()`は同期的かつ即座にObservableが発生する.`Observable.subscribeOn()`の引数としてschedulerを指定するとどんなコンテキストでsubscribeするかをスケジューリングすることができる.

`Observable.observeOn()`はどのようなコンテキストで通知が送信されるかをスケジューリングすることができる.source Observableとdestination Observerの間のmediatorが指定されたSchedulerを使ってdestination Observerをコールする.

`bufferTime`や`debounceTime`,`throttleTime`,`timeInterval`といった時間に関連するObservable instance methodも引数にSchedulerタイプを指定することができる.デフォルトは`Rx.Observable.async`となっている.

ReplaySubjectも時間を扱うのでSchedulerを指定することができる.デフォルトは与えられたclockに対してqueue Schedulerを試用する.
