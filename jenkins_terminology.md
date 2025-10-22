# Jenkins Pipeline 用語集
## 新人エンジニア向け - わかりやすい解説

---

## 📚 目次

1. 基本概念
2. アーキテクチャ用語
3. Pipeline構成要素
4. Jenkinsfile用語
5. 実行環境用語
6. よく出てくるカタカナ語

---

## 1. 基本概念

### Pipeline（パイプライン）
**意味**: ソースコードから本番環境までの自動化された流れ

**例え話**:
```
工場の製造ライン
原材料 → 加工 → 検査 → 梱包 → 出荷

ソフトウェア開発
コード → ビルド → テスト → デプロイ → 本番
```

**Jenkins Pipelineの例**:
```groovy
pipeline {
    agent any
    stages {
        stage('Build') { ... }    // ビルド工程
        stage('Test') { ... }     // 検査工程
        stage('Deploy') { ... }   // 出荷工程
    }
}
```

---

### CI/CD Pipeline
**CI (Continuous Integration)**: 継続的インテグレーション
- コードを頻繁に統合
- 自動ビルド・テスト

**CD (Continuous Delivery/Deployment)**: 継続的デリバリー/デプロイ
- 本番環境へのリリースを自動化

**全体像**:
```
開発者がコミット
    ↓
CI: 自動ビルド・テスト
    ↓
CD: 自動デプロイ
    ↓
本番環境で稼働
```

---

## 2. アーキテクチャ用語

### Master（マスター）
**意味**: Jenkinsの司令塔となるメインサーバー

**役割**:
- Jenkins UIを提供
- ジョブの設定を保存
- ビルドのスケジューリング
- Agent（Worker）に指示を出す

**イメージ**:
```
オーケストラの指揮者
各演奏者（Agent）に指示を出す
```

---

### Agent / Worker（エージェント / ワーカー）
**意味**: 実際にジョブを実行する作業者（サーバー）

**なぜ必要？**
- Masterの負荷を分散
- 異なる環境で実行可能
- スケーラビリティ向上

**構成例**:
```
┌─────────────┐
│   Master    │ ← ユーザーがアクセス
│  (司令塔)   │
└──────┬──────┘
       │ 指示
   ────┴────
   │        │
┌──▼──┐  ┌──▼──┐
│Agent│  │Agent│ ← 実際にビルド実行
│  1  │  │  2  │
└─────┘  └─────┘
```

**Jenkinsfileでの指定**:
```groovy
pipeline {
    agent any  // どのAgentでもOK
    // または
    agent {
        label 'linux'  // "linux"ラベルのAgentで実行
    }
}
```

---

### Node（ノード）
**意味**: Jenkins環境で使えるマシン全体（Master + Agent）

**関係性**:
```
Node = Master + Agent(s)

Node（全体）
├── Master Node（1台）
└── Agent Nodes（複数台）
```

---

### Offload（オフロード）
**意味**: 負荷を他のマシンに移す

**わかりやすい例**:
```
❌ 悪い例（オフロードなし）
社長が全ての仕事を自分でやる
→ パンクする

✅ 良い例（オフロード）
社長：指示を出す（Master）
社員：実作業をする（Agent）
→ 効率的
```

**Jenkins での実例**:
```
Master: ビルドの管理だけ
   ↓ （オフロード）
Agent 1: Pythonプロジェクトのビルド
Agent 2: Javaプロジェクトのビルド
Agent 3: テスト実行
```

---

## 3. Pipeline構成要素

### Stage（ステージ）
**意味**: Pipelineの大きな区切り（工程）

**イメージ**:
```
料理の手順
Stage 1: 材料準備
Stage 2: 調理
Stage 3: 盛り付け
Stage 4: 提供
```

**Jenkins での例**:
```groovy
stages {
    stage('ソースコード取得') { ... }
    stage('ビルド') { ... }
    stage('テスト') { ... }
    stage('デプロイ') { ... }
}
```

