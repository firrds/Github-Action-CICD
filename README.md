# 情報システム基礎演習 week2

## GitHub Actions と GitHub Pages を使った CI/CD 体験

## 今日の目的

**Node.js を使って簡単な Web アプリを作成し、GitHub Actions で自動テスト・自動ビルドを行い、GitHub Pages に自動デプロイする**ところまでを体験します。

- GitHub Actions が何を自動化する仕組みなのか説明できる
- CI と CD の違いを説明できる
- Node.js プロジェクトに対して GitHub Actions の workflow を作成できる
- push をきっかけにテストとビルドを自動実行できる
- ビルド結果を GitHub Pages に自動公開できる
- 自分でコードを修正して、再度 push し、サイトが更新される流れを確認できる

---

## 1. 全体像を理解する

今日の作業の流れ

```text
ローカルPCでコードを書く
        ↓
git add / commit / push
        ↓
GitHub にコードが送られる
        ↓
GitHub Actions が自動で起動する
        ↓
1. Node.js の実行環境を用意する
2. 依存関係をインストールする
3. テストを実行する
4. Web公開用ファイルを build する
5. build 結果を GitHub Pages に deploy する
        ↓
ブラウザで公開サイトを確認できる
```

### 1.1 CI と CD とは何か

- **CI (Continuous Integration)**
  - 継続的インテグレーション
  - 開発者がコードを push するたびに、**自動でテストや build を実行して、コードの健全性を確認する**考え方
- **CD (Continuous Delivery / Continuous Deployment)**
  - 継続的デリバリー / 継続的デプロイ
  - テスト済みのコードを、**自動で公開環境に届ける**考え方

今日の演習では、CI/CDは次のように役割分担します。

- **CI**: `npm test` と `npm run build`
- **CD**: build された `dist/` フォルダを GitHub Pages に公開

---

## 2. 今回作るサンプル Web アプリ

今回は、Node.js を使って **「今日のメッセージを表示するだけの非常に簡単な Web アプリ」** を作ります。

ただし、単に HTML を置くだけではなく、**Node.js の build スクリプトで HTML を組み立てて `dist/` フォルダへ出力する**構成にします。

これにより、以下を体験できます。

- Node.js を使った簡単な build
- build 前のソースと build 後の成果物の違い
- GitHub Actions による自動 build
- build 成果物を Pages へ自動 deploy する流れ

---

## 3. 前提条件

この教材は、**Windows 11 の PC** を使うことを想定しています。

### 3.1 必要なもの

- GitHub アカウント
- Windows 11 の PC
- Git がインストールされていること
- Node.js の **最新の LTS 版** がインストールされていること
- Visual Studio Code などのエディタ
- PowerShell を使ってコマンドを実行できること

### 3.2 Node.js 最新 LTS 版をインストールする

#### 3.2.1 この操作の目的

この教材では、Node.js を使ってローカルで build や test を実行します。  
そのため、最初に **Windows 11 上へ Node.js の最新 LTS 版をインストール**しておきます。

LTS は Long Term Support の略で、学習や授業で使うときに比較的安定して利用しやすい版です。

#### 3.2.2 インストール手順

