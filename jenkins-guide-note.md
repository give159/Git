# 【新人エンジニア向け】Jenkinsインストール完全ガイド

こんにちは！この記事では、CI/CDツール「Jenkins」のインストール方法を、新人エンジニアの方にも分かりやすく解説します。

## 📌 Jenkinsとは？

JenkinsはCI/CD（継続的インテグレーション/継続的デリバリー）を実現するための自動化ツールです。

**できること：**
- コードのビルド自動化
- テストの自動実行
- デプロイの自動化
- 定期的なジョブの実行

---

## 🔧 事前準備

インストール前に以下を確認してください：

- ✅ Javaのインストール（Java 11または17を推奨）
- ✅ 管理者権限
- ✅ 最低2GB以上のメモリ
- ✅ ディスク容量10GB以上（推奨）

---

## 💻 Mac でのインストール

### 手順1: Homebrewでインストール

```sh
# Javaがインストールされているか確認
java -version

# Javaがない場合はインストール（Java 11を推奨）
brew install openjdk@11

# Jenkins LTS版をインストール
brew install jenkins-lts

# Jenkinsをバックグラウンドサービスとして起動
brew services start jenkins-lts
```

### 手順2: ブラウザでアクセス

1. ブラウザで `http://localhost:8080` にアクセス
2. 初期パスワードを入力（次のコマンドで確認）

```sh
cat /Users/$(whoami)/.jenkins/secrets/initialAdminPassword
```

**参考リンク：** https://www.jenkins.io/download/lts/macos/

---

## 🐧 Linux（CentOS/Red Hat）でのインストール

```sh
# Jenkinsリポジトリの追加
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo

# GPGキーのインポート
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

# システムのアップグレード
sudo yum upgrade -y

# 必要なパッケージのインストール
sudo yum install epel-release java-11-openjdk-devel -y

# Jenkinsのインストール
sudo yum install jenkins -y

# サービスの起動
sudo systemctl daemon-reload
sudo systemctl start jenkins

# 起動確認
sudo systemctl status jenkins
```

**参考リンク：** https://www.jenkins.io/doc/book/installing/linux/#red-hat-centos

---

## ☁️ Amazon Linux 2 でのインストール

AWS EC2インスタンスでJenkinsを動かす場合の手順です。

```sh
# システムのアップデート
sudo yum update -y

# Jenkinsリポジトリの追加
sudo wget -O /etc/yum.repos.d/jenkins.repo \
    https://pkg.jenkins.io/redhat-stable/jenkins.repo

# GPGキーのインポート
sudo rpm --import https://pkg.jenkins.io/redhat-stable/jenkins.io.key

# システムのアップグレード
sudo yum upgrade -y

# EPELリポジトリの追加（重要！）
# これがないと "Requires: daemonize" エラーが発生します
sudo amazon-linux-extras install epel -y

# 再度アップデート
sudo yum update -y

# JenkinsとJavaのインストール
sudo yum install jenkins java-1.8.0-openjdk-devel -y

# サービスの起動
sudo systemctl daemon-reload
sudo systemctl start jenkins

# 起動確認
sudo systemctl status jenkins
```

### ✅ 成功時の出力例

```
● jenkins.service - LSB: Jenkins Automation Server
   Loaded: loaded (/etc/rc.d/init.d/jenkins; bad; vendor preset: disabled)
   Active: active (running) since Sun 2021-08-29 14:58:51 UTC; 2s ago
     Docs: man:systemd-sysv-generator(8)
  Process: 4751 ExecStart=/etc/rc.d/init.d/jenkins start (code=exited, status=0/SUCCESS)
   CGroup: /system.slice/jenkins.service
           └─4755 /etc/alternatives/java -Djava.awt.headless=true...
```

### 🔒 セキュリティグループの設定（AWS EC2の場合）

EC2のセキュリティグループで**ポート8080**を開放してください：

- Type: Custom TCP
- Port: 8080
- Source: My IP（または適切な範囲）

**参考リンク：** https://www.jenkins.io/doc/tutorials/tutorial-for-installing-jenkins-on-AWS/

---

## 🪟 その他のプラットフォーム

### Windows
https://www.jenkins.io/doc/book/installing/windows/

### Docker
https://www.jenkins.io/doc/book/installing/docker/

---

## 🎯 初期セットアップの流れ

### 1. 初期パスワード入力画面

初回アクセス時に初期パスワードが求められます。

```sh
# パスワードの確認方法
# Linux/Amazon Linux 2の場合
sudo cat /var/lib/jenkins/secrets/initialAdminPassword

# Macの場合
cat /Users/$(whoami)/.jenkins/secrets/initialAdminPassword
```

