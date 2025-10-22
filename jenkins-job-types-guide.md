# 【完全図解】Jenkins Job Types - Freestyle vs Pipeline vs Multi-branch Pipeline

こんにちは！「Jenkinsのジョブって種類がたくさんあって、どれを使えばいいの？」と悩んでいる新人エンジニアの方へ。この記事で、各ジョブタイプの違いと使い分けを徹底解説します！

---

## 📌 Jenkins Jobとは？

Jenkins Jobは、**ユーザーが定義した一連のタスク**のことです。

### 典型的なCI/CDワークフロー

```
1. 📥 Fetch
   ↓ Gitリポジトリから最新コードを取得
   
2. 🔨 Build
   ↓ コンパイル・ビルド（Ant, Maven, Gradle, Docker等）
   
3. 🧪 Test
   ↓ ユニットテスト・統合テストの実行
   
4. 📦 Archive
   ↓ ビルド成果物を保存
   
5. 📢 Notify
   ↓ 開発者に結果を通知（Slack, メール等）
```

---

## 🎯 Jenkins Job Typesの全体像

| タイプ | 難易度 | おすすめ度 | 用途 |
|--------|--------|-----------|------|
| **Freestyle Project** | ⭐ | ⭐⭐⭐ | シンプルなビルド |
| **Pipeline** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | 複雑なCI/CD |
| **Multi-branch Pipeline** | ⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ブランチごとのビルド |
| **Maven Project** | ⭐⭐ | ⭐⭐ | Mavenプロジェクト専用 |
| **Multi-Configuration** | ⭐⭐⭐ | ⭐⭐ | 複数環境でのテスト |
| **External Job** | ⭐⭐ | ⭐ | 外部プロセスの監視 |

---

## 🎨 1. Freestyle Project - 初心者向け

### 📌 特徴

**最もシンプルで柔軟なジョブタイプ**

- ✅ GUI で全て設定
- ✅ 学習コストが低い
- ✅ 小規模プロジェクト向け
- ❌ バージョン管理できない
- ❌ 複雑なワークフローには不向き

---

### 🚀 作成方法

```
1. New Item をクリック
2. ジョブ名を入力: my-freestyle-job
3. Freestyle project を選択
4. OK をクリック
```

---

### 🔧 設定項目の詳細解説

#### 1️⃣ General（一般設定）

**Build履歴の管理:**
```
┌─────────────────────────────────────┐
│ Discard old builds                  │
├─────────────────────────────────────┤
│ Strategy: Log Rotation              │
│ ├─ Days to keep builds: 30日       │
│ ├─ Max # of builds to keep: 50     │
│ ├─ Days to keep artifacts: 7日     │
│ └─ Max # to keep with artifacts: 10│
└─────────────────────────────────────┘

💡 ディスク容量を節約するため、
   古いビルドは自動削除がおすすめ！
```

---

**Build管理の詳細設定:**

| 項目 | 説明 | 推奨値 |
|------|------|--------|
| **Quiet period** | ビルド間の待機時間（秒） | 5秒 |
| **Retry Count** | Git checkoutの再試行回数 | 3回 |
| **Disable this project** | ジョブを無効化 | - |

**Quiet periodの使い方:**
```
コミット1 → [5秒待機] → ビルド開始
コミット2 ↗

複数のコミットをまとめてビルド可能
```

**Retry Countの使い方:**
```
Git checkout失敗
↓
再試行 1回目
↓ 失敗
再試行 2回目
↓ 失敗
再試行 3回目
↓ 成功！

ネットワークの一時的な問題に対応
```

---

#### 2️⃣ Source Code Management（ソースコード管理）

**Git の設定:**
```
┌─────────────────────────────────────┐
│ Source Code Management              │
├─────────────────────────────────────┤
│ ○ None                              │
│ ● Git                               │
│                                     │
│ Repository URL:                     │
│ https://github.com/user/repo.git    │
│                                     │
│ Credentials:                        │
│ [github-credentials ▼]              │
│                                     │
│ Branches to build:                  │
│ */main                              │
└─────────────────────────────────────┘
```

