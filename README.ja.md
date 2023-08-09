[![FIWARE Banner](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/fiware.png)](https://www.fiware.org/developers)
[![NGSI v2](https://img.shields.io/badge/NGSI-v2-5dc0cf.svg)](https://fiware-ges.github.io/orion/api/v2/stable/)

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Historic-Context-NIFI.svg)](https://opensource.org/licenses/MIT)

[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

このチュートリアルは [FIWARE Draco](https://fiware-draco.readthedocs.io/en/latest/)
の紹介です。
[Apache NIFI](https://nifi.apache.org) を使用してコンテキスト・データを
サードパーティのデータベースに永続化してコンテキストの履歴ビューを作成するために
使用される Generic Enabler です。チュートリアルでは、
[以前](https://github.com/FIWARE/tutorials.IoT-Agent) のチュートリアルで接続した
IoT センサをアクティブにし、 それらのセンサからの測定値をさらに分析するために
データベースに保存します。

チュートリアルは全体を通して [cUrl](https://ec.haxx.se/) コマンドを使いますが、
[Postman documentation](https://fiware.github.io/tutorials.Historic-Context-NIFI/)
も利用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/9658043920d9be43914a)
[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/FIWARE/tutorials.Historic-Context-NIFI/tree/NGSI-v2)

## コンテンツ

<details>
<summary>詳細 <b>(クリックして拡大)</b></summary>

-   [Apache NIFI を使用したデータの永続性](#data-persistence-using-apache-nifi)
-   [アーキテクチャ](#architecture)
-   [前提条件](#prerequisites)
    -   [Docker と Docker Compose](#docker-and-docker-compose)
    -   [Cygwin for Windows](#cygwin-for-windows)
-   [起動](#start-up)
-   [MongoDB - コンテキスト・データをデータベースに永続化](#mongodb---persisting-context-data-into-a-database)
    -   [MongoDB - データベース・サーバの設定](#mongodb---database-server-configuration)
    -   [MongoDB - Draco の設定](#mongodb---draco-configuration)
    -   [MongoDB - 起動](#mongodb---start-up)
        -   [Draco サービスの健全性をチェック](#checking-the-draco-service-health)
        -   [コンテキスト・データの生成](#generating-context-data)
        -   [コンテキスト変更のサブスクライブ](#subscribing-to-context-changes)
    -   [MongoDB - データベースからデータを読み込む](#mongodb----reading-data-from-a-database)
        -   [MongoDB サーバ上で利用可能なデータベースを表示](#show-available-databases-on-the-mongodb-server)
        -   [サーバから履歴コンテキストを読み込む](#read-historical-context-from-the-server)
-   [PostgreSQL - コンテキスト・データをデータベースに永続化](#postgresql---persisting-context-data-into-a-database)
    -   [PostgreSQL - データベース・サーバの設定](#postgresql---database-server-configuration)
    -   [PostgreSQL - Draco の設定](#postgresql---draco-configuration)
    -   [PostgreSQL - 起動](#postgresql---start-up)
        -   [Draco サービスの健全性をチェック](#checking-the-draco-service-health-1)
        -   [コンテキスト・データの生成](#generating-context-data-1)
        -   [コンテキスト変更のサブスクライブ](#subscribing-to-context-changes-1)
    -   [PostgreSQL - データベースからデータを読み込む](#postgresql---reading-data-from-a-database)
        -   [PostgreSQL server サーバ上で利用可能なデータベースを表示](#show-available-databases-on-the-postgresql-server)
        -   [PostgreSQL server サーバから履歴コンテキストを読み込む](#read-historical-context-from-the-postgresql-server)
-   [MySQL - コンテキスト・データをデータベースに永続化](#mysql---persisting-context-data-into-a-database)
    -   [MySQL - データベース・サーバの設定](#mysql---database-server-configuration)
    -   [MySQL - Draco の設定](#mysql---draco-configuration)
    -   [MySQL - 起動](#mysql---start-up)
        -   [Draco サービスの健全性をチェック](#checking-the-draco-service-health-2)
        -   [コンテキスト・データの生成](#generating-context-data-2)
        -   [コンテキスト変更のサブスクライブ](#subscribing-to-context-changes-2)
    -   [MySQL - データベースからデータを読み込む](#mysql---reading-data-from-a-database)
        -   [MySQL サーバ上で利用可能なデータベースを表示](#show-available-databases-on-the-mysql-server)
        -   [MySQL サーバから履歴コンテキストを読み込む](#read-historical-context-from-the-mysql-server)
-   [マルチ・エージェント - 複数のデータベースへのコンテキスト・データの永続化](#multi-agent---persisting-context-data-into-a-multiple-databases)
    -   [マルチ・エージェント - 複数のデータベースのための Draco 設定](#multi-agent---draco-configuration-for-multiple-databases)
    -   [マルチ・エージェント - 起動](#multi-agent---start-up)
        -   [Draco サービスの健全性をチェック](#checking-the-draco-service-health-3)
        -   [コンテキスト・データの生成](#generating-context-data-3)
        -   [コンテキスト変更のサブスクライブ](#subscribing-to-context-changes-3)
    -   [マルチ・エージェント - 永続化データの読み込み](#multi-agent---reading-persisted-data)
-   [次のステップ](#next-steps)

</details>

<a name="data-persistence-using-apache-nifi"></a>

# Apache NIFI を使用したデータの永続性

> "Plots within plots, but all roads lead down the dragon’s gullet."
>
> — George R.R. Martin (A Dance With Dragons)

[FIWARE Draco](https://fiware-draco.readthedocs.io/en/latest/) は、履歴コンテキスト・データを一連のデータベースに
永続化できる代替の汎用イネーブラーです。**Cygnus** のように **Draco** は、**Orion Context Broker**から状態の変化を
サブスクライブし、データ・シンクに永続化する前に、そのデータを処理するための目標到達プロセスを提供できます。

前述のように、履歴コンテキスト・データの永続化は、ビッグデータ分析、傾向の発見、または外れ値の削除に役立ちます。
これを行うために使用するツールはニーズによって異なり、**Cygnus** とは異なり、** Draco ** は、手順を設定および監視
するためのグラフィカル・インターフェイスを提供します。

違いの要約を以下に示します:

| Draco                                                                           | Cygnus                                                                           |
| ------------------------------------------------------------------------------- | -------------------------------------------------------------------------------- |
| 通知用の NGSIv2 インターフェイスを提供                                          | 通知用の NGSIv1 インターフェイスを提供                                           |
| 構成可能なサブスクリプション・エンドポイント。ただし、デフォルトは `/v2/notify` | サブスクリプション・エンドポイントは `/notify` をリッスン                        |
| 単一のポートでリッスン                                                          | 入力ごとに別々のポートでリッスン                                                 |
| グラフィカル・インターフェイスで構成                                            | 構成ファイルを介して構成                                                         |
| Apache NIFI ベース                                                              | Apache Flume ベース                                                              |
| **Draco** の[ドキュメント](https://fiware-draco.readthedocs.io/en/latest/)      | **Cygnus** の[ドキュメント](https://fiware-cygnus.readthedocs.io/en/latest/)     |

#### デバイス・モニタ

このチュートリアルの目的のために、一連のダミー IoT デバイスが作成され、Context Broker に接続されます。使用している
アーキテクチャとプロトコルの詳細は、[IoT Sensors チュートリアル](https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-v2)
にあります。各デバイスの状態は、次の UltraLight デバイス・モニタの Web ページで確認できます:
`http://localhost:3000/device/monitor`

![FIWARE Monitor](https://fiware.github.io/tutorials.Historic-Context-Flume/img/device-monitor.png)

<a name="architecture"></a>

# アーキテクチャ

このアプリケーションは、[以前のチュートリアル](https://github.com/FIWARE/tutorials.IoT-Agent/)で作成したコンポーネントと
ダミー IoT デバイスをベースにしています。3 つの FIWARE コンポーネントを使用します。
[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/),
[IoT Agent for Ultralight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/),
コンテキスト・データをデータベースに永続化するための
[Draco Generic Enabler](https://fiware-draco.readthedocs.io/en/latest/) を導入しました。
Orion Context Broker と IoT Agent の両方が [MongoDB](https://www.mongodb.com/) テクノロジを利用して保持している情報の
永続性を維持しています。**MySQL**, **PostgreSQL**, **MongoDB** データベースのいずれかで、履歴コンテキスト・データを永続化
します。

したがって、全体のアーキテクチャーは以下の要素で構成されます :

-   3 つの **FIWARE Generic Enabler**:
    -   FIWARE [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) は、
        [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) を使用してリクエストを受信します
    -   FIWARE [IoT Agent for Ultralight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/) は、
        [Ultralight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
        フォーマットのダミー IoT デバイスからノース・バウンドの測定値を受信し、Context Broker がコンテキスト・エンティティの
        状態を変更するための [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) リクエストに変換します
-   FIWARE Draco はコンテキストの変更をサブスクライブし、データベース (**MySQL** , **PostgreSQL** , **MongoDB**)
    に保持します。
-   以下の**データベース**の 1 つ、2 つまたは 3 つ:
    -   基礎となる [MongoDB](https://www.mongodb.com/) データベース:
        -   **Orion Context Broker** が、データ・エンティティなどのコンテキスト・データ情報を保持し、サブスクリプション、
            レジストレーションするために使用します
        -   **IoT Agent** がデバイスの URL やキーなどのデバイス情報を保持するために使用します
        -   履歴コンテキスト・データを保持するためのデータ・シンクとして潜在的に使用します
    -   追加の [PostgreSQL](https://www.postgresql.org/) データベース:
        -   履歴データを保持するためのデータ・シンクとして潜在的に使用します
    -   追加の [MySQL](https://www.mysql.com/) データベース :
        -   履歴データを保持するためのデータ・シンクとして潜在的に使用します
-   3 つの**コンテキスト・プロバイダ**:
    -   **在庫管理フロントエンド**は、このチュートリアルで使用していません。これは以下を行います :
        -   店舗情報を表示し、ユーザーがダミー IoT デバイスと対話できるようにします
        -   各店舗で購入できる商品を表示します
        -   ユーザが製品を購入して在庫数を減らすことを許可します
    -   HTTP 上で動作する
        [Ultralight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
        プロトコルを使用して、[ダミー IoT デバイス](https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-v2)の
        セットとして機能する Web サーバ
    -   このチュートリアルでは、**コンテキスト・プロバイダの NGSI proxy** は使用しません。これは以下を行います:
        -   [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) を使用してリクエストを受信します
        -   独自の API を独自のフォーマットで使用して、公開されているデータ・ソースへのリクエストを行います
        -   [NGSI-v2](https://fiware.github.io/specifications/OpenAPI/ngsiv2) 形式でコンテキスト・データを Orion Context Broker
            に返します

要素間のすべての対話は HTTP リクエストによって開始されるため、エンティティはコンテナ化され、公開されたポートから実行されます。

チュートリアルの各セクションの具体的なアーキテクチャについては、以下で説明します。

<a name="prerequisites"></a>

# 前提条件

<a name="docker-and-docker-compose"></a>

## Dokcer と Docker Compose

物事を単純にするために、両方のコンポーネントが [Docker](https://www.docker.com)
を使用して実行されます。**Docker** は、さまざまコンポーネントをそれぞれの環境に
分離することを可能にするコンテナ・テクノロジです。

-   Docker Windows にインストールするには、[こちら](https://docs.docker.com/docker-for-windows/)の手順に従ってください
-   Docker Mac にインストールするには、[こちら](https://docs.docker.com/docker-for-mac/)の手順に従ってください
-   Docker Linux にインストールするには、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行するためのツールです。
[YAML file](https://github.com/FIWARE/tutorials.Historic-Context-NIFI/tree/master/docker-compose)
ファイルは、アプリケーションのために必要なサービスを構成するために使用します。
つまり、すべてのコンテナ・サービスは 1 つのコマンドで呼び出すことができます。
Docker Compose は、デフォルトで Docker for Windows と Docker for Mac の一部としてインストールされますが、Linux ユーザは
[ここ](https://docs.docker.com/compose/install/) に記載されている手順に従う必要があります。

次のコマンドを使用して、現在の **Docker** バージョンと **Docker Compose** バージョンを確認できます :

```console
docker-compose -v
docker version
```

Docker バージョン 20.10 以降と Docker Compose 1.29 以上を使用していることを確認し、必要に応じてアップグレードしてください。

<a name="cygwin-for-windows"></a>

## Cygwin for Windows

シンプルな bash スクリプトを使用してサービスを開始します。Windows ユーザは [cygwin](http://www.cygwin.com/) をダウンロードして、
Windows 上の Linux ディストリビューションと同様のコマンドライン機能を提供する必要があります。

<a name="start-up"></a>

# 起動

開始する前に、必要な Docker イメージをローカルで取得または構築しておく必要があります。リポジトリを複製し、以下のコマンドを
実行して必要なイメージを作成してください:

```console
git clone https://github.com/fiware/tutorials.Historic-Context-NIFI.git
cd tutorials.Historic-Context
git checkout NGSI-v2

./services create
```

その後、リポジトリ内で提供される [services](https://github.com/FIWARE/tutorials.Historic-Context-NIFI/blob/NGSI-v2/services)
の Bash スクリプトを実行することによって、コマンドラインからすべてのサービスを初期化できます :

```console
./services <command>
```

ここで、`<command>` は、有効にしたいデータベースによって異なります。このコマンドは、以前のチュートリアルのシード・データを
インポートし、起動時にダミー IoT センサをプロビジョニングします。

> :information_source: **注:** クリーンアップをやり直したい場合は、次のコマンドを使用して再起動することができます :
>
> ```console
> ./services stop
> ```

<a name="mongodb---persisting-context-data-into-a-database"></a>

# MongoDB - コンテキスト・データをデータベースに永続化

MongoDB テクノロジを使用して履歴コンテキスト・データを永続化することは、Orion Context Broker および IoT Agent に関連するデータを
保持するために、MongoDB インスタンスを既に使用しているため、設定が比較的簡単です。MongoDB インスタンスは標準で `27017` ポートを
リッスンしており、全体的なアーキテクチャは以下のようになります。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/mongo-draco-tutorial.png)

<a name="mongodb---database-server-configuration"></a>

## MongoDB - データベース・サーバの設定

```yaml
mongo-db:
    image: mongo:4.2
    hostname: mongo-db
    container_name: db-mongo
    ports:
        - "27017:27017"
    networks:
        - default
```

<a name="mongodb---draco-configuration"></a>

## MongoDB - Draco の設定

```yaml
draco:
    image: ging/fiware-draco:1.1.0
    container_name: draco
    depends_on:
        - mongo-db
    environment:
        - NIFI_WEB_HTTP_PORT=9090
    ports:
        - "9090:9090"
        - "5050:5050"
    healthcheck:
        test: curl --fail -s http://localhost:9090/nifi-api/system-diagnostics || exit 1
```

`draco` コンテナは、2つのポートでリッスンしています :

-   Draco の サブスクリプション・ポート -` 5050` は、サービスが
    Orion Context Broker からの通知をリッスンするポートです
-   Draco の Web インターフェース - `9090` は、プロセッサの設定のためだけに
    公開されています

<a name="mongodb---start-up"></a>

## MongoDB - Start up

**MongoDB** データベースのみでシステムを起動するには、次のコマンドを実行します:

```console
./services mongodb
```

次に、ブラウザに移動し、このURL `http://localhost:9090/nifi` を使用して Draco を開きます。

NiFi GUI の上部にあるコンポーネント・ツールバーに行き、
テンプレート・アイコンを見つけて、Draco ユーザ・スペースの中に
ドラッグ＆ドロップします。この時点で、利用可能なすべてのテンプレートの
リストとともにポップアップが表示されます。テンプレート "MONGO-TUTORIAL"
を選択してください。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/mongo-tutorial-template.png)

すべてのプロセッサを選択し
(shift キーを押しながらすべてのプロセッサをクリック)、
スタート・ボタンをクリックして起動します。
これで、各プロセッサのステータス・アイコンが赤から緑に変わりました。

<a name="checking-the-draco-service-health"></a>

### Draco サービスの健全性をチェック

Draco が起動したら、公開された draco ポート `/nifi-api/system-diagnostics` に
HTTP リクエストを送信することでステータスを確認できます。
レスポンスが空白の場合は、通常 Draco が実行されていないか、
別のポートでリッスンしているためです。

#### :one: リクエスト :

```console
curl -X GET \
  'http://localhost:9090/nifi-api/system-diagnostics'
```

#### レスポンス :

レスポンスは次のようになります :

```json
{
    "systemDiagnostics": {
        "aggregateSnapshot": {
            "totalNonHeap": "value",
            "totalNonHeapBytes": 0,
            "usedNonHeap": "value",
            "usedNonHeapBytes": 0,
            "freeNonHeap": "value",
            "freeNonHeapBytes": 0,
            "maxNonHeap": "value",
            "maxNonHeapBytes": 0,
            "nonHeapUtilization": "value",
            "totalHeap": "value",
            "totalHeapBytes": 0,
            "usedHeap": "value",
            "usedHeapBytes": 0,
            "freeHeap": "value",
            "freeHeapBytes": 0,
            "maxHeap": "value",
            "maxHeapBytes": 0,
            "heapUtilization": "value",
            "availableProcessors": 0,
            "processorLoadAverage": 0.0,
            "totalThreads": 0,
            "daemonThreads": 0,
            "uptime": "value",
            "flowFileRepositoryStorageUsage": {},
            "contentRepositoryStorageUsage": [{}],
            "provenanceRepositoryStorageUsage": [{}],
            "garbageCollection": [{}],
            "statsLastRefreshed": "value",
            "versionInfo": {}
        },
        "nodeSnapshots": [{}]
    }
}
```

> **トラブルシューティング** : レスポンスが空白の場合はどうしますか？
>
> -   docker コンテナが実行されていることを確認するには
>
> ```bash
> docker ps
> ```
>
> いくつかのコンテナが動いているのが確認できます。
> draco が実行されていない場合は、必要に応じてコンテナを再起動できます。

<a name="generating-context-data"></a>

### コンテキスト・データの生成

このチュートリアルでは、コンテキストが定期的に更新されているシステムを
監視する必要があります。これを行うためにダミー IoT センサを使用できます。
`http://localhost:3000/device/monitor` で、デバイス・モニタ・ページを開き、
**Smart Door** のロックを解除して、**Smart Lamp** をオンにします。
これは、ドロップ・ダウン・リストから適切なコマンドを選択して `send`
ボタンを押すことで実行できます。
デバイスからの測定値のストリームは、同じページに表示されます :

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/door-open.gif)

<a name="subscribing-to-context-changes"></a>

### コンテキスト変更のサブスクライブ

動的なコンテキスト・システムが立ち上がったら、**Draco**
にコンテキストの変更を通知する必要があります。

これは、Orion Context Broker の `/v2/subscription` エンドポイントに
POST リクエストを送信することによって行われます。

-   `fiware-service` と `fiware-servicepath` ヘッダは、これらの設定を使用して
    プロビジョニングされているので、接続された IoT センサからの測定値のみを
    リッスンするためのサブスクリプションをフィルタリングするために
    使用されています
-   リクエストのボディ中の `idPattern` は、Draco
    に、すべてのコンテキスト・データの変更が通知されることを保証します
-   通知 `url` は Draco Listen HTTP Processor の設定された、
    `Base Path と Listening port` と一致する必要があります
-   `throttling` 値は変更がサンプリングされるレートを定義します

#### :two: リクエスト :

```console
curl -iX POST \
  'http://localhost:1026/v2/subscriptions' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "description": "Notify Draco of all context changes",
  "subject": {
    "entities": [
      {
        "idPattern": ".*"
      }
    ]
  },
  "notification": {
    "http": {
      "url": "http://draco:5050/v2/notify"
    }
  },
  "throttling": 5
}'
```

ご覧のとおり、コンテキスト・データを永続化するために使用されるデータベースは、
サブスクリプションの詳細には影響しません。各データベースで同じです。
レスポンスは **201 - Created** になります。

サブスクリプションが作成されている場合は、`/v2/subscriptions`エンドポイントに
GET リクエストを発行することによってそれが起動しているかどうかを確認できます。

#### :three: リクエスト :

```console
curl -X GET \
  'http://localhost:1026/v2/subscriptions/' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /'
```

#### レスポンス :

```json
[
    {
        "id": "5b39d7c866df40ed84284174",
        "description": "Notify Draco of all context changes",
        "status": "active",
        "subject": {
            "entities": [
                {
                    "idPattern": ".*"
                }
            ],
            "condition": {
                "attrs": []
            }
        },
        "notification": {
            "timesSent": 158,
            "lastNotification": "2018-07-02T07:59:21.00Z",
            "attrs": [],
            "http": {
                "url": "http://draco:5050/v2/notify"
            },
            "lastSuccess": "2018-07-02T07:59:21.00Z"
        },
        "throttling": 5
    }
]
```

レスポンスの `notification` セクション内に、
サブスクリプションの健全性を説明するいくつかの追加 `attributes` があります。

サブスクリプションの基準が満たされている場合は、`timesSent` は、`0`
より大きい必要があります。値が0の場合は、サブスクリプションの `subject`
が正しくないか、サブスクリプションが間違った `fiware-service-path`
または `fiware-service` ヘッダで作成されたことを示します。

`lastNotification` は最近のタイムスタンプであるべきです -
そうでない場合、デバイスは定期的にデータを送信していません。
**Smart Door** のロックを解除して **Smart Lamp** をオンにするのを
忘れないでください。

`lastSuccess` は `lastNotification`の日付と一致しなければなりません -
そうでない場合は **Draco** は適切にサブスクリプションを受けていません。
ホスト名とポートが正しいことを確認してください。

最後に、サブスクリプションの `status` が `active` であることを確認します -
有効期限が切れたサブスクリプションは起動しません。

<a name="mongodb----reading-data-from-a-database"></a>

## MongoDB - データベースからデータを読み込む

コマンド・ラインから MongoDB データを読み込むには、
コマンド・ライン・プロンプトを表示するために、`mongo` ツールにアクセスして
`mongo` イメージのインタラクティブなインスタンスを実行する必要があります。

```console
docker run -it --network fiware_default  --entrypoint /bin/bash mongo
```

次に示すように、コマンドラインを使用して実行中の `mongo-db`
データベースにログインできます :

```bash
mongosh --host mongo-db
```

<a name="show-available-databases-on-the-mongodb-server"></a>

### MongoDB サーバ上で利用可能なデータベースを表示

使用可能なデータベースのリストを表示するには、
以下のようにステートメントを実行してください :

#### クエリ :

```
show dbs
```

#### 結果 :

```
admin          0.000GB
iotagentul     0.000GB
local          0.000GB
orion          0.000GB
orion-openiot  0.000GB
sth_openiot    0.000GB
```

結果は、FIWARE プラットフォームによって作成された4つのデータベースと共に、
**MongoDB** によってデフォルトでセットアップされた2つのデータベース
`admin` と `local` を含みます。Orion Context Broker は各 `fiware-service`
に対して2つの別々のデータベース・インスタンスを作成しました。

-   Store エンティティは `fiware-service`を定義せずに作成されたため
    `orion` データベース内に保持されますが、IoTデバイスのエンティティは
    `openiot` `fiware-service` ヘッダを使用して作成され別々に保持されます。
    IoT Agent は IoT センサのデータを `iotagentul` と呼ばれる別の
    **MongoDB** データベースに保持するように初期化されました

Draco を Orion Context Brokerにサブスクリプションした結果、`sth_openiot`
という新しいデータベースが作成されました。履歴コンテキストを保持している
Mongo DB データベースのデフォルト値は、`sth_` プレフィックスとそれに続く
`fiware-service` ヘッダで構成されています。したがって、`sth_openiot` は、
IoT デバイスの履歴コンテキストを保持しています。

<a name="read-historical-context-from-the-server"></a>

### サーバから履歴コンテキストを読み込む

#### クエリ :

```
use sth_openiot
show collections
```

#### 結果 :

```
switched to db sth_openiot

sth_/_Door:001_Door
sth_/_Door:001_Door.aggr
sth_/_Lamp:001_Lamp
sth_/_Lamp:001_Lamp.aggr
sth_/_Motion:001_Motion
sth_/_Motion:001_Motion.aggr
```

`sth_openiot` 内を見ると、一連のテーブルが作成されているのがわかります。
 テーブルの名前は `sth_` プレフィックスとそれに続く `fiware-servicepath`
ヘッダとそれに続くエンティティ ID から成ります。各エンティティに対して
2つのテーブルが作成されます - `.aggr` テーブルは後のチュートリアルで
アクセスされるいくつかの集約データを保持します。
raw データは `.aggr` サフィックスなしでテーブルで見つかります。

履歴データは、各テーブル内のデータを見ることで確認できます。
デフォルトでは、各行には単一の属性のサンプリング値が含まれます。

#### クエリ :

```
db["sth_/_Door:001_Door"].find().limit(10)
```

#### 結果 :

```
{ "_id" : ObjectId("5b1fa48630c49e0012f7635d"), "recvTime" : ISODate("2018-06-12T10:46:30.897Z"), "attrName" : "TimeInstant", "attrType" : "ISO8601", "attrValue" : "2018-06-12T10:46:30.836Z" }
{ "_id" : ObjectId("5b1fa48630c49e0012f7635e"), "recvTime" : ISODate("2018-06-12T10:46:30.897Z"), "attrName" : "close_status", "attrType" : "commandStatus", "attrValue" : "UNKNOWN" }
{ "_id" : ObjectId("5b1fa48630c49e0012f7635f"), "recvTime" : ISODate("2018-06-12T10:46:30.897Z"), "attrName" : "lock_status", "attrType" : "commandStatus", "attrValue" : "UNKNOWN" }
{ "_id" : ObjectId("5b1fa48630c49e0012f76360"), "recvTime" : ISODate("2018-06-12T10:46:30.897Z"), "attrName" : "open_status", "attrType" : "commandStatus", "attrValue" : "UNKNOWN" }
{ "_id" : ObjectId("5b1fa48630c49e0012f76361"), "recvTime" : ISODate("2018-06-12T10:46:30.836Z"), "attrName" : "refStore", "attrType" : "Relationship", "attrValue" : "Store:001" }
{ "_id" : ObjectId("5b1fa48630c49e0012f76362"), "recvTime" : ISODate("2018-06-12T10:46:30.836Z"), "attrName" : "state", "attrType" : "Text", "attrValue" : "CLOSED" }
{ "_id" : ObjectId("5b1fa48630c49e0012f76363"), "recvTime" : ISODate("2018-06-12T10:45:26.368Z"), "attrName" : "unlock_info", "attrType" : "commandResult", "attrValue" : " unlock OK" }
{ "_id" : ObjectId("5b1fa48630c49e0012f76364"), "recvTime" : ISODate("2018-06-12T10:45:26.368Z"), "attrName" : "unlock_status", "attrType" : "commandStatus", "attrValue" : "OK" }
{ "_id" : ObjectId("5b1fa4c030c49e0012f76385"), "recvTime" : ISODate("2018-06-12T10:47:28.081Z"), "attrName" : "TimeInstant", "attrType" : "ISO8601", "attrValue" : "2018-06-12T10:47:28.038Z" }
{ "_id" : ObjectId("5b1fa4c030c49e0012f76386"), "recvTime" : ISODate("2018-06-12T10:47:28.081Z"), "attrName" : "close_status", "attrType" : "commandStatus", "attrValue" : "UNKNOWN" }
```

通常の **MongoDB** のクエリ構文は、適切なフィールドと値をフィルタ処理する
ために使用できます。たとえば、**Motion Sensor** の `id=Motion:001_Motion`
が累積している速度を読み取るには、次のようにクエリを作成します :

#### クエリ :

```
db["sth_/_Motion:001_Motion"].find({attrName: "count"},{_id: 0, attrType: 0, attrName: 0 } ).limit(10)
```

#### 結果 :

```
{ "recvTime" : ISODate("2018-06-12T10:46:18.756Z"), "attrValue" : "8" }
{ "recvTime" : ISODate("2018-06-12T10:46:36.881Z"), "attrValue" : "10" }
{ "recvTime" : ISODate("2018-06-12T10:46:42.947Z"), "attrValue" : "11" }
{ "recvTime" : ISODate("2018-06-12T10:46:54.893Z"), "attrValue" : "13" }
{ "recvTime" : ISODate("2018-06-12T10:47:00.929Z"), "attrValue" : "15" }
{ "recvTime" : ISODate("2018-06-12T10:47:06.954Z"), "attrValue" : "17" }
{ "recvTime" : ISODate("2018-06-12T10:47:15.983Z"), "attrValue" : "19" }
{ "recvTime" : ISODate("2018-06-12T10:47:49.090Z"), "attrValue" : "23" }
{ "recvTime" : ISODate("2018-06-12T10:47:58.112Z"), "attrValue" : "25" }
{ "recvTime" : ISODate("2018-06-12T10:48:28.218Z"), "attrValue" : "29" }
```

MongoDB クライアントを終了して対話モードを終了するには、
次のコマンドを実行します :

```console
exit
```

```console
exit
```

<a name="postgresql---persisting-context-data-into-a-database"></a>

# PostgreSQL - コンテキスト・データをデータベースに永続化

履歴コンテキスト・データを **PostgreSQL** などの代替データベースに
永続化するには、PostgreSQL サーバをホストする追加のコンテナが必要になります。
このデータのためにデフォルトの Docker イメージを使用できます。PostgreSQL
インスタンスは標準で `5432` ポートをリッスンしており、
全体的なアーキテクチャは以下のようになります。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/postgres-draco-tutorial.png)

MongoDB コンテナには Orion Context Broker とIoT Agent に関連するデータを
保持する必要があるため、2つのデータベースを持つシステムができました。

<a name="postgresql---database-server-configuration"></a>

## PostgreSQL - データベース・サーバの設定

```yaml
postgres-db:
    image: postgres:latest
    hostname: postgres-db
    container_name: db-postgres
    expose:
        - "5432"
    ports:
        - "5432:5432"
    networks:
        - default
    environment:
        - "POSTGRES_PASSWORD=password"
        - "POSTGRES_USER=postgres"
        - "POSTGRES_DB=postgres"
```

`postgres-db` コンテナは、単一ポートで待機しています :

-   ポート `5432` は PostgreSQL サーバのデフォルト・ポートです。
    公開されているので、必要に応じて `pgAdmin4` ツールを実行して
    データベースのデータを表示することもできます

次に示すように、`postgres-db` コンテナは環境変数によって駆動されます :

| キー              | 値         | 説明                                        |
| ----------------- | ---------- | ------------------------------------------- |
| POSTGRES_PASSWORD | `password` | PostgreSQL データベース・ユーザのパスワード |
| POSTGRES_USER     | `postgres` | PostgreSQL データベース・ユーザのユーザ名   |
| POSTGRES_DB       | `postgres` | PostgreSQL データベースの名前               |

> :information_source: **注** : このようにプレーン・テキストの環境変数で
> ユーザ名とパスワードを渡すことはセキュリティ上のリスクです。
> これはチュートリアルでは受け入れ可能な方法ですが、実稼働環境では、
> [Docker Secrets](https://blog.docker.com/2017/02/docker-secrets-management/)
> を適用することでこのリスクを回避できます。

<a name="postgresql---draco-configuration"></a>

## PostgreSQL - Draco の設定

```yaml
draco:
    image: ging/fiware-draco:1.1.0
    container_name: draco
    depends_on:
        - postgres-db
    environment:
        - NIFI_WEB_HTTP_PORT=9090
    ports:
        - "9090:9090"
        - "5050:5050"
    healthcheck:
        test: curl --fail -s http://localhost:9090/nifi-api/system-diagnostics || exit 1
```

`draco` コンテナは、2つのポートでリッスンしています :

-   Draco の サブスクリプション・ポート -` 5050` は、サービスが
    Orion Context Broker からの通知をリッスンするポートです
-   Draco の Web インターフェース - `9090` は、プロセッサの設定のためだけに
    公開されています

<a name="postgresql---start-up"></a>

## PostgreSQL - 起動

**PostgreSQL** データベースでシステムを起動するには、次のコマンドを実行します:

```console
./services postgres
```

次に、ブラウザに移動し、このURL `http://localhost:9090/nifi` を使用して Draco を開きます。

NiFi GUI の上部にあるコンポーネント・ツールバーに行き、
テンプレート・アイコンを見つけて、Draco ユーザ・スペースの中に
ドラッグ＆ドロップします。この時点で、利用可能なすべてのテンプレートの
リストとともにポップアップが表示されます。テンプレート "POSTGRESQL-TUTORIAL"
を選択してください。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/postgres-tutorial-template.png)

プロセッサを起動する前に、PostgreSQL のパスワードを設定し、”DBCConnectionPool"
コントローラを有効にする必要があります。そのためには、次の指示に従ってください :

1.  Draco GUI ユーザ・スペースの任意の部分を右クリックしてから
    "configure" をクリックします
    ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step1.png)

2.  "Controller Services" タブに移動します。この時点で、コントローラのリストが
    表示されるはずです。"DBCConnectionPool" コントローラを見つけます

3.  "DBCPConnectionPool" の "configuration" ボタンをクリックしてください
    ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step2.png)

4.  "controller Properties" タブに移動し、パフワード・フィールドに "password"
    と入力してから変更を適用します
    ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/controller-postgresql.png)

5.  雷のアイコンをクリックしてプロセッサを有効にしてから "enable"
    をクリックし、コントローラの "configuration" ページを閉じます

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step4.png)

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step5.png)

6.  すべてのプロセッサを選択し
    (shift キーを押しながらすべてのプロセッサをクリック)、
    スタート・ボタンをクリックして起動します。
    これで、各プロセッサのステータス・アイコンが赤から緑に変わりました

<a name="checking-the-draco-service-health-1"></a>

### Draco サービスの健全性をチェック

Draco が起動したら、公開された draco ポート `/nifi-api/system-diagnostics` に
HTTP リクエストを送信することでステータスを確認できます。
レスポンスが空白の場合は、通常 Draco が実行されていないか、
別のポートでリッスンしているためです。

#### :one: リクエスト :

```console
curl -X GET \
  'http://localhost:9090/nifi-api/system-diagnostics'
```

#### レスポンス :

レスポンスは次のようになります :

```json
{
    "systemDiagnostics": {
        "aggregateSnapshot": {
            "totalNonHeap": "value",
            "totalNonHeapBytes": 0,
            "usedNonHeap": "value",
            "usedNonHeapBytes": 0,
            "freeNonHeap": "value",
            "freeNonHeapBytes": 0,
            "maxNonHeap": "value",
            "maxNonHeapBytes": 0,
            "nonHeapUtilization": "value",
            "totalHeap": "value",
            "totalHeapBytes": 0,
            "usedHeap": "value",
            "usedHeapBytes": 0,
            "freeHeap": "value",
            "freeHeapBytes": 0,
            "maxHeap": "value",
            "maxHeapBytes": 0,
            "heapUtilization": "value",
            "availableProcessors": 0,
            "processorLoadAverage": 0.0,
            "totalThreads": 0,
            "daemonThreads": 0,
            "uptime": "value",
            "flowFileRepositoryStorageUsage": {},
            "contentRepositoryStorageUsage": [{}],
            "provenanceRepositoryStorageUsage": [{}],
            "garbageCollection": [{}],
            "statsLastRefreshed": "value",
            "versionInfo": {}
        },
        "nodeSnapshots": [{}]
    }
}
```

> **トラブルシューティング** : 回答が空白の場合はどうしますか？

> dockerコンテナが実行されていることを確認するには
> ```bash
> docker ps
> ```
> いくつかのコンテナが動いているのが見えるでしょう。dracoが実行されていない場合は、必要に応じてコンテナを再起動できます。

<a name="generating-context-data-1"></a>

### コンテキスト・データの生成

このチュートリアルでは、コンテキストが定期的に更新されているシステムを
監視する必要があります。これを行うためにダミー IoT センサを使用できます。
`http://localhost:3000/device/monitor` で、デバイス・モニタ・ページを開き、
**Smart Door** のロックを解除して、**Smart Lamp** をオンにします。
これは、ドロップ・ダウン・リストから適切なコマンドを選択して `send`
ボタンを押すことで実行できます。
デバイスからの測定値のストリームは、同じページに表示されます :

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/door-open.gif)

<a name="subscribing-to-context-changes-1"></a>

### コンテキスト変更のサブスクライブ

動的なコンテキスト・システムが立ち上がったら、**Draco**
にコンテキストの変更を通知する必要があります。

これは、Orion Context Broker の `/v2/subscription` エンドポイントに
POST リクエストを送信することによって行われます。

-   `fiware-service` と `fiware-servicepath` ヘッダは、これらの設定を使用して
    プロビジョニングされているので、接続された IoT センサからの測定値のみを
    リッスンするためのサブスクリプションをフィルタリングするために
    使用されています
-   リクエストのボディ中の `idPattern` は、Draco
    に、すべてのコンテキスト・データの変更が通知されることを保証します
-   通知 `url` は Draco Listen HTTP Processor の設定された、
    `Base Path と Listening port` と一致する必要があります
-   `throttling` 値は変更がサンプリングされるレートを定義します

#### :five: リクエスト :

```console
curl -iX POST \
  'http://localhost:1026/v2/subscriptions' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "description": "Notify Draco of all context changes",
  "subject": {
    "entities": [
      {
        "idPattern": ".*"
      }
    ]
  },
  "notification": {
    "http": {
      "url": "http://draco:5050/v2/notify"
    }
  },
  "throttling": 5
}'
```

ご覧のとおり、コンテキスト・データを永続化するために使用されるデータベースは、
サブスクリプションの詳細には影響しません。各データベースで同じです。
レスポンスは **201 - Created** になります。

<a name="postgresql---reading-data-from-a-database"></a>

## PostgreSQL - データベースからデータを読み込む

コマンドラインから PostgreSQL データを読み込むには、`postgres`
クライアントにアクセスする必要があります。これを行うには、次のように
コネクション文字列を指定して `postgresql-client`
イメージの対話型インスタンスを実行してコマンドライン・プロンプトを取得します :

```console
docker run -it --rm  --network fiware_default jbergknoff/postgresql-client \
   postgresql://postgres:password@postgres-db:5432/postgres
```

<a name="show-available-databases-on-the-postgresql-server"></a>

### PostgreSQL server サーバ上で利用可能なデータベースを表示

使用可能なデータベースのリストを表示するには、
次のようにステートメントを実行してください :

#### クエリ :

```
\list
```

#### 結果 :
```
   Name    |  Owner   | Encoding |  Collate   |   Ctype    |   Access privileges
-----------+----------+----------+------------+------------+-----------------------
 postgres  | postgres | UTF8     | en_US.utf8 | en_US.utf8 |
 template0 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
 template1 | postgres | UTF8     | en_US.utf8 | en_US.utf8 | =c/postgres          +
           |          |          |            |            | postgres=CTc/postgres
(3 rows)
```

結果は、2つのテンプレートデータベース `template0` と `template1` と
docker コンテナが起動された時の `postgres` データベースの設定を含みます。

使用可能なスキーマのリストを表示するには、
次のようにステートメントを実行してください :

#### クエリ :

```
\dn
```

#### 結果 :

```
  List of schemas
  Name   |  Owner
---------+----------
 openiot | postgres
 public  | postgres
(2 rows)
```

Draco の Orion Context Broker へのサブスクリプションの結果として、`openiot`
という新しいスキーマが作成されました。スキーマの名前は `fiware-service`
ヘッダと一致します。したがって、IoT デバイスの履歴コンテキストが保持されます。

<a name="read-historical-context-from-the-postgresql-server"></a>

### PostgreSQL server サーバから履歴コンテキストを読み込む

ネットワーク内で Docker コンテナを実行すると、実行中のデータベースに関する情報を
取得することができます。

#### クエリ :

```sql
SELECT table_schema,table_name
FROM information_schema.tables
WHERE table_schema ='openiot'
ORDER BY table_schema,table_name;
```

#### 結果 :

```
 table_schema |    table_name
--------------+-------------------
 openiot      | door_001_door
 openiot      | lamp_001_lamp
 openiot      | motion_001_motion
(3 rows)
```

`table_schema` はコンテキスト・データとともに提供される
`fiware-service` ヘッダと一致します :

テーブル内のデータを読み取るには、次に示すように select 文を実行します :

#### クエリ :

```sql
SELECT * FROM openiot.motion_001_motion limit 10;
```

#### 結果 :

```
  recvtimets   |         recvtime         | fiwareservicepath |  entityid  | entitytype |  attrname   |   attrtype   |        attrvalue         |                                    attrmd
---------------+--------------------------+-------------------+------------+------------+-------------+--------------+--------------------------+------------------------------------------------------------------------------
 1528803005491 | 2018-06-12T11:30:05.491Z | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:30:05.423Z | []
 1528803005491 | 2018-06-12T11:30:05.491Z | /                 | Motion:001 | Motion     | count       | Integer      | 7                        | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:30:05.423Z"}]
 1528803005491 | 2018-06-12T11:30:05.491Z | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:30:05.423Z"}]
 1528803035501 | 2018-06-12T11:30:35.501Z | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:30:35.480Z | []
 1528803035501 | 2018-06-12T11:30:35.501Z | /                 | Motion:001 | Motion     | count       | Integer      | 10                       | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:30:35.480Z"}]
 1528803035501 | 2018-06-12T11:30:35.501Z | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:30:35.480Z"}]
 1528803041563 | 2018-06-12T11:30:41.563Z | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:30:41.520Z | []
 1528803041563 | 2018-06-12T11:30:41.563Z | /                 | Motion:001 | Motion     | count       | Integer      | 12                       | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:30:41.520Z"}]
 1528803041563 | 2018-06-12T11:30:41.563Z | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:30:41.520Z"}]
 1528803047545 | 2018-06-12T11:30:47.545Z | /
```

通常の **PostgreSQL** のクエリ構文は適切なフィールドと値をフィルタするために
使用できます。たとえば、**Motion Sensor** `id=Motion:001_Motion`
が累積している速度を読み取るには、次のようにクエリを作成します :

#### クエリ :

```sql
SELECT recvtime, attrvalue FROM openiot.motion_001_motion WHERE attrname ='count'  limit 10;
```

#### 結果 :

```
         recvtime         | attrvalue
--------------------------+-----------
 2018-06-12T11:30:05.491Z | 7
 2018-06-12T11:30:35.501Z | 10
 2018-06-12T11:30:41.563Z | 12
 2018-06-12T11:30:47.545Z | 13
 2018-06-12T11:31:02.617Z | 15
 2018-06-12T11:31:32.718Z | 20
 2018-06-12T11:31:38.733Z | 22
 2018-06-12T11:31:50.780Z | 24
 2018-06-12T11:31:56.825Z | 25
 2018-06-12T11:31:59.790Z | 26
(10 rows)
```

Postgres クライアントを終了して対話モードを終了するには、
以下を実行してください :

```console
\q
```

その後、コマンドラインに戻ります。

<a name="mysql---persisting-context-data-into-a-database"></a>

# MySQL - コンテキスト・データをデータベースに永続化

同様に、履歴コンテキスト・データを **MySQL** に保存するには、MySQL サーバを
ホストする追加のコンテナが必要です。この場合も、このデータのデフォルトの
Docker イメージを使用できます。MySQL インスタンスは標準で `3306`
ポートをリッスンしており、全体的なアーキテクチャは以下のようになります。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/mysql-draco-tutorial.png)

MongoDB コンテナは Orion Context Broker と IoT Agent に関連するデータを
保持する必要があるため、ここでも2つのデータベースを持つシステムがあります。

<a name="mysql---database-server-configuration"></a>

## MySQL - データベース・サーバの設定

```yaml
mysql-db:
    restart: always
    image: mysql:5.7
    hostname: mysql-db
    container_name: db-mysql
    expose:
        - "3306"
    ports:
        - "3306:3306"
    networks:
        - default
    environment:
        - "MYSQL_ROOT_PASSWORD=123"
        - "MYSQL_ROOT_HOST=%"
```

> :information_source: **注** : このようにプレーン・テキストの環境変数で
> ユーザ名とパスワードを渡すことはセキュリティ上のリスクです。
> これはチュートリアルでは受け入れ可能な方法ですが、実稼働環境では、
> [Docker Secrets](https://blog.docker.com/2017/02/docker-secrets-management/)
> を適用することでこのリスクを回避できます。

`mysql-db` コンテナは、単一ポートで待機しています :

-   ポート `3306` は MySQL サーバのデフォルトポートです。必要に応じて
    他のデータベース・ツールを実行してデータを表示することもできます

次に示すように、`mysql-db` コンテナは環境変数によって制御されます :

| キー                | 値         | 説明                                                                                                                                                                                                 |
| ------------------- | ---------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| MYSQL_ROOT_PASSWORD | `123`.     | MySQLの `root` アカウントに設定されているパスワードを指定します                                                                                                                                       |
| MYSQL_ROOT_HOST     | `postgres` | デフォルトでは、MySQL は `root'@'localhost` アカウントを作成します。このアカウントはコンテナ内からのみ接続できます。この環境変数を設定すると、他のホストからの root 接続が許可されます                |

<a name="mysql---draco-configuration"></a>

## MySQL - Draco の設定

```yaml
draco:
    image: ging/fiware-draco:1.1.0
    container_name: draco
    depends_on:
        - mysql-db
    environment:
        - NIFI_WEB_HTTP_PORT=9090
    ports:
        - "9090:9090"
        - "5050:5050"
    healthcheck:
        test: curl --fail -s http://localhost:9090/nifi-api/system-diagnostics || exit 1
```

`draco` コンテナは、2つのポートでリッスンしています :

-   Draco の サブスクリプション・ポート -` 5050` は、サービスが
    Orion Context Broker からの通知をリッスンするポートです
-   Draco の Web インターフェース - `9090` は、プロセッサの設定のためだけに
    公開されています

<a name="mysql---start-up"></a>

## MySQL - 起動

システムで **MySQL** データベースを起動するには、次のコマンドを実行します :

```console
./services mysql
```

次に、ブラウザに移動し、このURL `http://localhost:9090/nifi` を使用して Draco を開きます。

NiFi GUI の上部にあるコンポーネント・ツールバーに行き、
テンプレート・アイコンを見つけて、Draco ユーザ・スペースの中に
ドラッグ＆ドロップします。この時点で、利用可能なすべてのテンプレートの
リストとともにポップアップが表示されます。テンプレート "MYSQL-TUTORIAL"
を選択してください。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/draco-template1.png)

プロセッサを起動する前に、MySQL のパスワードを設定し、"DBCConnectionPool"
コントローラを有効にする必要があります。そのためには、指示に従ってください。

1.  Draco GUI ユーザ・スペースの任意の部分を右クリックしてから "configure"
    をクリックします
    ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step1.png)

2.  "Controller Services" タブに移動します。この時点で、コントローラのリストが
    表示されるはずです。"DBCConnectionPool" コントローラが見つかります

3.  "DBCPConnectionPool"の "configuration" ボタンをクリックしてください
    ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step2.png)

4.  "controller Properties" タブに移動し、パスワード・フィールドに "123"
    を入力してから、変更を適用します
    ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step3.png)

5.  雷のアイコンをクリックしてプロセッサを有効にしてから "enable" をクリックし、
    コントローラの "configuration" ページを閉じます
    ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step4.png)

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step5.png)

6.  すべてのプロセッサを選択し
    (shift キーを押しながらすべてのプロセッサをクリック)、
    スタート・ボタンをクリックして起動します。
    これで、各プロセッサのステータス・アイコンが赤から緑に変わりました

<a name="checking-the-draco-service-health-2"></a>

### Draco サービスの健全性をチェック

Draco が起動したら、公開された draco ポート `/nifi-api/system-diagnostics` に
HTTP リクエストを送信することでステータスを確認できます。
レスポンスが空白の場合は、通常 Draco が実行されていないか、
別のポートでリッスンしているためです。

#### :one: リクエスト :

```console
curl -X GET \
  'http://localhost:9090/system-diagnostics'
```

#### レスポンス :

レスポンスは次のようになります :

```json
{
    "systemDiagnostics": {
        "aggregateSnapshot": {
            "totalNonHeap": "value",
            "totalNonHeapBytes": 0,
            "usedNonHeap": "value",
            "usedNonHeapBytes": 0,
            "freeNonHeap": "value",
            "freeNonHeapBytes": 0,
            "maxNonHeap": "value",
            "maxNonHeapBytes": 0,
            "nonHeapUtilization": "value",
            "totalHeap": "value",
            "totalHeapBytes": 0,
            "usedHeap": "value",
            "usedHeapBytes": 0,
            "freeHeap": "value",
            "freeHeapBytes": 0,
            "maxHeap": "value",
            "maxHeapBytes": 0,
            "heapUtilization": "value",
            "availableProcessors": 0,
            "processorLoadAverage": 0.0,
            "totalThreads": 0,
            "daemonThreads": 0,
            "uptime": "value",
            "flowFileRepositoryStorageUsage": {},
            "contentRepositoryStorageUsage": [{}],
            "provenanceRepositoryStorageUsage": [{}],
            "garbageCollection": [{}],
            "statsLastRefreshed": "value",
            "versionInfo": {}
        },
        "nodeSnapshots": [{}]
    }
}
```

> **トラブルシューティング** : レスポンスが空白の場合はどうしますか？
>
> -   docker コンテナが実行されていることを確認するには
>
> ```bash
> docker ps
> ```
>
> いくつかのコンテナが動いているのが確認できます。
> draco が実行されていない場合は、必要に応じてコンテナを再起動できます。

<a name="generating-context-data-2"></a>

### コンテキスト・データの生成

このチュートリアルでは、コンテキストが定期的に更新されているシステムを
監視する必要があります。これを行うためにダミー IoT センサを使用できます。
`http://localhost:3000/device/monitor` で、デバイス・モニタ・ページを開き、
**Smart Door** のロックを解除して、**Smart Lamp** をオンにします。
これは、ドロップ・ダウン・リストから適切なコマンドを選択して `send`
ボタンを押すことで実行できます。
デバイスからの測定値のストリームは、同じページに表示されます :

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/door-open.gif)

<a name="subscribing-to-context-changes-2"></a>

### コンテキスト変更のサブスクライブ

動的なコンテキスト・システムが立ち上がったら、**Draco**
にコンテキストの変更を通知する必要があります。

これは、Orion Context Broker の `/v2/subscription` エンドポイントに
POST リクエストを送信することによって行われます。

-   `fiware-service` と `fiware-servicepath` ヘッダは、これらの設定を使用して
    プロビジョニングされているので、接続された IoT センサからの測定値のみを
    リッスンするためのサブスクリプションをフィルタリングするために
    使用されています
-   リクエストのボディ中の `idPattern` は、Draco
    に、すべてのコンテキスト・データの変更が通知されることを保証します
-   通知 `url` は Draco Listen HTTP Processor の設定された、
    `Base Path と Listening port` と一致する必要があります
-   `throttling` 値は変更がサンプリングされるレートを定義します

#### :seven: リクエスト :

```console
curl -iX POST \
  'http://localhost:1026/v2/subscriptions' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "description": "Notify Draco of all context changes",
  "subject": {
    "entities": [
      {
        "idPattern": ".*"
      }
    ]
  },
  "notification": {
    "http": {
      "url": "http://draco:5050/v2/notify"
    }
  },
  "throttling": 5
}'
```

ご覧のとおり、コンテキスト・データを永続化するために使用されるデータベースは、
サブスクリプションの詳細には影響しません。各データベースで同じです。
レスポンスは **201 - Created** になります。

<a name="mysql---reading-data-from-a-database"></a>

## MySQL - データベースからデータを読み込む

コマンドラインから MySQL データを読み込むには、 `mysql` クライアントに
アクセスする必要があります。これを行うには、コマンドラインプロンプトを
表示するために、表示されているようにコネクション文字列を指定して `mysql`
のイメージの対話型インスタンスを実行します :

```console
docker exec -it  db-mysql mysql -h mysql-db -P 3306  -u root -p123
```

<a name="show-available-databases-on-the-mysql-server"></a>

### MySQL サーバ上で利用可能なデータベースを表示

使用可能なデータベースのリストを表示するには、
以下のようにステートメントを実行してください :

#### クエリ :

```sql
SHOW DATABASES;
```

#### 結果 :

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| openiot            |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

使用可能なスキーマのリストを表示するには、
以下のようにステートメントを実行してください :

#### クエリ :

```sql
SHOW SCHEMAS;
```

#### 結果 :

```
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| openiot            |
| performance_schema |
| sys                |
+--------------------+
5 rows in set (0.00 sec)
```

Draco の Orion Context Broker へのサブスクリプションの結果として、`openiot`
と呼ばれる新しいスキーマが作成されました。スキーマの名前は `fiware-service`
ヘッダと一致します - それゆえ` openiot` は IoT
デバイスの履歴コンテキストを保持します。

<a name="read-historical-context-from-the-mysql-server"></a>

### MySQL サーバから履歴コンテキストを読み込む

ネットワーク内で Docker コンテナを実行すると、
実行中のデータベースに関する情報を取得することができます。

#### クエリ :

```sql
SHOW tables FROM openiot;
```

#### 結果 :

```
 table_schema |    table_name
--------------+-------------------
 openiot      | door_001_door
 openiot      | lamp_001_lamp
 openiot      | motion_001_motion
(3 rows)
```

`table_schema` はコンテキスト・データとともに提供される
`fiware-service` ヘッダと一致します :

テーブル内のデータを読み取るには、次に示すように select 文を実行します :

#### クエリ :

```sql
SELECT * FROM openiot.Motion_001_Motion limit 10;
```

#### 結果 :

```
+---------------+-------------------------+-------------------+------------+------------+-------------+--------------+--------------------------+------------------------------------------------------------------------------+
| recvTimeTs    | recvTime                | fiwareServicePath | entityId   | entityType | attrName    | attrType     | attrValue                | attrMd                                                                       |
+---------------+-------------------------+-------------------+------------+------------+-------------+--------------+--------------------------+------------------------------------------------------------------------------+
| 1528804397955 | 2018-06-12T11:53:17.955 | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:53:17.923Z | []                                                                           |
| 1528804397955 | 2018-06-12T11:53:17.955 | /                 | Motion:001 | Motion     | count       | Integer      | 3                        | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:53:17.923Z"}] |
| 1528804397955 | 2018-06-12T11:53:17.955 | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:53:17.923Z"}] |
| 1528804403954 | 2018-06-12T11:53:23.954 | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:53:23.928Z | []                                                                           |
| 1528804403954 | 2018-06-12T11:53:23.954 | /                 | Motion:001 | Motion     | count       | Integer      | 5                        | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:53:23.928Z"}] |
| 1528804403954 | 2018-06-12T11:53:23.954 | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:53:23.928Z"}] |
| 1528804409970 | 2018-06-12T11:53:29.970 | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:53:29.948Z | []                                                                           |
| 1528804409970 | 2018-06-12T11:53:29.970 | /                 | Motion:001 | Motion     | count       | Integer      | 7                        | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:53:29.948Z"}] |
| 1528804409970 | 2018-06-12T11:53:29.970 | /                 | Motion:001 | Motion     | refStore    | Relationship | Store:001                | [{"name":"TimeInstant","type":"ISO8601","value":"2018-06-12T11:53:29.948Z"}] |
| 1528804446083 | 2018-06-12T11:54:06.83  | /                 | Motion:001 | Motion     | TimeInstant | ISO8601      | 2018-06-12T11:54:06.062Z | []                                                                           |
+---------------+-------------------------+-------------------+------------+------------+-------------+--------------+--------------------------+------------------------------------------------------------------------------+
```

通常の **MySQL** クエリ構文は適切なフィールドと値をフィルタするために
使用できます。たとえば、**Motion Sensor**  `id=Motion:001_Motion`
が累積している速度を読み取るには、次のようにクエリを作成します :

#### クエリ :

```sql
SELECT recvtime, attrvalue FROM openiot.Motion_001_Motion WHERE attrname ='count' LIMIT 10;
```

#### 結果 :

```
+-------------------------+-----------+
| recvtime                | attrvalue |
+-------------------------+-----------+
| 2018-06-12T11:53:17.955 | 3         |
| 2018-06-12T11:53:23.954 | 5         |
| 2018-06-12T11:53:29.970 | 7         |
| 2018-06-12T11:54:06.83  | 12        |
| 2018-06-12T11:54:12.132 | 13        |
| 2018-06-12T11:54:24.177 | 14        |
| 2018-06-12T11:54:36.196 | 16        |
| 2018-06-12T11:54:42.195 | 18        |
| 2018-06-12T11:55:24.300 | 23        |
| 2018-06-12T11:55:30.350 | 25        |
+-------------------------+-----------+
10 rows in set (0.00 sec)
```

MySQL クライアントを終了して対話モードを終了するには、次のコマンドを実行します :

```console
\q
```

その後、コマンド・ラインに戻ります。

<a name="multi-agent---persisting-context-data-into-a-multiple-databases"></a>
# マルチ・エージェント - 複数のデータベースへのコンテキスト・データの永続化

同時に複数のデータベースにデータを投入するように Draco
を設定することも可能です。前の3つの例のアーキテクチャを組み合わせて、
データを複数のシンクに格納するように Draco を設定することができます。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/multiple-draco-tutorial.png)

これで、データ永続化用に PostgreSQL と MySQL、Orion Context Broker および
IoT Agent に関連するデータを保持するためとデータ永続化用に MongoDB の
3つのデータベースを持つシステムができました。

<a name="multi-agent---draco-configuration-for-multiple-databases"></a>

## マルチ・エージェント - 複数のデータベースのための Draco 設定

```yaml
draco:
    image: ging/fiware-draco:1.1.0
    container_name: draco
    depends_on:
        - mysql-db
        - mongo-db
        - postgres-db
    environment:
        - NIFI_WEB_HTTP_PORT=9090
    ports:
        - "9090:9090"
        - "5050:5050"
    healthcheck:
        test: curl --fail -s http://localhost:9090/nifi-api/system-diagnostics || exit 1
```

`draco` コンテナは、2つのポートでリッスンしています :

-   Draco の サブスクリプション・ポート -` 5050` は、サービスが
    Orion Context Broker からの通知をリッスンするポートです
-   Draco の Web インターフェース - `9090` は、プロセッサの設定のためだけに
    公開されています

<a name="multi-agent---start-up"></a>

## マルチ・エージェント - 起動

**複数**のデータベースでシステムを起動するには、次のコマンドを実行します :

```console
./services multiple
```

次に、ブラウザに移動し、このURL `http://localhost:9090/nifi` を使用して Draco を開きます。

NiFi GUI の上部にあるコンポーネント・ツールバーに行き、
テンプレート・アイコンを見つけて、Draco ユーザ・スペースの中に
ドラッグ＆ドロップします。この時点で、利用可能なすべてのテンプレートの
リストとともにポップアップが表示されます。テンプレート "MULTIPLE-SINKS-TUTORIAL"
を選択してください。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/multiple-tutorial-template.png)

そして、MySQL と PostgreSQLの各コネクション・コントローラ "DBCPConnectionPool"
にパスワードを設定するためのプロセスを繰り返します。

すべてのプロセッサを選択し
(shift キーを押しながらすべてのプロセッサをクリック)、
スタート・ボタンをクリックして起動します。
これで、各プロセッサのステータス・アイコンが赤から緑に変わりました。

<a name="checking-the-draco-service-health-3"></a>

### Draco サービスの健全性をチェック

Draco が起動したら、公開された draco ポート `/nifi-api/system-diagnostics` に
HTTP リクエストを送信することでステータスを確認できます。
レスポンスが空白の場合は、通常 Draco が実行されていないか、
別のポートでリッスンしているためです。

#### :one: リクエスト :

```console
curl -X GET \
  'http://localhost:9090/nifi-api/system-diagnostics'
```

#### レスポンス :

レスポンスは次のようになります :

```json
{
    "systemDiagnostics": {
        "aggregateSnapshot": {
            "totalNonHeap": "value",
            "totalNonHeapBytes": 0,
            "usedNonHeap": "value",
            "usedNonHeapBytes": 0,
            "freeNonHeap": "value",
            "freeNonHeapBytes": 0,
            "maxNonHeap": "value",
            "maxNonHeapBytes": 0,
            "nonHeapUtilization": "value",
            "totalHeap": "value",
            "totalHeapBytes": 0,
            "usedHeap": "value",
            "usedHeapBytes": 0,
            "freeHeap": "value",
            "freeHeapBytes": 0,
            "maxHeap": "value",
            "maxHeapBytes": 0,
            "heapUtilization": "value",
            "availableProcessors": 0,
            "processorLoadAverage": 0.0,
            "totalThreads": 0,
            "daemonThreads": 0,
            "uptime": "value",
            "flowFileRepositoryStorageUsage": {},
            "contentRepositoryStorageUsage": [{}],
            "provenanceRepositoryStorageUsage": [{}],
            "garbageCollection": [{}],
            "statsLastRefreshed": "value",
            "versionInfo": {}
        },
        "nodeSnapshots": [{}]
    }
}
```

> **トラブルシューティング** : レスポンスが空白の場合はどうしますか？
>
> -   docker コンテナが実行されていることを確認するには
>
> ```bash
> docker ps
> ```
>
> いくつかのコンテナが動いているのが確認できます。
> draco が実行されていない場合は、必要に応じてコンテナを再起動できます。

<a name="generating-context-data-3"></a>

### コンテキスト・データの生成

このチュートリアルでは、コンテキストが定期的に更新されているシステムを
監視する必要があります。これを行うためにダミー IoT センサを使用できます。
`http://localhost:3000/device/monitor` で、デバイス・モニタ・ページを開き、
**Smart Door** のロックを解除して、**Smart Lamp** をオンにします。
これは、ドロップ・ダウン・リストから適切なコマンドを選択して `send`
ボタンを押すことで実行できます。
デバイスからの測定値のストリームは、同じページに表示されます :

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/door-open.gif)

<a name="subscribing-to-context-changes-3"></a>

### コンテキスト変更のサブスクライブ

動的なコンテキスト・システムが立ち上がったら、**Draco**
にコンテキストの変更を通知する必要があります。

これは、Orion Context Broker の `/v2/subscription` エンドポイントに
POST リクエストを送信することによって行われます。

-   `fiware-service` と `fiware-servicepath` ヘッダは、これらの設定を使用して
    プロビジョニングされているので、接続された IoT センサからの測定値のみを
    リッスンするためのサブスクリプションをフィルタリングするために
    使用されています
-   リクエストのボディ中の `idPattern` は、Draco
    に、すべてのコンテキスト・データの変更が通知されることを保証します
-   通知 `url` は Draco Listen HTTP Processor の設定された、
    `Base Path と Listening port` と一致する必要があります
-   `throttling` 値は変更がサンプリングされるレートを定義します

#### :nine: リクエスト :

```console
curl -iX POST \
  'http://localhost:1026/v2/subscriptions' \
  -H 'Content-Type: application/json' \
  -H 'fiware-service: openiot' \
  -H 'fiware-servicepath: /' \
  -d '{
  "description": "Notify Draco of all context changes",
  "subject": {
    "entities": [
      {
        "idPattern": ".*"
      }
    ]
  },
  "notification": {
    "http": {
      "url": "http://draco:5050/v2/notify"
    }
  },
  "throttling": 5
}'
```

ご覧のとおり、コンテキスト・データを永続化するために使用されるデータベースは、
サブスクリプションの詳細には影響しません。各データベースで同じです。
レスポンスは **201 - Created** になります。

<a name="multi-agent---reading-persisted-data"></a>

## マルチ・エージェント - 永続化データの読み込み

データベースから永続データを読み取るには、このチュートリアルの前のセクションを参照してください。

<a name="next-steps"></a>

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか
？このシリーズ
の[他のチュートリアル](https://www.letsfiware.jp/fiware-tutorials)を読むことで見
つけることができます

---

## License

[MIT](LICENSE) © 2018-2023 FIWARE Foundation e.V.
