# Cloudflare WorkersにPythonがやってきた
<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a>
<small>This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.</small>

## はじめに

### 自己紹介
* Ryuji Tsutsui@ryu22e
* さくらインターネット株式会社所属
* Python歴は13年くらい（主にDjango）
* Python Boot Camp、Shonan.py、GCPUG Shonanなどコミュニティ活動もしています
* 著書（共著）：『[Python実践レシピ](https://gihyo.jp/book/2022/978-4-297-12576-9)』

### 今日話したいこと
* Cloudflare WorkersでPythonが使えるようになった話
* デモを交えて実際に動かしてみる
* 中の仕組みについても解説

### このトークで得られること
* Cloudflare Workersの概要がわかる
* Cloudflare WorkersでPythonを使う上で必要な設定、仕様のクセがわかる
* Cloudflare WorkersでPythonが動く仕組みを知ることで、仕様のクセがある理由を理解できる

### トークの構成
* Cloudflare Workersとは
* Cloudflare WorkersでPythonを使う方法
* Cloudflare WorkersでPythonが動く仕組み

## Cloudflare Workersとは

### Cloudflare Workersの概要
* サーバーレスアプリケーションをデプロイできるプラットフォーム
* 世界中にあるサーバーにコードをデプロイし、クライアントは物理的に近いサーバーからレスポンスを受け取る
* コールドスタートが排除されている
* 類似サービスにAWS Lamnda@Edgeがある

### コールドスタートについて補足
* しばらく実行されていない関数を実行しなければならない状況をコールドスタートという
* Cloudflare Workersはコールドスタートが排除されているため、高速なレスポンスが期待できる

### Cloudflare WorkersとAWS Lambda@Edgeの違い
| Cloudflare Workers | AWS Lambda@Edge |
| --- | --- |
| 無料枠がある | 無料枠がない |
| デプロイが1分程度で完了 | TODO 要調査 |
| JavaScriptはChrome V8エンジンで直接動作 | Node.jsで動作 |

### Chrome V8について補足

## Cloudflare WorkersでPythonを使う方法

## Cloudflare WorkersでPythonが動く仕組み
