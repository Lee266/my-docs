# SSH

## File

- id_rsa: 秘密鍵ファイル
- id_rsa.pub: 公開鍵ファイル
- authorized_keys: 公開鍵を登録しておくファイル

## SSH生成コマンド

```bash
ssh-keygen -t ed25519 -C ""
ssh-keygen -t rsa -b 4096 -C ""
```

- 注意
  - ""の中は他のキーを区別する為のコメントでemailなどが使われる。

## Mac

- SSHの設定ファイルの確認
  § ls -l ~/.ssh/config
- ファイルがない場合
  § mkdir ^/.ssh
  § touch ~/.ssh/config
- SSHキー作成
  § SSH生成コマンドを実行
  § 特に指定がなければ全てEnter
- 公開鍵をコピーする
  § pbcopy < ~/.ssh/[file_name].pub

## Windows

- SSHの設定ファイルの確認
  - dir .ssh/config
- ファイルがない場合
  - mkdir ^/.ssh
  - touch ~/.ssh/config
- SSHキー作成
  - SSH生成コマンドを実行
  - 特に指定がなければ全てEnter
- 公開鍵をコピーする
  - Get-Content .ssh/[file_name].pub | clip

### server側に直接送る場合

```bash
# check ssh server
username@hostname

scp ~/.ssh/[file_name].pub username@hostname:~/.ssh/register_key
ssh username@hostname
cd .ssh
cat  cat register_key >> authorized_keys
chmod 600 authorized_keys
rm register_key
```

### SSH config

```bash
# ~/.ssh/config

Host
  HostName
  Port
  User
  IdentityFile
```

- Host: アクセス先の変更名前
- Hostname: アクセス先のサーバのホスト名orIPアドレス
- Port: SSHのポート番号(default 22)
- IdentityFile: 秘密鍵のファイルrul
- User: アクセス先のサーバーのアクセスしたいユーザー名

## サーバー側(接続先)

### SSHサーバーのインストールと事前作業

#### SSHサーバーのインストール

```bash
sudo apt -y install openssh-server
```

#### SSHサーバー動作確認

```bash
systemctl status ssh

#if stop ssh server
systemctl restart ssh
```

実際に確認する

```bash
# username
whoaim
# hostname
hostname

ssh username@hostname
# or
ssh username@192.234.234.234
```

### ssh設定ファイルの変更

- ポートの番号の変更: SSH のウェルノウンポートである22番号ポートを使用すると攻撃対象になるので、1024~65535にする。
  - Well-Known ports: 0~1023
  - registered ports: 1024~49151
  - dynamic ports: 49152~65535
- Root Userへのログインを禁止: SSHで直接Rootユーザーとしてログインできると安全ではないために、SSHからログインできないようにする
- パスワードによるログインを禁止: パスワードによるログインを許可すると、総当たり攻撃の的になるのでSSHでログインする方法を鍵のみにする

```bash
# /etc/ssh/sshd_config
# change port number
Port <number>

# not access root form ssh
- PermitRootLogin prohibit-password
+ PermitRootLogin no

# not use password
- #PasswordAuthentication yes
+ PasswordAuthentication no
```

#### 構文チェック

```bash
sudo sshd -t
```

### ファイヤーウォールの設定の変更

- SSHのポート番号を変更したので、ファイヤーウォールの設定を変更する
- デフォルト22番号を遮断して、設定した番号を通過

```bash
sudo ufw deny ssh
# set your setting port number
sudo ufw allow <number>
sudo ufw enable

# check port
sudo ufw status
```

#### デーモンの再起動

- SSH と UFW のデーモンを再起動する。

- ただし、いきなり再起動してしまうと、もし設定が間違っていてログインできなくなったときにサーバから締め出されてしまいます。そこで、まずは SSH デーモンだけを再起動して、別のシェルでログインする。

##### sshの再起動

```bash
sudo systemctl restart ssh

# check
sudo systemctl status ssh
```

###### UFWの再起動

```bash
ssh username@globalipadd -p number -i
```

```bash
sudo ufw reload
```