**画面での見え方**:
```
[ソースコード取得] → [ビルド] → [テスト] → [デプロイ]
     ✓              ✓          ✓         実行中...
```

---

### Step（ステップ）
**意味**: Stage内の具体的なコマンド・処理

**関係性**:
```
Pipeline
├── Stage 1
│   ├── Step 1
│   ├── Step 2
│   └── Step 3
└── Stage 2
    ├── Step 1
    └── Step 2
```

**実例**:
```groovy
stage('Build') {
    steps {
        // Step 1
        sh 'npm install'
        
        // Step 2
        sh 'npm run build'
        
        // Step 3
        echo 'Build completed!'
    }
}
```

---

### Phase（フェーズ）
**意味**: Stageとほぼ同義（開発の段階）

**典型的なフェーズ**:
```
1. Build Phase（ビルドフェーズ）
2. Test Phase（テストフェーズ）
3. Deploy Phase（デプロイフェーズ）
```

---

## 4. Jenkinsfile用語

### Jenkinsfile（ジェンキンスファイル）
**意味**: Pipelineの設定をコードで記述したファイル

**メリット**:
- バージョン管理可能
- チームで共有可能
- 再利用可能

**配置場所**:
```
my-project/
├── src/
├── tests/
├── README.md
└── Jenkinsfile  ← リポジトリのルートに配置
```

---

### Declarative Pipeline（宣言型パイプライン）
**意味**: 構造化された書き方（初心者向け）

**特徴**:
- 読みやすい
- 書きやすい
- エラーが少ない

**例**:
```groovy
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'npm install'
            }
        }
    }
}
```

---

### Scripted Pipeline（スクリプト型パイプライン）
**意味**: 柔軟な書き方（上級者向け）

**特徴**:
- Groovyの知識が必要
- 自由度が高い
- 複雑な処理も可能

**例**:
```groovy
node {
    stage('Build') {
        if (env.BRANCH_NAME == 'master') {
            sh 'npm run build:prod'
        } else {
            sh 'npm run build:dev'
        }
    }
}
```

**どちらを使う？**
```
初心者・標準的な処理
  → Declarative Pipeline（推奨）

複雑な条件分岐・ループ
  → Scripted Pipeline
```

---

## 5. 実行環境用語

### Workspace（ワークスペース）
**意味**: ビルドが実行される作業ディレクトリ

**場所**:
```
Linux:
/var/lib/jenkins/workspace/job-name/

Windows:
C:\Program Files\Jenkins\workspace\job-name\
```

**中身**:
```
workspace/my-job/
├── .git/
├── src/
├── tests/
├── package.json
└── Jenkinsfile
```

---

### Environment Variables（環境変数）
**意味**: ビルド中に使える変数

**Jenkinsの組み込み変数**:
```groovy
echo "${BUILD_NUMBER}"      // ビルド番号: 123
echo "${JOB_NAME}"          // ジョブ名: my-project
echo "${WORKSPACE}"         // 作業ディレクトリ
echo "${GIT_COMMIT}"        // Gitコミットハッシュ
```

**カスタム変数**:
```groovy
environment {
    APP_NAME = 'my-app'
    VERSION = '1.0.0'
}
```

---

### Credentials（クレデンシャル）
**意味**: パスワードや秘密鍵などの認証情報

**管理場所**:
```
Manage Jenkins
  → Manage Credentials
    → 安全に暗号化して保存
```

**使用例**:
```groovy
environment {
    // Jenkins Credentialsから取得
    AWS_ACCESS_KEY_ID = credentials('aws-key-id')
}
```

---

### Stash / Unstash（スタッシュ / アンスタッシュ）
**意味**: ファイルを一時保存して、後で取り出す

**イメージ**:
```
Stage 1で作成したファイル
    ↓ stash（保存）
[一時保管庫]
    ↓ unstash（取り出し）
Stage 2で使用
```