### 2. プラグイン選択画面

**「Install suggested plugins」を選択**（初心者におすすめ）

これで必要な基本プラグインが自動的にインストールされます。

### 3. 管理者ユーザー作成

以下の情報を入力します：
- ユーザー名
- パスワード
- メールアドレス

### 4. 完了！

Jenkinsのダッシュボードが表示されます 🎉

---

## ⚠️ よくある問題と解決方法

### 問題1: ポート8080が既に使用されている

```sh
# ポート8080を使用しているプロセスを確認
lsof -i :8080

# プロセスを停止するか、Jenkinsを別のポートで起動
```

### 問題2: Jenkinsが起動しない

```sh
# ログを確認
# Linux/Amazon Linux 2
sudo tail -f /var/log/jenkins/jenkins.log

# Mac
tail -f /usr/local/var/log/jenkins/jenkins.log

# サービスの状態確認
sudo systemctl status jenkins  # Linux
brew services list              # Mac
```

### 問題3: メモリ不足エラー

- 最低2GB以上の空きメモリを確保
- 他のアプリケーションを終了してから再試行
- EC2インスタンスの場合はt2.small以上を推奨

---

## 🔧 【上級者向け】Credentials プラグインのインストールエラー対処法

Credentials plugin v2.6のインストール時にエラーが発生する場合の対処法です。

**注意**: 通常は不要な作業です。エラーが出た場合のみ実行してください。

### 原因
プラグインの依存関係の問題が原因です。

### 解決方法

```sh
# 作業用ディレクトリに移動
cd ~

# プラグイン管理ツールをダウンロード
wget https://github.com/jenkinsci/plugin-installation-manager-tool/releases/download/2.11.0/jenkins-plugin-manager-2.11.0.jar

# Jenkinsの場所を確認
sudo find / -type f -name "jenkins.war"
# 出力例: /usr/lib/jenkins/jenkins.war

# 古いバージョン(v2.5)をダウンロード
java -jar jenkins-plugin-manager-*.jar \
  --war /usr/lib/jenkins/jenkins.war \
  --plugins credentials:2.5 \
  -d .

# プラグインファイルを配置
sudo cp credentials.jpi /var/lib/jenkins/plugins/

# Jenkinsを再起動
sudo systemctl restart jenkins  # Linux/Amazon Linux 2
brew services restart jenkins-lts  # Mac
```

**参考リンク：**
- https://github.com/jenkinsci/plugin-installation-manager-tool#getting-started
- https://stackoverflow.com/a/14953877/1528958
- https://plugins.jenkins.io/credentials/#releases

---

## 📚 インストール後にやること

### 1. セキュリティ設定の確認

「Manage Jenkins」→「Configure Global Security」で以下を確認：
- 匿名ユーザーのアクセスを制限
- 適切な認証方法の設定

### 2. 最初のジョブを作成してみよう

1. 「New Item」をクリック
2. 「Freestyle project」を選択
3. 「Build」セクションで「Execute shell」を追加
4. 以下のコマンドを入力：

```sh
echo "Hello Jenkins!"
date
```

5. 「Save」→「Build Now」で実行

### 3. 学習リソース

- [Jenkins公式ドキュメント](https://www.jenkins.io/doc/)
- [Jenkins入門チュートリアル](https://www.jenkins.io/doc/tutorials/)
- [Jenkinsパイプライン入門](https://www.jenkins.io/doc/book/pipeline/)

---

## 🆘 困ったときは

### デバッグの基本手順

1. **エラーメッセージを確認**
   - ログファイル: `/var/log/jenkins/jenkins.log`
   - ブラウザのコンソール（F12キー）

2. **Google検索**
   - エラーメッセージをそのまま検索
   - 「Jenkins」+ エラー内容で検索

3. **公式リソース**
   - [Jenkins公式フォーラム](https://community.jenkins.io/)
   - [Stack Overflow](https://stackoverflow.com/questions/tagged/jenkins)

---

## ✨ まとめ

Jenkinsのインストールは、環境に応じた適切な手順を踏めば難しくありません。

**ポイント：**
- ✅ 環境に合った手順を選ぶ
- ✅ ログを確認する習慣をつける
- ✅ 公式ドキュメントを参照する
- ✅ 小さく始めて徐々に学ぶ

この記事が皆さんのJenkins導入の助けになれば幸いです！

---

💬 質問や改善点があれば、コメント欄でお気軽にお知らせください。

#Jenkins #CI/CD #DevOps #新人エンジニア #インフラ #自動化