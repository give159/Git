# Jenkins ローカルPC セットアップ＆ビルド実践ガイド
## 初期設定からビルド実行まで

---

## 📋 本ガイドの流れ

1. **環境準備** - Java・Jenkinsのインストール
2. **初期設定** - 管理者アカウント・プラグイン
3. **ジョブ作成** - GitリポジトリURL指定
4. **ビルド実行** - 手動・自動ビルド
5. **トラブルシューティング**

---

## 🖥️ 1. 環境準備（Windows）

### 1-1. Javaのインストール

**ダウンロード**
- Oracle JDK 11 または OpenJDK 11 をダウンロード
- URL: https://adoptium.net/

**インストール手順**
```
1. インストーラーを実行
2. インストールパスを確認（例：C:\Program Files\Java\jdk-11）
3. 環境変数PATHに追加
```

**確認コマンド**
```cmd
java -version
```

**正常な出力例**
```
openjdk version "11.0.16" 2022-07-19
OpenJDK Runtime Environment (build 11.0.16+8)
```

---

### 1-2. Jenkinsのインストール（Windows）

**方法1: Windowsインストーラー（推奨）**

1. **ダウンロード**
   - https://www.jenkins.io/download/
   - 「Windows」を選択
   - `jenkins.msi` をダウンロード

2. **インストール実行**
   ```
   - インストーラーを実行
   - インストール先: C:\Program Files\Jenkins（デフォルト）
   - サービスとして自動起動
   ```

3. **サービス確認**
   ```cmd
   # サービス管理を開く
   services.msc
   
   # 「Jenkins」サービスが「実行中」であることを確認
   ```

---

**方法2: WAR ファイル（手動起動）**

1. **ダウンロード**
   ```
   https://www.jenkins.io/download/
   → Generic Java package (.war) をダウンロード
   ```

2. **起動**
   ```cmd
   cd C:\jenkins
   java -jar jenkins.war --httpPort=8080
   ```

3. **確認**
   ```
   ブラウザで http://localhost:8080 にアクセス
   ```

---

## 🖥️ 1. 環境準備（Mac）

### 1-1. Homebrewのインストール（未導入の場合）

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

### 1-2. Javaのインストール

```bash
# Homebrewでインストール
brew install openjdk@11

# パスを通す
echo 'export PATH="/usr/local/opt/openjdk@11/bin:$PATH"' >> ~/.zshrc
source ~/.zshrc

# 確認
java -version
```

### 1-3. Jenkinsのインストール

```bash
# Jenkinsインストール
brew install jenkins-lts

# 起動
brew services start jenkins-lts

# 確認
brew services list
```

**アクセス**
```
http://localhost:8080
```

---

## 🖥️ 1. 環境準備（Linux/Ubuntu）

### 1-1. Javaのインストール

```bash
# パッケージリスト更新
sudo apt update

# OpenJDK 11インストール
sudo apt install openjdk-11-jdk -y

# 確認
java -version
```

### 1-2. Jenkinsのインストール

```bash
# Jenkinsリポジトリキー追加
curl -fsSL https://pkg.jenkins.io/debian-stable/jenkins.io-2023.key | sudo tee \
  /usr/share/keyrings/jenkins-keyring.asc > /dev/null

# リポジトリ追加
echo deb [signed-by=/usr/share/keyrings/jenkins-keyring.asc] \
  https://pkg.jenkins.io/debian-stable binary/ | sudo tee \
  /etc/apt/sources.list.d/jenkins.list > /dev/null

# インストール
sudo apt update
sudo apt install jenkins -y

# 起動
sudo systemctl start jenkins
sudo systemctl enable jenkins

# ステータス確認
sudo systemctl status jenkins
```

---

## 🔑 2. 初期設定

### 2-1. 初回アクセス

**ブラウザでアクセス**
```
http://localhost:8080
```

**初期管理者パスワードの取得**

**Windows:**
```cmd
type "C:\Program Files\Jenkins\secrets\initialAdminPassword"
```

