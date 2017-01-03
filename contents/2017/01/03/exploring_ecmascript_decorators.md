Exploring EcmaScript Decoratorsを読んだときのメモ
===

### materials

- Exploring EcmaScript Decorators
  - https://medium.com/google-developers/exploring-es7-decorators-76ecb65fb841#.3ercj63kr
- tc39/proposal-decorators
  - https://github.com/tc39/proposal-decorators

## The Decorator Pattern

Pythonにおいては,decoratorsは高階関数(higer-order functions)を呼び出すためのシンプルなsyntaxを提供する.別の関数を引数に取る関数で,元の関数の振舞いを明示的に修正することなしに拡張することができる.

Decoratorsの使用例としては,

- memoizeによる関数実行結果のキャッシュによる高速化
- アクセスコントロールと認証の強制
- 計測関数
- ロギング
- 律速

などが挙げられる.

## Decorators in ES

提案中のdecoratorsはclass,properties,object literalが対象とされている.

### Decorating a property

see also

- readonly-decorators.js

decorator libraryとしてcore-decorators.jsというのもある.

- https://github.com/jayphelps/core-decorators.js

### Decorating a class

現在策定中の仕様によると,classのdecoratorはconstructorを受け取る.

see also

- class-decorator.js

### ES2016 Decorators and Mixins

詳しくは以下の記事を参照のこと.

- Using ES.later Decorators as Mixins
  - http://raganwald.com/2015/06/26/decorators-in-es7.html
- Functional Mixins in ECMAScript 2015
  - http://raganwald.com/2015/06/17/functional-mixins.html

mixin関数を定義してdecoratorsを使ってclassに振舞いを追加することができる.

see also

- decoratos-as-mixins.js

## Conclusion

decoratorsは以下のメソッドを使って実現するメタプログラミングのように感じた.

- [Object.getOwnPropertyDecriptor()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/getOwnPropertyDescriptor)
- [Object.defineProperty()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)
- [Reflect.ownKeys()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Reflect/ownKeys)

#### Appendix

- sample example
  - https://github.com/rkaneko/learn-es-decorators