**複数ブランチの指定:**
```
*/main          ← mainブランチ
*/develop       ← developブランチ
*/feature/*     ← すべてのfeatureブランチ
```

---

#### 3️⃣ Build Triggers（ビルドトリガー）

**自動ビルドの設定方法**

##### オプション1: Build after other projects（他プロジェクト後）

```
┌─────────────────────────────────────┐
│ Projects to watch:                  │
│ frontend-build, backend-build       │
│                                     │
│ Trigger only if build is:           │
│ ● Stable                            │
│ ○ Unstable                          │
│ ○ Failed                            │
└─────────────────────────────────────┘

使用例:
frontend-build ✅
backend-build  ✅
      ↓
integration-test 🚀 自動起動
```

---

##### オプション2: Build periodically（定期ビルド）

**Cron記法で設定:**
```
Schedule:
H 2 * * *     ← 毎日深夜2時ごろ（負荷分散）
H/15 * * * *  ← 15分ごと
0 8 * * 1-5   ← 平日の朝8時
```

**Cron記法の解説:**
```
* * * * *
│ │ │ │ └── 曜日 (0-7, 0と7は日曜)
│ │ │ └──── 月 (1-12)
│ │ └────── 日 (1-31)
│ └──────── 時 (0-23)
└────────── 分 (0-59)

H = ハッシュ（負荷を分散）
例: H 2 * * * → 2:00-2:59の間のランダムな時刻
```

---

##### オプション3: Build when a change is pushed to Bitbucket

**Webhook連携（推奨！）**

```
従来の方法（Poll SCM）:
Jenkins → 5分ごとにチェック → Bitbucket
         ↑
      オーバーヘッド大

推奨の方法（Webhook）:
Bitbucket → 変更を即座に通知 → Jenkins
           ↑
        効率的！
```

**Bitbucketの設定:**
```
1. リポジトリ → Settings → Webhooks
2. Add webhook
3. URL: http://your-jenkins/bitbucket-hook/
4. Triggers: ✅ Repository push
5. Save
```

**Jenkinsの設定:**
```
Build Triggers:
✅ Build when a change is pushed to Bitbucket
```

---

##### オプション4: Poll SCM（定期チェック）

```
Schedule:
H/5 * * * *   ← 5分ごとにGitをチェック

Git リポジトリに変更があった場合のみビルド

⚠️ 注意: Webhookが使えない場合の代替手段
```

---

#### 4️⃣ Build Environment（ビルド環境）

**便利なオプション:**

##### ✅ Delete workspace before build starts
```
ビルド前にワークスペースをクリーンアップ
↓
前回のビルドの影響を受けない
↓
クリーンなビルド環境

必要なプラグイン: Workspace Cleanup Plugin
```

---

##### ✅ Use secret text(s) or file(s)
```
パスワードやAPIキーを安全に使用

例:
┌─────────────────────────────────┐
│ Bindings:                       │
│ ├─ Username and password        │
│ │  Username Variable: GIT_USER  │
│ │  Password Variable: GIT_PASS  │
│ │  Credentials: github-creds    │
│ └─ Secret text                  │
│    Variable: API_KEY            │
│    Credentials: openai-api-key  │
└─────────────────────────────────┘

ビルドステップで使用:
echo $GIT_USER     → リダクトされる
curl -H "Authorization: Bearer $API_KEY"
```

必要なプラグイン: **Credentials Binding Plugin**

---

##### ✅ Abort the build if it's stuck
```
無限ループやハングアップを防ぐ

┌─────────────────────────────────┐
│ Timeout strategy:               │
│ ● Absolute                      │
│   Timeout: 30 minutes           │
│                                 │
│ Timeout actions:                │
│ ✅ Abort the build              │
└─────────────────────────────────┘

30分経過しても終わらない場合、
自動的にビルドを中止

必要なプラグイン: Build Timeout Plugin
```

