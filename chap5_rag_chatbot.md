# DifyでRAGチャットボットアプリを作ろう

前章で簡単なチャットボットアプリを作ってみました。
モデルが学習済の情報を元にチャットボットが回答するアプリでした。
ここでは、モデルが学習していない文書のデータを元にチャットボットが回答する「RAGチャットボット」を作成してみましょう。

## RAGの仕組み

RAG (Retrieval-Augmented Generation: 検索拡張生成) とは、質問に対応するドキュメントと質問文を生成AIに読み込ませて、ドキュメントの内容に従った回答文を生成する手法です。
「情報の検索」と「大規模言語モデルによる文章生成」を組み合わせたものになります。
これによって、例えば「社内情報に対応したAIチャットボット」を簡単に実現することができます。

![RAGの仕組み](/images/books/pyml-niigata-dify/rag_architecture.png "RAGの仕組み")

RAGに必要なモデルは、Claudeのような大規模言語モデルの他に、「埋め込み (Embeddings)」モデルが必要になります。これは文書のテキスト文字列を数値ベクトルに変換するためのモデルです。
RAGでは、このモデルとセットでベクトルストア上に文書のテキスト文字列を保存することで、テキスト文字列の検索を効率よく行うことができます。

またDifyでは、「Rerank」モデルも利用する必要があります。これはベクトル検索の結果として抽出された関連性のある文字列をさらに評価し、関連性の高い順序に並び替えます。
RAGにおいては必須ではありませんが、Difyで利用する場合にRerankモデルを指定しないとエラーになるため、利用します。

## モデルの有効化

今回RAGで利用するモデルは、Claudeの他に「埋め込み(Embeddings)」と「Rerank」のモデルを利用します。

Amazon Bedrockを利用する場合は、「埋め込み」は「Amazon Titan Text Embeddings V2」を、「Rerank」は「Cohere Rerank 3.5」を今回利用するので、各々のモデルの有効化を行います。

Bedrockを利用しない場合は、OpenAIのEmbbeddingsモデルと、Cohereの無料で使えるRerankモデルをそれぞれ使います。

### Amazon Bedrockを利用する場合

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
7. 「Amazon」の「Titan Text Embeddings V2」は有効化済であることを確認し、「Cohere」の「Rerank 3.5」にチェックを入れて、「変更を保存」をクリック
    ![Bedrock モデル有効化 Titan Text Embeddings V2](/images/books/pyml-niigata-dify/bedrock_models_titan_embed_v2.png "Bedrock モデル有効化 Titan Text Embeddings V2")
    ![Bedrock モデル有効化 Cohere Rerank 3.5](/images/books/pyml-niigata-dify/bedrock_models_cohere_rerank_35.png "Bedrock モデル有効化 Cohere Rerank 3.5")
8. アクセスが付与される間に、以降の作業を進めます。

#### Difyのモデルプロバイダを設定

1. Dify.aiにログインし、画面右上にあるユーザアイコンをクリックして「設定」をクリックします。
2. 左側メニューの「モデルプロバイダー」をクリックします。
3. Amazon Bedrockモデルの「モデルを表示 \>」をクリックします。
4. 以下のモデルを有効化します。
    - `amazon.titan-embed-text-v2:0`
    - `cohere.rerank-v3-5:0`
    - 有効化されていなければトグルスイッチを有効にしてください。
    ![Dify Bedrock モデル有効化 埋め込み Rerank](/images/books/pyml-niigata-dify/dify_bedrock_models_embed_rerank.png "Dify Bedrock モデル有効化 埋め込み Rerank")

### Amazon Bedrockを利用しない場合

#### Cohereアカウントの作成

今回はCohereの無料アカウントを作成して、Rerankモデルを無料利用枠の範囲で利用します。