**使用例**:
```groovy
stage('Build') {
    steps {
        sh 'npm run build'
        // dist/ フォルダを保存
        stash name: 'build-files', includes: 'dist/**'
    }
}

stage('Deploy') {
    steps {
        // 保存したファイルを取り出す
        unstash 'build-files'
        sh 'ls -l dist/'
    }
}
```

---

## 6. よく出てくるカタカナ語

### Trigger（トリガー）
**意味**: ビルドを開始するきっかけ

**種類**:
```
1. Manual（手動）
   → 「ビルド実行」ボタンをクリック

2. SCM Polling（定期チェック）
   → 5分ごとにGitをチェック

3. Webhook（即座に通知）
   → GitHubにpushしたら即座にビルド開始

4. Scheduled（スケジュール）
   → 毎日夜中2時にビルド
```

---

### Artifact（アーティファクト / 成果物）
**意味**: ビルドで生成されたファイル

**例**:
```
- コンパイル済みのバイナリ
- JARファイル
- Dockerイメージ
- テストレポート
- ビルドログ
```

**保存例**:
```groovy
post {
    always {
        // 成果物を保存
        archiveArtifacts artifacts: 'dist/**/*'
    }
}
```

---

### Checkout（チェックアウト）
**意味**: Gitリポジトリからソースコードを取得

**自動 vs 手動**:
```
自動（デフォルト）:
pipeline開始時に自動でcheckout

手動:
skipDefaultCheckout(true) を使う場合
```

**例**:
```groovy
stage('Checkout') {
    steps {
        // Gitからコードを取得
        checkout scm
    }
}
```

---

### Post Actions（ポストアクション）
**意味**: ビルド後に実行する処理

**タイミング**:
```groovy
post {
    always {
        // 成功・失敗に関わらず常に実行
        echo 'Cleanup...'
    }
    success {
        // 成功時のみ
        echo 'Build succeeded!'
    }
    failure {
        // 失敗時のみ
        echo 'Build failed!'
        mail to: 'team@example.com'
    }
}
```

---

### Parallel Execution（並列実行）
**意味**: 複数の処理を同時に実行

**メリット**:
- ビルド時間の短縮
- 効率的なリソース利用

**イメージ**:
```
直列実行（遅い）:
[Test A] → [Test B] → [Test C]
  10分      10分      10分
  合計: 30分

並列実行（速い）:
[Test A]
[Test B]  ← 同時実行
[Test C]
  合計: 10分
```

**例**:
```groovy
stage('Tests') {
    parallel {
        stage('Unit Tests') {
            steps { sh 'npm run test:unit' }
        }
        stage('Integration Tests') {
            steps { sh 'npm run test:integration' }
        }
        stage('E2E Tests') {
            steps { sh 'npm run test:e2e' }
        }
    }
}
```

---

### Multibranch Pipeline（マルチブランチパイプライン）
**意味**: ブランチごとに自動でPipelineを作成

**仕組み**:
```
Git Repository
├── main ブランチ
│   └── Jenkinsfile → Jenkins Pipeline 1
├── develop ブランチ
│   └── Jenkinsfile → Jenkins Pipeline 2
└── feature/new-ui ブランチ
    └── Jenkinsfile → Jenkins Pipeline 3
```

**メリット**:
- ブランチごとに自動テスト
- Pull Requestの品質確認
- 設定の手間が少ない

---

## 7. Docker関連用語

### Docker Image（ドッカーイメージ）
**意味**: アプリケーションの実行環境をパッケージ化したもの

**例**:
```
node:14-alpine     → Node.js 14環境
python:3.11-slim   → Python 3.11環境
maven:3.8-jdk-11   → Maven + Java 11環境
```

---

### Docker Container（ドッカーコンテナ）
**意味**: Docker Imageから作られた実行中の環境

**関係性**:
```
Docker Image（設計図）
    ↓ 実行
Docker Container（実際の部屋）
```