---

##### ✅ Add timestamps to the Console Output
```
ログに時刻を表示

Before:
Started by user admin
Building in workspace...

After:
[2025-10-23 10:30:15] Started by user admin
[2025-10-23 10:30:16] Building in workspace...
[2025-10-23 10:32:45] Build completed

💡 デバッグが格段に楽になる！

必要なプラグイン: Timestamper Plugin
```

---

#### 5️⃣ Build（ビルドステップ）

**実際の作業を定義する最重要セクション！**

```
┌─────────────────────────────────┐
│ Add build step ▼                │
├─────────────────────────────────┤
│ ├─ Execute shell                │
│ ├─ Execute Windows batch        │
│ ├─ Invoke Ant                   │
│ ├─ Invoke Gradle script         │
│ ├─ Invoke top-level Maven       │
│ └─ Docker Build and Publish     │
└─────────────────────────────────┘
```

---

##### 例1: Execute shell

**Node.jsプロジェクトのビルド:**
```bash
#!/bin/bash
set -e  # エラーで停止

echo "=== Installing dependencies ==="
npm ci

echo "=== Running linter ==="
npm run lint

echo "=== Running tests ==="
npm test -- --coverage

echo "=== Building project ==="
npm run build

echo "=== Build completed successfully ==="
```

---

##### 例2: Invoke Gradle script

**Androidアプリのビルド:**
```
Tasks: clean assembleRelease
Build File: (空欄でOK - build.gradleを自動検出)
```

---

##### 例3: Docker Build and Publish

**コンテナイメージのビルド:**
```
Repository Name: mycompany/myapp
Tag: ${BUILD_NUMBER}
Docker Host URI: (空欄でローカル)
Registry credentials: dockerhub-credentials
```

---

#### 6️⃣ Post-build Actions（ビルド後のアクション）

**ビルド完了後の処理を定義**

```
┌─────────────────────────────────┐
│ Add post-build action ▼         │
├─────────────────────────────────┤
│ ├─ Archive the artifacts        │
│ ├─ Publish JUnit test result    │
│ ├─ E-mail Notification          │
│ ├─ Slack Notifications          │
│ ├─ Git Publisher                │
│ └─ Build other projects         │
└─────────────────────────────────┘
```

---

##### 📦 Archive the artifacts

**ビルド成果物を保存:**
```
Files to archive: dist/**/*.zip, build/*.jar
Excludes: **/*.tmp

結果:
ビルドページから成果物をダウンロード可能
```

---

##### 🧪 Publish JUnit test result report

**テスト結果を可視化:**
```
Test report XMLs: test-results/**/*.xml

結果:
┌──────────────────────────────┐
│ Test Result                  │
│ ├─ 150 tests                │
│ ├─ 148 passed ✅            │
│ ├─ 2 failed ❌              │
│ └─ 0 skipped                │
└──────────────────────────────┘
```

---

##### 📧 E-mail Notification

**ビルド結果をメール通知:**
```
Recipients: dev-team@example.com
Send e-mail for every unstable build: ✅
```

---

##### 💬 Slack Notifications

**Slackチャンネルに通知:**
```
Slack Channel: #jenkins
Notify:
✅ Build Success
✅ Build Failure
✅ Build Unstable
```

---

##### 🏷️ Git Publisher

**GitにBuildステータスをプッシュ:**
```
✅ Push Only If Build Succeeds
✅ Merge Results
Tags to push: build-${BUILD_NUMBER}
Target remote name: origin

結果:
Git commit にビルドステータスが表示される
```

---

##### 🔗 Build other projects

**次のジョブを起動:**
```
Projects to build: deploy-to-staging

Trigger only if build is stable: ✅

ワークフロー:
build-job ✅
    ↓
deploy-to-staging 🚀 自動起動
```

---

### 📊 Freestyle Projectの実例

**Node.jsアプリのビルド設定:**