1. cohereのサイトにアクセスします。
    - [https://cohere.com/](https://cohere.com/)
2. 「Sign in」をクリックします。
3. 画面右上にある「Sign up」をクリックします。
4. 必要事項を入力して「Sign up」をクリックします。
    - 「Googleアカウント」または「GitHubアカウント」を用いても良いです
    - アカウント作成の際に表示される質問には、適宜回答しておきます。
5. cohere dashboard画面が表示されたら、左側メニューの「API Keys」をクリックします。
6. すでに「Trial Keys」が「default」の名前で1つ作成されていると思います。そのKeyの目玉アイコン「👁️」をクリックして、表示されたKeyの文字列をメモしておきます。
    ![Cohere API Key](/images/books/pyml-niigata-dify/cohere_api_key.png "Cohere API Key")

#### Difyのモデルプロバイダを設定

1. Dify.aiにログインし、画面右上にあるユーザアイコンをクリックして「設定」をクリックします。
2. 左側メニューの「モデルプロバイダー」をクリックします。
3. OpenAIモデルの「モデルを表示 \>」をクリックします。
4. 以下のモデルを有効化します。
    - `text-embedding-3-large`
    - `text-embedding-3-small`
    - 有効化されていなければトグルスイッチを有効にしてください。
    ![Dify OpenAI モデル有効化 埋め込み](/images/books/pyml-niigata-dify/dify_openai_models_embed.png "Dify OpenAI モデル有効化 埋め込み")
5. 「モデルプロバイダーをインストールする」の中から「Cohere」にカーソルを合わせ「インストール」をクリックします。
    - 画面上部の「モデル」欄に追加されます。
6. CohereモデルのAPI-KEYの「セットアップ」をクリックします。
    ![Dify Cohere 初期設定](/images/books/pyml-niigata-dify/dify_cohere_init.png "Dify Cohere 初期設定")
7. 以下の項目を入力し「保存」をクリックします。
    - API Key: CohereのAPIキー
    ![Dify Cohere API Keyセットアップ](/images/books/pyml-niigata-dify/dify_cohere_api_key.png "Dify Cohere API Keyセットアップ")

8. 「モデルの表示」をクリックします
9. 「rerank-v3.5」が有効化になっていることを確認します。
    ![Dify Cohere Rerankモデル有効化](/images/books/pyml-niigata-dify/dify_cohere_models_rerank.png "Dify Cohere Rerankモデル有効化")


## ナレッジの作成

Difyには、文書を管理するための「ナレッジ」という機能があります。
今回ナレッジとして使う文書をダウンロードします。

- [kintai.pdf](tmp/pyml-niigata-dify/kintai.pdf) (勤怠管理システム 従業員利用 マニュアル)

1. Difyの画面上部にある「ナレッジ」をクリックし、「ナレッジベースを作成」をクリックします。
    ![Dify ナレッジベース作成](/images/books/pyml-niigata-dify/dify_knowledge_base_create.png "Dify ナレッジベース作成")
2. 以下の設定を行い「次へ」をクリックします。
    - データソース: 「テキストファイルからインポート」を選択
    - テキストファイルをアップロード: 先ほどダウンロードした「kintai.pdf」をアップロード
        - ファイルは複数アップロードできます
    ![Dify ナレッジベース作成 データソース](/images/books/pyml-niigata-dify/dify_knowledge_base_datasource.png "Dify ナレッジベース作成 データソース")
    - 「空のナレッジベースを作成します」をクリックして、ナレッジベース名を指定します
        - ナレッジベース名: 社内文書
    ![Dify ナレッジベース作成 データソース ナレッジベース名](/images/books/pyml-niigata-dify/dify_knowledge_base_datasource_knowledge_base_name.png "Dify ナレッジベース作成 データソース ナレッジベース名")
3. 以下の設定を行い「保存して処理」をクリックします。
    - ここではテキストの前処理方法と検索方法を設定します
    - チャンク設定
        - 「親子」を選択
        - コンテキスト用親チャンク > 段落 > 最大チャンク長: 1536 characters
        - 検索用子チャンク > チャンク識別子: `\n\n`
        - 検索用子チャンク > 最大チャンク長: 512 characters
    - インデックス方法: 高品質
    ![Dify ナレッジベース テキスト処理設定1](/images/books/pyml-niigata-dify/dify_knowledge_base_create_config1.png "Dify ナレッジベース テキスト処理設定1")
    - 埋め込みモデル:
        - Bedrockの場合: `amazon.titan-embed-text-v2:0`
        - OpenAIの場合: `text-embedding-3-large`
    ![Dify ナレッジベース テキスト処理設定2](/images/books/pyml-niigata-dify/dify_knowledge_base_create_config2.png "Dify ナレッジベース テキスト処理設定2")
    - 検索設定
        - 「ハイブリッド検索」を選択
        - 「Rerankモデル」を選択
        - Bedrockの場合: `cohere.rerank-v3-5:0`
        - Cohereの場合: `rerank-v3.5`
        - トップK: 5
    ![Dify ナレッジベース テキスト処理設定3](/images/books/pyml-niigata-dify/dify_knowledge_base_create_config3.png "Dify ナレッジベース テキスト処理設定3")
    ![Dify ナレッジベース テキスト処理設定4](/images/books/pyml-niigata-dify/dify_knowledge_base_create_config4.png "Dify ナレッジベース テキスト処理設定4")
4. ナレッジベースが作成され、しばらくすると埋め込みが完了すればOKです。
    ![Dify ナレッジベース作成完了](/images/books/pyml-niigata-dify/dify_knowledge_base_create_complete.png "Dify ナレッジベース作成完了")


ちなみに、RAGの精度を向上させる方法の解説記事がAWSブログにありましたので、ご紹介します。興味のある方はご確認ください。

- [RAGの精度を向上させる Advanced RAG on AWSの道標](https://aws.amazon.com/jp/blogs/news/a-practical-guide-to-improve-rag-systems-with-advanced-rag-on-aws/)

## RAGチャットボットの作成

1. Difyの画面上部にある「スタジオ」をクリックし、「アプリを作成する」欄の「最初から作成」をクリックします。
    ![Dify チャットボット作成](/images/books/pyml-niigata-dify/dify_chatbot_create.png "Dify チャットボット作成")
2. 「アプリタイプを選択」の中から小さく書かれた「初心者向けの基本的なアプリタイプ」をクリックし「チャットボット」をクリックします。
3. 「アプリのアイコンと名前」のテキストボックスに適当なアプリ名を入力して「作成する」をクリックします。
    ![Dify RAGチャットボット作成 初期画面](/images/books/pyml-niigata-dify/dify_rag_chatbot_init.png "Dify RAGチャットボット作成 初期画面")
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
6. 「プロンプト」欄に、チャットボットの指示として、検索してユーザの質問に回答する指示を与えます。
    ```text
    ユーザの入力に回答するために、社内文書を検索して関連する文書を確認した上で回答するようにしてください。
    ```
    ![Dify RAGチャットボット プロンプト](/images/books/pyml-niigata-dify/dify_rag_chatbot_prompt.png "Dify RAGチャットボット プロンプト")
7. コンテキスト欄の「+ 追加」をクリックし、利用するナレッジ「社内文書」を選択し、「追加」をクリックします。
    ![Dify RAGチャットボット コンテキスト](/images/books/pyml-niigata-dify/dify_rag_chatbot_context.png "Dify RAGチャットボット コンテキスト")
8. デバッグとプレビュー欄下部にある「有効な機能」の「管理 →」をクリックし、「会話の開始」を有効化して「オープナーを書く」をクリックします。
9. チャットボット側からの最初のメッセージを設定します。さらにオプションも追加します。設定したら「保存」をクリックします。
    - メッセージ
        ```
        こんにちは！社内文書を検索して、あなたの質問に回答します。私は現在、以下の文書に対する知識を持っています。

        - 勤怠管理システム 従業員利用 マニュアル

        選択肢から質問したり、自由に入力して質問することができます。
        ```
    - オプション
        - 有給休暇の申請方法は？
    ![Dify RAGチャットボット オープナー](/images/books/pyml-niigata-dify/dify_rag_chatbot_opener.png "Dify RAGチャットボット オープナー")
10. 機能欄の右側にある「×」をクリックして元の画面に戻り、画面右側にあるプレビュー画面でRAGチャットを体験してみましょう。
    ![Dify RAGチャットボット デバッグ](/images/books/pyml-niigata-dify/dify_rag_chatbot_debug.png "Dify RAGチャットボット デバッグ")
11. 「公開する」をクリックして、サイトを最新状態で更新し、サイトを開いて確認してみましょう。
    ![Dify RAGチャットボット アプリ起動](/images/books/pyml-niigata-dify/dify_rag_chatbot_app.png "Dify RAGチャットボット アプリ起動")

### 補足: RAGでの検索結果の確認

検索結果がどのようなものになっていたかを確認することができます。

プレビュー画面でRAGチャットを実行し、チャットボットからの回答にマウスカーソルを合わせて「ドキュメントアイコン」の画像をクリックします。

![Dify RAGチャットボット 検索結果確認1](/images/books/pyml-niigata-dify/dify_rag_chatbot_retrieve1.png "Dify RAGチャットボット 検索結果確認1")

プロンプトログが表示されます。この中で `<context>` タグで囲まれた部分がナレッジ検索結果として抽出された内容になります。

![Dify RAGチャットボット 検索結果確認2](/images/books/pyml-niigata-dify/dify_rag_chatbot_retrieve2.png "Dify RAGチャットボット 検索結果確認2")

## RAGチャットボットにFAQから回答させる

よくある質問に対しては、定型的にFAQから回答させることで正確性・応答速度などを高め、ユーザの満足度を向上できます。

### FAQの追加と利用

1. 画面左側のログアイコンをクリックして、ログ画面を表示し、「注釈」をクリックします。
    ![Dify チャットボット ログ 注釈](/images/books/pyml-niigata-dify/dify_chatbot_log_annotation.png "Dify チャットボット ログ 注釈")
2. 「注釈の返信」を有効化します。
    ![Dify チャットボット ログ 注釈 返信](/images/books/pyml-niigata-dify/dify_chatbot_log_annotation_faq.png "Dify チャットボット ログ 注釈 返信")
3. 以下の設定をして「保存して有効にする」をクリックします。
    - スコア閾値: 0.80
    - 埋め込みモデル:
        - Bedrockの場合: `amazon.titan-embed-text-v2:0`
        - OpenAIの場合: `text-embedding-3-large`
    ![Dify チャットボット ログ 注釈 返信 設定](/images/books/pyml-niigata-dify/dify_chatbot_log_annotation_faq_settings.png "Dify チャットボット ログ 注釈 返信 設定")
4. 「注釈を追加」をクリックします。
5. 以下の内容を設定し「追加」をクリックします。
    - 質問
        ```
        育児休暇の方法は？
        ```
    - 回答
        ```
        育児休暇については、勤怠管理システム外での手続きが必要になります。人事部門 (内線: 1234) までご連絡ください。
        ```
    ![Dify チャットボット ログ 注釈 返信 追加](/images/books/pyml-niigata-dify/dify_chatbot_log_annotation_faq_add.png "Dify チャットボット ログ 注釈 返信 追加")
6. オーケストレーション画面に戻り、デバッグ画面で確認してみましょう。
    ![Dify チャットボット ログ 注釈 返信 デバッグ](/images/books/pyml-niigata-dify/dify_chatbot_log_annotation_faq_debug.png "Dify チャットボット ログ 注釈 返信 デバッグ")
7. 「公開する」をクリックして、サイトを最新状態で更新し、サイトを開いて確認してみましょう。
