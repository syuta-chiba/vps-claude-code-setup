# ConoHa VPS + Claude Code + スマホ Remote Control 手順書（Windows版）

## 必要なもの
- ConoHa アカウント（https://www.conoha.jp）
- Claude Pro アカウント（月額$20）
- Windows PC（初回セットアップのみ）
- スマホ（iOS / Android）にClaudeアプリをインストール済み

---

## STEP 1: ConoHa VPSでサーバーを作る

1. https://cp.conoha.jp にアクセスしてログイン
2. 左メニューの「**サーバー追加**」をクリック
3. サービスは「**VPS**」を選択（最初から選択されている）
4. イメージタイプの「**OS**」タブはそのままにして、下にスクロール
5. 「**オプションを見る**」をクリック
6. 「**スタートアップスクリプト**」の「**テンプレート**」をクリック
7. ドロップダウンから「**Claude Code**」を選択
8. プランを「**4GB（3,968円/月）**」に変更（※4GB未満は動かない）
9. 「**rootパスワード**」を入力（英数字記号混じり9文字以上）→ **必ずメモしておく**
10. 右側の「**追加**」ボタンをクリック
11. サーバーリストに「起動中」と表示されるまで待つ（1〜2分）

---

## STEP 2: セキュリティグループを変更する（重要！）

ConoHaのデフォルト設定は外部からのSSH接続をブロックしています。変更が必要です。

1. 左メニューの「**サーバー**」をクリック
2. 作成したサーバー名をクリック
3. 「**ネットワーク情報**」セクションを開く
4. 「**セキュリティグループ**」の右にある **鉛筆アイコン（編集）** をクリック
5. 「**default**」を削除して「**IPv4v6-SSH**」を追加
6. 「**保存**」をクリック

---

## STEP 3: rootで初回SSH接続する

セットアップ用に、まず**1回だけ**rootでログインします。

PowerShell（Windowsのターミナル）を開いて以下を実行：

```
ssh root@（サーバーのIPアドレス）
```

※IPアドレスはConoHaのサーバーリスト画面で確認できます（例：160.251.215.139）

接続確認のメッセージが出たら：
```
yes
```
と入力してEnter

パスワードを聞かれたら：
→ STEP 1で設定したrootパスワードを入力してEnter
（※入力しても画面に何も表示されないが正常）

`root@vm-XXXXXXXX:~#` と表示されればログイン成功

---

## STEP 4: 一般ユーザーを作成する（重要・セキュリティ向上）

rootのまま運用すると、不正アクセス時にサーバー全体が乗っ取られるリスクがあります。**sudoが使える一般ユーザー**を作って、以降はそのユーザーで使います。

引き続きroot状態のシェルで以下を順に実行：

### 4-1. ユーザーを追加する

```
adduser claude-code-user
```

※`claude-code-user` の部分は好きな名前にしてOK。以降の手順もその名前で読み替える。

対話プロンプトが順に出てきます：

- **New password** → 任意の強いパスワードを設定（必ずメモ）
- **Retype new password** → 同じパスワードを再入力
- **Full Name** などの追加情報 → 全部 **Enter** でスキップでOK
- **Is the information correct? [Y/n]** → `Y` を入力してEnter

### 4-2. sudoグループに追加する

```
usermod -aG sudo claude-code-user
```

### 4-3. 確認

```
id claude-code-user
```

出力例：
```
uid=1000(claude-code-user) gid=1000(claude-code-user) groups=1000(claude-code-user),27(sudo)
```

`groups` のところに `sudo` が入っていればOK。

### 4-4. rootからログアウト

```
exit
```

PowerShellの画面に戻ります。

---

## STEP 5: 新しいユーザーで再ログインする

PowerShellで再度SSH接続。今度は **作ったユーザー名** で：

```
ssh claude-code-user@（サーバーのIPアドレス）
```

STEP 4-1で設定した**そのユーザーのパスワード**を入力。

```
claude-code-user@vm-XXXXXXXX:~$
```

と表示されればOK。
（プロンプトが `#` ではなく `$` になっているのは、一般ユーザーになった証拠）

---

## STEP 6: tmuxを起動する

tmuxはSSHを切っても中のプログラムが動き続けるツールです。

```
sudo apt install tmux -y
```

初回 `sudo` のパスワードを聞かれたら、**STEP 4-1 で設定した一般ユーザーのパスワード**を入力。

インストール後、tmuxセッションを作成：

```
tmux new -s claude
```

画面下に緑のバーが出てくれば成功

---

## STEP 7: Claude Codeを起動・認証する

ConoHa の「Claude Code」スタートアップスクリプトを使っていれば、すでに `claude` コマンドはインストール済みです。

```
claude
```

「このフォルダを信頼しますか？」と聞かれたら：
→ キーボードで「**1**」を押してEnter（Yes, I trust this folder）

「Welcome back ○○！」と表示されれば起動成功

### `claude: command not found` と出た場合

スタートアップスクリプトが効いていない、または別ユーザーの環境に入っていない可能性があります。以下で自分のユーザーにインストール：

```
sudo npm install -g @anthropic-ai/claude-code
```

その後もう一度 `claude` を実行。

---

## STEP 8: Remote Controlを有効にしてスマホと接続する

Claude Codeの画面で：

```
/remote-control
```

と入力してEnter

画面に以下のように表示される：
```
/remote-control is active · Code in CLI or at
https://claude.ai/code/session_XXXXXXXXXX
```

このURLをスマホのClaudeアプリで開く
→ スマホからVPS上のClaude Codeに接続完了！

---

## 毎日の使い方

### 普段（VPSが動き続けている場合）
→ スマホのClaudeアプリからそのまま接続できる

### VPSが再起動した場合（月1回あるかないか）
PCでSSH接続して以下を実行：

```
ssh claude-code-user@（IPアドレス）
tmux attach -t claude
```

tmuxが消えていた場合：

```
tmux new -s claude
claude
/remote-control
```

---

## トラブルシューティング

### SSH接続できない場合
→ ConoHaのセキュリティグループを確認（STEP 2を再確認）

### `Permission denied (publickey,password)` と出る
→ パスワードを間違えている可能性。STEP 4-1 で設定したパスワードを確認

### `sudo: a password is required` と出る
→ `sudo` のパスワードは「一般ユーザーのパスワード」（rootパスワードではない）。STEP 4-1 のものを入力

### `claude: command not found` と出る
```
sudo npm install -g @anthropic-ai/claude-code
```
を実行

### tmuxに戻りたい場合
```
tmux attach -t claude
```

### tmuxから一時的に抜けたい場合（セッションは残る）
`Ctrl + B` を押してから `D` を押す

---

## まとめ

- **VPS（ConoHa）** → 24時間動き続けるサーバー
- **一般ユーザー（claude-code-user）** → rootより安全なログイン用ユーザー
- **tmux** → SSH切断後もClaude Codeを動かし続ける
- **Remote Control** → スマホからVPS上のClaude Codeを操作
- **Claude Codeへの指示** → スマホのClaudeアプリから自然言語で送るだけ
