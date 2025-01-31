# Cloudflare WorkersにPythonがやってきた

Ryuji Tsutsui

PyCon mini Shizuoka 2024 continue資料

<a rel="license" href="http://creativecommons.org/licenses/by/4.0/"><img alt="Creative Commons License" style="border-width:0" src="https://i.creativecommons.org/l/by/4.0/88x31.png" /></a>
<small>This work is licensed under a <a rel="license" href="http://creativecommons.org/licenses/by/4.0/">Creative Commons Attribution 4.0 International License</a>.</small>

## はじめに

### 自己紹介
* Ryuji Tsutsui@ryu22e
* さくらインターネット株式会社所属
* Python歴は14年くらい（主にDjango）
* Python Boot Camp、Shonan.py、GCPUG Shonanなどコミュニティ活動もしています
* 著書（共著）：『[Python実践レシピ](https://gihyo.jp/book/2022/978-4-297-12576-9)』

### 今日話したいこと
* Cloudflare WorkersでPythonが使えるようになった話
* デモを交えて実際に動かしてみる
* 中の仕組みについても解説

### このトークの対象者
* Pythonの基礎的な文法がわかる人

### このトークで得られること
* Cloudflare Workersの概要がわかる
* Cloudflare WorkersでPythonを使う上で必要な設定、仕様のクセがわかる
* Cloudflare WorkersでPythonが動く仕組みを知ることで、仕様のクセがある理由を理解できる

### トークの構成
* Cloudflare Workersとは
* Cloudflare WorkersでPythonを使う方法
* Cloudflare WorkersでPythonが動く仕組み

### トークの元ネタ
2024年7月に私が書いた以下の記事をベースにしています。

[Cloudflare WorkersでサーバーレスPythonアプリを構築してみよう | gihyo.jp](https://gihyo.jp/article/2024/07/monthly-python-2407)

## Cloudflare Workersとは

### Cloudflare Workersの概要
Cloudflare Workersとは、サーバーレスアプリケーションをデプロイできるプラットフォーム。

### Cloudflare Workersの特徴(1)
プログラマーは世界中にあるサーバー（エッジ環境）にコードをデプロイし、ユーザーは物理的に近いサーバーからレスポンスを受け取る。

```{revealjs-break}
```

```{figure} cloudflare-workers-image.*
:alt: イメージ図

イメージ図
```

### Cloudflare Workersの特徴(2)
コールドスタートが排除されている。

* しばらく実行されていない関数を実行しなければならない状況をコールドスタートという
* Cloudflare Workersはコールドスタートが排除されているため、高速なレスポンスが期待できる

### Cloudflare Workersの類似サービス
類似サービスにAWS Lamnda@Edgeがある。

### Cloudflare WorkersがAWS Lambda@Edgeと異なる点
* 無料枠がある
* JavaScriptを高速に実行するためのチューニングがされている

参考: [サーバーレスコンピューティングがパフォーマンスを改善する方法とは?| Lambdaのパフォーマンス | Cloudflare](https://www.cloudflare.com/ja-jp/learning/serverless/serverless-performance/)

### Cloudflare Workersがサポートする言語
* JavaScript（TypeScript）
* WebAssembly（WASM）のバイナリにビルドできる言語（Rust、C、C++、Kotlin、Goなど）
* Python←NEW!

## Cloudflare WorkersでPythonを使う方法
### 必要なもの
* Node.js 16.17.0以上
* npx

### Cloudflare Workersを簡単に試す方法（デモ）
公式のサンプルコードを使うと簡単に試すことができる。
```{revealjs-code-block} shell

% git clone https://github.com/cloudflare/python-workers-examples.git
% cd python-workers-examples/01-hello
% npx wrangler@latest dev
```

### デプロイもやってみる（デモ）
デプロイは以下のコマンドで行う。

```{revealjs-code-block} shell
% npx wrangler@latest deploy
```

### Wranglerとは何か
* Cloudflare Workersの開発者ツール
* ローカルサーバーの立ち上げ、デプロイ、環境変数の設定など、Workersプロジェクトの管理を行う

### ちなみに、wrangleの意味は
1. 争う、口論する
2. 世話をする

おそらく2の意味で使われている。

### Hello worldアプリを構成している各ファイルについて解説
以下について解説する。

* src/entry.py: アプリケーションのソースコード
* wrangler.toml: プロジェクトの設定ファイル

### src/entry.pyの中身

```{revealjs-code-block} python
from js import Response  # ←これは何？

async def on_fetch(request, env):
    return Response.new("Hello world!")
```

### jsモジュールとは何か
* PythonからJavaScript APIを呼び出すためのモジュール
* 標準モジュールではなく、Cloudflare Workersの独自モジュール
* `Headers`や`fetch`なども呼べる

### jsモジュールの使用例

```{revealjs-code-block} python
from js import Headers, Response, fetch, console, URL

API_URL = "https://httpbin.org"

async def on_fetch(request, env):
    # /old にアクセスされた場合は/にリダイレクト
    # JavaScriptだとnew URL(request.url)と書くが、Pythonにnew演算子はないのでこう書く
    url = URL.new(request.url)
    if url.pathname == "/old":
        return Response.redirect(url.origin, 307)

    # JSON形式でレスポンスを返すためのヘッダーを設定
    headers = Headers.new({"content-type": "application/json; charset=utf-8"}.items())

    # fetch()関数を使ってAPIサーバーにリクエストを送信
    res = await fetch(f"{API_URL}/ip")

    # レスポンスの内容をコンソールに出力
    console.log(res)

    return Response.new(res.body, headers=headers)
```

### jsモジュールのサンプルコード
[以下のサンプルコード](https://github.com/ryu22e/python-workers-examples/tree/main/js-sample)を参照。
```{revealjs-code-block} shell

% git clone https://github.com/ryu22e/python-workers-examples.git
% cd python-workers-examples/js-sample
% # 設定方法はREADME.mdを参照
```

### その他のjsモジュールの使用例
以下の公式サイトを参照。

[Examples | Cloudflare Workers docs](https://developers.cloudflare.com/workers/examples/)

### Q. PythonなのになぜJavaScriptのAPIを使うの？

この疑問に答えるには、Cloudflare WorkersでPythonが動く仕組みを知る必要があるので、一旦置いておいてください。

### wrangler.tomlの中身
```{revealjs-code-block} toml

# Workersプロジェクト名
name = "hello-python"
# エントリーポイント
main = "src/entry.py"
# 互換性フラグ（ランタイムの特定の機能を有効化）
compatibility_flags = ["python_workers"]
# 互換性日付（ランタイムのバージョン番号のようなもの）
compatibility_date = "2024-03-29"
```

### 環境変数を参照するには
`env`引数を使う（osモジュールでは参照できない）。

```{revealjs-code-block} python
from js import Response

async def on_fetch(request, env):
    return Response.new(f"My name is {env.MY_NAME}.\nSECRET_KEY: {env.SECRET_KEY}")
```

### 環境変数を定義するには(1)
公開してもよい値の場合、wrangler.tomlに書く。

```{revealjs-code-block} toml
name = "environment-variables"
main = "src/entry.py"
compatibility_flags = ["python_workers"]
compatibility_date = "2024-03-29"

# ↓ここに環境変数を書く
[vars]
MY_NAME = "Ryuji Tsutsui"
```

### 環境変数を定義するには(2)
秘密の値（例: APIキー）の場合、ローカルではプロジェクト直下の.dev.varsファイルに書く。
```{revealjs-code-block} text
SECRET_KEY="local_value"
```

```{revealjs-break}
```
本番環境は`npx wrangler secret put {環境変数名}`で設定。

```{revealjs-code-block} shell
% npx wrangler secret put SECRET_KEY

 ⛅️ wrangler 3.78.8
-------------------

✔ Enter a secret value: … ****************
🌀 Creating the secret for the Worker "environment-variables"
✨ Success! Uploaded secret SECRET_KEY
```

### 実際に環境変数を定義・参照してみる（時間があればデモ）
[以下のサンプルコード](https://github.com/ryu22e/python-workers-examples/tree/main/environment-variables)を参照。

```{revealjs-code-block} shell
% git clone https://github.com/ryu22e/python-workers-examples.git
% cd python-workers-examples/environment-variables
% # 設定方法はREADME.mdを参照
```

### Cloudflare D1を使ったシンプルなAPI（時間があればデモ）
Cloudflare D1とは

* SQLiteベースのサーバーレスデータベース
* Cloudflareのエッジ環境にSQLiteのリードレプリカが配置されることで、高速な読み込みを実現

```{revealjs-break}
```
データベースの作り方は以下の通り。

```{revealjs-code-block} shell
% # データベース「bookshelf」の作成
% npx wrangler d1 create bookshelf
% # ↑出力された内容をwrangler.tomlに追記
% # テーブルの作成（ローカル）
% npx wrangler d1 execute bookshelf --local --file=./schema.sql
% # テーブルの作成（本番）
% npx wrangler d1 execute bookshelf --remote --file=./schema.sql
```

```{revealjs-break}
```
DBアクセスのサンプルコードは以下の通り。

```{revealjs-code-block} python
async def on_fetch(request, env):
    ...
    # INSERT文
    await (
        env.DB.prepare("INSERT INTO books (title, description) VALUES (?, ?)")
        .bind(title, description)
        .run()
    )
    # SELECT文
    r = await env.DB.prepare("SELECT * from books").all()
    print(r.results)
    ...
```

```{revealjs-break}
```
[以下のサンプルコード](https://github.com/ryu22e/python-workers-examples/tree/main/simple-api)を参照。

```{revealjs-code-block} shell
% git clone https://github.com/ryu22e/python-workers-examples.git
% cd python-workers-examples/simple-api
% # 設定方法はREADME.mdを参照
```


### Built-in packagesとは
* Cloudflare Workersで提供されているPythonパッケージ
* requirements.txtにパッケージ名を記述することで利用できる
* 予め用意されたパッケージのみ利用可能

### requirements.txtの記述例
```{revealjs-code-block} text

fastapi
```

### FastAPIのコード例
```{revealjs-code-block} python
from fastapi import FastAPI

# この関数の定義は必ず必要
async def on_fetch(request, env):
    import asgi

    return await asgi.fetch(app, request, env)

# これ以降は普通のFastAPIのコード
app = FastAPI()

@app.get("/")
async def root(req: Request):
    ...
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

### Q. Built-in packagesに自分が使いたいパッケージがない……
A. Built-in packagesのmicropipを使えば、他のパッケージも使える（ただし、これにも制限がある）。

### micropipのコード例(1)
```{revealjs-code-block} python
import micropip

# FastAPIの設定は省略

@app.get("/example")
async def example(req: Request):
    await micropip.install("beautifulsoup4==4.12.3")

    # beautifulsoup4はbuilt-in packagesにはないが
    # micropipでインストールできる
    from bs4 import BeautifulSoup
    ...
```

### micropipのコード例(2)
```{revealjs-code-block} python
import micropip

# FastAPIの設定は省略
@app.get("/example")
async def example(req: Request):
    """micropip.install()が失敗する例"""
    # pandas==2.2.2はpure Pyhon wheelがないため、
    # micropipでインストールできずエラーになる
    # 参考: https://pyodide.org/en/stable/usage/faq.html#why-can-t-micropip-find-a-pure-python-wheel-for-a-package
    # （wheelとはPythonコードを1個のファイルにまとめたアーカイブ）
    await micropip.install("pandas==2.2.2")
    ...
```

### Built-in packagesを使ったAPI（時間があればデモ）
[以下のサンプルコード](https://github.com/ryu22e/python-workers-examples/tree/main/built-in-sample)を参照。

```{revealjs-code-block} shell

% git clone https://github.com/ryu22e/python-workers-examples.git
% cd python-workers-examples/built-in-sample
% # 設定方法はREADME.mdを参照
```

### 残念なお知らせ
この発表時点では、Built-in packagesは本番環境にデプロイできない。

```{revealjs-code-block} shell
% npx wrangler@latest deploy
(省略)
✘ [ERROR] A request to the Cloudflare API (/accounts/****/workers/scripts/built-in-sample) failed.

  You cannot yet deploy Python Workers that depend on packages defined in requirements.txt. Support
  for Python packages is coming soon. [code: 10021]

  If you think this is a bug, please open an issue at:
  https://github.com/cloudflare/workers-sdk/issues/new/choose
```

## Cloudflare WorkersでPythonが動く仕組み
### Q. WASMをサポートしないPythonがなぜ動くの？
* Cloudflare WorkersはJavaScript(TypeScript)またはWebAssembly（WASM）をサポートしている
* しかし、PythonにはコードをWASMにコンパイルする機能がない
* では、なぜPythonが動くのか？

### A.PyodideがPythonコードを解釈して実行している
* [Pyodide](https://pyodide.org/en/stable/)とは、CPythonのWASM実装
* Cloudflare Workersのランタイムである[workerd](https://github.com/cloudflare/workerd)には、Pyodideが組み込まれている

### PyodideでPythonを動かせるということは……
* Rubyのruby.wasm、PHPのphp-wasmなどを使って、他の言語も動かせるのでは？
* （あくまで私の空想です）

### Pyodideの技術的制限
* OpenSSLなどのCライブラリに依存する標準パッケージに利用制限がある
* 軽量化のため削除されている標準パッケージがある

参考: [Standard Library provided to Python Workers](https://developers.cloudflare.com/workers/languages/python/stdlib/)

### サードパーティパッケージも何でも使えるわけではない
* 一部のBuilt-in packagesはCloudflare Workersで動かすためにパッチが当てられている
* 本番環境へのデプロイがなかなかできるようにならないのは、この制限が関係しているのかも？

### jsモジュールが存在する理由
* workerdは、現状ではJavaScript(TypeScript)またはWASMのランタイムとして作られている
* これにPythonを加えると、1から実装することになって大変
* そこで、FFI（Foreign Function Interface）を提供して、PythonからJavaScriptのAPIを呼べるようにした
* jsモジュールは、PythonでもJavaScriptと同等の実装ができるように提供されている次善の策

### 以下の公式ブログも参照
[Bringing Python to Workers using Pyodide and WebAssembly](https://blog.cloudflare.com/python-workers)

### jsモジュールは正直使いにくいが……
* JavaScriptとPythonの流儀の違いがあるため、違和感を感じることがある
* `request.json()`で辞書型で扱えるのを期待していたら属性アクセスが求められたりして、混乱する
* Built-in packagesがこの使いにくさを緩和してくれることを期待

## 最後に
### まとめ
* Cloudflare Workersはサーバーレスアプリケーションをデプロイできるプラットフォーム
* JavaScript(TypeScript)またはWASMをサポートしているが、Pythonも使えるようになった
* Pythonが動くのは、WASM実装のPythonインタプリタPyodideを使っているため

### ご清聴ありがとうございました
```{figure} thank-you-for-your-attention.*
:alt: AIが考えた「Cloudflare Workers in Python」

AIが考えた「Cloudflare Workers in Python」
```
