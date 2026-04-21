# GitHub Pages 共同編集実験（リーダー用）

## 1. この資料の役割

この資料は、グループリーダーが **共同編集用の新しいリポジトリを作成し、初期状態を整えて公開するまで** の手順をまとめたものです。

この実験では、**今まで作ってきた既存のソースコードをそのまま上書き修正するのではなく、別フォルダにコピーして、共同編集専用の別プロジェクトとして作成**します。

---

## 2. 作業の全体像

リーダーは次の流れで準備します。

1. 既存の教材プロジェクトを別フォルダへコピーする
2. コピー先を共同編集版に修正する
3. 新しい GitHub リポジトリを作成する
4. その新しいリポジトリへ push する
5. GitHub Pages と Actions の動作を確認する
6. メンバーに新しいリポジトリの URL を共有する

---

## 3. ステップ 1: 既存プロジェクトを別フォルダへコピーする

### 3.1 この操作の目的

元の教材を壊さずに、共同編集実験専用のプロジェクトを作るためです。

### 3.2 例

たとえば、元のフォルダが次のようになっているとします。

```text
node-pages-cicd-sample
```

これを、別フォルダとして次のようにコピーします。

```text
node-pages-cicd-prtest
```

---

## 4. ステップ 2: `src/messages.json` を追加する

### 4.1 この操作の目的

全員のメッセージを 1 つのデータファイルで管理するためです。

### 4.2 `src/messages.json`

新しく `src/messages.json` を作成し、次の内容を書いてください。

```json
[
  {
    "name": "Leader",
    "message": "Welcome to our group GitHub Pages site!"
  }
]
```

### 4.3 このファイルの意味

- `name`: 誰のメッセージかを表す
- `message`: GitHub Pages 上に表示したい本文

最初はリーダーのメッセージだけを入れておきます。後でメンバーが PR を使って、この配列に自分のデータを追加します。

---

## 5. ステップ 3: HTML テンプレートを共同編集版に変更する

既存の `src/index.template.html` を、次の内容で上書きしてください。

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
      <p class="lead">GitHub Actions・GitHub Pages・Pull Request を使った共同編集学習用アプリです。</p>

      <section class="card">
        <h2>Group Messages</h2>
        <ul class="message-list">
          {{MESSAGE_LIST}}
        </ul>
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

---

## 6. ステップ 4: CSS を共同編集版に少し拡張する

既存の `src/style.css` の末尾に、次を追加してください。

```css
.message-list {
  list-style: none;
  padding-left: 0;
  margin: 0;
}

.message-list li {
  padding: 12px 16px;
  margin-bottom: 12px;
  background: #eef4ff;
  border-left: 6px solid #0b57d0;
  border-radius: 8px;
}

.message-name {
  font-weight: bold;
}
```

---

## 7. ステップ 5: build スクリプトを共同編集版に変更する

既存の `scripts/build.mjs` を、次の内容で上書きしてください。

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
const messagesPath = path.join(srcDir, "messages.json");

const template = fs.readFileSync(templatePath, "utf-8");
const messages = JSON.parse(fs.readFileSync(messagesPath, "utf-8"));

const title = "Node.js CI/CD Group Sample";
const heading = "GitHub Actions + GitHub Pages + Pull Request Demo";
const builtAt = new Date().toISOString();
const commitSha = process.env.GITHUB_SHA || "local-build";

const messageListHtml = messages
  .map(
    (item) => `\n<li><span class="message-name">${escapeHtml(item.name)}</span>: ${escapeHtml(item.message)}</li>`
  )
  .join("");

const outputHtml = template
  .replaceAll("{{TITLE}}", title)
  .replaceAll("{{HEADING}}", heading)
  .replaceAll("{{MESSAGE_LIST}}", messageListHtml)
  .replaceAll("{{BUILT_AT}}", builtAt)
  .replaceAll("{{COMMIT_SHA}}", commitSha);

fs.rmSync(distDir, { recursive: true, force: true });
fs.mkdirSync(distDir, { recursive: true });

fs.writeFileSync(path.join(distDir, "index.html"), outputHtml, "utf-8");
fs.copyFileSync(appJsPath, path.join(distDir, "app.js"));
fs.copyFileSync(styleCssPath, path.join(distDir, "style.css"));

console.log("Build completed successfully.");

function escapeHtml(value) {
  return String(value)
    .replaceAll("&", "&amp;")
    .replaceAll("<", "&lt;")
    .replaceAll(">", "&gt;")
    .replaceAll('"', "&quot;")
    .replaceAll("'", "&#39;");
}
```

---

## 8. ステップ 6: テストを共同編集版に追加する

新しく `test/messages.test.mjs` を作成し、次を書いてください。

```javascript
import test from "node:test";
import assert from "node:assert/strict";
import fs from "node:fs";

