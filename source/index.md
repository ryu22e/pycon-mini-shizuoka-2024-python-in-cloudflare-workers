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

### Cloudflare WorkersがAWS Lambda@Edgeよりも優れている点
* 無料枠がある
* デプロイが高速（1分程度で完了）
* JavaScriptを高速に実行するためのチューニングがされている

## Cloudflare WorkersでPythonを使う方法
### Cloudflare Workersを簡単に試す方法（デモ）
公式のサンプルコードを使うと簡単に試すことができる。
```{revealjs-code-block} shell

% git clone https://github.com/cloudflare/python-workers-examples.git
% cd python-workers-examples/01-hello
% npx wrangler@latest dev
```

### Wranglerとは何か
* Cloudflare Workersの開発者ツール
* ローカルサーバーの立ち上げ、デプロイ、環境変数の設定など、Workersプロジェクトの管理を行う

### src/entry.pyの中身

```{revealjs-code-block} python
from js import Response  # ←これは何？

async def on_fetch(request, env):
    return Response.new("Hello world!")
```

### jsモジュールとは何か
* PythonからJavaScript APIを呼び出すためのモジュール
* `Headers`や`fetch`なども呼べる

### Q. PythonなのになぜJavaScriptのAPIを使うの？

この疑問に答えるには、Cloudflare WorkersでPythonが動く仕組みを知る必要があるので、一旦置いておいてください。

### wrangler.tomlの中身
```{revealjs-code-block} toml

name = "hello-python"
main = "src/entry.py"
compatibility_flags = ["python_workers"]
compatibility_date = "2024-03-29"
```

### wrangler.tomlの各項目について説明
* name: Workersプロジェクト名
* main: エントリーポイント
* compatibility_flags: 互換性フラグ
* compatibility_date: 互換性日付

### 環境変数を参照するには（デモ）
以下のサンプルコードを参照。

```{revealjs-code-block} shell
$ git clone https://github.com/ryu22e/python-workers-examples.git
$ cd python-workers-examples/environment-variables
$ # 設定方法はREADME.mdを参照
```

### Cloudflare D1を使ったシンプルなAPI（デモ）
以下のサンプルコードを参照。

```{revealjs-code-block} shell
$ git clone https://github.com/ryu22e/python-workers-examples.git
$ cd python-workers-examples/simple-api
$ # 設定方法はREADME.mdを参照
```

### built-in packagesとは
* Cloudflare Workersで提供されているPythonパッケージ
* requirements.txtにパッケージ名を記述することで利用できる

### requirements.txtの記述例
```{revealjs-code-block} text

fastapi
```

### Q. requirements.txtって普通こう書かない？
```{revealjs-code-block} text

# パッケージ名の右側にバージョンを指定する
fastapi==0.112.0
```

### A. Cloudflare Workersでは別の方法でバージョンを指定する
wrangler.tomlの以下項目によってパッケージのバージョンが決まる。

* compatibility_flags: 互換性フラグ
* compatibility_date: 互換性日付

### サポートするパッケージとバージョンの一覧

以下の公式ドキュメントで確認できる。

<https://developers.cloudflare.com/workers/languages/python/packages/#supported-packages>

### Cloudflare Workersを簡単に試す方法（デモ）
公式のサンプルコードを使うと簡単に試すことができる。
```{revealjs-code-block} shell

% git clone https://github.com/cloudflare/python-workers-examples.git
% cd python-workers-examples/03-fastapi
% npx wrangler@latest dev
```

### 残念なお知らせ
この発表時点では、built-in packagesは本番環境にデプロイできない。

### FastAPIとLangChainを組み合わせたAPI（デモ）
以下のサンプルコードを参照。

```{revealjs-code-block} shell

$ git clone https://github.com/cloudflare/python-workers-examples.git
$ cd python-workers-examples/built-in-sample
$ # 設定方法はREADME.mdを参照
```

## Cloudflare WorkersでPythonが動く仕組み
### Q. WASMをサポートしないPythonがなぜ動くの？
* Cloudflare WorkersはJavaScript(TypeScript)またはWebAssembly（WASM）をサポートしている
* しかし、PythonにはコードをWASMにコンパイルする機能がない
* では、なぜPythonが動くのか？

### A.PyodideがPythonコードを解釈して実行している
* [Pyodide](https://pyodide.org/en/stable/)とは、CPythonのWASM実装
* Cloudflare Workersのランタイムである[workerd](https://github.com/cloudflare/workerd)には、Pyodideが組み込まれている

### workerdの実装について
* 現状ではJavaScript(TypeScript)またはWASMのランタイムとして作られている
* これにPythonのを加えると、1から実装することになって大変
* そこで、FFI（Foreign Function Interface）を提供して、PythonからJavaScriptのAPIを呼べるようにした
* 前述したjsモジュールを通してJavaScriptのAPIを呼び出せる

### 以下の公式ブログも参照
[Bringing Python to Workers using Pyodide and WebAssembly](https://blog.cloudflare.com/python-workers)