```
【General】
✅ Discard old builds (30日, 50ビルド)

【Source Code Management】
● Git
Repository: https://github.com/mycompany/myapp.git
Credentials: github-credentials
Branch: */main

【Build Triggers】
✅ Build when a change is pushed to Bitbucket

【Build Environment】
✅ Delete workspace before build starts
✅ Add timestamps to the Console Output
✅ Abort the build if it's stuck (30分)

【Build】
Execute shell:
  npm ci
  npm run lint
  npm test
  npm run build

【Post-build Actions】
├─ Archive artifacts: dist/**/*
├─ Publish JUnit: test-results/**/*.xml
└─ Slack Notification: #jenkins
```

---

## 🔧 2. Pipeline - 現代的な方法（推奨）

### 📌 特徴

**コードでビルドプロセスを定義（Infrastructure as Code）**

- ✅ Jenkinsfileでバージョン管理
- ✅ 複雑なワークフローを表現可能
- ✅ 再利用・共有が容易
- ✅ コードレビュー可能
- ✅ 並列実行が簡単
- ❌ 学習コストがやや高い

---

### 🚀 作成方法

```
1. New Item をクリック
2. ジョブ名を入力: my-pipeline-job
3. Pipeline を選択
4. OK をクリック
```

---

### 📝 Jenkinsfileの基本構造

```groovy
pipeline {
    agent any  // どこで実行するか
    
    stages {
        stage('Checkout') {
            steps {
                // ステップの内容
            }
        }
    }
    
    post {
        always {
            // 必ず実行
        }
    }
}
```

---

### 🎯 実践例：完全なCI/CDパイプライン

```groovy
pipeline {
    agent any
    
    // 環境変数
    environment {
        NODE_ENV = 'production'
        DOCKER_IMAGE = 'mycompany/myapp'
        REGISTRY = 'https://registry.hub.docker.com'
    }
    
    // ビルド保持ポリシー
    options {
        buildDiscarder(logRotator(
            numToKeepStr: '50',
            daysToKeepStr: '30',
            artifactNumToKeepStr: '10'
        ))
        timestamps()  // タイムスタンプを追加
        timeout(time: 30, unit: 'MINUTES')  // タイムアウト
    }
    
    stages {
        // ステージ1: コードの取得
        stage('Checkout') {
            steps {
                checkout scm
                sh 'git log -1'
            }
        }
        
        // ステージ2: 依存関係のインストール
        stage('Install Dependencies') {
            steps {
                sh 'npm ci'
            }
        }
        
        // ステージ3: 並列テスト
        stage('Tests') {
            parallel {
                stage('Lint') {
                    steps {
                        sh 'npm run lint'
                    }
                }
                stage('Unit Tests') {
                    steps {
                        sh 'npm run test:unit'
                    }
                    post {
                        always {
                            junit 'test-results/unit/*.xml'
                        }
                    }
                }
                stage('Integration Tests') {
                    steps {
                        sh 'npm run test:integration'
                    }
                    post {
                        always {
                            junit 'test-results/integration/*.xml'
                        }
                    }
                }
            }
        }
        
        // ステージ4: ビルド
        stage('Build') {
            steps {
                sh 'npm run build'
            }
        }
        
        // ステージ5: Dockerイメージ作成
        stage('Docker Build') {
            steps {
                script {
                    dockerImage = docker.build("${DOCKER_IMAGE}:${BUILD_NUMBER}")
                }
            }
        }
        
        // ステージ6: レジストリにプッシュ
        stage('Docker Push') {
            when {
                branch 'main'  // mainブランチのみ
            }
            steps {
                script {
                    docker.withRegistry(REGISTRY, 'dockerhub-credentials') {
                        dockerImage.push("${BUILD_NUMBER}")
                        dockerImage.push("latest")
                    }
                }
            }
        }
        
        // ステージ7: デプロイ
        stage('Deploy') {
            when {
                branch 'main'
            }
            steps {
                withCredentials([
                    sshUserPrivateKey(
                        credentialsId: 'ssh-credentials',
                        keyFileVariable: 'SSH_KEY',
                        usernameVariable: 'SSH_USER'
                    )
                ]) {
                    sh '''
                        ssh -i $SSH_KEY $SSH_USER@production-server.com \
                        "docker pull ${DOCKER_IMAGE}:${BUILD_NUMBER} && \
                         docker-compose up -d"
                    '''
                }
            }
        }
    }
    
    // ビルド後の処理
    post {
        success {
            slackSend(
                color: 'good',
                message: "✅ Build SUCCESS: ${env.JOB_NAME} #${env.BUILD_NUMBER}"
            )
        }
        failure {
            slackSend(
                color: 'danger',
                message: "❌ Build FAILED: ${env.JOB_NAME} #${env.BUILD_NUMBER}\n${env.BUILD_URL}"
            )
        }
        always {
            // 成果物を保存
            archiveArtifacts artifacts: 'dist/**/*', fingerprint: true
            // ワークスペースをクリーンアップ
            cleanWs()
        }
    }
}
```

