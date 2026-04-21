# GitHub Pages 共同編集実験（メンバー用）

## 1. この資料の役割

この資料は、グループメンバーが **リーダーの新しい共同編集用リポジトリを fork し、自分のメッセージを PR で追加するまで** の手順をまとめたものです。

ここで使うのは、**元の教材リポジトリではありません**。  
**リーダーが今回新しく作成した共同編集用リポジトリ** を fork してください。

---

## 2. 作業の全体像

メンバーは次の流れで作業します。

1. リーダーが作成した新しいリポジトリを fork する
2. 自分の fork を clone する
3. `upstream` としてリーダーの新しいリポジトリを登録する
4. 作業用 branch を作る
5. `src/messages.json` に自分のメッセージを追加する
6. テストと build を実行する
7. commit して fork へ push する
8. Pull Request を作成する

---

## 3. ステップ 1: リーダーの新しいリポジトリを fork する

### 3.1 この操作の目的

リーダーの `main` ブランチへ直接 push せず、**自分のコピー（fork）上で変更してから PR を送る**ためです。

### 3.2 操作手順

1. ブラウザで、**リーダーが今回新しく作成した共同編集用リポジトリ** を開く
2. 右上の **Fork** をクリックする
3. 自分の GitHub アカウント側にコピーを作成する
4. 自分の fork リポジトリを開く
5. **Code** ボタンから URL をコピーする

---

## 4. ステップ 2: 自分の fork を clone する

PowerShell で次を実行します。

```powershell
git clone https://github.com/YOUR_ACCOUNT/node-pages-cicd-prtest.git
cd node-pages-cicd-group-sample
```

---

## 5. ステップ 3: `upstream` を設定する

リーダーの新しい共同編集用リポジトリを `upstream` として登録します。

```powershell
git remote add upstream https://github.com/LEADER_ACCOUNT/node-pages-cicd-prtest.git
git remote -v
```

### 5.1 この設定の意味

- `origin`: 自分の fork リポジトリ
- `upstream`: リーダーの新しい共同編集用リポジトリ

---

## 6. ステップ 4: 作業用ブランチを作る

自分の名前に合わせて、次のようなブランチ名を作ります。

```powershell
git switch -c add-message-tanaka
```

または、次でも構いません。

```powershell
git checkout -b add-message-tanaka
```

---

## 7. ステップ 5: `src/messages.json` に自分のメッセージを追加する

### 7.1 追加前の例

```json
[
  {
    "name": "Leader",
    "message": "Welcome to our group GitHub Pages site!"
  }
]
```

### 7.2 追加後の例

```json
[
  {
    "name": "Leader",
    "message": "Welcome to our group GitHub Pages site!"
  },
  {
    "name": "Tanaka",
    "message": "Pull requests make collaborative development safer and clearer."
  }
]
```

### 7.3 編集時の注意

- JSON では最後のカンマや引用符の抜けに注意する
- `name` と `message` は空欄にしない
- 他の人の行を消さない

---

## 8. ステップ 6: ローカルで確認する

変更後、PowerShell で次を実行します。

```powershell
npm test
npm run build
```

成功したら、`dist/index.html` を開いて自分のメッセージが表示されることを確認します。

---

## 9. ステップ 7: commit して fork へ push する

```powershell
git add src/messages.json
git commit -m "Add Tanaka message"
git push -u origin add-message-tanaka
```

---

## 10. ステップ 8: Pull Request を作成する

### 10.1 操作手順

1. 自分の fork リポジトリを GitHub で開く
2. `Compare & pull request` が表示されたらクリックする
3. 比較対象を確認する
   - **base repository**: リーダーの新しい共同編集用リポジトリ
   - **base branch**: `main`
   - **head repository**: 自分の fork
   - **compare branch**: `add-message-自分の名前`
4. タイトルを入力する
   - 例: `Add Tanaka message`
5. 本文を書く
6. **Create pull request** をクリックする

### 10.2 PR 本文の例

```text
I added my message to src/messages.json.
Please review and merge this pull request.
```

---

## 11. ステップ 9: 必要に応じて最新状態を取り込む

ほかのメンバーの PR が先に取り込まれると、自分の branch が古くなることがあります。  
その場合は、`upstream` の最新状態を取り込んでください。

```powershell
git fetch upstream
git switch main
git merge upstream/main
git switch add-message-tanaka
git merge main
```

---

## 12. 起こりやすいトラブル

### 12.1 JSON のカンマミス
- `npm test` や `npm run build` が失敗する

### 12.2 コンフリクト
- 同じ場所を複数人が編集すると、自動マージできないことがある

### 12.3 間違ったリポジトリを fork してしまう
- 元の教材リポジトリではなく、**リーダーが新しく作った共同編集用リポジトリ** を fork しているか確認する

---

## 13. まとめ

メンバーは、**リーダーが新しく作成した共同編集用リポジトリを fork し、そのリポジトリに対して PR を送る**ことが役割です。
