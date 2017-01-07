Learn redux-observable
===

### materials

- https://redux-observable.js.org/

# Epics

Epicはredux-observableにおいて根本となる概念.

actionのStreamを受け取って,actionのStreamを返す関数.

```js
function (action$: Observable<Action>, store: Store): Observable<Action>;
```

# Examples

## Invoking Web APIs asynchronously

- Web API module

```js
'use strict';

require('isomorphic-fetch');

function fetchUsers() {
  return fetch(`/apis/users`)
    .then(response => response.json());
}

module.exports = {
  fetchUsers
};
```

- Web API epic module

```js
'use strict';

// You should import RxJS in an entry file for using RxJS operators
const Rx = require('rxjs');

const actionTypes = require('./path/2/action-types.js');
const webApi = require('./path/2/web-api.js');

const fetchUsersEpic = action$ => (
  action$.ofType(actionTypes.FETCH_USERS)
    .do(action => console.log(action))  // debug
    .mergeMap(action => Rx.Observable.fromPromise(webApi.fetchUsers()))
    .map(json => {
        const {users} = json;
        return {
          type: actionTypes.RECEIVE_USERS,
          payload: {users}
        };
    })
    .do(action => console.log(action))  // debug
);

module.exports = {
  fetchUsersEpic
};
```

- redux reducer module

```js
'use strict';

const actionTypes = require('./path/2/action-types.js');

const initialState = {
  users: []
};

function reducer(state = initialState, action) {
  const {type, payload} = action;
  switch (type) {
  case actionTypes.RECEIVE_USERS:
    const {users} = payload;
    return {...state, users};
  default:
    return state;
  }
}

module.exports = reducer;
```

- redux store module

```js
'use strict';

const {applyMiddleware, createStore} = require('redux');
const {combineEpics, createEpicMiddleware} = require('redux-observable');

const reducer = require('./path/2/reducer.js');

const {fetchUsersEpic} = require('./path/2/epic.js');

const rootEpic = combineEpics(
  fetchUsersEpic
);
const epicMiddleware = createEpicMiddleware(rootEpic);

const store = applyMiddleware(
  epicMiddleware
)(createStore)(reducer);

module.exports = store;
```

あとはViewからactionをdispatchするhandlerが呼ばれると,epicMiddlewareによってepicが処理される.

注意したいのは,actionがdispatchされると当然reducerにもactionが伝わるので,該当actionの処理を書いていれば当然その処理が走るし,書いていないければdefaultの処理が走る.

> REMEMBER: When an Epic receives an action, it has already been run through your reducers and the state updated.

Epic内で`store.dispatch()`を呼ぶのは控えたほうが良いみたい.anti-patternsと考えられていて将来的にはAPIからなくなる可能性あり.

> Using store.dispatch() inside your Epic is a handy escape hatch for quick hacks, but use it sparingly. It's considered an anti-pattern and we may remove it from future releases.

# Accessing the Store's State

epic関数の中で同期的にstoreのcurrent stateにアクセスすることもできる.

```js
const incrementIfOddEpic = (action$, store) => (
  action$.ofType(INCREMENT_IF_ODD)
    .filter(() => store.getState().counter % 2 === 1)
    .map(() => {type: INCREMENT});
```

# Combining Epics

redux-observableのcombineEpics関数はObservable.merge(epic...)のWrapper.

# Cancellation

例えば,`Rx.Observable.takeUntil()`とactionをdispatchすることを組み合わせて以下のようにもできる.

```js
const fetchUsersEpic = action$ =>
  action$.ofType(actionTypes.FETCH_USERS)
    .mergeMap(action =>
      Rx.Observable.fromPromise(webApi.fetchUsers())
        .map(json => {
          const {users} = json;
          return {
            type: actionTypes.RECEIVE_USERS,
            payload: {users}
          };
        })
        .takeUntil(action$.ofType(actionTypes.FETCH_USER_CANCELLED))
    );
```

ポイントとしては,メインのEpicを`takeUntil`するのではなく,ajaxコールのStreamを`takeUntil`しているところ.Epic自体をキャンセルしたいのではなく,非同期処理のみをキャンセルしたいというユースケースを想定している.

# Error Handling

エラー処理もEpicの主要件である.もっともシンプルな例は,epicの中でcatchしてerror情報を含むactionをemitすること.

