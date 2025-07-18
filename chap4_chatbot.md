# Difyで簡単なチャットボットアプリを作ろう

前章でモデルプロバイダの設定までできていると思いますので、
ここで簡単なチャットボットアプリを作ってみましょう。

## チャットボットの作成

1. Difyのホーム画面にて「全て」もしくは「チャットボット」を選択し、「アプリを作成する」欄の「最初から作成」をクリックします。
    ![Dify チャットボット作成](/images/books/pyml-niigata-dify/dify_chatbot_create.png "Dify チャットボット作成")
2. 「アプリタイプを選択」の中から小さく書かれた「初心者向けの基本的なアプリタイプ」をクリックし「チャットボット」をクリックします。
3. 「アプリのアイコンと名前」のテキストボックスに適当なアプリ名を入力して「作成する」をクリックします。
    ![Difyチャットボット作成 初期画面](/images/books/pyml-niigata-dify/dify_chatbot_init.png "Difyチャットボット作成 初期画面")
4. オーケストレーション画面が表示されます。右側のモデル名をクリックしましょう。
    ![Dify チャットボット オーケストレーション](/images/books/pyml-niigata-dify/dify_chatbot_orchestration_home.png "Dify チャットボット オーケストレーション")
5. 以下の設定をします。
    - モデル: Anthropic Claude
    - パラメータ
        - Bedrock Model: Claude 3.7 Sonnet
        - Use Cross-Region Inference: True (デフォルト)
        - 他デフォルトのままでOKです
    ![Dify チャットボット モデル設定](/images/books/pyml-niigata-dify/dify_chatbot_model_settings.png "Dify チャットボット モデル設定")
    - OpenAIを利用する場合は、モデルを「gpt-4o-mini」あたりを指定しましょう。
        - パラメータは「プリセットの読み込み」からお好きな設定を指定しましょう。
6. 「デバッグとプレビュー」欄下の「Botと話す」内で、適当にBotと会話できるか確認しましょう。
    ![Dify チャットボット デバッグ](/images/books/pyml-niigata-dify/dify_chatbot_debug.png "Dify チャットボット デバッグ")
7. 「公開する」をクリックして「更新を公開」をクリックしましょう。
    ![Dify チャットボット 公開](/images/books/pyml-niigata-dify/dify_chatbot_deploy.png "Dify チャットボット 公開")
8. 再度「公開する」をクリックして「アプリを実行」をクリックしましょう。
    - 実際のDifyのチャットボットアプリケーションが利用できます。
    ![Dify チャットボット 実行](/images/books/pyml-niigata-dify/dify_chatbot_run.png "Dify チャットボット 実行")

これで、チャットボットアプリの完成です。
簡単な手順でアプリを利用できます。

ちなみに、オーケストレーション内の「プロンプト」にシステムプロンプトを記載できるので、チャットボット特有の振る舞いをプロンプトで指定できます。

## チャットボットの管理

オーケストレーションの左側にある、チャットボット画像のアイコン等をクリックすると、チャットボットアプリケーションの管理ができます。

![Dify チャットボット 管理設定メニュー](/images/books/pyml-niigata-dify/dify_chatbot_admin_menu.png "Dify チャットボット 管理設定メニュー")

### チャットボットの基本情報

ここでは、チャットボット名の変更などを行うことができます。チャットボットアプリケーションを停止することも可能です。

![Dify チャットボット 基本情報](/images/books/pyml-niigata-dify/dify_chatbot_basic_settings.png "Dify チャットボット 基本情報")

### オーケストレーション

デフォルトの表示画面です。
チャットボットの振る舞いを定義し、デバッグできます。

![Dify チャットボット オーケストレーション](/images/books/pyml-niigata-dify/dify_chatbot_orchestration_home.png "Dify チャットボット オーケストレーション")

### チャットアプリAPI

チャットアプリをAPIとして利用する方法が記載されています。APIキーもここで払い出せます。

![Dify チャットボット API](/images/books/pyml-niigata-dify/dify_chatbot_api_home.png "Dify チャットボット API")

### ログ

アプリケーションの実行ログを確認できます。

![Dify チャットボット ログ](/images/books/pyml-niigata-dify/dify_chatbot_log_home.png "Dify チャットボット ログ")

### 監視

アプリケーションの利用状況を確認できます。
サードパーティ製のサービスにトレース情報を取り込んで管理することもできます。

![Dify チャットボット 監視](/images/books/pyml-niigata-dify/dify_chatbot_trace_home.png "Dify チャットボット 監視")

## 補足: テキストジェネレーターとは何か？

チャットボットと似た機能で「テキストジェネレーター」があります。

![Difyテキストジェネレーター作成 初期画面](/images/books/pyml-niigata-dify/dify_textgenerator_init.png "Difyテキストジェネレーター作成 初期画面")

「テキストジェネレーター」は与えられた入力プロンプトを元に、文章を生成するアプリタイプです。
チャットボットと違い、会話形式ではなく「1回実行したら結果を返す」タイプのアプリになります。

テキストジェネレーターでは「変数」を使って、以下のようにキーワードを入れると文章を作らせるといったことができます。

![Dify テキストジェネレーター サンプル](/images/books/pyml-niigata-dify/dify_textgenerator_sample.png "Dify テキストジェネレーター サンプル")
