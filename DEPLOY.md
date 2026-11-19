# サービス紹介LP デプロイ手順（Coolify ＋ 独自サブドメイン）

公開URL（目標）：**https://lp.s-toru.com**
構成：静的HTML 1枚（`index.html`）を nginx で配信（`Dockerfile` 同梱・isaka-lp と同方式）

---

## 0. 前提・値

| 項目 | 値 |
|---|---|
| VPS IP | `163.44.103.9` |
| Coolify UI | http://163.44.103.9:8000 （Email: shittoru.ad@gmail.com） |
| Coolify project | My first project（`awydvhju4mbnqml4bj0s77mn`）/ env: production |
| GitHub org | `shittoruad-gif`（公開リポジトリ） |
| 公開ドメイン | `lp.s-toru.com` |
| 保有ドメイン | `s-toru.com`（Squarespace／旧Google Domains） |

> Coolify API は外部遮断（iptables）。API操作は VPS へ SSH してから `localhost:8000` に対して実行する。
> 自動デプロイ cron は threads-studio のみ監視 → このアプリは push 後に**手動デプロイAPI**を叩く必要あり。

---

## 1. GitHub にリポジトリを作成して push

```bash
cd /Users/kabushikikaishashitsutoru/CODE/service-lp
gh repo create shittoruad-gif/service-lp --public --source=. --remote=origin --push
```

（既に git init・初回コミット済み。`gh auth status` で認証を確認してから実行）

---

## 2. DNS：サブドメインを追加（無料）

Squarespace の DNS設定 → **カスタムレコードを追加**（既存レコードは触らない）：

| Host | Type | Data |
|---|---|---|
| `lp` | `A` | `163.44.103.9` |

- 既存メール（Google Workspace）のレコードはそのまま。`lp` を新規追加するだけ。
- 編集時に Google Workspace の再認証ダイアログが出る場合あり（要・本人操作）。
- 反映確認：`dig +short lp.s-toru.com`（→ `163.44.103.9` が返ればOK、数分〜）

---

## 3. Coolify にアプリを作成

UI（http://163.44.103.9:8000）から：

1. project「My first project」内で **+ New → Application**
2. **Public Repository** → `https://github.com/shittoruad-gif/service-lp`、Branch `main`
3. **Build Pack: Dockerfile**
4. 作成後、**Domains（FQDN）** に `https://lp.s-toru.com` を設定（https:// 付きで入れると Let's Encrypt 自動発行）
5. **Deploy** 実行

### （参考）SSH＋API で手動デプロイする場合

```bash
ssh -i ~/.ssh/id_ed25519 root@163.44.103.9
# VPS内で（<UUID> は作成したアプリのUUIDに置換）
curl -s -H "Authorization: Bearer $(cat ~/.claude/secrets/coolify.token)" \
  "http://localhost:8000/api/v1/deploy?uuid=<UUID>&force=true"
```

---

## 4. 確認

- `https://lp.s-toru.com` をPC・スマホ両方で開く（SSL 鍵マーク・全セクション表示・各ボタンのメール起動）
- 以降の更新は `git push` → 手順3の手動デプロイAPI（または UI の Redeploy）

---

## 更新時メモ

- 中身を直す → `index.html` を編集 → commit → push → Coolify で Redeploy
- 文言・価格・リンクはすべて `index.html` 内（外部依存なし）
