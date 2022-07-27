# AWS総合演習

## 演習の全体像

Web3層アーキテクチャ（3Tier）でナレッジシステムを開発する。ローカル（データ層はクラウド）上で開発をし、全体的に完成したら順次クラウドにデプロイする。

<img src="img/01_01.jpg">

<br><br>

ナレッジシステムは、以下の機能を実装する。

- ナレッジ一覧機能（初期表示）
- ナレッジ追加機能
- ナレッジ削除機能

<img src="img/01_02.jpg">

<br><br>

## プロジェクト構成

### プレゼンテーション層

ローカル上に下記の構成でファイルを作成する。

<img src="img/01_03.jpg" width="400">

<br>

### アプリケーション層

Eclipse上にknowledge-apiプロジェクトを用意し、以下の構成でファイルを作成する。

<img src="img/01_04.jpg" width="600">

<br>

### データ層

開発段階からRDS上にMySQLを用意し、アプリケーション層と接続する。

<br><br>

## テーブル定義

**KNOWLEDGEテーブル**

| 論理名 | 物理名 | データ型 | PK | UK | FK | Not Null | Default |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| ID | ID | INT(11) | ◯ |  |  |  | 自動採番 |
| CATEGORY | カテゴリー | VARCHAR(30) |  |  |  | ◯ |  |
| KNOWLEDGE | ナレッジ | TEXT |  |  |  | ◯ |  |
| UPDATE_AT | 更新日 | TIMESTAMP |  |  |  | ◯ | 現在日時 |

```sql
CREATE DATABASE IF NOT EXISTS knowledge;

USE knowledge

CREATE TABLE IF NOT EXISTS KNOWLEDGE (
   ID INT(11) AUTO_INCREMENT PRIMARY KEY
  ,CATEGORY VARCHAR(30) NOT NULL
  ,KNOWLEDGE TEXT NOT NULL
  ,UPDATE_AT TIMESTAMP DEFAULT CURRENT_TIMESTAMP NOT NULL
);

INSERT INTO KNOWLEDGE (CATEGORY, KNOWLEDGE) VALUES ('Java', 'Javaは言語として高性能で、OSに依存せずに動作すること、開発効率と保守性が高いこと、ライブラリが充実していることなどのメリットがあります。');
INSERT INTO KNOWLEDGE (CATEGORY, KNOWLEDGE) VALUES ('Java', 'Javaのバージョンは半年に1度、3月と9月にリリースされ、3年に1度、長期サポート（LTS）版がリリースされます。');
INSERT INTO KNOWLEDGE (CATEGORY, KNOWLEDGE) VALUES ('AWS', 'AWS（Amazon Web Services）は、Amazon社が提供するクラウドサービスプラットフォームで、2021年9月時点で220を超えるサービスを提供しています。');
```

<br>

### RDSの設定

- MySQL WorkbenchでRDSに接続する。
    - ユーザ名：root
    - パスワード：P4ssw0rd
    - スキーマ：knowledge
- KNOWLEDGEテーブルの作成とデータの用意をする。（各ルームの代表者）

<br><br>

## 機能仕様

### 1. ナレッジ一覧機能

#### 処理仕様

<img src="img/02_01.jpg">

| ファイル名 | 処理内容 |
|:---:|-----|
| knowledge.html | ロード時に非同期通信でListKnowledgeServletにGET通信し、ナレッジデータを全件取得する。 |
| FindKnowledgeServlet.java | URLパターン：/find<br>HTTPメソッド：GET<br>KnowledgeDaoのfindAll()メソッドを呼び出し、ナレッジデータを全件取得する。取得したナレッジデータをJSON形式に変換し、レスポンスのボディ部に書き出す。 |
| KnowledgeDao.java | メソッド名：findAll()<br>引数の型：なし<br>戻り値の型：Knowledgeのリスト<br>KNOWLEDGEテーブルのデータを全件取得する。取得したデータをListに格納し、返却する。 |

<br>

### 2. ナレッジ登録機能

