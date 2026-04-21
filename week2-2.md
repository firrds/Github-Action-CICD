# GitHub Pages 共同編集実験（全体向け）

## 1. この実験の概要

この実験では、3〜4人のグループで GitHub Pages を共同編集します。  
グループリーダーが **共同編集用の新しいリポジトリ** を作成し、グループメンバーはそのリポジトリを **fork** して、自分の変更を **Pull Request (PR)** で提案します。

リーダーが PR を確認して `main` に merge すると、GitHub Actions が再実行され、GitHub Pages に最新内容が反映されます。

---

## 2. この実験の学習目標

この実験では、次のことを体験します。

- 既存の教材を別の共同編集用プロジェクトとして再構成する
- 他人のリポジトリを fork して変更提案を送る
- Pull Request を作成する
- リーダーが差分をレビューして merge する
- `main` への反映をきっかけに GitHub Actions が動く
- マージされた結果が GitHub Pages に自動で反映される

---

## 3. 役割分担

1 グループ 3〜4 人で、次のように役割を分けます。

### 3.1 グループリーダー
- 既存の教材のソースコードを別フォルダへコピーする
- 共同編集用に内容を修正する
- 新しい GitHub リポジトリを作成する
- そのリポジトリへ push する
- メンバーから届いた PR を確認する
- 問題がなければ merge する

### 3.2 グループメンバー
- リーダーが作成した新しい GitHub リポジトリを fork する
- 自分のメッセージを追加する
- PR を送る

---

## 4. この実験で実現したい最終状態

GitHub Pages 上で、複数人のメッセージが一覧で表示される状態を目指します。

```text
Group Messages
- Leader: Welcome to our group GitHub Pages site!
- Alice: CI/CD is interesting!
- Bob: Pull requests help us review changes.
- Carol: GitHub Pages updates automatically.
```

---

## 5. 共同編集版で追加・変更するファイル

この実験では、主に次のファイルを追加・変更します。

```text
src/
├─ index.template.html   ← 一部変更
├─ messages.json         ← 新規追加
scripts/
└─ build.mjs             ← 一部変更
test/
└─ messages.test.mjs     ← 新規追加
```

---

## 6. 実験全体の流れ

```text
リーダーが既存プロジェクトを別フォルダへコピーする
  ↓
共同編集版に修正する
  ↓
新しい GitHub リポジトリを作成して push する
  ↓
メンバーがその新しいリポジトリを fork する
  ↓
各メンバーが branch を作って変更する
  ↓
Pull Request を送る
  ↓
リーダーがレビューして merge する
  ↓
main が更新される
  ↓
GitHub Actions が自動実行される
  ↓
GitHub Pages が更新される
```

---

## 7. 次の作業

- リーダーは [**leader.md**](leader.md)
- メンバーは [**member.md**](member.md)

この全体向け資料は、実験の目的と役割分担を確認するためのものです。