**Jenkins での使用**:
```groovy
agent {
    docker {
        image 'python:3.11-slim'
    }
}
```

---

### Docker Volume（ドッカーボリューム）
**意味**: コンテナ間でファイルを共有する仕組み

**使用例**:
```groovy
agent {
    docker {
        image 'node:14-alpine'
        // ホストとコンテナ間でフォルダ共有
        args '-v $HOME/cache:/root/.npm'
    }
}
```

---

## 8. 実践的な用語の組み合わせ

### 典型的なPipelineの流れ
```
1. Trigger（トリガー）
   GitHub Webhookでビルド開始

2. Checkout（チェックアウト）
   Gitからソースコード取得
   → Workspace に配置

3. Agent（エージェント）選択
   適切なNodeでビルド実行

4. Stages（ステージ）実行
   Build → Test → Deploy

5. 各Stage内でSteps（ステップ）実行
   具体的なコマンドを実行

6. Artifacts（成果物）保存
   ビルド結果を保存

7. Post Actions（ポストアクション）
   通知やクリーンアップ
```

---

## 9. 初心者が混乱しやすい用語

### Agent vs Node
```
Agent:
- ジョブを実行する「役割」
- Jenkinsfileで agent any と指定

Node:
- 実際の「マシン」
- MasterもNodeの一つ
```

### Stage vs Step
```
Stage:
- 大きな区切り（工程）
- UI上でも表示される

Step:
- 具体的なコマンド
- Stage の中に複数ある
```

### Build vs Job vs Pipeline
```
Build:
- 1回の実行
- "Build #123" のように番号付き

Job:
- Jenkinsの設定単位
- 「何をするか」を定義

Pipeline:
- Job の一種
- Jenkinsfileで定義されたJob
```

---

## 10. よく使う英単語の意味

### Execute（エグゼキュート）
**意味**: 実行する
```
execute a job = ジョブを実行する
```

### Provision（プロビジョン）
**意味**: 準備する・提供する
```
provision an agent = Agentを準備する
```

### Poll（ポール）
**意味**: 定期的にチェックする
```
poll SCM = Gitリポジトリを定期的にチェック
```

### Deploy（デプロイ）
**意味**: 配備する・デプロイする
```
deploy to production = 本番環境にデプロイ
```

### Checkout（チェックアウト）
**意味**: 取り出す
```
checkout source code = ソースコードを取得
```

---

## 📚 まとめ

### 最重要用語トップ10
1. **Pipeline**: 自動化の流れ全体
2. **Master/Agent**: 司令塔と作業者
3. **Stage**: 大きな工程
4. **Step**: 具体的なコマンド
5. **Jenkinsfile**: Pipeline設定ファイル
6. **Workspace**: 作業ディレクトリ
7. **Trigger**: ビルド開始のきっかけ
8. **Artifact**: ビルド成果物
9. **Environment Variables**: 環境変数
10. **Post Actions**: ビルド後の処理

### 学習のコツ
- **まずは基本用語を覚える**
- **実際にJenkinsfileを書いて試す**
- **公式ドキュメントを参照**
- **分からない用語はその都度調べる**

---

## 🎯 確認問題

### Q1: 次の用語を説明してください
1. Master
2. Agent
3. Offload

### Q2: Stage と Step の違いは？

### Q3: Stash の用途は？

### Q4: Multibranch Pipeline とは？

---

## 📖 参考リソース

**公式ドキュメント**:
- Jenkins Pipeline Syntax: https://www.jenkins.io/doc/book/pipeline/syntax/
- Jenkins Glossary: https://www.jenkins.io/doc/book/glossary/

**学習リソース**:
- Jenkins 公式チュートリアル
- Pipeline Examples
- Community Forums

---

これで Jenkins Pipeline の用語が理解できました！ 🎉
実際にJenkinsfileを書きながら、少しずつ慣れていきましょう！