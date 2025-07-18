# Difyの基本的な設定

## Difyアカウントを登録

今回はSaaS版の[Dify.ai](https://dify.ai/jp)を利用します。
Dify.aiのサイト [https://dify.ai/jp](https://dify.ai/jp) にアクセスし、「始める」をクリックしましょう。

「GitHubで続行」「Googleで続行」「メールアドレスとコードで続行」でDifyアカウント作成できますので、お好きな方法で行います。
(私は「Googleで続行」からGoogleアカウントを使ってDifyアカウントを作成しました)

![Difyアカウント作成・ログイン](/images/books/pyml-niigata-dify/account1.png "Difyアカウント作成・ログイン")

## Difyのホーム画面

Difyアカウントが作成できましたら、ホーム画面が表示されます。

![Difyホーム画面](/images/books/pyml-niigata-dify/workspace_home.png "Difyホーム画面")

## モデルプロバイダの設定

今回はAmazon Bedrockを経由してAnthropic Claudeを利用することをベースにハンズオンを進めます。

AWSアカウントがない方は、OpenAIのモデルを利用します。
Dify.aiのSANDBOXアカウントは、OpenAIのモデルを200クレジット分無料で利用可能ですので、それを利用します。

### Amazon Bedrockを利用する

Amazon Bedrockを利用する場合は、まず対象リージョンで利用する基盤モデルの有効化を行います。また、Bedrockの基盤モデルを実行する権限を持つIAMユーザのAPIキーを取得する必要があります。
Amazon Bedrockでは、IAMユーザ以外の[APIキー払い出しにも対応](https://dev.classmethod.jp/articles/amazon-bedrock-api-keys-feature-added/)しましたが、Difyではその機能にまだ対応していないようです。(2025年7月現在)

今回はオレゴンリージョン (us-west-2) でBedrockを利用します。
東京リージョンでも同じように利用可能です。

#### Bedrock基盤モデルの有効化

1. [AWSのWebサイト](https://aws.amazon.com/jp/)にアクセスし、
    サイト右上にある「コンソールへログイン」をクリックします。
2. 管理用のIAMユーザでログインしましょう。
3. AWSマネジメントコンソールのトップページが開いたら、サービスから「Amazon Bedrock」をクリックします。
4. AWSマネジメントコンソールで右側に「米国 (オレゴン)」と表示されているか確認します。
    - 別のリージョン名が表示されていない場合は、リージョン名をクリックし「オレゴン us-west-2」を選択します。
5. 左側メニューから「モデルアクセス」をクリックします。
    - 左側メニュー非表示の場合、左側上にある「3本線」のアイコンをクリックして表
示しましょう。
6. 「モデルアクセスを変更」をクリックします。
7. 「Anthropic」の「Claude」の各モデルにチェックを入れて、「変更を保存」をクリック
    - 今回は `Claude 3.7 Sonnet` が有効化されていればOKです
    - Claudeのモデルをまだ使ったことがない方は先に「ユースケースの詳細」を送信する必要があります
        - 参考: [参考: Amazon Bedrock をマネジメントコンソールからちょっと触ってみたいときは Base Models（基盤モデル）へのアクセスを設定しましょう | DevelopersIO](https://dev.classmethod.jp/articles/if-you-want-to-try-out-amazon-bedrock-from-the-management-console-you-can-set-up-access-to-base-models/)
        - 参考: [Amazon BedrockのIAMポリシーとユースケースの詳細を申告してからチャットを試すまで #AWS - Qiita](https://qiita.com/mkin/items/2f60566d15777557d533#%E3%83%A6%E3%83%BC%E3%82%B9%E3%82%B1%E3%83%BC%E3%82%B9%E3%81%AE%E8%A9%B3%E7%B4%B0%E3%82%92%E9%80%81%E4%BF%A1)
    ![Bedrock モデル有効化](/images/books/pyml-niigata-dify/bedrock_models.png "Bedrock モデル有効化")
8. アクセスが付与される間に、以降の作業を進めます。

#### IAMユーザの作成

1. AWSマネジメントコンソールで「サービス」から「IAM」をクリックします。
    - Identity and Access Managementのダッシュボード画面が表示されます。
2. 「ユーザー」をクリックし、「ユーザーを追加」をクリックします。
    「ユーザーを追加」画面が表示されます。
3. 以下の項目を入力し「次へ」をクリックします。
    - ユーザ名: difyBedrockUser
4. 以下の項目を入力し「次へ」をクリックします。
    - 許可のオプション: 「ポリシーを直接アタッチする」を選択
    - 許可ポリシー: 「AmazonBedrockFullAccess」の左側チェックボックスにチェックを入れる
        - 検索ボックスに「bedrock」と入力すると、Amazon Bedrockに関するポリシーが一覧で表示されるので、その中から選択すると良い
5. 内容を確認し「ユーザの作成」をクリックします。
6. ユーザが作成されたら、作成したユーザ名をクリックして詳細画面に遷移します。
7. 「セキュリティ認証情報」タブをクリックし、「アクセスキーを作成」をクリックします。
8. 「ユースケース」は適当でも良いのですが、今回は「サードパーティサービス」を選択し、「上記のレコメンデーションを理解し、アクセスキーを作成します」にチェックを入れ、「次へ」をクリックします。
9. 「説明タグ」に「dify」と入力し、「アクセスキーを作成」をクリックします。
10. アクセスキーが作成されたら「アクセスキー」および「シークレットアクセスキー」をメモしておきます。可能なら「.csvファイルをダウンロード」をクリックしてCSVファイルとしてローカルPCに保存しておくと良いでしょう
    - アクセスキーおよびシークレットアクセスキーは、外部に漏らさないようにお願いします
11. メモが終わったら「完了」をクリックしましょう。

#### Difyのモデルプロバイダを設定

1. Dify.aiにログインし、画面右上にあるユーザアイコンをクリックして「設定」をクリックします。
2. 左側メニューの「モデルプロバイダー」をクリックします。
3. 「モデルプロバイダーをインストールする」の中から「Amazon Bedrock」にカーソルを合わせ「インストール」をクリックします。
    - 画面上部の「モデル」欄に追加されます。
4. Amazon BedrockモデルのAPI-KEYの「セットアップ」をクリックします。
5. 以下の項目を入力し「保存」をクリックします。
    - Access Key: IAMユーザの「アクセスキー」
    - Secret Access Key: IAMユーザの「シークレットアクセスキー」
    - AWS Region: US West (Oregon)
    - Bedrock Endpoint URL: 空欄のまま何も入力しません
    - Available Model Name: `us.anthropic.claude-3-7-sonnet-20250219-v1:0`
        - 利用するモデルの1つを指定すればOK
            - 接続検証用に1つ指定するだけなので、検証不要であれば指定しなくてもOKです
        - Claude 3.7 Sonnet等はCross-region Infarenceが有効になっているので、クロスリージョン推論プロファイルIDを指定します
            - https://docs.aws.amazon.com/ja_jp/bedrock/latest/userguide/inference-profiles-support.html
            - 上記サイトの「US Anthropic Claude 3.7 Sonnet」内に記載あります
            - 東京リージョンの場合は「APAC Anthropic Claude 3.7 Sonnet」内に記載あります
            - またはAmazon Bedrockのマネジメントコンソール上の「Cross-region inference」をクリックすると `Inference profile ID` にて記載あります
        - Cross-resion Infarence対応してないモデルの場合は、Amazon Bedrockのマネジメントコンソール上の「Model catalog」内にて該当モデルを選択して、 `Model ID` にて記載あります
    - Bedrock Proxy URL: 何も入力しなくてOK
    ![Dify Bedrock セットアップ](/images/books/pyml-niigata-dify/dify_bedrock_setup2.png "Dify Bedrock セットアップ")
6. 設定が合っていればエラーなく「変更が正常に行われました」と表示されて設定内容が保存されます。
7. Amazon Bedrockモデルの「モデルを表示 \>」をクリックします。
8. 「Anthropic Claude」が有効化されているか確認してください。
    - 有効化されていなければトグルスイッチを有効にしてください。
    ![Dify Bedrock モデル有効化](/images/books/pyml-niigata-dify/dify_bedrock_models.png "Dify Bedrock モデル有効化")


これで、モデルプロバイダの設定が完了しました。

#### モデルプロバイダの設定画面の表記が異なる場合

Amazon Bedrockモデルプラグインのバージョンが異なる可能性があります。
バージョンアップしておきましょう。

1. Dify.aiの画面右上にある「プラグイン」をクリックします。
2. 「Amazon Bedrock」をクリックします。
3. 右側にプラグインのプロパティが表示されるので、バージョン番号をクリックします。
4. バージョン「0.0.25」をクリックして「インストール」をクリックします。
    - Amazon Bedrockプラグインのバージョンが「0.0.25」になっていればOKです。

![DifyプラグインBedrockバージョン画面](/images/books/pyml-niigata-dify/dify_bedrock_versions.png "DifyプラグインBedrockバージョン画面")

### OpenAIを利用する

ここでは、AWSアカウントがない方向けに、OpenAIのモデルを利用する手順を示します。
Dify.aiのSANDBOXアカウントでは、OpenAIのモデルを200クレジット分無料で利用可能ですので、今回はそれを利用します。これはOpenAIのAPIアカウントが無くても、利用可能です。

#### Difyのモデルプロバイダを設定

1. Dify.aiにログインし、画面右上にあるユーザアイコンをクリックして「設定」をクリックします。
2. 左側メニューの「モデルプロバイダー」をクリックします。
3. 「モデルプロバイダーをインストールする」の中から「OpenAI」にカーソルを合わせ「インストール」をクリックします。
    - 画面上部の「モデル」欄に追加されます。
4. OpenAIモデルの「モデルを表示 \>」をクリックします。
5. 「gpt-4o-mini」が有効化されているか確認してください。
    - 有効化されていなければトグルスイッチを有効にしてください。
    ![Dify OpenAI モデル有効化](/images/books/pyml-niigata-dify/dify_openai_models.png "Dify OpenAI モデル有効化")


これで、モデルプロバイダの設定が完了しました。


