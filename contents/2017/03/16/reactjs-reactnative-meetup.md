React.js meetup × React Native meetup
===

- https://react-native-meetup.connpass.com/event/49024/

---

# パネルディスカッション

- @yosuke\_furukawa
- @koba04
- @yoshitosi
- @janus\_wel

## What's React Native

- Native experience
- Learn once, write anywhere
- Keeping development velocity

Web上でコンポネントを動かすことも可能だが、Nativeで提供できる体験にはかなわない。
基礎的な技術を学び直すことなくプラットフォーム固有の個々のアプリを開発する。

## React Nativeに注目したきっかけ

- Recruit
  - 1・2年前から注目はしてた。
  - Reactを全面的に会社で採用。
  - React開発者のバックアップが得られそう。
  - Nativeの開発者もそもそも足りていない。
  - React Nativeに注目している開発者が身近にいた。
- Togetter
  - TogetterではiPhone/AndroidのNativeエンジニアは社内にいない。
  - Webの開発者はいたが、React NativeがAndroid対応したあたりからやり始めた。最初はReact Native Androidの方はバグもあった。
  - 現在は2人でiPhone/Androidアプリをリリース。
- CureApp
  - CureAppでは初代CTOがJS好き。アプリはtitaniumだったけど辛くなってきた。
  - 別のスタンドアローンのプロトタイプをReact Nativeで実装して見せたら感触が良かった。
  - そこから2・3人で開発を初めて今にいたる。

## Learn once, write anywhereは実際どうなの?

- Reactから入るひとにはしきいは高くないという解釈。
- Nativeへの繋ぎこみ部分だけ薄く提供されているのでNative機能を検索すればつくれてしまう。

## React Nativeでどこまで共通化できる?

- Platformをまたぐ共通化コンポーネントはものすごく低レベルのボタンぐらいしかWrite anywhereは期待していない。
- Write anywhereよりもReact開発者がなんとなく開発できるというところには期待している。
- React Native WebはViewを抽象化してWebでもNativeでも使えるというものがある。
- 画面レベルでのコンポーネントは分ける必要がある。Platformごとにエントリーポイントが異なる?
- TogetterではPlatformごとのコンポーネントによせるということはまだできていない。アプリごとにコードはほぼ同じ。
- WebとNativeでの共通化はどう考えている?
  - PhoneGapで昔は考えたことあるがもうやめた。
  - React Nativeの採用をCureAppでは考えている。というのもアプリの中にDSLを使ってそれをパースしてごにょるみたいな部分がある。この部分のデバッグがNativeだときついのでReact Native Webならどうかと。実際にやっているひとの話を聴いた感じだとイケるかもしれないと考えている。
- パフォーマンスは?
  - CureAppでアプリをためしにつくったときは60fpsぐらい出た。
  - WebのPhoneGapやTitaniumよりは全然パフォーマンスがよい。

## バージョンアップはどう?

- 今のWebのReactの問題は複雑なViewの差分計算が同期的に行われていてパフォーマンスがでない点。
- 16からはReact fiberでこの問題を非同期処理で行うようにする。フラグで管理。

## Reactってあと何年使える?

- Reactが死ぬのと一緒にReact Nativeも死ぬと思う? > React Native使っている勢はどう思っている?
- Togetterでは何年使えるというよりは速くリリースできることが良かった。
  - アプリ作って見るとページへの滞在時間が3倍になったとかが素早くフィードバックを得られた。
  - これでアプリをつくれば価値が出せそうだという所感を得たのでNativeアプリ開発がんばってみるかというフェーズ。
- furukawa氏もReactとReact Nativeの寿命は一緒かなと考えている。
  - 何年使えるっていうよりも、Web開発者がNativeの知るきっかけになると思っていてそこにやる価値はあると思っている。
- React NativeのcontributorsはFacebook外に多い。
- CureAppのアプリでは、患者さんを賢くサポートするという機能が求められる。
  - これを各Platformの言語で書くとリソースが追いつかなくなってしまう。
  - これをJavaScriptとDDDで書くことでスタートアップが取れる戦略としてはありかと思っている。
- Backendも

## 結局、React Nativeやりますか？

- DeNAのng-coreはReactNativeと野望は一緒だった。
  - ng-coreはとんざしてしまったけど、React Nativeはng-coreが超えられなかった課題を超えられるかには着目している。
  - Closed sourceだった React Native OSSなのでこれは○
  - Platformのアプリ審査が必要だけどこれはベンダー側がOKしないので課題としては残ったまま。
  - Unityのようなオーサリングツールとの相性や繋ぎこみがないと本格的なゲームはつらい。
- 今のNative engineerにReact Nativeの普及や導入を手伝ってもらう流れにならないと大きな波にはなりづらいかも。
- Windowsのコードプッシュ?はどう?
  - バグ回収とか早いリリースにはよさそうだけど、悪いコードに差し替えるとかいう事例が出てくるとベンダーにバンくらう可能性はある。
- ReactエンジニアがWebの強みからNativeへスキルを伸ばしていくというのは価値があると思う。


# React Hot Loaderを使って開発をさらに加速させる

- stateを保持したままコード更新を反映させることができる。

# Inside Bdash

- BdashではDomainLogicはView以外のデータとってくるとかの処理全般と定義。
- BdashではAction creatorsからDomain Logicを呼んでる。
- BdashではStoreの分割はページ(Container)単位。
  - この設計だとページをまたいでデータを共有するのが面倒。
  - ページ間で共有するデータはDomain LogicにStore(LocalStorageとかをバックエンドに)として保持。
  - nestedなstateの更新ユーティリティとしてimmupを作った。

# HyperApp - 1KBのビューライブラリ

- react, preact, inferno, mithril, ember, vueに似ている。
- hyperapp = vdom + redux/elm的なstate管理 + router
- 〜1kb
- Reactはheavyすぎ
- vdomエンジンが欲しかった
  - OSSのやつはhyperappに適した奴がなかったので自作
- dependenciesは0
- 300行のコードベース
- next?
  - vdomエンジンのパフォーマンスのためにrequestAnimationFrameを使う。


# 小回りの効くWebViewの使い方


# React nativeでfirebaseのネイティブSDKを操作する 


# Androiderから見るReact Native

- dev env
  - WebStorm
- React nativeをやっているとアプリに必要なコンポーネントを洗い出してComponentとして提供されているか、Native API bridgeを探す作業のループ。
- 良いと思ったところ
  - ごりごりにパフォーマンスだしたいところはNativeで、面倒な作りあきた部分をReact Nativeで作ってくれると同居できる。
- Native固有の動きをReact Nativeで実装できるの?