---

### 📊 Blue Oceanでの表示

```
┌──────┐  ┌────────┐  ┌────────────────────┐  ┌──────┐
│Check │─→│Install │─→│      Tests         │─→│Build │
│out   │  │Deps    │  │  ┌────┐ ┌────┐    │  │      │
│10秒  │  │30秒    │  │  │Lint│ │Unit│    │  │1分   │
└──────┘  └────────┘  │  │15秒│ │20秒│    │  └──────┘
                       │  └────┘ └────┘    │
                       │  ┌────────┐       │
                       │  │Integ   │       │
                       │  │25秒    │       │
                       │  └────────┘       │
                       └────────────────────┘
          ↓
┌────────────┐  ┌─────────┐  ┌────────┐
│Docker Build│─→│Docker   │─→│Deploy  │
│2分         │  │Push     │  │30秒    │
└────────────┘  │1分      │  └────────┘
                └─────────┘
```

---

## 🌿 3. Multi-branch Pipeline - ブランチごとのビルド（最推奨）

### 📌 特徴

**各ブランチを自動検出して個別にビルド**

- ✅ ブランチごとに自動でジョブ作成
- ✅ プルリクエストも自動ビルド
- ✅ Jenkinsfileをリポジトリで管理
- ✅ ブランチ戦略に最適
- ✅ 大規模プロジェクトに最適
- ❌ 初期設定がやや複雑

---

### 🚀 作成方法

```
1. New Item をクリック
2. ジョブ名を入力: my-multibranch-pipeline
3. Multibranch Pipeline を選択
4. OK をクリック
```

---

### ⚙️ 設定方法

#### 1. Branch Sourcesの設定

```
Branch Sources
├─ Add source → Git または GitHub
└─ 設定:
    Repository URL: https://github.com/user/repo.git
    Credentials: github-credentials
    
    Behaviours:
    ├─ Discover branches
    ├─ Discover pull requests from origin
    └─ Clean before checkout
```

---

#### 2. Build Configurationの設定

```
Build Configuration
├─ Mode: by Jenkinsfile
└─ Script Path: Jenkinsfile  ← リポジトリ内のパス
```

---

#### 3. Scan Multibranch Pipeline Triggersの設定

```
Scan Multibranch Pipeline Triggers
└─ Periodically if not otherwise run
    Interval: 1時間

または

└─ Scan by webhook  ← 推奨！
```

---

### 📝 リポジトリ構成

```
your-repo/
├─ Jenkinsfile          ← パイプライン定義
├─ src/
│  └─ ...
├─ tests/
│  └─ ...
└─ README.md
```

---

### 🎯 Multibranch用Jenkinsfile

