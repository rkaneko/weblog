Ecmascript async functionsについて
===

### Materials

- [Tips for using async functions](http://www.2ality.com/2016/10/async-function-tips.html)

# 1. Promisesを知る

async functionsのベースとなっているのはPromisesである.
特に,Promisesをベースとしていない既存コードとasync functionsを組み合わせるときには,Promisesを直接使う以外の選択肢はない.

XHRをPromise化するにはcallback内でPromiseをresolve/rejectをする.
callback内ではfunctionの値をreturnしたりもthrowもできない.

async functionsのcommon coding styleは次のようになる.

- 非同期なprimitivesを構築するためにPromisesを使う.
- 非同期関数経由でそのprimitivesを使う.

# 2. Async functionsは同期的に始まり,非同期に解決される

async functionsはどのように実行されるか?

0. async functionsの結果は常にPromise.async functionsの実行開始時にこのPromise値は生成される.
0. bodyが実行される.実行は`return`や`throw`を経由して永久的に終了する.もしくは`await`経由で一時的に終了する.この場合は通常後の処理が継続される.
0. Promise値が返却される.

see also

- src/async-func.js

# 3. Promisesを返すものはwrapされない

Promiseのresolveは通常の命令となる.

async functionsの中でPromise値を返しても,そのPromise値はPromiseでwrapされることはない.

see also

- src/async-func-using-promise.js

async functionsを入れ子にした場合はどうなるか?

次の２つのコードはだいたい同じようなイメージなる.ただし前者のほうがパフォーマンスが良い.

```js
async function asyncAnotherFunc() {
  return 5;
}

async function asyncFunc() {
  return asyncAnotherFunc();
}

asyncFunc()
  .then(x => console.log(x));

// output: 5
```

```js
async function asyncAnotherFunc() {
  return 5;
}

async function asyncFunc() {
  return await asyncAnotherFunc();
}

asyncFunc()
  .then(x => console.log(x));

// output: 5
```

see also

- src/async-func-nested.js


# 4. `await`を忘れては行けないケース

async function内でasync functionを呼び出す場合には`await`をつけないと意図した挙動にならない.

```js
async function asyncFunc() {
  // 関数の戻り値が欲しいが...
  const value = otherAsyncFunc(); // missing await
}
```

ESLint [require-await](http://eslint.org/docs/rules/require-await)ではasync functionsで意図しない処理の値がasync functionsの戻り値になるのを防ぐために`await`を書くように強制できる.

`await`はasync functionsが戻り値を返さない時にも意味をなす.呼び出し元に呼び出しが終了したことを伝えるシグナルとしてPromiseを使用することもできる.

async functionsのbodyでPromisesをawaitすると,awaitしているPromisesの処理が終わってから,async functionsが実行されることが保証される.

see also

- src/await-promises.js

## ちなみに

実行エンジンでは,async functions内でPromisesをawaitしているasync functionsを呼び出す処理では,catchを書かないと怒られる.

```
(node:24554) UnhandledPromiseRejectionWarning: Unhandled promise rejection (rejection id: 2): TypeError: Promise resolver undefined is not a function
```

# 5. "fire and forget"を行う場合は`await`をつける必要はない

非同期処理をトリガーするだけで終了には興味がないケースではawaitを書く必要がない.

```js
async function asyncFunc() {
  const writer = openFile('someFile.txt');
  writer.write('hello');  // 書き込みはAPIが正しい順序で行うことを保証してくれている
  writer.write('world');  // 書き込み終了のタイミングには関心がないのでawaitを書く必要はない
  await writer.close();  // このasyncFunc()関数内でclose完了を保証したいので,await
}

```

最終行の`await write.close();`では`asyncFunc()`が`writer.close();`が終了した後に実行が終了されることを保証している.

closeすることを保証することなく,returnすれば(Promisesでwrapされていない)Promisesを返すこともできる.

```js
async function asyncFunc() {
  const writer = opneFile('someFile.txt');
  writer.write('hello');
  writer.write('world');
  return write.close();
}
```

どちらのパターンもPros/Consがある.Dr.Axel的にはawait版のほうが理解しやすいとの意見.

# 6. Parallelism

次のコードは`asyncFunc1()`,`asyncFunc2`がシーケンシャルに実行される.

see also

- src/async-func-sequential.js

次のコードは`asyncFunc1`,`asyncFunc2`がパラレルで実行される.

see also

- src/async-func-parallel.js


#### Appendix

- examples
  - https://github.com/rkaneko/learn-es-async-functions