**Mac/Linux:**
```bash
sudo cat /var/lib/jenkins/secrets/initialAdminPassword
```

**出力例:**
```
a1b2c3d4e5f6g7h8i9j0k1l2m3n4o5p6
```

このパスワードをコピーしてブラウザに貼り付け

---

### 2-2. プラグインのインストール

**「Install suggested plugins」を選択（推奨）**

自動的にインストールされる主要プラグイン：
- Git plugin
- Pipeline plugin
- Credentials plugin
- Mailer plugin
- etc.

**インストール完了まで待機（5-10分程度）**

---

### 2-3. 管理者ユーザー作成

**入力項目:**
```
ユーザー名: admin
パスワード: [安全なパスワード]
パスワード再入力: [同じパスワード]
フルネーム: Admin User
メールアドレス: admin@example.com
```

**「Save and Continue」をクリック**

---

### 2-4. Jenkins URL設定

**Jenkins URLの確認**
```
http://localhost:8080/
```

- ローカル環境のみで使用する場合はデフォルトのままでOK
- チーム内で共有する場合は、IPアドレスに変更
  ```
  例: http://192.168.1.100:8080/
  ```

**「Save and Finish」→「Start using Jenkins」**

---

## 🎯 3. 最初のジョブ作成（Freestyle Project）

### 3-1. 新規ジョブの作成

**手順:**
```
1. Dashboardで「新規ジョブ作成」をクリック
2. ジョブ名を入力: my-first-build
3. 「フリースタイル・プロジェクトのビルド」を選択
4. 「OK」をクリック
```

---

### 3-2. GitリポジトリURLの指定

**ソースコード管理セクション**

1. **「Git」を選択**

2. **Repository URL を入力**
   ```
   # パブリックリポジトリの例
   https://github.com/username/repository.git
   ```

3. **Credentialsの設定（プライベートリポジトリの場合）**
   ```
   - 「追加」→「Jenkins」をクリック
   - Kind: Username with password
   - Username: GitHubユーザー名
   - Password: Personal Access Token
   - ID: github-credentials
   - 「追加」をクリック
   - ドロップダウンから作成したCredentialsを選択
   ```

4. **ブランチの指定**
   ```
   Branch Specifier: */main
   または
   Branch Specifier: */master
   ```

---

### 3-3. ビルドステップの追加

**「ビルド」セクション**

1. **「ビルド手順の追加」をクリック**

2. **Windows の場合: 「Windowsバッチコマンドの実行」を選択**
   ```cmd
   echo "Build started"
   echo "Current directory: %CD%"
   dir
   echo "Build completed successfully!"
   ```

3. **Mac/Linux の場合: 「シェルの実行」を選択**
   ```bash
   #!/bin/bash
   echo "Build started"
   echo "Current directory: $(pwd)"
   ls -la
   echo "Build completed successfully!"
   ```

---

### 3-4. ビルド後の処理（オプション）

**成果物の保存例:**
```
ビルド後の処理 → 成果物の保存
ファイル: dist/**/*
```

**メール通知:**
```
ビルド後の処理 → E-mail通知
受信者: your-email@example.com
```

**設定を保存**
```
画面下部の「保存」ボタンをクリック
```

---

## 🚀 4. ビルドの実行

### 4-1. 手動ビルド実行

**方法1: Dashboardから**
```
1. Dashboard → ジョブ名の右側の▶アイコンをクリック
2. ビルドが開始される
```

**方法2: ジョブページから**
```
1. ジョブ名をクリック
2. 左メニューの「ビルド実行」をクリック
```

---

### 4-2. ビルド状況の確認

**ビルド履歴**
```
左メニュー「ビルド履歴」に表示される
例: #1, #2, #3...
```

**ステータスの色**
- 🔵 **青（成功）**: ビルド成功
- 🔴 **赤（失敗）**: ビルド失敗
- 🟡 **黄（不安定）**: テスト失敗があるが、ビルドは完了
- ⚪ **灰色**: 実行待ち

---

### 4-3. ビルドログの確認

