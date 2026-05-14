# ConoHa VPS + Claude Code セットアップエージェント（Mac版）

## あなたの役割
このファイルを読んだら、ユーザーのConoHa VPSにClaude CodeをセットアップしてスマホからRemote Controlで接続できる環境を自動で構築してください。

**重要：** 以前は root のままセットアップしていましたが、現在は **一般ユーザーを作成してsudoで運用** する方針に変わりました。以下の手順では新規ユーザー作成ステップを含めています。

---

## 事前確認（必ずユーザーに聞くこと）

作業を開始する前に以下を確認してください：

1. **ConoHaのログイン情報**
   - メールアドレス
   - パスワード

2. **希望するrootパスワード**（VPS用、英数字記号混じり9文字以上）

3. **希望する一般ユーザー名とパスワード**（Claude Code を運用するためのユーザー）
   - ユーザー名のデフォルト案：`claude-code-user`
   - パスワード：英数字記号混じり9文字以上

4. **Claudeアカウントのメールアドレス**（Claude Code認証用）

確認が取れたら作業を開始してください。

---

## STEP 1: ConoHa VPSを作成する

ブラウザで以下を操作してください：

1. https://cp.conoha.jp を開いてログイン
2. 左メニューの「サーバー追加」をクリック
3. サービスは「VPS」を選択
4. イメージタイプは「OS」タブのまま
5. 下にスクロールして「オプションを見る」をクリック
6. 「スタートアップスクリプト」の「テンプレート」をクリック
7. ドロップダウンから「Claude Code」を選択
8. プランを「4GB」に変更
9. rootパスワードを入力（ユーザーから聞いたもの）
10. 「追加」ボタンをクリック
11. サーバーリストで「起動中」になるまで待つ

**IPアドレスをメモする**（後で使用）

---

## STEP 2: セキュリティグループを変更する

1. 左メニューの「サーバー」をクリック
2. 作成したサーバー名をクリック
3. 「ネットワーク情報」を開く
4. 「セキュリティグループ」の鉛筆アイコンをクリック
5. 「default」を削除
6. 「IPv4v6-SSH」を追加
7. 「保存」をクリック

---

## STEP 3: rootで初回SSH接続して一般ユーザーを作成する

ターミナル（Command + Space で「ターミナル」と検索して開く）で以下を順番に実行してください：

```bash
# SSH接続（rootで一度だけ）
ssh root@{IPアドレス}
# 「yes」と入力してEnter（初回接続時のみ）
# rootパスワードを入力
```

ログインできたら、引き続き以下を実行して一般ユーザーを作成：

```bash
# ユーザーを非対話的に作成（{ユーザー名}と{パスワード}は事前確認で聞いた値）
useradd -m -s /bin/bash {ユーザー名}
echo '{ユーザー名}:{パスワード}' | chpasswd

# sudoグループに追加
usermod -aG sudo {ユーザー名}

# 確認（groupsに sudo が含まれていればOK）
id {ユーザー名}

# rootからログアウト
exit
```

`useradd` の対話プロンプトを避けるため `adduser` ではなく `useradd + chpasswd` を使う点に注意。

---

## STEP 4: 一般ユーザーで再ログインしてClaude Codeをセットアップする

ターミナルで以下を順番に実行：

```bash
# 新しいユーザーでSSH接続
ssh {ユーザー名}@{IPアドレス}
# STEP 3で設定したユーザーのパスワードを入力

# tmuxインストール（sudoパスワードはユーザーのパスワード）
sudo apt install tmux -y

# tmuxセッション作成
tmux new -s claude

# Claude Codeが入っているか確認
which claude

# 入っていなければインストール（ConoHaのテンプレートで入っているはず）
# sudo npm install -g @anthropic-ai/claude-code

# Claude Code起動
claude
# 「1」を押してEnter（Yes, I trust this folder）
# Claudeアカウントでログイン認証を行う

# Remote Control有効化
/remote-control
```

---

## STEP 5: ユーザーにURLを伝える

`/remote-control`実行後に表示されるURLをユーザーに伝えてください：

```
https://claude.ai/code/session_XXXXXXXXXX
```

このURLをスマホのClaudeアプリで開けば接続完了です。

---

## 完了後にユーザーに伝えること

- スマホのClaudeアプリからURLを開けば接続できます
- VPSは24時間動き続けるので基本的にそのまま使えます
- VPS再起動後はターミナルで以下を実行してください：
  ```
  ssh {ユーザー名}@{IPアドレス}
  tmux attach -t claude
  ```
- tmuxが消えていた場合：
  ```
  tmux new -s claude
  claude
  /remote-control
  ```

---

## エラー対応

### SSH接続できない場合
→ セキュリティグループが「IPv4v6-SSH」になっているか確認

### `Permission denied (publickey,password)` と出る
→ 一般ユーザーのパスワードを間違えている可能性。STEP 3 で設定した値を確認

### `sudo: a password is required` と出る
→ `sudo` のパスワードは「一般ユーザーのパスワード」。rootパスワードではない

### `claude: command not found` の場合
```
sudo npm install -g @anthropic-ai/claude-code
```
を実行

### tmuxに戻りたい場合
```
tmux attach -t claude
```

---

## ⚠️ セキュリティ上の注意点

**必ずユーザーに以下を伝えてください：**

1. **ConoHaのパスワードについて**
   ConoHaのログインパスワードはCowork経由でAnthropicのサーバーを通過する可能性があります。セットアップ完了後は**必ずConoHaのパスワードを変更してください**。

2. **rootでのSSH接続を無効化することを推奨**
   一般ユーザーを作って sudo で運用しているので、`/etc/ssh/sshd_config` で `PermitRootLogin no` に変更し、`sudo systemctl restart ssh` を実行すれば、root への直接 SSH を完全に塞げます。本番運用ではこの追加対応を推奨してください。

3. **Remote ControlのセッションURLについて**
   表示されるURLを知っている人は誰でもVPS上のClaude Codeに接続できます。**URLを他人に共有しないでください**。チャット画面にURLが表示された場合は速やかに削除してください。

4. **VPSの各種パスワードについて**
   rootパスワードと一般ユーザーのパスワードは安全な場所に保管してください。紛失するとサーバーにアクセスできなくなります。