#### 処理仕様

<img src="img/02_02.jpg">

| ファイル名 | 処理内容 |
|:---:|-----|
| knowledge.html | 追加ボタン押下時に非同期通信でCreateKnowledgeServletにPOST通信し、ナレッジデータを登録する。POST通信時、入力されたカテゴリー、ナレッジをURLSearchParamsオブジェクトで送る。<br>登録処理後、ナレッジを再度全件検索し、カテゴリーとナレッジの入力値をクリアする。 |
| CreateKnowledgeServlet.java | URLパターン：/create<br>HTTPメソッド：POST<br>KnowledgeDaoのcreate()メソッドを呼び出し、リクエスト情報（カテゴリー、ナレッジ）を基にナレッジデータを登録する。 |
| KnowledgeDao.java | メソッド名：create()<br>引数の型：Knowledge<br>戻り値の型：なし<br>KNOWLEDGEテーブルにデータを登録する。 |

<br>

### 3. ナレッジ削除機能

#### 処理仕様

<img src="img/02_03.jpg">

| ファイル名 | 処理内容 |
|:---:|-----|
| knowledge.html | 削除ボタン押下時に非同期通信でDeleteKnowledgeServletにPOST通信し、ナレッジデータを削除する。POST通信時、削除対象ナレッジのIDをURLSearchParamsオブジェクトで送る。<br>削除処理後、ナレッジを再度全件検索する。 |
| DeleteKnowledgeServlet.java | URLパターン：/delete<br>HTTPメソッド：POST<br>KnowledgeDaoのdelete()メソッドを呼び出し、対象のナレッジデータを削除する。 |
| KnowledgeDao.java | メソッド名：delete()<br>引数の型：int<br>戻り値の型：なし<br>KNOWLEDGEテーブルから引数IDのデータを削除する。 |

<br><br>

## 演習手順

### 1. ナレッジシステムの開発

- 機能仕様を基にナレッジシステムを開発する。
    - ナレッジ一覧機能
    - ナレッジ追加機能
    - ナレッジ削除機能

<br>

### 2. アプリケーション層のデプロイ

- 以下の仕様でEC2インスタンスを起動する。
    - タグ：自分の番号_knowledge_api_1
    - AMI：Ubuntu Server 22.04 LTS
    - インスタンスタイプ：t2.micro
    - キーペア：新規作成する
        - キーペア名：自分の番号_knowledge_key
        - タイプ：ED25519
        - 形式：.pem
    - VPC：デフォルト
    - サブネット：ap-northeast-1a
    - セキュリティグループ：新規作成する
        - セキュリティグループ名：自分の番号_knowledge_security
        - ssh(22)：0.0.0.0/0、http(80)：0.0.0.0/0、tcp(8080)：0.0.0.0/0
    - ストレージ：10GiB
- アプリケーション層（Servlet）のwarファイルを生成する。
- インスタンスに接続し、以下のソフトウェアをインストールし、適切な設定でwarファイルをデプロイ、公開する。
    - openjdk-11-jre
    - tomcat9
    - apache2
- HTTPクライアントツール（Talend API Tester）で公開したアプリケーション層に対してデータを送信し、想定通りにデータの取得や登録ができることを確認する。
    - `http://[パブリックIPアドレス]/knowledge-api/find`
    - `http://[パブリックIPアドレス]/knowledge-api/create`
    - `http://[パブリックIPアドレス]/knowledge-api/delete`
- 構築したEC2インスタンスのイメージ（AMI）を作成する。
    - イメージ名：自分の番号_knowledge_api_image
    - イメージの説明：任意の説明文
    - ボリュームサイズ：10

<br>

### 3. プレゼンテーション層のデプロイ

- 以下の仕様でknowledge.htmlをS3にホスティングする。
    - バケット名：自分の番号-knowledge
- CloudFrontを作成し、S3と連携する。
- エンドポイントにアクセスし、ナレッジシステムが正常に動作することを確認する。
    - ナレッジ一覧機能
    - ナレッジ登録機能
    - ナレッジ削除機能