```js
const fetchUsersEpic = action$ =>
  action$.ofType(actionTypes.FETCH_USERS)
    .mergeMap(action =>
      Rx.Observable.fromPromise(webApi.fetchUsers())
        .map(json => {
          const {users} = json;
          return {
            type: actionTypes.RECEIVE_USERS,
            payload: {users}
          };
        })
        .catch(err => Observable.of({
          type: actionTypes.FETCH_USERS_REJECTED,
          payload: {error},
          error: true
        }))
    );
```

注意点は,`action$.ofType()`のStreamに`.catch()`を書いてしまうと,一度catchが呼ばれるとそのepicは二度と新たなactionをlistenしなくなってしまう.これはRxJSの`error: err => doSomething()`が一度しか呼ばれないという仕様に呼応している.

# Writing Tests

## Epic with invoking asynchronously Web APIs

非同期APIリクエストを行うEpicのテストのやり方を考えてみる.

テスト対象モジュールは,

```js
const fetchUsersEpic = action$ => (
  action$.ofType(actionTypes.FETCH_USERS)
    .do(action => console.log(action))  // debug
    .mergeMap(action => Rx.Observable.fromPromise(webApi.fetchUsers()))
    .map(json => {
        const {users} = json;
        return {
          type: actionTypes.RECEIVE_USERS,
          payload: {users}
        };
    })
    .do(action => console.log(action))  // debug
);
```

テスティングライブラリは次のような構成.

- [ava](https://github.com/avajs/ava)
  - test runner
  - assertion lib (power-assert)
  - support async/await, Observable test
- [fetch-mock](https://github.com/wheresrhys/fetch-mock)
  - fetch (isomorphic-fetch) APIのmock
- [redux-mock-store](https://github.com/arnaudbenard/redux-mock-store)
  - redux storeのmock

```js
'use strict';

const test = require('test');
const fetchMock = require('fetch-mock');
const configureMockStore = require('redux-mock-store').default;

const Rx = require('rxjs');
const [combineEpics, createEpicMiddleware} = require('redux-observable');

const actionTypes = require('./path/2/action-types.js');
const {fetchUsersEpic} = require('./path/2/epic.js');

let epicMiddleware;
let rootEpic;
let mockStore;
let store;
test.before(t => {
  rootEpic = combineEpics(
    fetchUsersEpic
  );
  epicMiddleware = createEpicMiddleware(rootEpic);
  mockStore = configureMockStore([epicMiddleware]);
});

test.beforeEach(t => {
  store = mockStore();
});

test.afterEach(t => {
  fetchMock.restore();
  epicMiddleware.replaceEpic(rootEpic);
});

test('If you dispatch action: FETCH_USERS then dispatch action: RECEIVE_USERS asynchronously', t => {
  // setup
  const responseJson = {
    users: [
      {
        name: 'Bob',
        age: 25
      }
    ]
  };
  fetchMock.get(/apis\/users/, {
    body: responseJson
  });

  // exec
  store.dispatch({
    type: actionTypes.FETCH_USERS
  });

  // verify
  t.truthy(fetchMock.lastCall(/apis\/users/));
  return Rx.Observable.interval(500)
    .take(4)
    .mapTo(store.getActions())
    .first(actualActions => actualActions.length === 2)
    .map(actualActions => {
      return t.deepEqual(
        actualActions,
        [
          {
            type: actionTypes.FETCH_USERS
          },
          {
            type: actionTypes.RECEIVE_USERS,
            payload: {
              users: responseJson.users
            }
          }
        ]
      );
    });
});
```

Obsrevable使わないで以下のように書くと,storeにdisapatchされたactionが`FETCH_USERS`のみになっていることがあって,非同期されるdispatchされる`RECEIVE_USERS`がassert時にはまだstoreにdispatchされていないことがあってテストが失敗する.

```js
// verify
t.truthy(fetchMock.lastCall(/apis\/users/));
t.deepEqual(
  store.getActions(),
  [
    {
      type: actionTypes.FETCH_USERS
    },
    {
      type: actionTypes.RECEIVE_USERS,
      payload: {
        users: responseJson.users
      }
    }
  ]
);
```

#### Appendix

- https://github.com/rkaneko/react-examples/tree/master/redux-observable-example