1. Node.js の公式ダウンロードページ(https://nodejs.org/ja/download)を開く
2. **LTS** と表示されている Windows 用インストーラーを選ぶ
   - 多くの大学の実習 PC では **Windows Installer (.msi) / 64-bit** を選べばよいです
3. ダウンロードした `.msi` ファイルを実行する
4. セットアップ画面で **Next** を押して進める
5. ライセンスに同意する
6. インストール先は特に理由がなければ既定値のままでよい
7. **Automatically install the necessary tools...** のような追加ツール導入項目は、この教材では必須ではありません
8. **Install** を押してインストールを完了する
9. インストール後、**PowerShell をいったん閉じて開き直す**

#### 3.2.3 インストール確認

PowerShell を開いて、次を実行してください。

```powershell
node --version
npm --version
```

どちらもバージョン番号が表示されれば成功です。

### 3.3 Git の確認

PowerShell を開いて、次を実行してください。

```powershell
git --version
```

Git のバージョンが表示されれば準備完了です。

### 3.4 この教材で使うターミナルについて

この教材では、**Windows Terminal 上の PowerShell** または **PowerShell** を標準の実行環境として扱います。  
そのため、コマンド例も原則として PowerShell 形式で記載します。

---

## 4. 完成後のフォルダ構成

最終的なフォルダ構成は次のようになります。

```text
node-pages-cicd-sample/
├─ .github/
│  └─ workflows/
│     └─ deploy.yml
├─ scripts/
│  └─ build.mjs
├─ src/
│  ├─ index.template.html
│  ├─ app.js
│  └─ style.css
├─ test/
│  └─ message.test.mjs
├─ package.json
├─ package-lock.json
└─ README.md
```

> `dist/` フォルダは build 後に生成されるため、最初は存在しません。

---

## 5. ステップ 1: 新しい GitHub リポジトリを作成する

### 5.1 この操作の目的

ここでは、作成する Web アプリのコードを保存するための GitHub リポジトリを準備します。

GitHub Actions も GitHub Pages も、基本的には **GitHub 上のリポジトリを基準に動く**ため、最初にリポジトリが必要です。

### 5.2 操作手順

1. GitHub (https://www.github.com)にログインする
2. 右上の **New repository** をクリックする
3. Repository name に次を入力する

```text
node-pages-cicd-sample
```

4. Public を選ぶ
5. `Add a README file` は **オフ** にする
   - 今回はローカルで最初から作るため
6. **Create repository** をクリックする

### 5.3 なぜ Public を推奨するのか

GitHub Pages は現在、**public リポジトリでは GitHub Free で利用可能**です。private リポジトリでも利用できるケースはありますが、学習教材としては public の方が設定確認がしやすいです。

---

## 6. ステップ 2: ローカルにプロジェクトを作る

### 6.1 この操作の目的

ここでは、ローカル PC 上に Node.js プロジェクトを作成します。

### 6.2 操作手順

任意の作業フォルダで、以下を実行してください。

```powershell
mkdir node-pages-cicd-sample
cd node-pages-cicd-sample
```

Git を初期化します。

```powershell
git init
```

Node.js の設定ファイルを作ります。

```powershell
npm init -y
```

### 6.3 この操作の意味

- `git init`
  - このフォルダを Git 管理対象にする
- `npm init -y`
  - `package.json` を自動生成する
  - Node.js プロジェクトとして扱えるようになる

---

## 7. ステップ 3: 必要なフォルダを作成する

```powershell
mkdir src
mkdir scripts
mkdir test
mkdir .github
mkdir .github/workflows
```

---

## 8. ステップ 4: `package.json` を書き換える

### 8.1 この操作の目的

ここでは、

- build コマンド
- test コマンド
- モジュール形式

を設定します。

### 8.2 `package.json` の完成形

現在の `package.json` を、次の内容で上書きしてください。

```json
{
  "name": "node-pages-cicd-sample",
  "version": "1.0.0",
  "description": "A simple Node.js web app for learning GitHub Actions and GitHub Pages CI/CD.",
  "type": "module",
  "scripts": {
    "build": "node scripts/build.mjs",
    "test": "node --test"
  },
  "keywords": [
    "github-actions",
    "github-pages",
    "ci",
    "cd",
    "nodejs"
  ],
  "author": "",
  "license": "MIT"
}
```

### 8.3 各設定の意味

- `"type": "module"`
  - `import` / `export` を使えるようにする
- `"scripts"`
  - `npm run build` や `npm test` で実行される内容を定義する
- `"build": "node scripts/build.mjs"`
  - Node.js で build 用スクリプトを実行する
- `"test": "node --test"`
  - Node.js 標準のテストランナーを使ってテストする

---

## 9. ステップ 5: Web アプリの元になるファイルを作る

今回は、テンプレート HTML・JavaScript・CSS を用意し、Node.js で build して `dist/` に出力します。

### 9.1 `src/index.template.html`

新しく `src/index.template.html` を作成し、次を書いてください。

```html
<!DOCTYPE html>
<html lang="ja">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>{{TITLE}}</title>
    <link rel="stylesheet" href="./style.css" />
  </head>
  <body>
    <main class="container">
      <h1>{{HEADING}}</h1>
      <p class="lead">GitHub Actions と GitHub Pages を使った CI/CD 学習用アプリです。</p>

      <section class="card">
        <h2>今日のメッセージ</h2>
        <p id="message">{{MESSAGE}}</p>
      </section>

      <section class="card">
        <h2>Build Information</h2>
        <ul>
          <li><strong>Built at:</strong> {{BUILT_AT}}</li>
          <li><strong>Commit SHA:</strong> {{COMMIT_SHA}}</li>
        </ul>
      </section>
    </main>

    <script src="./app.js"></script>
  </body>
</html>
```

### 9.2 `src/app.js`

新しく `src/app.js` を作成し、次を書いてください。

```javascript
console.log("Sample web app loaded successfully.");
```

### 9.3 `src/style.css`

新しく `src/style.css` を作成し、次を書いてください。

```css
* {
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: Arial, "Hiragino Kaku Gothic ProN", "Yu Gothic", sans-serif;
  background: #f4f7fb;
  color: #222;
}

.container {
  max-width: 900px;
  margin: 0 auto;
  padding: 48px 24px;
}

h1 {
  margin-bottom: 12px;
}

.lead {
  margin-bottom: 24px;
  font-size: 1.05rem;
}

.card {
  background: #ffffff;
  border-radius: 12px;
  padding: 24px;
  margin-bottom: 20px;
  box-shadow: 0 6px 18px rgba(0, 0, 0, 0.08);
}

#message {
  font-size: 1.3rem;
  font-weight: bold;
  color: #0b57d0;
}

ul {
  padding-left: 20px;
}
```

### 9.4 この時点の意味

この段階では、まだ Pages に公開する完成 HTML はありません。

今あるのは、**テンプレートと素材**です。

次の build スクリプトで、これらから `dist/index.html` を作ります。

---

## 10. ステップ 6: build スクリプトを作る

### 10.1 この操作の目的

ここでは、Node.js を使って

- テンプレート HTML を読み込む
- プレースホルダを実際の値に置き換える
- `dist/` フォルダを生成する
- 公開に必要なファイルをコピーする

という build を行います。

### 10.2 `scripts/build.mjs`

新しく `scripts/build.mjs` を作成し、次のコードを書いてください。

```javascript
import fs from "node:fs";
import path from "node:path";
import { fileURLToPath } from "node:url";

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);
const projectRoot = path.resolve(__dirname, "..");

const srcDir = path.join(projectRoot, "src");
const distDir = path.join(projectRoot, "dist");

const templatePath = path.join(srcDir, "index.template.html");
const appJsPath = path.join(srcDir, "app.js");
const styleCssPath = path.join(srcDir, "style.css");

const template = fs.readFileSync(templatePath, "utf-8");

const title = "Node.js CI/CD Sample";
const heading = "GitHub Actions + GitHub Pages Demo";
const message = "Push をきっかけに自動テスト・自動デプロイされています。";
const builtAt = new Date().toISOString();
const commitSha = process.env.GITHUB_SHA || "local-build";

const outputHtml = template
  .replaceAll("{{TITLE}}", title)
  .replaceAll("{{HEADING}}", heading)
  .replaceAll("{{MESSAGE}}", message)
  .replaceAll("{{BUILT_AT}}", builtAt)
  .replaceAll("{{COMMIT_SHA}}", commitSha);

fs.rmSync(distDir, { recursive: true, force: true });
fs.mkdirSync(distDir, { recursive: true });

fs.writeFileSync(path.join(distDir, "index.html"), outputHtml, "utf-8");
fs.copyFileSync(appJsPath, path.join(distDir, "app.js"));
fs.copyFileSync(styleCssPath, path.join(distDir, "style.css"));

console.log("Build completed successfully.");
```

### 10.3 コードの意味

#### 10.3.1 テンプレートを読む

```javascript
const template = fs.readFileSync(templatePath, "utf-8");
```

`index.template.html` を文字列として読み込んでいます。

#### 10.3.2 置換する値を決める

```javascript
const builtAt = new Date().toISOString();
const commitSha = process.env.GITHUB_SHA || "local-build";
```

- `builtAt`
  - build 実行時刻
- `commitSha`
  - GitHub Actions 上ならコミット ID が入る
  - ローカルでは `local-build` になる

#### 10.3.3 テンプレート文字列を置換する

```javascript
const outputHtml = template
  .replaceAll("{{TITLE}}", title)
  ...
```

プレースホルダを実際の文字列に置き換えています。

#### 10.3.4 `dist/` を作る

```javascript
fs.rmSync(distDir, { recursive: true, force: true });
fs.mkdirSync(distDir, { recursive: true });
```

古い build 結果を消してから、新しい `dist/` を作っています。

#### 10.3.5 build 結果を書き込む

```javascript
fs.writeFileSync(path.join(distDir, "index.html"), outputHtml, "utf-8");
```

公開用の完成 HTML を `dist/index.html` に出力しています。

---

## 11. ステップ 7: テストコードを作る

### 11.1 この操作の目的

CI では、push のたびにテストが自動実行されることが重要です。

今回は簡易的に、**教材で使うメッセージ文字列が想定どおりか** を確認するテストを作ります。

### 11.2 `test/message.test.mjs`

新しく `test/message.test.mjs` を作成し、次を書いてください。

```javascript
import test from "node:test";
import assert from "node:assert/strict";

const message = "Push をきっかけに自動テスト・自動デプロイされています。";

test("message should mention automatic deployment", () => {
  assert.match(message, /自動デプロイ/);
});

test("message should not be empty", () => {
  assert.notEqual(message.trim(), "");
});
```

### 11.3 このテストの意味

- 1 つ目のテスト
  - メッセージの中に「自動デプロイ」が含まれているか確認
- 2 つ目のテスト
  - 空文字列ではないか確認

現実の開発ではもっと実用的なテストを書きますが、今回は **「push したら自動で test が走る」ことを体験する** のが目的です。

---

## 12. ステップ 8: まずローカルで build と test を確認する

### 12.1 この操作の目的

GitHub Actions に任せる前に、**ローカルで正しく動くかを確認**します。

### 12.2 操作手順

テストを実行します。

```powershell
npm test
```

成功すると、`ok` が表示されます。

次に build を実行します。

```powershell
npm run build
```

### 12.3 build 成功後に確認すること

次のファイルが生成されていることを確認してください。

```text
dist/
├─ index.html
├─ app.js
└─ style.css
```

### 12.4 `dist/index.html` をブラウザで確認する

Windows 11 では `dist/index.html` をダブルクリックして既定のブラウザで開いてください。

表示されれば、ローカル build は成功です。

---

## 13. ステップ 9: `.gitignore` を作る

### 13.1 この操作の目的

学習目的では `dist/` を commit しなくても構いません。

今回の構成では、**GitHub Actions が毎回 build して Pages に公開する**ので、`dist/` は Git 管理対象から外した方が分かりやすいです。

### 13.2 `.gitignore`

新しく `.gitignore` を作成し、次を書いてください。

```gitignore
dist/
node_modules/
```

### 13.3 なぜ `dist/` を ignore するのか

- `dist/` は build のたびに生成できる
- ソースコードと成果物を分離できる
- 「Git に入れるべきもの」と「build により生まれるもの」の違いを理解しやすい

---

## 14. ステップ 10: GitHub Actions の workflow を作る

### 14.1 この操作の目的

ここが教材の中心です。

workflow を定義することで、GitHub に push したとき自動で以下を実行します。

1. ソースコードを取得
2. Node.js を準備
3. テスト実行
4. build 実行
5. Pages 用 artifact をアップロード
6. GitHub Pages へ deploy

### 14.2 `/.github/workflows/deploy.yml`

新しく `.github/workflows/deploy.yml` を作成し、次を書いてください。

```yaml
name: Deploy Node.js app to GitHub Pages
env:
  FORCE_JAVASCRIPT_ACTIONS_TO_NODE24: true

on:
  push:
    branches:
      - main
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: true

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repository
        uses: actions/checkout@v5

      - name: Configure GitHub Pages
        uses: actions/configure-pages@v5

      - name: Set up Node.js
        uses: actions/setup-node@v5
        with:
          node-version: 24
          cache: npm

      - name: Install dependencies
        run: npm ci

      - name: Run tests
        run: npm test

      - name: Build project
        run: npm run build

      - name: Upload Pages artifact
        uses: actions/upload-pages-artifact@v5
        with:
          path: ./dist

  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    needs: build

    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v5
```

---

## 15. ステップ 11: workflow の内容を詳しく理解する

### 15.1 `on`

```yaml
on:
  push:
    branches:
      - main
  workflow_dispatch:
```

意味:

- `main` ブランチに push されたら自動実行
- `workflow_dispatch` により、GitHub の画面から手動実行も可能

### 15.2 `permissions`

```yaml
permissions:
  contents: read
  pages: write
  id-token: write
```

意味:

- `contents: read`
  - リポジトリ内容を読めるようにする
- `pages: write`
  - GitHub Pages へ deploy する権限
- `id-token: write`
  - Pages デプロイ時に必要

### 15.3 `concurrency`

```yaml
concurrency:
  group: "pages"
  cancel-in-progress: true
```

意味:

- 連続で push されたとき、古い deploy を途中で止めて最新を優先する

### 15.4 `jobs.build`

build ジョブでは、テストと build を行います。

#### checkout

```yaml
- name: Checkout repository
  uses: actions/checkout@v5
```

GitHub 上のリポジトリ内容を runner に取得します。

#### configure-pages

```yaml
- name: Configure GitHub Pages
  uses: actions/configure-pages@v5
```

Pages 用の各種設定やメタデータを準備します。

#### setup-node

```yaml
- name: Set up Node.js
  uses: actions/setup-node@v4
  with:
    node-version: 24
    cache: npm
```

Node.js 24 系（LTS 系）を runner 上で使えるようにします。

`cache: npm` は、依存関係のキャッシュを利用して workflow を高速化しやすくする設定です。

#### install dependencies

```yaml
- name: Install dependencies
  run: npm ci
```

ロックファイルに基づいて、再現性の高い方法で依存関係を入れます。

> 今回は依存パッケージがないため実質ほぼ何もしませんが、実務に近い形にするために入れています。

#### run tests

```yaml
- name: Run tests
  run: npm test
```

push のたびにテストを実行します。

#### build project

```yaml
- name: Build project
  run: npm run build
```

`dist/` を生成します。

#### upload pages artifact

```yaml
- name: Upload Pages artifact
  uses: actions/upload-pages-artifact@v4
  with:
    path: ./dist
```

Pages に公開するため、build 結果を artifact としてアップロードします。

### 15.5 `jobs.deploy`

`deploy` ジョブは、`build` ジョブの完了後に実行されます。

```yaml
needs: build
```

これにより、**テストや build が失敗した場合は deploy されません**。

```yaml
- name: Deploy to GitHub Pages
  id: deployment
  uses: actions/deploy-pages@v4
```

アップロード済み artifact を GitHub Pages に公開します。

---

## 16. ステップ 12: `npm ci` のために lock ファイルを作る

### 16.1 この操作の目的

workflow で `npm ci` を使うには、通常 `package-lock.json` が必要です。

### 16.2 操作手順

以下を実行してください。

```powershell
npm install
```

依存関係がなくても、`package-lock.json` が生成されます。

### 16.3 なぜ `npm ci` を使うのか

`npm ci` は `package-lock.json` を前提に動き、`npm install` よりも **再現性が高く、CI 向き** です。

---

## 17. ステップ 13: GitHub に push する

### 17.1 この操作の目的

ここで初めて GitHub Actions が動きます。

### 17.2 GitHub リポジトリを remote として追加する

GitHub で作成した自分の URL に置き換えて実行してください。

```powershell
git remote add origin https://github.com/YOUR_ACCOUNT/node-pages-cicd-sample.git
```

### 17.3 初回コミットと push

```powershell
git add .
git commit -m "Create Node.js CI/CD sample app"
git branch -M main
git push -u origin main
```

### 17.4 この操作の意味

- `git add .`
  - 変更ファイルを commit 対象にする
- `git commit`
  - 変更内容を履歴として確定する
- `git branch -M main`
  - メインブランチを `main` にする
- `git push`
  - GitHub に送信する

---

## 18. ステップ 14: GitHub Actions の実行結果を確認する

### 18.1 操作手順

1. GitHub リポジトリを開く
2. 上部の **Actions** タブを開く
3. `Deploy Node.js app to GitHub Pages` を選ぶ
4. workflow 実行結果を確認する

### 18.2 見るべきポイント

- `build` ジョブが成功しているか
- `deploy` ジョブが成功しているか
- 途中で `npm test` が成功しているか
- `Upload Pages artifact` が成功しているか

### 18.3 失敗したときに見る場所

各 step をクリックするとログが見られます。

特に次を確認してください。

- YAML のインデントミス
- ファイル名のスペルミス
- `package-lock.json` がない
- `main` ブランチに push していない

---

## 19. ステップ 15: GitHub Pages を有効にする

### 19.1 この操作の目的

GitHub Actions で deploy するためには、リポジトリ側で Pages の公開元を設定する必要があります。

### 19.2 操作手順

1. GitHub リポジトリの **Settings** を開く
2. 左メニューの **Pages** をクリックする
3. **Build and deployment** セクションを確認する
4. **Source** で **GitHub Actions** を選ぶ

### 19.3 この設定の意味

これは「Pages の公開元は branch ではなく、Actions workflow に任せます」という設定です。

---

## 20. ステップ 16: 公開サイトを確認する

### 20.1 操作手順

workflow が成功すると、GitHub Pages の URL が表示されます。

一般的には次の形式です。

```text
https://YOUR_ACCOUNT.github.io/node-pages-cicd-sample/
```

ブラウザでアクセスし、ページが表示されることを確認してください。

### 20.2 確認ポイント

- 画面に見出しが表示される
- 今日のメッセージが表示される
- Built at が表示される
- Commit SHA が表示される

### 20.3 ここで理解すべきこと

このページは、ローカルで直接公開しているわけではありません。

**GitHub に push → Actions が build → Pages に deploy → 公開 URL で閲覧**

という流れで成り立っています。

---

## 21. ステップ 17: 修正して再 deploy を体験する

### 21.1 この操作の目的

CI/CD の価値は、**コード修正から公開までの流れが自動化されること**にあります。

### 21.2 `scripts/build.mjs` のメッセージを変更する

次の部分を探してください。

```javascript
const message = "Push をきっかけに自動テスト・自動デプロイされています。";
```

例えば、次のように変更します。

```javascript
const message = "GitHub Actions により、テストとデプロイが自動実行されています。";
```

### 21.3 テストも合わせて変更する

`test/message.test.mjs` のテストは元のメッセージ文字列に依存しているため、同じく修正します。

完成形は次です。

```javascript
import test from "node:test";
import assert from "node:assert/strict";

const message = "GitHub Actions により、テストとデプロイが自動実行されています。";

test("message should mention deployment", () => {
  assert.match(message, /デプロイ/);
});

test("message should not be empty", () => {
  assert.notEqual(message.trim(), "");
});
```

### 21.4 ローカル確認後に push

```powershell
npm test
npm run build
git add .
git commit -m "Update displayed message"
git push
```

### 21.5 何が起こるか

- push を検知して workflow が再実行される
- テストが通る
- build し直される
- Pages が再公開される
- 公開サイトの表示が更新される

これが CI/CD の基本体験です。

---

## 22. 失敗を意図的に起こして学ぶ

学習では、成功だけでなく失敗も重要です。

### 22.1 パターン A: テストを失敗させる

`test/message.test.mjs` を次のように変更します。

```javascript
import test from "node:test";
import assert from "node:assert/strict";

const message = "GitHub Actions により、テストとデプロイが自動実行されています。";

test("message should mention CI only", () => {
  assert.match(message, /存在しない文字列/);
});
```

push するとどうなるでしょうか。

#### 期待される結果

- `build` ジョブ内の `Run tests` が失敗する
- `deploy` は実行されない
- 公開サイトは更新されない

#### 学べること

CI は「壊れたコードを公開しないための門番」として働く

### 22.2 パターン B: YAML を壊す

たとえばインデントを崩すと workflow 自体が正常に解釈されません。

#### 学べること

GitHub Actions では **YAML の構文がとても重要** である

---

## 23. よくあるトラブルと対処

### 23.1 `npm ci` が失敗する

**原因**: `package-lock.json` がない可能性が高い

**対処**:

```bash
npm install
git add package-lock.json
git commit -m "Add package-lock.json"
git push
```

### 23.2 Actions は成功したのに Pages が表示されない

**確認ポイント**:

- `Settings` → `Pages` → `Source` が `GitHub Actions` になっているか
- workflow の `deploy` ジョブが成功しているか
- URL が正しいか
- 反映に少し時間がかかっていないか

### 23.3 404 になる

**原因候補**:

- `dist/index.html` が正しく生成されていない
- artifact に `dist/` を指定できていない
- リポジトリ名を含む URL を間違えている

### 23.4 workflow が動かない

**確認ポイント**:

- push 先が `main` ブランチか
- `.github/workflows/deploy.yml` の場所が正しいか
- YAML のインデントが崩れていないか

---

## 24. まとめ

この教材では、Node.js の簡単な Web アプリを題材にして、GitHub Actions と GitHub Pages による CI/CD の基本を体験しました。

重要なポイントは次の通りです。

- **GitHub Actions は push をきっかけに処理を自動実行できる**
- **CI はテストや build によりコードの品質を確認する**
- **CD は build 結果を公開環境へ届ける**
- **GitHub Pages を使うと静的サイトを簡単に公開できる**
- **テストに失敗すれば deploy させない、という流れを設計できる**

この考え方は、将来 React、Vue、Next.js、Astro などを扱うときにも基本的に同じです。

---

## 25. 参考資料（公式）

- GitHub Docs: Understanding GitHub Actions  
  https://docs.github.com/en/actions/get-started/quickstart
- GitHub Docs: Building and testing Node.js  
  https://docs.github.com/en/actions/tutorials/build-and-test-code/nodejs
- GitHub Docs: Using custom workflows with GitHub Pages  
  https://docs.github.com/en/pages/getting-started-with-github-pages/using-custom-workflows-with-github-pages
- GitHub Docs: Configuring a publishing source for your GitHub Pages site  
  https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site



---