**Console Output（コンソール出力）の見方**

```
1. ビルド番号（例: #1）をクリック
2. 左メニューの「Console Output」をクリック
3. ログが表示される
```

**ログの例:**
```
Started by user admin
Running as SYSTEM
Building in workspace /var/jenkins_home/workspace/my-first-build
Cloning the remote Git repository
Cloning repository https://github.com/username/repository.git
 > git init /var/jenkins_home/workspace/my-first-build
 > git fetch --tags --progress https://github.com/username/repository.git
Checking out Revision abc123def456
[my-first-build] $ /bin/sh -xe /tmp/jenkins1234567890.sh
+ echo Build started
Build started
+ pwd
/var/jenkins_home/workspace/my-first-build
+ ls -la
total 48
drwxr-xr-x  8 jenkins jenkins 4096 Jan 10 12:00 .
drwxr-xr-x  3 jenkins jenkins 4096 Jan 10 11:59 ..
-rw-r--r--  1 jenkins jenkins  123 Jan 10 12:00 README.md
+ echo Build completed successfully!
Build completed successfully!
Finished: SUCCESS
```

---

## 🔄 5. 自動ビルド設定

### 5-1. GitHubからの自動ビルド（Webhook）

**前提条件:**
- Jenkinsがインターネットからアクセス可能
- または ngrok 等のトンネリングツール使用

**ローカル環境での実現方法（ngrok使用）**

1. **ngrokのインストール**
   ```bash
   # ngrokダウンロード
   https://ngrok.com/download
   
   # 実行
   ngrok http 8080
   ```

2. **公開URLの取得**
   ```
   例: https://abc123.ngrok.io → Jenkins
   ```

3. **Jenkins設定**
   ```
   ジョブ設定 → ビルド・トリガ
   「GitHub hook trigger for GITScm polling」にチェック
   ```

4. **GitHub Webhook設定**
   ```
   GitHub Repository → Settings → Webhooks
   Payload URL: https://abc123.ngrok.io/github-webhook/
   Content type: application/json
   ```

---

### 5-2. 定期ビルド（スケジュール）

**ジョブ設定 → ビルド・トリガ**

**「定期的に実行」にチェック**

**スケジュール記法（Cron形式）:**
```
# 毎日午前2時
0 2 * * *

# 平日の午後6時
0 18 * * 1-5

# 15分ごと
H/15 * * * *

# 毎時0分
H * * * *
```

**フォーマット:**
```
分 時 日 月 曜日
```

---

### 5-3. SCMポーリング

**ジョブ設定 → ビルド・トリガ**

**「SCMをポーリング」にチェック**

**スケジュール:**
```
# 5分ごとにGitをチェック
H/5 * * * *
```

**動作:**
- 指定間隔でGitリポジトリをチェック
- 変更があればビルド実行
- Webhookより負荷が高い

---

## 🔧 6. 実践例：Pythonプロジェクトのビルド

### 6-1. Pythonのインストール

**Windows:**
```
1. https://www.python.org/downloads/ からインストーラーをダウンロード
2. インストール時に「Add Python to PATH」にチェック
3. インストール完了後、確認:
```
```cmd
python --version
pip --version
```

**Mac:**
```bash
# Homebrewでインストール
brew install python@3.11

# 確認
python3 --version
pip3 --version
```

**Linux:**
```bash
# Python3とpipのインストール
sudo apt update
sudo apt install python3 python3-pip python3-venv -y

# 確認
python3 --version
pip3 --version
```

---

### 6-2. 仮想環境の準備（推奨）



---

## 🛠️ 10. Pythonビルドのトラブルシューティング

### 問題1: pip install エラー

**症状:** `ERROR: Could not install packages`

**原因と解決策:**

**1. ネットワーク問題**
```bash
# タイムアウト時間を延長
pip install --timeout=120 -r requirements.txt

# ミラーサイトを使用
pip install -i https://pypi.org/simple -r requirements.txt
```

