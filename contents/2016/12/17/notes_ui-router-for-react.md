UI-Router for React
===

# New concepts

いくつかのコンセプトとそれを実現するためのUI-Routerの機能を見ていく.

## Resolve data

ユーザがSPAにおけるstateを行ったり来たりするときに,アプリケーションはREST endpointのようなサーバAPIを使ってdataを取得する必要がある場合がある.

`resolve`プロパティを定義することで、stateは必要なdataを特定できる.ユーザが`resolve`プロパティを持つstateをactivateすると,UI-Routerはstateをactivateする前にデータをfetchする.fetchされたdataは`props`経由でcomponentに渡される.

`resolve`

```js
resolve: [
  {
    token: 'people',
    resolveFn: () => PeopleService.getAllPeople()
  }
]
```

- `resolve`は`array.<object>`.
- arrayのElementにfetchされるべきdataをobjectで定義する.
  - `token`プロパティはロードされたdataのためのDI token.
  - `resolveFn`プロパティはdata取得のためのpromiseを返す関数.返されたpromiseから取得されたdataはDI tokenに代入される.
  - `deps`プロパティには`resolveFn`の依存関係を定義する.`resolveFn`の関数の引数としてDI tokenを指定する.`array.<string>`で指定する.

DI tokenについて

ソースを軽く読んでみた感じだと、`UIView`の子componentに[Reactのcontext](https://facebook.github.io/react/docs/context.html)の仕組みを使ってpropsに渡していっているみたい？詳しくはReact力が足りなくてわからなかった。

https://github.com/ui-router/react/blob/9745555cff70dc1cb7fa510ada1e2644a21bfbd8/src/components/UIView.tsx

`resolve`の`token`に`'people'`を指定し、`resolveFn`によって取得されたdataは各componentの`props`に渡される。

```js
People.propTypes = {
  resolves: PropTypes.shape({
    people: PropTypes.arrayOf(PropTypes.object)
  })
}
```

## State Parameters

RESTのendpointがリソースに対する識別子などを持つ場合に,この情報を元にdataをfetchしたい場合がある。

`/people/:personId` のようなURLで`personId`がpeopleの識別子を表す場合に,`personId`を使ってデータを取得したいようなケース.

personのstateは次のようになる.

```js
const person = {
  name: 'person',
  url: '/people/:personId',
  component: Person,
  resolve: [{
    token: 'person',
    deps: ['$transition$'],
    resolveFn: (trans) => PeopleService.getPerson(trans.params().personId)
  }]
}
```

`deps`プロパティに`'$transition$'`を指定することで,`resolveFn`に`Transition` objectがinjectされる.Transition objectを経由してURL resourceにアクセスすることができる.
`Transition` objectは現在のhistoryの状態遷移についての情報を保持する特別なinjectableなobject.

refs: https://developer.mozilla.org/en-US/docs/Web/API/History_API

## Linking with params

UI-router for reactで管理するhistoryのstateに対して,ｈｔｍｌの`<a>`のようなtagではstatの変化eventを発火させることができない.そのために`<UISerf>`といったReact componentが提供されている.これを利用してrouterにstate変化event(ここでは,person detailページに遷移するというevent)を発火させる.

```js
const ToPerson = (props) => {
  return (
    <UISerf to="person" params={{ personId: props.person.id }}>
      <a>{props.person.name}</a>
    </UISerf>
  );
};
```
