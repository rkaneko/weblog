React RxJS using recompose
===

[RxJS](https://github.com/ReactiveX/rxjs)をReactと組わせる際にどういうふうにやるかについて調べた時のメモ.
いくつか方法はあるみたいだけどどのように組み合わせるかを知りたかったのでミニマルにできそうな[recompose](https://github.com/acdlite/recompose/)をこのエントリではとりあげる.

RxJSのmanualにもrx-react-componentを利用したサンプルがある.

- http://reactivex.io/rxjs/manual/tutorial.html#react

### 前提

- node v6.9.0
- RxJS v5.0.2
- react v15.4.1
- recompose v0.21.2

### 対象読者

- ReactやRxJSについては特に説明しないので,そのあたりの知識がある人

### recomposeについて

Higher-order Components(HoCs)と呼ばれる,React Component(Base Component)を受け取り,新しいReact Component(Enhanced Component)を返す関数を集めたライブラリがrecomposeである.

```js
const EnhancedComponent = hoc(BaseComponent)
```

HoCsの使用例としては,

react-reduxの`connect()`関数が挙げられる.connect関数でWrapされたReact Componentは`SomeComponent.contextTypes`に親から受け取りたい値の定義が追加され,[React Context](https://facebook.github.io/react/docs/context.html)の仕組みが適用されて,親Componentから指定した値が暗黙的にpropsとして渡されるといったことを実現することができる.

react-reduxのconnect()関数とProvider Componentを利用することによってstateの変化をSubscribeする責務をライブラリ側が担ってくれるというメリットを享受することができる.

HoCsの別の利用例としては,レンダリングパフォーマンスチューニングをする例も挙げられる.`pure()`というHoCで,React ComponentのLifeCycleメソッドの一つである`shouldComponentUpdate(nextProps)`をpropsのshallowEqualsで判断するような拡張を行う.

### recomposeを使ったReactとRxJSの組み合わせ

recomposeは[Observable utilities](https://github.com/acdlite/recompose/blob/c7efea4dd5de14975d7ae32cd37396eec72c087e/docs/API.md#observable-utilities)として,ReactとStreamライブラリを組み合わせるAPI群を提供している.

- `createEventHandler()`
- `componentFromStream()`
- `mapPropsStream()`

#### `setObservableConfig()`

Observable utilitiesはデフォルトでは[ES Observable proposal](https://github.com/tc39/proposal-observable)の実装Observableを使うようになっている.RxJSのObservableをStreamとして使用するために設定を行う必要がある.

```js
'use strict';

const Rx = require('rxjs');
const {setObservableConfig} = require('recompose');
setObservableConfig({
  fromESObservable: Rx.Observable.from
});
```

設定プロパティは`fromESObservable`の他にも`toESObservable`もあるが,今回使うことはなかったし,定義しなければ引数を受け取ってそのまま引数を返す関数としてデフォルトの設定になっているの何もしていない.

#### `createEventHandler()`

`createEventHandler()`はhandler(publisher)とhandlerがpublishする値をStreamとして扱うStream(Observable)のPairが提供される.

```js
'use strict';

const React = require('react');
const {Component} = React;
const {createEventhandler} = require('recompose');

const Counter = ({count = 0, handleIncrement}) => {
  return (
    <div>
      Count: {count}
      <button type="button" onClick={handleIncrement}>+</button>
    <div>
  );
};

class App extends Component {
  constructor(props) {
    super(props);
    this.state = {count: 0};
  }

  componentDidMount() {
    const {handler, stream} = creatEventHandler();
    this.incrementSubscription = stream.mapTo(1).startWith(0).subscribe({
      next: v => {
        const count = this.state.count + v;
        const nextState = Object.assign({}, this.state, {count});
        this.setState(nextState);
      }
    });
    this.handleIncrement = handler;
  }

  componentWillUnmount() {
    if (this.incrementSubscription) {
      incrementSubscription.unsubscribe();
    }
  }

  render() {
    const {count} = this.state;
    const counterProps = {count, handleIncrement: this.handleIncrement};
    return (
      <Counter {...counterProps} />
    );
  }
}

module.exports = App;
```

適切なReact Lifecycleメソッド内でのStream関連のリソース解放をしている.Streamのリソース解放やその他のReact Lifecycleメソッドにおけるpropsのsubscribe等の責務を抽象化したHoCが`componentFromStream()`にあたる.

#### `componentFromStream()`

```js
'use strict';

const Rx = require('rxjs');
const {createEventHandler, componentFromStream} = require('recompose');

const Counter = require('./path/2/counter.js');

const {stream, handler: handleIncrement} = createEventHandler();
const increment$ = stream.mapTo(1).startWith(0);

const count$ = increment$
  .scan((count, diff) => count + diff, 0);

const EnhancedComponent = componentFromStream(props$ => {
  return props$.combineLatest(
    count$,
    (props, count) => {
      const nextProps = {...props, count, handleIncrement};
      return (
        <Counter {...nextProps} />
      );
    }
  );
});

module.exports = EnhancedComponent;
```

`componentFromStream()`には,Componentに渡ってくるpropsのStream(`props$`)を受け取り,React ElementのStreamを返す関数(recompose内ではpropsToVdomと表現されている)を引数として渡す.

`componentFromStream()`が行う処理は,

- `props`のStream(`props$`)とこのStreamの値をpublish(emit)するPublisher(`propsEmitter`)を生成し,「propsのStreamを受け取ってReact ElementのStreamを返す関数(`propsToVdom`)」に`props$`を渡してReact ElementのStream(`vdom$`)を生成する.
- React Lifecycleメソッド`componentWillMount`内で`vdom$`のsubscribeを開始し,Subscriptionを生成する.subscribe関数ではpublishが呼ばれるたびにReact Elementが渡ってくるのでその値を`setState()`する.また,initial propsを`propsEmitter`でpublish(emit)する.
- React Lifecycleメソッド`componentWillReceiveProps(nextProps)`内でpropsを受け取ったタイミングでpropsを`propsEmitter`でpublish(emit)する.
- React Lifecycleメソッド`componentWillUnmount()`内でSubscriptionをunsubscribeする.
- `render()`メソッドではstateのReact Elementを描画する.

#### `mapPropsStream()`

これは内部的には,`componentFromStream()`を使っている.

```js
'use strict';

const {createEventHandler, mapPropsStream} = require('recompose');

const Counter = require('./path/2/counter.js');

const {stream, handler: handleIncrement} = createEventHandler();
const increment$ = stream.mapTo(1).startWith(0);

const count$ = increment$
  .scan((count, diff) => count + diff, 0);

const withCount = mapPropsStream(props$ => {
  return props$.combineLatest(
    count$,
    (props, count) => {
      const nextProps = {...props, count, handleIncrement};
      return {...nextProps};
    }
  );
};

const EnhancedCounter = withCount(props => {
  return <Counter {...props} />
});

module.exports = EnhancedCounter;
```

`mapPropsStream()`には「受け取るpropsのStream(`props$`)を受け取り,新たなpropsのStreamを返す関数」を渡す.これで「propsを受け取ってReact Elementを返す関数」(例では`withCount()`)が生成される.

## まとめ

- 小さめのサンプルでReactとRxJSの組み合わせの仕組みを見た.
- recomposeのObservable utilitiesを使うことで,Streamのリソース解放や適切なタイミングでのpublish等を気にする必要がなくなる.
- React SFCとHoCによるコンポーネント化の具体例を見た.


ある程度複雑なアプリじゃないとRxJS使うメリットは無いけど,Reactとの組み合わせ方は把握できたので,もっと複雑な例で素振りしてみたい.
