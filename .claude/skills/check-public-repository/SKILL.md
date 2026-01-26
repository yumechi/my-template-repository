---
name: check-public-repository
description: リポジトリを公開する前に、機密情報や不適切なデータが含まれていないかチェックします
---

# リポジトリ公開前チェック

このスキルはリポジトリを公開する前に、以下の項目を確認します。

## チェック項目

### 1. 必須ファイルの存在確認
- [ ] `README.md` が存在すること
- [ ] `LICENSE` または `LICENCE` ファイルが存在すること

### 2. 機密情報のチェック
以下のパターンがリポジトリに含まれていないことを確認してください。

#### 鍵・認証情報
- [ ] 秘密鍵ファイル（`*.pem`, `*.key`, `id_rsa`, `id_ed25519` など）
- [ ] AWS アクセスキー（`AKIA` で始まる文字列）
- [ ] API キー・トークン（`api_key`, `apikey`, `access_token`, `secret_key` など）
- [ ] `.env` ファイルに実際の認証情報が含まれていないこと

#### パスワード
- [ ] ハードコードされたパスワード（`password =`, `passwd =`, `pwd =` など）
- [ ] データベース接続文字列に含まれるパスワード

#### 個人的な環境パス
- [ ] `/Users/<username>/` のような個人のホームディレクトリパス
- [ ] `/home/<username>/` のような個人のホームディレクトリパス
- [ ] `C:\Users\<username>\` のような Windows の個人パス

#### メールアドレス
- [ ] ダミー以外の実際のメールアドレス（`example.com`, `example.org` ドメイン以外）
- [ ] 個人のメールアドレス

#### その他の個人情報
- [ ] 電話番号
- [ ] 住所
- [ ] クレジットカード番号のようなパターン
- [ ] マイナンバーのようなパターン

## チェック手順

1. `README.md` と `LICENSE`/`LICENCE` ファイルの存在を確認
2. `git ls-files` でトラッキングされているファイル一覧を取得
3. 機密情報のパターンを grep で検索
4. 検出された問題をレポート

## 実行例

```bash
# 必須ファイルの確認
ls README.md LICENSE LICENCE 2>/dev/null

# 秘密鍵ファイルの検索
git ls-files | grep -E '\.(pem|key)$|id_rsa|id_ed25519'

# AWS キーのパターン検索
git grep -E 'AKIA[A-Z0-9]{16}'

# パスワードパターンの検索
git grep -iE '(password|passwd|pwd)\s*[=:]\s*["\x27][^"\x27]+'

# 個人パスの検索
git grep -E '/(Users|home)/[a-zA-Z0-9_-]+/'

# メールアドレスの検索（example.com/org 以外）
git grep -E '[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}' | grep -v 'example\.\(com\|org\)'
```

## 注意事項

- `.gitignore` に含まれているファイルはチェック対象外ですが、誤ってコミットされていないか確認してください
- 偽陽性（false positive）が発生する場合があります。検出結果を手動で確認してください
- このチェックは完全ではありません。公開前に必ず手動でも確認してください