const messages = JSON.parse(fs.readFileSync("./src/messages.json", "utf-8"));

test("messages should be an array", () => {
  assert.equal(Array.isArray(messages), true);
});

test("messages should contain at least one entry", () => {
  assert.ok(messages.length >= 1);
});

test("each entry should have name and message", () => {
  for (const item of messages) {
    assert.equal(typeof item.name, "string");
    assert.equal(typeof item.message, "string");
    assert.notEqual(item.name.trim(), "");
    assert.notEqual(item.message.trim(), "");
  }
});
```

---

## 9. ステップ 7: ローカルで動作確認する

PowerShell で次を実行します。

```powershell
npm test
npm run build
```

### 9.1 確認すること

- テストが成功するか
- build が成功するか
- `dist/index.html` を開くとリーダーのメッセージが表示されるか

---

## 10. ステップ 8: 新しい GitHub リポジトリを作成する

### 10.1 この操作の目的

共同編集実験専用のリモートリポジトリを作るためです。  
元の教材用リポジトリとは **別のリポジトリ** にしてください。

### 10.2 リポジトリ名の例

```text
node-pages-cicd-prtest
```

### 10.3 GitHub 上で行うこと

1. GitHub にログインする
2. **New repository** をクリックする
3. 新しい名前でリポジトリを作成する
4. 必要なら Public にする
5. README の自動作成はしない設定でもよい

---

## 11. ステップ 9: コピーしたプロジェクトを新しいリポジトリへ push する

### 11.1 すでに Git 管理されている場合

コピー元の `.git` がそのまま含まれている場合は、`origin` の向き先を新しいリポジトリへ変更してから push してください。

```powershell
git remote -v
git remote set-url origin https://github.com/YOUR_ACCOUNT/node-pages-cicd-prtest.git
git remote -v
git add .
git commit -m "Prepare collaborative GitHub Pages project"
git push -u origin main
```

### 11.2 Git 管理を新しく始める場合

必要に応じて、コピー先で初期化し直しても構いません。

```powershell
git init
git branch -M main
git add .
git commit -m "Prepare collaborative GitHub Pages project"
git remote add origin https://github.com/YOUR_ACCOUNT/node-pages-cicd-prtest.git
git push -u origin main
```

---

## 12. ステップ 10: GitHub Pages と Actions を確認する

### 12.1 確認すること

- Actions が成功しているか
- GitHub Pages にリーダーのメッセージが表示されるか

### 12.2 この時点の意味

この時点で、**メンバーが fork して参加するための共同編集用の新しい受け皿** が完成しています。

---

## 13. ステップ 11: メンバーに新しいリポジトリ URL を共有する

メンバーには、**元の教材リポジトリではなく、今回新しく作成した共同編集用リポジトリ** の URL を共有してください。

共有時には、次のように伝えると分かりやすいです。

```text
この新しいリポジトリを fork してください。
元の教材リポジトリではなく、共同編集実験用のリポジトリを使います。
```

---

## 14. ステップ 12: メンバーからの Pull Request をレビューする

### 14.1 リーダーの確認ポイント

- `src/messages.json` だけが変更されているか
- JSON の形式が壊れていないか
- 不適切な表現や空欄がないか
- 既存のメッセージを誤って削除していないか

### 14.2 merge の流れ

1. **Pull requests** を開く
2. 対象の PR をクリックする
3. 差分を確認する
4. 問題がなければ **Merge pull request** をクリックする
5. 必要に応じて **Confirm merge** をクリックする

---

## 15. ステップ 13: GitHub Pages への反映を確認する

PR が `main` に merge されると、新しい `push` が発生し、GitHub Actions が再実行されます。  
その後、GitHub Pages の内容が更新され、メンバーのメッセージが一覧に追加されます。

---

## 16. 起こりやすいトラブル

### 16.1 JSON のカンマミス
- `npm test` や `npm run build` が失敗する
- Actions でも JSON parse error になる

### 16.2 同じ行を複数人が編集してコンフリクトする
- PR が自動マージできない
- メンバーに `upstream/main` の取り込みを案内する

### 16.3 Pages に反映されない
- 共同編集用の新しいリポジトリに本当に merge したか
- Actions が失敗していないか
- Pages の設定が正しいか

---

## 17. まとめ

リーダーは、**元の教材を壊さずに別プロジェクトとして共同編集版を立ち上げ、その新しいリポジトリを中心にチーム開発を進める役割** を担います。