```groovy
pipeline {
    agent any
    
    environment {
        // ブランチ名を使った動的な設定
        DEPLOY_ENV = "${env.BRANCH_NAME == 'main' ? 'production' : 'staging'}"
    }
    
    stages {
        stage('Info') {
            steps {
                echo "Building branch: ${env.BRANCH_NAME}"
                echo "Deploy environment: ${DEPLOY_ENV}"
            }
        }
        
        stage('Build') {
            steps {
                sh 'npm ci'
                sh 'npm run build'
            }
        }
        
        stage('Test') {
            steps {
                sh 'npm test'
            }
        }
        
        stage('Deploy') {
            when {
                // mainとdevelopのみデプロイ
                anyOf {
                    branch 'main'
                    branch 'develop'
                }
            }
            steps {
                script {
                    if (env.BRANCH_NAME == 'main') {
                        sh './deploy-production.sh'
                    } else {
                        sh './deploy-staging.sh'
                    }
                }
            }
        }
    }
}
```

---

### 🌳 ブランチ戦略との統合

**Git Flow の例:**

```
main (本番)
├─ Jenkinsfile → production環境にデプロイ
│
develop (開発)
├─ Jenkinsfile → staging環境にデプロイ
│
feature/new-feature
├─ Jenkinsfile → ビルド＆テストのみ
│
hotfix/bug-fix
└─ Jenkinsfile → hotfix環境にデプロイ
```

**自動的に以下のジョブが作成される:**
```
my-multibranch-pipeline/
├─ main
├─ develop
├─ feature-new-feature
└─ hotfix-bug-fix
```

---

### 🔔 プルリクエストの自動ビルド

**GitHubの場合:**

```groovy
// Jenkinsfileに追加
when {
    changeRequest()  // PRの場合のみ実行
}
```

**表示:**
```
GitHub PR #42
├─ Jenkins: Build #15 ✅ SUCCESS
├─ コードカバレッジ: 85%
└─ マージ可能
```

---

## 📊 比較表：どれを選ぶべき？

| 項目 | Freestyle | Pipeline | Multi-branch |
|------|-----------|----------|--------------|
| **設定方法** | GUI | コード | コード |
| **バージョン管理** | ❌ | ✅ | ✅ |
| **ブランチ対応** | ❌ | 手動 | ✅ 自動 |
| **並列実行** | ❌ | ✅ | ✅ |
| **複雑なワークフロー** | ❌ | ✅ | ✅ |
| **学習コスト** | 低 | 中 | 中〜高 |
| **保守性** | 低 | 高 | 高 |
| **チーム開発** | ❌ | ✅ | ✅ |
| **おすすめ度** | ⭐⭐⭐ | ⭐⭐⭐⭐⭐ | ⭐⭐⭐⭐⭐ |

---

## 🎓 選び方のフローチャート

```
プロジェクトの規模は？
├─ 小規模（個人プロジェクト）
│  └─ Freestyle Project
│
├─ 中規模（チーム開発）
│  ├─ ブランチ戦略を使っている？
│  │  ├─ はい → Multi-branch Pipeline
│  │  └─ いいえ → Pipeline
│  └─
│
└─ 大規模（複数チーム）
   └─ Multi-branch Pipeline 一択！
```

---

## 💡 実践的な使い分け

### ケース1: 個人の学習プロジェクト
```
👉 Freestyle Project

理由:
- GUIで直感的
- 素早く試せる
- 学習コストが低い
```

---

### ケース2: スタートアップのWebアプリ
```
👉 Pipeline

理由:
- コードで管理できる
- チームで共有しやすい
- CI/CDの基礎を学べる
```

---

### ケース3: 大規模なマイクロサービス
```
👉 Multi-branch Pipeline

理由:
- 各サービスに個別のJenkinsfile
- feature/hotfix/releaseブランチを自動管理
- PRごとに自動テスト
```

---

### ケース4: レガシーシステムの移行
```
👉 Freestyle → Pipeline → Multi-branch

段階的な移行:
1. まずFreestyleで動作確認
2. Pipelineに変換
3. ブランチ戦略導入後、Multi-branch化
```

---

## 🛠️ 他のJob Types

### Maven Project

**Mavenプロジェクト専用**