<br>

### 4. ロードバランサーの導入

<span style="color:red;">※作成数に上限があるため、動作確認後は上記3.までの状態に戻す。</span>

- 2.で作成したAMIを基にEC2インスタンスを作成する。
    - タグ：自分の番号_knowledge_api_2
    - AMI：自作のAMI
    - インスタンスタイプ：t2.micro
    - キーペア：自分の番号_knowledge_key
    - VPC：デフォルト
    - サブネット：ap-northeast-1c
    - セキュリティグループ：自分の番号_knowledge_security
    - ストレージ：10GiB
- ロードバランサーを作成する。
    - Load balancer name：自分の番号_elb_knowledge
    - Mappings：ap-northeast-1a、ap-northeast-1c
    - Security Groups：新規作成する
        - セキュリティグループ名：自分の番号_knowledge_security_elb
        - http(80)：0.0.0.0/0
    - Target group：新規作成する
        - Target group name：999-knowledge-target-group
        - Targets：自分の番号_knowledge_api_1、自分の番号_knowledge_api_2
- 2つのEC2インスタンスのアクセス制御を施す。
    - ロードバランサーからのみアクセスできるようにする。
    - HTTPクライアントツール（Talend API Tester）で、各EC2インスタンスに直接アクセスできないことを確認する。
- index.htmlを修正する。
    - ロードバランサー経由でアプリケーション層にアクセスする。
    - 正常にナレッジシステムが動作することを確認する。

<br>

## 5. Lambda用アプリケーションの開発

- アプリケーション層（Java）
    - Eclipse上にknowledge-lambdaプロジェクトを作成する。
        - Mavenプロジェクト
        - Create a simple project
        - Group Id：jp.knowledge
        - Artifact Id：knowledge-lambda
        - Packaging：jar
    - 以下の仕様で部分一致検索をする機能を実装する。

        | パッケージ名.クラス名 | 処理 |
        |---|---|
        | jp.knowledge.api.SearchKnowledge | Inputの型：Map<String, String><br>Outputの型：List\<Knowledge\><br>リクエスト情報（検索キーワード）を基にナレッジの部分一致検索をする。取得したデータをリストに格納し、返却する。 |
        | jp.knowledge.dao.KnowledgeDao | メソッド名：findLikeKeyword()<br>KNOWLEDGEテーブルから引数のキーワードでナレッジの部分一致検索をし、紐付くナレッジデータを取得する。 |
    - 正常に動作することを確認したら、AWS Lambdaにアップロードする。
        - 関数名：自分の番号_FindKnowledgeByKeyword
        - knowledge-lambda.jarをアップロードする。
        - JSONで任意のIDを指定しテストする。指定したキーワードに部分一致するナレッジデータを返ってくることを確認する。
    - Lambda関数「自分の番号_FindEmployee」のトリガーとなるAPI Gatewayを作成する。
        - API name：自分の番号_FindKnowledgeByKeyword-API
        - メソッド：GET
            - クエリパラメータを送信できるように設定する。

<br>

- プレゼンテーション層（knowledge.html）
    - 「ナレッジ一覧」のタイトル部分の下に以下のコードを追記する。
        ```html
        (中略)
        <div class="my-3 ml-2 text-info">
          <h3>ナレッジ一覧</h3>
        </div>

        <!-- 下記コードを追加 -->
        <div>
          <form class="form-group form-inline">
            <input type="text" class="form-control" id="keyword" placeholder="検索キーワード">
            <button type="button" class="btn btn-secondary">検索</button>
          </form>
        </div>
        (中略)
        ```
    - 削除ボタン押下時に非同期通信でAPI GatewayのエンドポイントにGET通信し、ナレッジデータを取得する。GET通信時、検索キーワードをクエリパラメータで送る。
検索後、ナレッジ一覧（id=knowledgesの要素の中身）をクリアし、取得したナレッジid=knowledgesの要素に追加する。