**2. 権限問題**
```bash
# ユーザー領域にインストール
pip install --user -r requirements.txt

# 仮想環境を使用（推奨）
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

**3. バージョン競合**
```bash
# 依存関係の確認
pip check

# キャッシュクリア
pip cache purge
pip install --no-cache-dir -r requirements.txt
```

---

### 問題2: ModuleNotFoundError

**症状:** `ModuleNotFoundError: No module named 'xxx'`

**解決策:**

**1. 仮想環境が有効化されているか確認**
```bash
# 仮想環境の確認
which python  # 仮想環境のpythonを指しているか

# 再度有効化
source venv/bin/activate  # Mac/Linux
venv\Scripts\activate     # Windows
```

**2. 依存関係の再インストール**
```bash
pip install -r requirements.txt
```

**3. PYTHONPATHの設定**
```bash
export PYTHONPATH="${PYTHONPATH}:${WORKSPACE}/src"
```

---

### 問題3: pytest実行エラー

**症状:** `pytest: command not found`

**解決策:**

```bash
# pytestのインストール確認
pip list | grep pytest

# インストールされていない場合
pip install pytest

# または直接実行
python -m pytest tests/
```

---

### 問題4: 仮想環境作成エラー

**症状:** `The virtual environment was not created successfully`

**解決策:**

**Ubuntu/Debian:**
```bash
# venvモジュールのインストール
sudo apt install python3-venv
```

**その他のLinux:**
```bash
# python3-devのインストール
sudo yum install python3-devel  # CentOS/RHEL
```

---

## 🛠️ 11. トラブルシューティング（全般）

### 問題1: Jenkinsが起動しない

**症状:** ブラウザでアクセスできない

**確認事項:**

**Windows:**
```cmd
# サービス確認
services.msc → Jenkinsサービスの状態を確認

# ポート確認
netstat -an | find "8080"
```

**Mac/Linux:**
```bash
# サービス状態確認
sudo systemctl status jenkins

# ポート確認
sudo lsof -i :8080
```

**解決策:**
- サービスを再起動
- ポート8080が他のプログラムで使用されていないか確認
- ファイアウォール設定を確認

---

### 問題2: Git clone/fetch エラー

**症状:** `Failed to connect to repository`

**原因:**
- URLが間違っている
- 認証情報が正しくない
- ネットワーク接続の問題

**解決策:**

1. **URLの確認**
   ```
   正: https://github.com/username/repo.git
   誤: github.com/username/repo
   ```

2. **認証情報の確認**
   ```
   - Personal Access Token の権限を確認
   - Token が有効期限切れでないか確認
   ```

3. **手動確認**
   ```bash
   # コマンドラインで確認
   git clone https://github.com/username/repo.git
   ```

---

### 問題3: ビルドスクリプトエラー

**症状:** `npm: command not found`

**原因:**
- Node.jsがインストールされていない
- PATHが通っていない

**解決策:**

**Jenkinsの環境変数を確認:**
```
Manage Jenkins → System Configuration → System
環境変数セクションでPATHを追加
```

**ビルドスクリプトでPATHを明示:**
```bash
# Mac/Linux
export PATH=/usr/local/bin:$PATH
node --version
```

```cmd
# Windows
set PATH=C:\Program Files\nodejs;%PATH%
node --version
```

---

### 問題4: 権限エラー

**症状:** `Permission denied`

**原因:**
- Jenkinsユーザーの権限不足

**解決策:**

**Linux:**
```bash
# ワークスペースの権限変更
sudo chown -R jenkins:jenkins /var/lib/jenkins/workspace

