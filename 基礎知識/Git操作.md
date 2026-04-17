# Git 基本操作

---

## 1. Git とは

ファイルの変更履歴を記録・管理するバージョン管理システム。

```
ローカル環境（PC）             リモート（GitHub など）
┌──────────────────────┐      ┌─────────────────┐
│ 作業ディレクトリ       │      │                 │
│   ↓ git add          │      │   origin/main   │
│ ステージングエリア     │      │                 │
│   ↓ git commit       │      │                 │
│ ローカルリポジトリ     │ push │                 │
│                      │ ───→ │                 │
│                      │ ←─── │                 │
│                      │ pull │                 │
└──────────────────────┘      └─────────────────┘
```

---

## 2. 初期設定

```bash
# ユーザー名とメールを設定（初回のみ）
git config --global user.name "田中太郎"
git config --global user.email "tanaka@example.com"

# 設定を確認
git config --list
```

---

## 3. リポジトリの作成・クローン

```bash
# 新しいリポジトリを作成
git init

# GitHub などからリポジトリをコピー
git clone https://github.com/ユーザー名/リポジトリ名.git
```

---

## 4. 基本の流れ（add → commit → push）

```bash
# 変更状態を確認
git status

# 変更差分を確認
git diff

# ファイルをステージングに追加
git add ファイル名
git add .         # 全ファイルを追加（注意：不要なファイルも入る場合あり）

# コミット（変更を記録）
git commit -m "変更内容の説明"

# リモートに送信
git push origin main
```

---

## 5. ブランチ操作

ブランチ = 作業の「分岐」。本番コードを変えずに新機能を開発できる。

```bash
# ブランチ一覧を確認
git branch        # ローカル
git branch -a     # リモートも含む

# ブランチを作成して切り替え
git switch -c feature/新機能

# 既存ブランチに切り替え
git switch main

# ブランチを削除（マージ済み）
git branch -d feature/新機能

# ブランチを強制削除（マージ前でも）
git branch -D feature/新機能
```

---

## 6. マージ

ブランチの変更を別のブランチに取り込む。

```bash
# main に feature ブランチをマージ
git switch main
git merge feature/新機能

# マージコミットを作らずに取り込む（履歴が直線になる）
git merge --squash feature/新機能
git commit -m "新機能を追加"
```

---

## 7. リベース

コミット履歴をきれいに保つ。チームで使う場合はルールを確認してから使う。

```bash
# feature ブランチを main の最新に追いつかせる
git switch feature/新機能
git rebase main
```

### merge vs rebase

| 比較 | merge | rebase |
| --- | --- | --- |
| 履歴の形 | 分岐・合流がそのまま残る | 直線になる |
| 安全性 | 安全（履歴を書き換えない） | プッシュ済みのブランチには危険 |
| 使いどき | チーム開発での統合 | 個人ブランチを最新に追いつかせるとき |

---

## 8. コンフリクト（競合）の解消

同じ箇所を別々に変更した場合に発生する。

```bash
# コンフリクト発生
git merge feature/新機能
# CONFLICT (content): Merge conflict in ファイル名

# ファイルを開くと以下のようなマーカーが入っている
<<<<<<< HEAD
現在のブランチの内容
=======
マージしようとした内容
>>>>>>> feature/新機能
```

1. ファイルを編集して正しい内容に直す
2. マーカー（`<<<<<<<` など）を削除する
3. `git add ファイル名` でステージング
4. `git commit` でコンフリクト解消を記録

---

## 9. 変更を元に戻す

```bash
# まだ add していない変更を元に戻す
git restore ファイル名

# add したファイルをステージングから外す（変更は残る）
git restore --staged ファイル名

# 直前のコミットを取り消す（変更は作業ディレクトリに残る）
git reset HEAD~1

# 直前のコミットを取り消す（変更も消える）
git reset --hard HEAD~1

# コミットを取り消す新しいコミットを作る（安全・プッシュ済みでも使える）
git revert HEAD
```

---

## 10. ログと履歴確認

```bash
# コミット履歴を表示
git log

# 1行で表示
git log --oneline

# グラフ付きで表示
git log --oneline --graph --all

# 特定ファイルの変更履歴
git log --follow ファイル名

# 誰がどこを変更したか
git blame ファイル名
```

---

## 11. stash：作業を一時退避する

作業中に別ブランチに切り替えたいときに使う。

```bash
# 現在の変更を一時保存
git stash

# 退避した変更を戻す
git stash pop

# 退避リストを確認
git stash list

# 指定した退避を戻す
git stash apply stash@{1}
```

---

## 12. タグ：リリース管理

```bash
# タグを作成
git tag v1.0.0

# タグにメッセージを付ける
git tag -a v1.0.0 -m "初回リリース"

# タグをリモートに送信
git push origin v1.0.0

# タグ一覧
git tag

# タグを削除（ローカル）
git tag -d v1.0.0
```

---

## 13. .gitignore：管理対象外のファイルを指定

```gitignore
# Windows 系
Thumbs.db
desktop.ini

# ビルド出力
bin/
obj/

# 設定ファイル（秘密情報）
*.env
appsettings.Development.json

# IDE
.vs/
*.user
```

---

## 14. Git フロー（ブランチ戦略）

| ブランチ | 役割 |
| --- | --- |
| `main` | 本番リリース済みのコード |
| `develop` | 開発の統合ブランチ |
| `feature/xxx` | 新機能の開発 |
| `bugfix/xxx` | バグ修正 |
| `release/xxx` | リリース準備 |
| `hotfix/xxx` | 本番の緊急修正 |
