# 【AWS】検証！Powertools AWS Lambda(Python)

このREADMEは`AWSに触るなんてムリムリ！※無理じゃなかった？！2ndシーズン Advent Calendar 2025`の一環として作成されました。

## この記事のポイント

- Powertools AWS Lambda(Python)の基本的な使い方を理解する

## はじめに

この記事では「この前リリースされた機能って実際に動かすとどんな感じなんだろう」とか「もしかしたら内容次第では使えるかも？？」などAWSサービスの中でも特定の機能にフォーカスして検証していく記事です。主な内容としては実践したときのメモを中心に書きます。（忘れやすいことなど）
誤りなどがあれば書き直していく予定です。

今回はPowertools for AWS Lambda (Python)について検証します。

内容は[Tutorial - Powertools for AWS Lambda (Python)](https://docs.aws.amazon.com/powertools/python/latest/tutorial/)に沿って進めていきます。

## Powertools AWS Lambda(Python)とは

## 必要なもの

今回の検証で必要なものは以下のとおりです。

- AWS CLI
- AWS SAM CLI

## セットアップ方法

では、まずは環境をセットアップしていきましょう。

### AWS CLI のインストール

まずはAWS CLIをインストールします。最新版のAWS CLIを公式インストーラーでインストールします。

```bash
# 1. インストーラーをダウンロード
curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o "awscliv2.zip"

# 2. unzipがインストールされていない場合はインストール
sudo apt update && sudo apt install unzip -y  # Ubuntu/Debian系
# または
sudo yum install unzip -y                     # CentOS/RHEL系

# 3. ダウンロードしたファイルを展開
unzip awscliv2.zip

# 4. インストール実行
sudo ./aws/install

# 5. インストール確認
aws --version

# ダウンロードしたzipファイルと展開したディレクトリを削除してクリーンアップします。
rm  "awscliv2.zip"

# 解凍したディレクトリを削除
rm -rf aws
```

### AWS CLI の設定

AWS CLIの設定を行います。今回はAWS IAM Identity Center(旧AWS SSO)を使用してログインします。まずは以下のコマンドを実行して、SSOの設定を行います。

```bash
aws configure sso
```
設定時に以下の情報の入力が求められます：
- **SSO start URL**: 組織のSSO開始URL（例：`https://my-company.awsapps.com/start`）
- **SSO Region**: SSOが設定されているリージョン（例：`us-east-1`）
- **アカウント選択**: 利用可能なAWSアカウントから選択
- **ロール選択**: 選択したアカウントで利用可能なロールから選択
- **CLI default client Region**: デフォルトのAWSリージョン（例：`ap-northeast-1`）
- **CLI default output format**: 出力形式（`json`、`text`、`table`のいずれか）
- **CLI profile name**: プロファイル名（`default`とします。）

SSOの設定が完了したら、以下のコマンドでログインを実行します。

```bash
aws sso login
```

### AWS CLI の動作確認

AWS CLIが正しくインストールされ、SSOでログインできているか確認します。AWS STSで認証情報を確認します。

```bash
aws sts get-caller-identity
```

### AWS SAM CLI のインストール

[Install the AWS SAM CLI - AWS Serverless Application Model](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/install-sam-cli.html)に従って、セットアップを行います。

AWS SAM CLIをインストールします。以下のコマンドを実行してください。

```bash
wget https://github.com/aws/aws-sam-cli/releases/latest/download/aws-sam-cli-linux-x86_64.zip
unzip aws-sam-cli-linux-x86_64.zip -d sam-installation
sudo ./sam-installation/install
rm -rf aws-sam-cli-linux-x86_64.zip sam-installation
```

動作確認を行います。

```bash
sam --version
```

実行結果

```bash
SAM CLI, version 1.146.0
```

## 実際に使ってみよう

## まとめ

## 参考

- [powertools-lambda-python](https://github.com/aws-powertools/powertools-lambda-python?tab=readme-ov-file)
- [Tutorial - Powertools for AWS Lambda (Python)](https://docs.aws.amazon.com/powertools/python/latest/tutorial/)
- [AWS Lambda Powertools Python 入門 第 1 回 - buileers.flash✨ | AWS](https://aws.amazon.com/jp/builders-flash/202203/lambda-powertools-python-1/)

## ここから下はAWS CLIのインストールとSSOログイン手順ですので、必要に応じてご参照ください。

## AWS CLI インストールと SSO ログイン手順 (Linux環境)

このガイドでは、Linux環境でAWS CLIをインストールし、AWS SSOを使用してログインするまでの手順を説明します。

## 前提条件

- Linux環境（Ubuntu、CentOS、Amazon Linux等）
- インターネット接続
- 管理者権限（sudoが使用可能）
- AWS SSO が組織で設定済み
- Python 3.12.1

## AWS CLI のインストール

### 公式インストーラーを使用（推奨）

最新版のAWS CLI v2を公式インストーラーでインストールします。

```bash
# 1. インストーラーをダウンロード
curl "https://awscli.amazonaws.com/awscli-exe-linux-$(uname -m).zip" -o "awscliv2.zip"

# 2. unzipがインストールされていない場合はインストール
sudo apt update && sudo apt install unzip -y  # Ubuntu/Debian系
# または
sudo yum install unzip -y                     # CentOS/RHEL系

# 3. ダウンロードしたファイルを展開
unzip awscliv2.zip

# 4. インストール実行
sudo ./aws/install

# 5. インストール確認
aws --version

# ダウンロードしたzipファイルと展開したディレクトリを削除してクリーンアップします。
rm  "awscliv2.zip"

# 解凍したディレクトリを削除
rm -rf aws
```

## AWS SSO の設定とログイン

### 1. AWS SSO の設定

AWS SSOを使用するための初期設定を行います。

```bash
aws configure sso
```

設定時に以下の情報の入力が求められます：

- **SSO start URL**: 組織のSSO開始URL（例：`https://my-company.awsapps.com/start`）
- **SSO Region**: SSOが設定されているリージョン（例：`us-east-1`）
- **アカウント選択**: 利用可能なAWSアカウントから選択
- **ロール選択**: 選択したアカウントで利用可能なロールから選択
- **CLI default client Region**: デフォルトのAWSリージョン（例：`ap-northeast-1`）
- **CLI default output format**: 出力形式（`json`、`text`、`table`のいずれか）
- **CLI profile name**: プロファイル名（`default`とします。）

### 2. AWS SSO ログイン

設定完了後、以下のコマンドでログインを実行します。

```bash
aws sso login
```

ログイン時の流れ：
1. コマンド実行後、ブラウザが自動的に開きます
2. AWS SSOのログインページが表示されます
3. 組織のIDプロバイダー（例：Active Directory、Okta等）でログイン
4. 認証が成功すると、ターミナルに成功メッセージが表示されます

### 3. ログイン状態の確認

認証情報を確認します。

```bash
aws sts get-caller-identity
```

正常にログインできている場合、以下のような情報が表示されます：

```json
{
    "UserId": "AROAXXXXXXXXXXXXXX:username@company.com",
    "Account": "123456789012",
    "Arn": "arn:aws:sts::123456789012:assumed-role/RoleName/username@company.com"
}
```

## トラブルシューティング

### よくある問題と解決方法

#### 1. ブラウザが開かない場合

```bash
# 手動でブラウザを開く場合のURL確認
aws sso login --no-browser
```

表示されたURLを手動でブラウザで開いてください。

#### 2. セッションが期限切れの場合

```bash
# 再ログイン
aws sso login
```

#### 4. プロキシ環境での設定

プロキシ環境の場合、以下の環境変数を設定してください：

```bash
export HTTP_PROXY=http://proxy.company.com:8080
export HTTPS_PROXY=http://proxy.company.com:8080
export NO_PROXY=localhost,127.0.0.1,.company.com
```

## セキュリティのベストプラクティス

1. **定期的な認証情報の更新**: SSOセッションには有効期限があります。定期的に再ログインを行ってください。

2. **最小権限の原則**: 必要最小限の権限を持つロールを使用してください。

3. **プロファイルの分離**: 本番環境と開発環境で異なるプロファイルを使用してください。

4. **ログアウト**: 作業終了時は適切にログアウトしてください：
   ```bash
   aws sso logout --profile <プロファイル名>
   ```

## 参考リンク

- [AWS CLI ユーザーガイド](https://docs.aws.amazon.com/cli/latest/userguide/)
- [AWS SSO ユーザーガイド](https://docs.aws.amazon.com/singlesignon/latest/userguide/)
- [AWS CLI インストールガイド](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html)