# Docker使用時
sudo usermod -aG docker jenkins
sudo systemctl restart jenkins
```

**Windows:**
- Jenkinsサービスを管理者権限で実行
- フォルダの権限設定を確認

---

## 📊 12. Pythonビルド結果の確認ポイント

### チェックリスト

**✅ Pythonビルド成功の確認**
```
□ ステータスが青（成功）
□ すべてのテストがPASSED
□ Console Outputに "Finished: SUCCESS"
□ カバレッジが基準を満たしている
□ リント警告がない（または許容範囲内）
□ 仮想環境が正しく作成・使用されている
```

**❌ ビルド失敗時の確認ポイント**
```
□ pytest出力でFAILEDテストを確認
□ トレースバックでエラー箇所を特定
□ requirements.txtの依存関係を確認
□ Pythonバージョンの互換性を確認
□ 環境変数・PYTHONPATHを確認
```

**📊 テスト結果の見方**
```
======================== test session starts =========================
platform linux -- Python 3.11.0, pytest-7.4.0
collected 10 items

tests/test_main.py::test_add ........                          [ 80%]
tests/test_api.py::test_endpoint ..                            [100%]

========================= 10 passed in 0.50s =========================
```

---

## 🎯 13. Pythonプロジェクトでの次のステップ

### やってみよう（Python編）

**初級:**
- [ ] シンプルなPythonスクリプトのビルドジョブを作成
- [ ] pytestで簡単なテストを実行
- [ ] Console Outputでテスト結果を確認

**中級:**
- [ ] 仮想環境を使用したビルド設定
- [ ] カバレッジレポートの生成と保存
- [ ] リントチェック（flake8, black）の導入
- [ ] requirements.txtの依存関係管理

**上級:**
- [ ] Django/Flaskプロジェクトのビルド
- [ ] 複数のPythonバージョンでテスト
- [ ] Dockerを使用したビルド環境
- [ ] Pipelineに移行してJenkinsfile作成

**実践プロジェクト例:**
```python
# 簡単な電卓アプリ
# src/calculator.py
def add(a, b):
    return a + b

def divide(a, b):
    if b == 0:
        raise ValueError("Division by zero")
    return a / b

# tests/test_calculator.py
import pytest
from src.calculator import add, divide

def test_add():
    assert add(2, 3) == 5

def test_divide():
    assert divide(10, 2) == 5
    
def test_divide_by_zero():
    with pytest.raises(ValueError):
        divide(10, 0)
```

---

## 📚 まとめ（Python対応版）

### 本ガイドで学んだこと

1. **環境構築**: Jenkins・Java・Pythonのインストール
2. **初期設定**: 管理者作成・プラグイン導入
3. **ジョブ作成**: GitリポジトリURL指定
4. **Pythonビルド**: 仮想環境・pytest・カバレッジ
5. **ビルド実行**: 手動・自動ビルド
6. **トラブル対応**: Python特有の問題と解決策

### 🐍 Pythonプロジェクトでの重要ポイント

- **仮想環境の使用**: プロジェクトごとに環境を分離
- **requirements.txt**: 依存関係を明確に管理
- **pytest**: テスト自動化の標準ツール
- **コード品質**: black（フォーマット）、flake8（リント）
- **カバレッジ**: テストのカバー率を測定

### 🚀 実践のコツ

- **小さく始める**: まずシンプルなテストから
- **ログを読む**: pytest出力とConsole Outputを確認
- **試行錯誤**: エラーは学習の機会
- **記録する**: 設定とトラブル対応をドキュメント化
- **段階的に**: 機能を徐々に追加していく

---

## 🔗 参考リソース（Python追加版）

**公式ドキュメント:**
- Jenkins公式: https://www.jenkins.io/doc/
- pytest公式: https://docs.pytest.org/
- Python公式: https://docs.python.org/

**Python CI/CDリソース:**
- pytest ベストプラクティス
- Python パッケージングガイド
- tox（複数環境テスト）

**コミュニティ:**
- Stack Overflow: [jenkins][python] タグ
- pytest Discord
- Python Discord

**おすすめ書籍:**
- 「Jenkins実践入門」
- 「Pythonテスト入門」
- 「Python実践レシピ」

---

## ✨ お疲れさまでした！

これでローカルPCでのJenkins環境構築と**Pythonプロジェクト**のビルド実行ができるようになりました。

PythonとJenkinsを組み合わせて、品質の高いコードを効率的に開発しましょう！ 🎉🐍