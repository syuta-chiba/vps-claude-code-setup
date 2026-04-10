# ConoHa VPS + Claude Code セットアップエージェント（macOS版）

## あなたの役割
このファイルを読んだら、ユーザーのConoHa VPSにClaude CodeをセットアップしてスマホからRemote Controlで接続できる環境を自動で構築してください。

---

## 事前確認（必ずユーザーに聞くこと）

作業を開始する前に以下を確認してください：

1. **ConoHaのログイン情報**
   - メールアドレス
   - パスワード

2. **希望するrootパスワード**（VPS用、英数字記号混じり9文字以上）

3. **Claudeアカウントのメールアドレス**（Claude Code認証用）

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

## STEP 3: SSH接続してClaude Codeをセットアップする

ターミナル（Command + Space で「ターミナル」と検索して開く）で以下を順番に実行してください：

```bash
# SSH接続
ssh root@{IPアドレス}
# パスワードを聞かれたらrootパスワードを入力
# 初回接続時は「yes」と入力してEnter

# tmuxインストールと起動
apt install tmux -y
tmux new -s claude

# Claude Codeインストール
npm install -g @anthropic-ai/claude-code

# Claude Code起動
claude
# 「1」を押してEnter（Yes, I trust this folder）
# Claudeアカウントでログイン認証を行う

# Remote Control有効化
/remote-control
```

---

## STEP 4: ユーザーにURLを伝える

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
  ssh root@{IPアドレス}
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

### claude: command not foundの場合
```
npm install -g @anthropic-ai/claude-code
```
を再実行

### tmuxに戻りたい場合
```
tmux attach -t claude
```

---

## ⚠️ セキュリティ上の注意点

**必ずユーザーに以下を伝えてください：**

1. **ConoHaのパスワードについて**
   ConoHaのログインパスワードはCowork経由でAnthropicのサーバーを通過する可能性があります。セットアップ完了後は**必ずConoHaのパスワードを変更してください**。

2. **rootでのSSH接続について**
   このセットアップではrootユーザーで接続しています。不正アクセスされた場合サーバーが完全に乗っ取られるリスクがあります。本番環境での使用には一般ユーザーの作成とroot接続の無効化を推奨します。

3. **Remote ControlのセッションURLについて**
   表示されるURLを知っている人は誰でもVPS上のClaude Codeに接続できます。**URLを他人に共有しないでください**。チャット画面にURLが表示された場合は速やかに削除してください。

4. **VPSのrootパスワードについて**
   rootパスワードは安全な場所に保管してください。紛失するとサーバーにアクセスできなくなります。
