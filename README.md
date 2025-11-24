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

チュートリアルを参考に進めるので以下のサービスを利用します。

- AWS Lambda
- Amazon API Gateway
- Amazon CloudWatch
- AWS X-Ray

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

## 準備体操：AWS SAMでHello Worldアプリケーションを作成して動作確認

Powertools for AWS Lambda (Python)のチュートリアルに入る前に、AWS SAMでHello Worldアプリケーションを作成して動作確認を行います。
これからのやることのおおまかな流れは以下のとおりです。

- Pythonのバージョンを確認
- AWS SAMでHello Worldアプリケーションを作成
- ローカルでビルドとAPI起動

まずはPythonのバージョンを確認します。

```bash
python3 --version
```

今回はPython 3.12.1を使用します。このバージョンを使ってAWS SAMでHello Worldアプリケーションを作成します。
以下のコマンドを実行して、プロジェクトを作成します。

```bash
sam init --runtime python3.12 --dependency-manager pip --app-template hello-world --name powertools-quickstart
```

ディレクトリを移動します。

```bash
cd powertools-quickstart
```

ローカルでビルドとAPI起動を行います。

```bash
sam build && sam local start-api
```

`http://127.0.0.1:3000/hello`でアクセスします。
Web画面上に以下のようなメッセージが表示されたら成功です。

```text
{"message": "hello world"}
```

curlコマンドでアクセスする場合は以下のとおりです。※ターミナルを次の行で表示するために改行が入るようにしています。

```bash
curl http://127.0.0.1:3000/hello && echo ""
```

sam local invokeによる実行も試してみます。以下のコマンドを実行します。
イベントファイルとリソース指定で実行する例を示します。

```bash
sam local invoke HelloWorldFunction -e events/event.json
```

または短縮版で実行することもできます。

```bash
sam local invoke -e events/event.json
```

問題なく動作したら、次にデプロイを行います。以下のコマンドを実行します。

```bash
sam build && sam deploy --guided
```

いくつか質問が表示されるので、以下のように入力して進めてください。

```
Setting default arguments for 'sam deploy'
=========================================
Stack Name [powertools-quickstart]: 
AWS Region [ap-northeast-1]: 
#Shows you resources changes to be deployed and require a 'Y' to initiate deploy
Confirm changes before deploy [Y/n]: Y
#SAM needs permission to be able to create roles to connect to the resources in your template
Allow SAM CLI IAM role creation [Y/n]: Y
#Preserves the state of previously provisioned resources when an operation fails
Disable rollback [y/N]: y
HelloWorldFunction has no authentication. Is this okay? [y/N]: y
Save arguments to configuration file [Y/n]: y
SAM configuration file [samconfig.toml]: 
SAM configuration environment [default]: 
```

デプロイがはじまるのでしばらく待ちます。途中で`changeset`の確認が表示されるので`Y`を入力して進めてください。
デプロイが完了したら、以下のようなメッセージが表示されます。

```
Successfully created/updated stack - powertools-quickstart in ap-northeast-1
```

デプロイが完了したら、samのコマンドでデプロイしたLambda関数のエンドポイントを確認します。

```bash
sam list endpoints --output json
```

実行結果の`CloudEndpoint`という項目でProdとStageのURLが確認できます。

```json
  {
    "LogicalResourceId": "ServerlessRestApi",
    "PhysicalResourceId": "XXXXXXX",
    "CloudEndpoint": [
      "https://{PhysicalResourceId}.execute-api.ap-northeast-1.amazonaws.com/Prod",
      "https://{PhysicalResourceId}.execute-api.ap-northeast-1.amazonaws.com/Stage"
    ],
    "Methods": [
      "/hello['get']"
    ]
  }
```

`CloudEndpoint`のURLに`/hello`を付与してアクセスします。ブラウザまたはcurlコマンドでアクセスしてください。

```bash
curl https://{PhysicalResourceId}.execute-api.ap-northeast-1.amazonaws.com/Prod/hello && echo ""
```

実行結果

```json
{"message": "hello world"}
```

## 独自のルーター作成をPowertoolsで簡単に実装する

Hello Worldアプリケーションの動作確認ができたところで、次は独自のルーター作成をPowertoolsで簡単に実装してみます。
変更のおおまかな流れは以下のとおりです。

- template.ymlの修正
- app.pyの修正

### template.ymlの修正

URLパスをもう一つ追加する場合はResourcesセクションにもう1つの項目を追加するべきかのように思えますが
単にパスをもう1つ追加するだけであれば、`Events`セクションにもう1つのイベントを追加するだけ問題ありません。
具体的には以下のように修正します。

https://github.com/ymd65536/aws_lambda_powertools/blob/8d2f40da59b6b88fe64412d2bc3e84f6849392a8/sample/self_router/template.yaml#L13-L32

次にapp.pyの修正ですが、まずはpowertoolsを使わない場合のコード例を示します。

https://github.com/ymd65536/aws_lambda_powertools/blob/main/sample/self_router/app.py

これは最初に思いつく実装方法ですが、実際に運用してみると難しい点がいくつかあります。
Routerクラスを保守する必要があるのももちろんですが、もうひとつ例を挙げると、lambda_handlerで取得したリクエストメソッドがRouterクラスで動作するかどうかを理解するためにRouterクラスの実装を確認する必要がある点です。

これはlambda_handlerの中でリクエストメソッド(GET、POSTなど)を取得している点ことが原因と考えて良いでしょう。理想的にはlambda_handlerではリクエストメソッドを意識せずに処理できると良いです。

Lambda Powertoolsを使うと、これらの問題を解決できます。以下に修正例を示します。

https://github.com/ymd65536/aws_lambda_powertools/blob/main/sample/powertools_api_gateway/app.py

上記のコード例ではaws_lambda_powertools.event_handlerのAPIGatewayRestResolverを使うことで、lambda_handlerでリクエストメソッドを意識せずに処理できるようになっています。

モジュールインストールのためにrequirements.txtの修正は必要ですが、template.ymlの修正は不要です。

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
