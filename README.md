
# Zotero 用 LLM 論文サマライザー

LLM を用いて、Zotero 上の学術論文を自動要約し、ノートを生成します。

## 概要

Zotero に追加した論文を自動的に要約し、要約結果をノートとして保存するツールです。

### ワークフロー

本ツールは、[zotero-actions-tags](https://github.com/windingwind/zotero-actions-tags) 用のスクリプトとして実装されています。

1. Zotero に新しい論文が追加される
2. zotero-actions-tags が JavaScript スクリプトをトリガー
3. 論文の PDF ファイルパスと論文タイトルを取得
4. PDF をサーバーへ送信し、解析・分割を実行
5. ローカル環境で LLM API を呼び出し、論文を要約
6. 生成された Markdown をサーバーへ送信し、HTML に変換
7. HTML 化された要約を Zotero の論文ノートとして書き込み

![example](https://qyzhang-obsidian.oss-cn-hangzhou.aliyuncs.com/20250124100826.png)

---

## デプロイ | Setup

[zotero-actions-tags](https://github.com/windingwind/zotero-actions-tags) プラグインをインストールし、以下の画像のように設定してください。

![zotero-actions-tags settings](https://qyzhang-obsidian.oss-cn-hangzhou.aliyuncs.com/20250124094839.png)

![edit action](https://qyzhang-obsidian.oss-cn-hangzhou.aliyuncs.com/20250124095407.png)

---

## 設定 | Configuration

### 重要なポイント

* [ZotMoov](https://github.com/wileyyugioh/zotmoov) や
  [ZotFile](https://github.com/jlegewie/zotfile) を使って、論文 PDF を OneDrive などの同期ドライブに保存している場合は、
  **`only_link_file` を `true` に設定**してください。
* 使用する LLM API に応じて
  **`openaiBaseUrl` / `modelName` / `apiKey`** を設定してください。
* **`chunkSize`** は、使用するモデルのコンテキスト長に合わせて調整してください。
* **`stuffPrompt` / `mapPrompt` / `reducePrompt`** は、自分の使用言語に翻訳してください。
  ただし、**`{title}` と `{text}` は変更しないでください。**

### TL;DR（要点まとめ）

* ZotMoov / ZotFile で「Link to File」形式を使っている → `only_link_file = true`
* LLM API に合わせて `openaiBaseUrl`, `modelName`, `apiKey` を設定
* モデルのコンテキスト長に合わせて `chunkSize` を調整
* プロンプトは翻訳してよいが `{title}`, `{text}` はそのまま

---

## 設定項目（zotero_script.js）

`zotero_script.js` の先頭には、以下のような設定項目があります。

```js
let serverUrl = "https://paper_summarizer.jianyue.tech";
let only_link_file = false;
let timeout = 30;
let openaiBaseUrl = "https://dashscope.aliyuncs.com/compatible-mode/v1";
// Gemini
// let openaiBaseUrl = "https://generativelanguage.googleapis.com/v1beta/openai/";
let modelName = "qwen-plus-latest";
let apiKey = "sk-xxxxxxxxxxxxx";
let chunkSize = 64000;
let chunkOverlap = 1000;
let stuffPrompt = "";
let mapPrompt = "";
let reducePrompt = "";
```

### 各設定の説明（日本語）

* **`serverUrl`**
  PDF の解析および、要約後の Markdown を HTML に変換するためのサーバー URL。
  デフォルトでは作者が公開しているサーバーを使用。
  **機密性の高い論文を扱う場合は、自前でサーバーを立てることを推奨**。
  リポジトリを clone して `python server.py` を実行するだけで利用可能。

* **`only_link_file`**
  ZotMoov / ZotFile などを使い、PDF を Zotero に「Link to File」として保存している場合は `true`。
  それ以外の場合は `false`。

* **`timeout`**
  ZotMoov / ZotFile 使用時に、新しい論文追加後、PDF のダウンロード完了を待つ最大秒数。

* **`openaiBaseUrl`**
  OpenAI 互換 API のエンドポイント。
  使用するモデル提供元によって異なる。
  デフォルトは **通義千問（Qwen）**。
  詳細: [https://www.aliyun.com/product/bailian](https://www.aliyun.com/product/bailian)

* **`modelName`**
  API 呼び出し時に指定するモデル名。

* **`apiKey`**
  LLM の API キー。

* **`chunkSize`**
  モデルのコンテキストサイズ。
  PDF がこのサイズを超える場合は、map-reduce 方式で分割要約される。

* **`chunkOverlap`**
  map-reduce 方式における、分割チャンク間の重なりサイズ。

* **`stuffPrompt`**
  PDF 全体がコンテキスト内に収まる場合に使用するプロンプト。

* **`mapPrompt`**
  map-reduce 方式の **map フェーズ**で使用するプロンプト。

* **`reducePrompt`**
  map-reduce 方式の **reduce フェーズ**で使用するプロンプト。
