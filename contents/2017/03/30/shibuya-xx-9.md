Shibuya.XSS techtalk #9
===

# Edgeの話

- Edgeはソーシャルネットワーク型のマルウェアの取れる検出率が高かったのでChromeより安全と謳っている。
- なので探したｗ
- Windows 10 insider preview, Fast ring, slow ring
- Let's get started!
  - SOP
    - Edgeはportが変わってもSOとみなす
    - Edge側としては後方互換性のためで将来的にはスタンダードに合わせるかも
    - iframeからのredirectを誤認させる挙動
  - Edgeの独自機能を見ていく
    - Linkを踏ませてWeb noteを取らせると(local file)Web noteからデータを引っこ抜ける
    - location.href内にPCユーザ名があるのでそれを狙う
    - Web noteにはスクショとかData URLで引っこ抜ける
- 今年のFile関連脆弱性で注目しているのはEPUB
  - EPUBはzipアーカイブ
  - EPUBはWeb技術を使用している HTML,CSS(,JavaScript)
  - AppleのEPUBリーダはJSをサポートしている
  - 画像を使えば外部にリクエストを遅れてlocation.hrefも取れるのでユーザ名等が引っこ抜けた。
  - AdobeのEPUBリーダは後ろはIEっぽい
  - IEはPortが異なってもSOとみなすのでlocalhostの全サービスにアクセス可能
  - EdgeのEPUBリーダはiframe sandboxを使ってユーザコンテンツを表示しておりJSの実行を制限している
  - iframe sandboxをバイパスできればJSをEPUBで実行できる
  - EPUBリーダーによってはハッキングツールと化す恐れあり
  - ユーザはリーダーの実装を理解しないとデータを守れない
  - EPUBの仕様に従っているリーダはない(らしい)www
  - XXEの脆弱性も上がっていてサーバのデータも...

# WAFを自作した話

- Vurp
  - Webアプリをリバプロして脆弱性も持つWebアプリにできるものｗ
  - 何に使う?
    - セミナーのデモ
    - トレーニングでの演習環境

# XSSフィルターの使い方

- XSSフィルターの基本
  - ページを書き換えることによりXSSを防ぐ
  - XSSフィルター,Auditor,FFではoScriptNを使うと同等
  - IEやEdgeは#で置換するという動作
  - ChromeやSafariは一部を消す
  - NoScriptは事前に危険なリクエストを置換
  - 誤検知は仕組み上さけられない
  - この誤検知を利用して攻撃できないか?
  - フィルタの誤動作が起こることを利用する
- X-XSS-Protectionヘッダは1以外を利用しよう

# WebセキュリティとW3CとIETFの仕様

- W3C
  - Web Application Security WG
- CSP Level3
  - strict-dynamic
  - disown-opener
  - navigation-to
  - report-sample
- CSP Embedded Enforcement
  - 埋め込む側がiframeの中のコンテンツにCSPをつけるように要請する
- Clear Site Data
  - サーバからクライアントのCookieやキャッシュをクリアできる
- Suborigins
  - 同一オリジンで複数アプリケーションが提供される場合にサブオリジンを指定できる
  - Access-Control-Allow-Suborigin
- HSTS mi
- CORS and RFC1918
  - パブリックなネットワークから、ローカルネットワークへのCSRFをできないようにする
- Isolated Origins
  - HTTPヘッダでIsolation=1を受け取るとUAが特定の動作をするようになる
- Origin Policy
  - オリジン全体にポリシーを適応できるようにする
  - ヘッダにポリシー名を指定しファイルを置く
  - Sec-Origin-Policy: "policy-1"
  - CSPはレスポンスごとに設定する必要があるがだるいなど
- Cookie Prefixes
  - Cookieについてる属性を確かなものにする
  - Set-Cookie:__Scure-SID=blahblah
- Same-Site Cookie
  - laxとか
  - CSRF対策になる
- Let localhost be localhost
  - localhostをループバックアドレスにする仕様
    - secure contextで"localhost"をsecure contextにしたいというモチベーション
    - localhostがループバックアドレスを返すことをMUSTにしたい
- ブラウザのセキュリティ系Mailing Listはおすすめ
- Google のMike Westのレポジトリとアクティビティも良い
