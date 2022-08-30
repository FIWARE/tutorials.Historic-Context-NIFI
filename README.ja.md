# Data Persistence (NIFI)[<img src="https://img.shields.io/badge/NGSI-LD-d6604d.svg" width="90"  align="left" />]("https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.03.01_60/gs_cim009v010301p.pdf)[<img src="https://fiware.github.io/tutorials.CRUD-Operations/img/fiware.png" align="left" width="162">](https://www.fiware.org/)<br/>

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Historic-Context-NIFI.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://nexus.lab.fiware.org/repository/raw/public/badges/stackoverflow/fiware.svg)](https://stackoverflow.com/questions/tagged/fiware)
<br/> [![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)


このチュートリアルは、[FIWARE Draco](https://fiware-draco.readthedocs.io/en/latest/) の概要です。これは、
コンテキストの履歴ビューを作成する [Apache NIFI](https://nifi.apache.org) を使用してコンテキスト・データを
サードパーティ・データベースに永続化するために使用される代替の汎用イネーブラーです。
[以前のチュートリアル](https://github.com/FIWARE/tutorials.Historic-Context-Flume) と同じ方法で、ダミー
IoT センサをアクティブにして、それらのセンサからの測定値をデータベースに保持し、さらに分析します。

チュートリアルでは全体で [cUrl](https://ec.haxx.se/) コマンドを使用しますが、
 [Postman ドキュメント](https://fiware.github.io/tutorials.Historic-Context-NIFI/)としても利用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/9658043920d9be43914a)
[![Open in Gitpod](https://gitpod.io/button/open-in-gitpod.svg)](https://gitpod.io/#https://github.com/FIWARE/tutorials.Historic-Context-NIFI/tree/NGSI-LD)

## コンテンツ

<details>
<summary><strong>詳細</strong></summary>

-   [Apache NIFI を使用したデータの永続化](#data-persistence-using-apache-nifi)
-   [アーキテクチャ](#architecture)
-   [前提条件](#prerequisites)
    -   [Docker と Docker Compose](#docker-and-docker-compose)
    -   [Cygwin for Windows](#cygwin-for-windows)
-   [起動](#start-up)
-   [MongoDB - コンテキスト・データをデータベースに永続化](#mongodb---persisting-context-data-into-a-database)
    -   [MongoDB - データベース・サーバの構成](#mongodb---database-server-configuration)
    -   [MongoDB - Draco の構成](#mongodb---draco-configuration)
    -   [MongoDB - 起動](#mongodb---start-up)
        -   [Draco サービスの状態を確認](#checking-the-draco-service-health)
        -   [コンテキスト・データの生成](#generating-context-data)
        -   [コンテキスト変更のサブスクライブ](#subscribing-to-context-changes)
    -   [MongoDB - データベースからのデータの読み取り](#mongodb----reading-data-from-a-database)
        -   [MongoDB サーバで利用可能なデータベースを表示](#show-available-databases-on-the-mongodb-server)
        -   [サーバーから履歴コンテキストを読み取り](#read-historical-context-from-the-server)
-   [PostgreSQL - コンテキスト・データをデータベースに永続化](#postgresql---persisting-context-data-into-a-database)
    -   [PostgreSQL - データベース・サーバの構成](#postgresql---database-server-configuration)
    -   [PostgreSQL - Draco の構成](#postgresql---draco-configuration)
    -   [PostgreSQL - 起動](#postgresql---start-up)
        -   [Draco サービスの状態を確認](#checking-the-draco-service-health-1)
        -   [コンテキスト・データの生成](#generating-context-data-1)
        -   [コンテキスト変更のサブスクライブ](#subscribing-to-context-changes-1)
    -   [PostgreSQL - データベースからのデータの読み取り](#postgresql---reading-data-from-a-database)
        -   [PostgreSQL サーバで利用可能なデータベースを表示](#show-available-databases-on-the-postgresql-server)
        -   [PostgreSQL サーバから履歴コンテキストの読み取り](#read-historical-context-from-the-postgresql-server)
-   [MySQL - コンテキスト・データをデータベースに永続化](#mysql---persisting-context-data-into-a-database)
    -   [MySQL - データベース・サーバの構成](#mysql---database-server-configuration)
    -   [MySQL - Draco の構成](#mysql---draco-configuration)
    -   [MySQL - 起動](#mysql---start-up)
        -   [Draco サービスの状態を確認](#checking-the-draco-service-health-2)
        -   [コンテキスト・データの生成](#generating-context-data-2)
        -   [コンテキスト変更のサブスクライブ](#subscribing-to-context-changes-2)
    -   [MySQL - データベースからのデータの読み取り](#mysql---reading-data-from-a-database)
        -   [MySQL サーバで利用可能なデータベースを表示](#show-available-databases-on-the-mysql-server)
        -   [MySQL サーバから履歴コンテキストの読み取り](#read-historical-context-from-the-mysql-server)
-   [Multi-Agent - コンテキスト・データを複数のデータベースに永続化](#multi-agent---persisting-context-data-into-a-multiple-databases)
    -   [Multi-Agent - 複数データベース向けの Draco の構成](#multi-agent---draco-configuration-for-multiple-databases)
    -   [Multi-Agent - 起動](#multi-agent---start-up)
        -   [Draco サービスの状態を確認](#checking-the-draco-service-health-3)
        -   [コンテキスト・データの生成](#generating-context-data-3)
        -   [コンテキスト変更のサブスクライブ](#subscribing-to-context-changes-3)
    -   [Multi-Agent - 永続データの読み取り](#multi-agent---reading-persisted-data)
-   [次のステップ](#next-steps)

</details>

<a name="data-persistence-using-apache-nifi"></a>

# Apache NIFI を使用したデータの永続化

> "Plots within plots, but all roads lead down the dragon’s gullet."
>
> — George R.R. Martin (A Dance With Dragons)

[FIWARE Draco](https://fiware-draco.readthedocs.io/en/latest/) は、履歴コンテキスト・データを一連のデータベースに
永続化できる代替の汎用イネーブラーです。**Cygnus** のように、**Draco** は、**Orion Context Broker**からの状態変化を
サブスクライブし、データ・シンクに永続化する前にそのデータを処理するためのファネル (funnel) を提供できます。

前述のように、履歴コンテキスト・データの永続化は、ビッグデータ分析、傾向の発見、または外れ値の削除に役立ちます。
これを行うために使用するツールはニーズによって異なり、**Cygnus** とは異なり、**Draco** は、手順を設定および監視
するためのグラフィカル・インターフェイスを提供します。

違いの要約を以下に示します:

| Draco                                                                              | Cygnus                                                                        |
| ---------------------------------------------------------------------------------- | ----------------------------------------------------------------------------- |
| 通知用の NGSI-LD インターフェースを提供します                                      | 通知用の NGSIv2 インターフェイスを提供します                                  | 
| サブスクリプションのエンドポイントは構成可能ですが、デフォルトは `/v2/notify` です | サブスクリプションのエンドポイントは `/notify` をリッスンします               |
| 単一のポートでリッスンします                                                       | 入力ごとに別々のポートでリッスンします                                        |
| グラフィカル・インターフェイスで構成します                                         | 構成ファイルを介して構成します                                                |
| Apache NIFI をベースにしています                                                   | Apache Flume をベースにしています                                             |
| **Draco**は、[ここ](https://fiware-draco.readthedocs.io/en/latest/) にあります     | **Cygnus** [ここ](https://fiware-cygnus.readthedocs.io/en/latest/) にあります |

#### デバイス・モニタ

このチュートリアルの目的のために、一連のダミー農業用 IoT デバイスが作成され、Context Broker に接続されます。
使用されているアーキテクチャとプロトコルの詳細は、[IoT センサのチュートリアル](https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-LD)
に記載されています。各デバイスの状態は、次の場所にある UltraLight デバイス・モニタの Web ページで確認できます:
`http://localhost:3000/device/monitor`

![FIWARE Monitor](https://github.com/FIWARE/tutorials.Historic-Context-NIFI/blob/gh-pages/img/device-monitor.png)

<a name="architecture"></a>

# アーキテクチャ

このアプリケーションは、[以前のチュートリアル](https://github.com/FIWARE/tutorials.IoT-Agent/)で作成されたコンポーネントと
ダミー IoT デバイスに基づいて構築されています。3つのFIWAREコンポーネントを利用します。
[Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/), 
[IoT Agent for Ultralight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/) および、
コンテキスト・データをデータベースに永続化するための [Draco Generic Enabler](https://fiware-draco.readthedocs.io/en/latest/)
を導入します。

したがって、アーキテクチャ全体は次の要素で構成されます:

-   **FIWARE Generic Enablers**:

    -   [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/gitlab/NGSI-LD/NGSI-LD/raw/master/spec/updated/full_api.json)
        を使用してリクエストを受信する [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/)
    -   FIWARE [IoT Agent for UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/) は、
        [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/gitlab/NGSI-LD/NGSI-LD/raw/master/spec/updated/full_api.json)
        を使用してサウスバウンド・リクエストを受信し、それらをデバイスの
        [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
        コマンドに変換します
    -   FIWARE [Draco](https://fiware-draco.readthedocs.io/en/latest/) は、コンテキストの変更をサブスクライブしてデータベース
        (**MySQL** , **PostgreSQL** または **MongoDB**) に永続化します
-   **データベース**は、次の1つ、2つ、または3つ:
    -   基盤となる [MongoDB](https://www.mongodb.com/) データベース :
        -   Orion Context Broker が、データエンティティ、サブスクリプション、レジストレーションなどのコンテキスト・データ情報を
            保持するために使用します
        -   デバイスの URL やキーなどのデバイス情報を保持するために **IoT Agent** によって使用されます
        -   履歴コンテキスト・データを保持するためのデータ・シンクとして使用される可能性があります
    -   追加の [PostgreSQL](https://www.postgresql.org/) データベース :
        -   履歴コンテキスト・データを保持するためのデータ・シンクとして使用される可能性があります
    -   追加の [MySQL](https://www.mysql.com/) データベース :
        -   履歴コンテキスト・データを保持するためのデータ・シンクとして使用される可能性があります
-   3つの **コンテキスト・プロバイダ**:
    -   このチュートリアルでは、在庫管理フロントエンド (**Stock Management Frontend**) は使用しません。それは以下を行います:
        -   ストア情報を表示し、ユーザがダミー IoT デバイスと対話できるようにします
        -   各店舗で購入できる商品を表示します
        -   ユーザが製品を "buy" して在庫数を減らすことができるようにします
    -   Web サーバ は、HTTP 上で実行される
        [Ultralight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
        プロトコルを使用する
        [ダミー IoT デバイス](https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-v2)のセットとして機能します
-   **チュートリアル・アプリケーション**は、以下のことを行います:
    -   `@context` システム内のコンテキストエンティティを定義する静的ファイルを提供します
    -   HTTP 上で実行される
        [UltraLight 2.0](https://fiware-iotagent-ul.readthedocs.io/en/latest/usermanual/index.html#user-programmers-manual)
        プロトコルを使用して、ダミー
        [農業用 IoT デバイス](https://github.com/FIWARE/tutorials.IoT-Sensors/tree/NGSI-LD)のセットとして機能します

要素間のすべての相互作用は HTTP リクエストによって開始されるため、エンティティをコンテナ化して、公開されたポートから
実行できます。

チュートリアルの各セクションの特定のアーキテクチャについては、以下で説明します。

<a name="prerequisites"></a>

# 前提条件

<a name="docker-and-docker-compose"></a>

## Docker と Docker Compose

物事を単純にするために、すべてのコンポーネントが [Docker](https://www.docker.com) を使用して実行されます。**Docker**
は、さまざまコンポーネントをそれぞれの環境に分離することを可能にするコンテナ・テクノロジです。

-   Docker Windows にインストールするには、[こちら](https://docs.docker.com/docker-for-windows/)の手順に従ってください
-   Docker Mac にインストールするには、[こちら](https://docs.docker.com/docker-for-mac/)の手順に従ってください
-   Docker Linux にインストールするには、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行するためのツールです。
[YAML file](https://github.com/FIWARE/tutorials.Historic-Context-NIFI/blob/NGSI-LD/docker-compose/multiple.yml)
ファイルは、アプリケーションのために必要なサービスを構成するために使用します。つまり、すべてのコンテナ・サービスは
1 つのコマンドで呼び出すことができます。Docker Compose は、デフォルトで Docker for Windows と Docker for Mac の一部と
してインストールされますが、Linux ユーザは[ここ](https://docs.docker.com/compose/install/)に記載されている手順に従う
必要があります。

次のコマンドを使用して、現在の **Docker** バージョンと **Docker Compose** バージョンを確認できます :

```console
docker-compose -v
docker version
```

Docker バージョン 18.03 以降と Docker Compose 1.21 以上を使用していることを確認し、必要に応じてアップグレードして
ください。

<a name="cygwin-for-windows"></a>

## Cygwin for Windows

シンプルな bash スクリプトを使用してサービスを開始します。Windows ユーザは [cygwin](http://www.cygwin.com/) を
ダウンロードして、Windows 上の Linux ディストリビューションと同様のコマンドライン機能を提供する必要があります。

<a name="start-up"></a>

# 起動

開始する前に、必要な Docker イメージをローカルで取得または構築しておく必要があります。リポジトリを複製し、以下の
コマンドを実行して必要なイメージを作成してください :

```console
git clone https://github.com/fiware/tutorials.Historic-Context-NIFI.git
cd tutorials.Historic-Context-NIFI
git checkout NGSI-LD

./services create
```

その後、リポジトリ内で提供される
[services](https://github.com/FIWARE/tutorials.Historic-Context-NIFI/blob/NGSI-LD/services)
の Bash スクリプトを実行することによって、コマンドラインからすべてのサービスを初期化できます:

```console
./services <command>
```

ここで、`<command>` は、アクティブ化するデータベースによって異なります。このコマンドは、以前のチュートリアルから
シード・データをインポートし、起動時にダミー IoT センサをプロビジョニングします。

> :information_source: **注:** クリーンアップをやり直したい場合は、次のコマンドを使用して再起動することができます::
>
> ```console
> ./services stop
> ```

<a name="mongodb---persisting-context-data-into-a-database"></a>

# MongoDB - コンテキスト・データをデータベースに永続化

すでに MongoDB インスタンスを使用して Orion Context Broker と IoT Agent に関連するデータを保持しているため、MongoDB
テクノロジを使用して履歴コンテキスト・データを永続化することは比較的簡単に構成できます。MongoDB インスタンスは標準
`27017` ポートでリッスンしており、アーキテクチャ全体を以下に示します:

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/mongo-draco-tutorial.png)

<a name="mongodb---database-server-configuration"></a>

## MongoDB - データベース・サーバの構成

```yaml
mongo-db:
    image: mongo:3.6
    hostname: mongo-db
    container_name: db-mongo
    ports:
        - "27017:27017"
    networks:
        - default
    command: --bind_ip_all --smallfiles
```

<a name="mongodb---draco-configuration"></a>

## MongoDB - Draco の構成

```yaml
draco:
    image: ging/fiware-draco:1.3.5
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


`draco` コンテナは、2つのポートでリッスンしています:

-   Draco のサブスクリプション・ポート - `5050` はサービスが Orion Context Broker からの通知をリッスンするポート
-   Draco の Web インターフェイス- `9090` はプロセッサ (processors) を構成するためだけに公開されています

<a name="mongodb---start-up"></a>

## MongoDB - 起動

**MongoDB** データベースのみでシステムを起動するには、次のコマンドを実行します:

```console
./services mongodb
```

次に、ブラウザに移動し、このURL `http://localhost:9090/nifi` を使用して Draco を開きます。

次に、NiFi GUI の上部にあるコンポーネント・ツールバーに移動し、テンプレート・アイコンを見つけて、Draco
ユーザ・スペース内にドラッグ・アンド・ドロップします。この時点で、使用可能なすべてのテンプレートのリストを含む
ポップアップが表示されます。テンプレート MONGO-TUTORIAL を選択してください。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/mongo-tutorial-template.png)

すべてのプロセッサを選択し (Shift キーを押しながらすべてのプロセッサをクリック)、開始 (start) ボタンをクリックして
プロセッサを起動します。これで、各プロセッサのステータス・アイコンが赤から緑に変わったことがわかります。

<a name="checking-the-draco-service-health"></a>

### Draco サービスの状態を確認

Draco が実行されたら、公開された draco ポートに対して、`/nifi-api/system-diagnostics` に HTTP リクエストを送信して、
ステータスを確認できます。レスポンスがブランクの場合、これは通常、Draco が実行されていないか、別のポートでリッスン
していることが原因です。

#### :one: リクエスト:

```console
curl -X GET \
  'http://localhost:9090/nifi-api/system-diagnostics'
```

#### レスポンス:

レスポンスは次のようになります:

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

> **Troubleshooting:** レスポンスがブランクの場合はどうしますか？
>
> -   Docker コンテナが実行されていることを確認するには、
>
> ```bash
> docker ps
> ```
>
> いくつかのコンテナが実行されているのが確認できるはずです。Draco が実行されていない場合は、
> 必要に応じてコンテナを再起動できます。

<a name="generating-context-data"></a>

### コンテキスト・データの生成

このチュートリアルでは、コンテキストが定期的に更新されているシステムを監視する必要があります。ダミー IoT センサを
使用してこれを行うことができます。

農場周辺のさまざまな建物の詳細は、チュートリアル・アプリケーションにあります。
`http://localhost:3000/app/farm/urn:ngsi-ld:Building:farm001` を開いて、関連する充填センサとサーモスタットを備えた
建物を表示します。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/fmis.png)

納屋からいくつかの干し草を外し、サーモスタットを更新し、`http://localhost:3000/device/monitor` でデバイス・モニタ・
ページを開くと**Tractor** が起動し、**Smart Lamp** をオンにします。これは、ドロップ・ダウン・リストから適切な
コマンドを選択して `send` ボタンを押すことで実行できます。デバイスからの測定の流れは、同じページに表示されます。

<a name="subscribing-to-context-changes"></a>

### コンテキスト変更のサブスクライブ

動的コンテキストシステムが稼働したら、コンテキストの変更を **Draco** に通知する必要があります。

これは、Orion Context Broker の `/v1/subscriptions` エンドポイントに POST リクエストを行うことによって行われます。

-   `fiware-service` と `fiware-servicepath` ヘッダは、これらの設定を使用してプロビジョニングされているため、
    接続された IoT センサからの測定値のみをリッスンするようにサブスクリプションをフィルタリングするために使用されます
-   リクエスト・ボディの `idPattern` により、Draco にすべてのコンテキスト・データの変更が通知されます
-   通知は、Draco Listen HTTP プロセッサの URL 構成 `Base Path and Listening port` と一致する必要があります
-   `throttling` 値は、変更がサンプリングされるレートを定義します

#### :two: リクエスト:

```console
curl -L -X POST 'http://localhost:1026/ngsi-ld/v1/subscriptions/' \
-H 'Content-Type: application/ld+json' \
-H 'NGSILD-Tenant: openiot' \
--data-raw '{
  "description": "Notify me of all changes",
  "type": "Subscription",
  "entities" : [{"type" :"Device"}, {"type": "Tractor"}],
  "notification": {
    "format": "normalized",
    "endpoint": {
      "uri": "http://draco:5050/v2/notify",
      "accept": "application/json"
    }
  },
   "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld"
}'
```

ご覧のとおり、コンテキスト・データの永続化に使用されるデータベースは、サブスクリプションの詳細に影響を与えません。
各データベースで同じです。レスポンスは、**201 - Created** になります。

サブスクリプションが作成されている場合は、`/ngsi-ld/v1/subscriptions/` エンドポイントに対して GET リクエストを
行うことで、サブスクリプションが起動しているかどうかを確認できます。

#### :three: リクエスト:

```console
curl -X GET \
  'http://localhost:1026/ngsi-ld/v1/subscriptions/' \
  -H 'NGSILD-Tenant: openiot'
```

#### レスポンス:

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

レスポンスの `notification` セクション内に、サブスクリプションの状態を説明するいくつかの追加の `attributes`
が表示されます。

サブスクリプションの基準が満たされている場合、`timesSent` は `0` より大きくなければなりません。ゼロ値は、
サブスクリプションの `subject` が正しくないか、サブスクリプションが間違った `fiware-service-path` または
`fiware-service` ヘッダで作成されたことを示します。

`lastNotification` は最近のタイムスタンプである必要があります。そうでない場合、デバイスは定期的にデータを
送信していません。**Smart Door** のロックを解除し、**Smart Lamp** のスイッチを入れることを忘れないでください。

`lastSuccess` は、`lastNotification` の日付と一致する必要があります。そうでない場合、**Draco**は
サブスクリプションを正しく受信していません。 ホスト名とポートが正しいことを確認してください。

最後に、サブスクリプションの `status` が `active` であることを確認します。期限切れのサブスクリプションは起動しません。

<a name="mongodb----reading-data-from-a-database"></a>

## MongoDB - データベースからのデータの読み取り

コマンドラインから MongoDB データを読み取るには、`mongo` ツールにアクセスする必要があります。`mongo` イメージの
インタラクティブ・インスタンスを実行し、コマンドライン・プロンプトを起動します:

```console
docker run -it --network fiware_default  --entrypoint /bin/bash mongo
```

次に、次のようにコマンドラインを使用して、実行中の `mongo-db` データベースにログインできます:

```bash
mongo --host mongo-db
```

<a name="show-available-databases-on-the-mongodb-server"></a>

### MongoDB サーバで利用可能なデータベースを表示

使用可能なデータベースのリストを表示するには、次のようにステートメントを実行します:

#### クエリ:

```
show dbs
```

#### 結果:

```
admin          0.000GB
iotagentul     0.000GB
local          0.000GB
orion          0.000GB
orion-openiot  0.000GB
sth_openiot    0.000GB
```

結果には、**MongoDB** によってデフォルトで設定される2つのデータベース `admin` と `local` と、FIWARE
プラットフォームによって作成された4つのデータベースが含まれます。Orion Context Broker は、 `fiware-service`
ごとに2つの個別のデータベース・インスタンスを作成します。

-   `Store` エンティティは `fiware-service` を定義せずに作成されたため、`orion` データベース内に保持されますが、
     IoT デバイスのエンティティは `openiot` の `fiware-service` ヘッダを使用して作成されて個別に保持されます。
     IoT Agent は、IoT センサのデータを `iotagentul` と呼ばれる別の **MongoDB**データベースに保持するように
     初期化されました。

Draco を Orion Context Broker にサブスクリプションした結果、`sth_openiot`という新しいデータベースが作成されました。
履歴コンテキストを保持する **Mongo DB** データベースのデフォルト値は、`sth_` プレフィックスとそれに続く
`fiware-service` ヘッダで構成されます。したがって、`sth_openiot` は IoT デバイスの履歴コンテキストを保持します。

<a name="read-historical-context-from-the-server"></a>

### サーバーから履歴コンテキストを読み取り

#### クエリ:

```
use sth_openiot
show collections
```

#### 結果:

```
switched to db sth_openiot

sth_/_Door:001_Door
sth_/_Door:001_Door.aggr
sth_/_Lamp:001_Lamp
sth_/_Lamp:001_Lamp.aggr
sth_/_Motion:001_Motion
sth_/_Motion:001_Motion.aggr
```

`sth_openiot` 内を見ると、一連のテーブルが作成されていることがわかります。 各テーブルの名前は、`sth_`プレフィックスと
それに続く `fiware-servicepath` ヘッダとそれに続くエンティティ ID で構成されます。エンティティごとに2つのテーブルが
作成されます。`.aggr` テーブルは、後のチュートリアルでアクセスされるいくつかの集約データを保持します。 生データは、
`.aggr` サフィックスなしでテーブルに表示されます。

履歴データは、各テーブル内のデータを確認することで確認できます。デフォルトでは、各行には単一の属性のサンプル値が
含まれます。

#### クエリ:

```
db["sth_/_Door:001_Door"].find().limit(10)
```

#### 結果:

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

通常の **MongoDB** クエリ構文を使用して、適切なフィールドと値をフィルタリングできます。 たとえば、
`id=Motion:001_Motion` の **Motion Sensor** が蓄積する速度を読み取るには、次のようにクエリを実行します:

#### クエリ:

```
db["sth_/_Motion:001_Motion"].find({attrName: "count"},{_id: 0, attrType: 0, attrName: 0 } ).limit(10)
```

#### 結果:

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

MongoDB クライアントを終了してインタラクティブ・モードを終了するには、以下を実行します:

```console
exit
```

```console
exit
```

<a name="postgresql---persisting-context-data-into-a-database"></a>

# PostgreSQL - コンテキスト・データをデータベースに永続化

履歴コンテキスト・データを PostgreSQL などの代替データベースに永続化するには、PostgreSQL サーバをホストする追加の
コンテナが必要になります。このデータのデフォルトの Docker イメージを使用できます。PostgreSQL インスタンスは標準
`5432` ポートでリッスンしており、アーキテクチャ全体を以下に示します:

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/postgres-draco-tutorial.png)

Orion Context Broker と IoT Agent に関連するデータを保持するために MongoDB コンテナが引き続き必要であるため、
2つのデータベースを備えたシステムができました。

<a name="postgresql---database-server-configuration"></a>

## PostgreSQL - データベース・サーバの構成

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

`postgres-db` コンテナは、単一ポートで待機しています:

-   ポート `5432` は、PostgreSQL サーバのデフォルトのポートです。そのポートは公開されているため、必要に応じて`pgAdmin4`
    ツールを実行してデータベースのデータを表示することもできます

`postgres-db` コンテナは、次のような環境変数によって駆動されます:

| キー              | 値         | 説明                                        |
| ----------------- | ---------- | ------------------------------------------- |
| POSTGRES_PASSWORD | `password` | PostgreSQL データベース・ユーザのパスワード |
| POSTGRES_USER     | `postgres` | PostgreSQL データベース・ユーザのユーザ名   |
| POSTGRES_DB       | `postgres` | PostgreSQL データベース名                   |

> :information_source: **注:** このようなプレーン・テキストの環境変数でユーザ名とパスワードを渡すことは、セキュリティ
> 上のリスクです。これはチュートリアルでは許容できる方法ですが、本番環境では、
> [Docker Secrets](https://blog.docker.com/2017/02/docker-secrets-management/)
> を適用することでこのリスクを回避できます。

<a name="postgresql---draco-configuration"></a>

## PostgreSQL - Draco の構成

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

`draco` コンテナは、2つのポートでリッスンしています:

-   Draco のサブスクリプション・ポート - `5050` はサービスが Orion Context Broker からの通知をリッスンするポート
-   Draco の Web インターフェイス- `9090` はプロセッサ (processors) を構成するためだけに公開されています

<a name="postgresql---start-up"></a>

## PostgreSQL - 起動

**PostgreSQL** データベースを使用してシステムを起動するには、次のコマンドを実行します:

```console
./services postgres
```

次に、ブラウザに移動し、この URL `http://localhost:9090/nifi` を使用して Draco を開きます。

次に、NiFi GUI の上部にあるコンポーネント・ツールバーに移動し、テンプレート・アイコンを見つけて、Draco
ユーザ・スペース内にドラッグ・アンド・ドロップします。この時点で、使用可能なすべてのテンプレートのリストを含む
ポップアップが表示されます。テンプレート POSTGRESQL-TUTORIAL を選択してください。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/postgres-tutorial-template.png)

プロセッサを起動する前に、PostgreSQL パスワードを設定し、DBCConnectionPool コントローラを有効にする必要があります。
それを行うには、指示に従ってください:

1.  Draco GUI ユーザー・スペースの任意の部分を右クリックしてから、configure をクリックします
    ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step1.png)

2.  [Controller Services] タブに移動します。この時点で、コントローラのリストが表示されます。DBCConnectionPool
    コントローラを見つけます

3.  "DBCPConnectionPool" の設定ボタンをクリックします
    ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step2.png)

4.  コントローラの [Properties] タブに移動し、[password] フィールドに "password" を入力して、変更を適用します
    ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/controller-postgresql.png)

5.  サンダー・アイコンをクリックしてプロセッサを有効にし、[enable] をクリックして、コントローラの設定ページを
    閉じます

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step4.png)

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step5.png)

6.  すべてのプロセッサを選択し (Shift キーを押しながらすべてのプロセッサをクリック)、開始ボタンをクリックして
    プロセッサを起動します。これで、各プロセッサのステータス・アイコンが赤から緑に変わったことがわかります

<a name="checking-the-draco-service-health-1"></a>

### Draco サービスの状態を確認

Draco が実行されたら、公開された draco ポートに対して、`/nifi-api/system-diagnostics` に HTTP リクエストを送信して、
ステータスを確認できます。レスポンスがブランクの場合、これは通常、Draco が実行されていないか、別のポートでリッスン
していることが原因です。

#### :one: リクエスト:

```console
curl -X GET \
  'http://localhost:9090/nifi-api/system-diagnostics'
```

#### レスポンス:

レスポンスは次のようになります:

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

> **Troubleshooting:** レスポンスがブランクの場合はどうしますか？
>
> -   Docker コンテナが実行されていることを確認するには、
>
> ```bash
> docker ps
> ```
>
> いくつかのコンテナが実行されているのが確認できるはずです。Draco が実行されていない場合は、
> 必要に応じてコンテナを再起動できます。

<a name="generating-context-data-1"></a>

### コンテキスト・データの生成

このチュートリアルでは、コンテキストが定期的に更新されているシステムを監視する必要があります。 ダミー IoT センサを
使用してこれを行うことができます。`http://localhost:3000/device/monitor` でデバイス・モニタ・ページを開き、
**Smart Door** のロックを解除して、**Smart Lamp** をオンにします。 これは、ドロップ・ダウン・リストから適切な
コマンドを選択し、`send` ボタンを押すことで実行できます。デバイスからの測定の流れは、同じページで見ることができます:

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/door-open.gif)

<a name="subscribing-to-context-changes-1"></a>

### コンテキスト変更のサブスクライブ

動的コンテキストシステムが稼働したら、コンテキストの変更を **Draco** に通知する必要があります。

これは、Orion Context Broker の `/v1/subscriptions` エンドポイントに POST リクエストを行うことによって行われます。

-   `fiware-service` と `fiware-servicepath` ヘッダは、これらの設定を使用してプロビジョニングされているため、
    接続された IoT センサからの測定値のみをリッスンするようにサブスクリプションをフィルタリングするために使用されます
-   リクエスト・ボディの `idPattern` により、Draco にすべてのコンテキスト・データの変更が通知されます
-   `throttling` 値は、変更がサンプリングされるレートを定義します

#### :five: リクエスト:

```console
curl -L -X POST 'http://localhost:1026/ngsi-ld/v1/subscriptions/' \
-H 'Content-Type: application/ld+json' \
-H 'NGSILD-Tenant: openiot' \
--data-raw '{
  "description": "Notify Draco of all device and tractor changes",
  "type": "Subscription",
  "entities" : [{"type" :"Device"}, {"type": "Tractor"}],
  "notification": {
    "format": "normalized",
    "endpoint": {
      "uri": "http://draco:5050/v2/notify",
      "accept": "application/json"
    }
  },
   "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld"
}'
```

ご覧のとおり、コンテキスト・データの永続化に使用されるデータベースは、サブスクリプションの詳細に影響を与えません。
各データベースで同じです。レスポンスは、**201 - Created** になります。

<a name="postgresql---reading-data-from-a-database"></a>

## PostgreSQL - データベースからのデータの読み取り

コマンドラインから PostgreSQL データを読み取るには、`postgres` クライアントにアクセスする必要があります。これを行うには、
次に示されているように接続文字列を提供する `postgresql-client` イメージのインタラクティブ・インスタンスを実行して、
コマンドライン・プロンプトを起動します:

```console
docker run -it --rm  --network fiware_default jbergknoff/postgresql-client \
   postgresql://postgres:password@postgres-db:5432/postgres
```

<a name="show-available-databases-on-the-postgresql-server"></a>

### PostgreSQL サーバで利用可能なデータベースを表示

使用可能なデータベースのリストを表示するには、次のようにステートメントを実行します:

#### クエリ:

```
\list
```

#### 結果:

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

結果には、2つのテンプレートデータベース `template0` と `template1` および docker コンテナが起動されたときの
`postgres` データベースのセットアップが含まれます。

使用可能なスキーマのリストを表示するには、次のようにステートメントを実行します:

#### クエリ:

```
\dn
```

#### 結果:

```
  List of schemas
  Name   |  Owner
---------+----------
 openiot | postgres
 public  | postgres
(2 rows)
```

Draco を Orion Context Broker にサブスクリプションした結果、`openiot` と呼ばれる新しいスキーマが作成されました。
スキーマの名前は `fiware-service` ヘッダと一致します。したがって、`openiot` は IoT デバイスの履歴コンテキストを
保持します。

<a name="read-historical-context-from-the-postgresql-server"></a>

### PostgreSQL サーバから履歴コンテキストの読み取り

ネットワーク内で Docker コンテナを実行すると、実行中のデータベースに関する情報を取得できます。

#### クエリ:

```sql
SELECT table_schema,table_name
FROM information_schema.tables
WHERE table_schema ='openiot'
ORDER BY table_schema,table_name;
```

#### 結果:

```
 table_schema |    table_name
--------------+-------------------
 openiot      | door_001_door
 openiot      | lamp_001_lamp
 openiot      | motion_001_motion
(3 rows)
```

`table_schema` は、コンテキスト・データで提供される `fiware-service` ヘッダと一致します:

テーブル内のデータを読み取るには、次のように select ステートメントを実行します:

#### クエリ:

```sql
SELECT * FROM openiot.motion_001_motion limit 10;
```

#### 結果:

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

通常の **PostgreSQL** クエリ構文を使用して、適切なフィールドと値をフィルタリングできます。たとえば、
`id=Motion:001_Motion` の **Motion Sensor** が蓄積する速度を読み取るには、次のようにクエリを実行します:

#### クエリ:

```sql
SELECT recvtime, attrvalue FROM openiot.motion_001_motion WHERE attrname ='count'  limit 10;
```

#### 結果:

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

Postgres クライアントを終了してインタラクティブ・モードを終了するには、以下を実行します:

```console
\q
```

その後、コマンドラインに戻ります。

<a name="mysql---persisting-context-data-into-a-database"></a>

# MySQL - コンテキスト・データをデータベースに永続化

同様に、履歴コンテキスト・データを **MySQL** に永続化するには、MySQL サーバをホストする追加のコンテナが再び必要に
なります。このデータのデフォルトの Docker イメージを使用できます。MySQL インスタンスは標準の `3306` ポートでリッスン
しており、全体的なアーキテクチャを以下に示します:

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/mysql-draco-tutorial.png)

Orion Context Broker と IoT Agent に関連するデータを保持するために MongoDB コンテナが引き続き必要であるため、
ここでも2つのデータベースを備えたシステムがあります。

<a name="mysql---database-server-configuration"></a>

## MySQL - データベース・サーバの構成

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

> :information_source: **注:** デフォルトの `root` ユーザを使用し、このような環境変数にパスワードを表示することは、
> セキュリティ上のリスクです。これはチュートリアルでは許容できる方法ですが、本番環境では、
> [Docker Secrets](https://blog.docker.com/2017/02/docker-secrets-management/)
> を適用することでこのリスクを回避できます。

`mysql-db` コンテナは、単一ポートで待機しています:

-   ポート `3306` は、MySQL サーバのデフォルトのポートです。公開されているため、必要に応じて他のデータベース・ツールを
    実行してデータを表示することもできます

`mysql-db` コンテナは、環境変数によって駆動されます:

| キー                | 値                 | 説明                                                                                                                                                                                      |
| ------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| MYSQL_ROOT_PASSWORD | `123`              | MySQL `root` アカウントに設定されるパスワードを指定します                                                                                                                                 |
| MYSQL_ROOT_HOST     | `root'@'localhost` | デフォルトでは、MySQLは `root'@'localhost` アカウントを作成します。このアカウントは、コンテナ内からのみ接続できます。この環境変数を設定すると、他のホストからの root 接続が可能になります |

<a name="mysql---draco-configuration"></a>

## MySQL - Draco の構成

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

`draco` コンテナは、2つのポートでリッスンしています:

-   Draco のサブスクリプション・ポート - `5050` はサービスが Orion Context Broker からの通知をリッスンするポート
-   Draco の Web インターフェイス- `9090` はプロセッサ (processors) を構成するためだけに公開されています

<a name="mysql---start-up"></a>

## MySQL - 起動

**MySQL** データベースを使用してシステムを起動するには、次のコマンドを実行します:

```console
./services mysql
```

次に、ブラウザに移動し、この URL `http://localhost:9090/nifi` を使用して Draco を開きます。

次に、NiFi GUI の上部にあるコンポーネント・ツールバーに移動し、テンプレート・アイコンを見つけて、Draco
ユーザ・スペース内にドラッグ・アンド・ドロップします。この時点で、使用可能なすべてのテンプレートのリストを含む
ポップアップが表示されます。テンプレート MYSQL-TUTORIAL を選択してください。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/draco-template1.png)

プロセッサを起動する前に、MySQL パスワードを設定し、DBCConnectionPool コントローラを有効にする必要があります。
それを行うには、指示に従ってください:

1.  Draco GUI ユーザー・スペースの任意の部分を右クリックしてから、configure をクリックします
    ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step1.png)

2.  [Controller Services] タブに移動します。この時点で、コントローラのリストが表示されます。DBCConnectionPool
    コントローラを見つけます

3.  "DBCPConnectionPool" の設定ボタンをクリックします
    ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step2.png)

4.  コントローラの [Properties] タブに移動し、[password] フィールドに "password" を入力して、変更を適用します
    ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step3.png)

5.  サンダー・アイコンをクリックしてプロセッサを有効にし、[enable] をクリックして、コントローラの設定ページを
    閉じます。 ![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step4.png)

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/step5.png)

6.  すべてのプロセッサを選択し (Shift キーを押しながらすべてのプロセッサをクリック)、開始ボタンをクリックして
    プロセッサを起動します。これで、各プロセッサのステータス・アイコンが赤から緑に変わったことがわかります


<a name="checking-the-draco-service-health-2"></a>

### Draco サービスの状態を確認

Draco が実行されたら、公開された draco ポートに対して、`/system-diagnostics` に HTTP リクエストを送信して、
ステータスを確認できます。レスポンスがブランクの場合、これは通常、Draco が実行されていないか、別のポートでリッスン
していることが原因です。

#### :one: リクエスト:

```console
curl -X GET \
  'http://localhost:9090/system-diagnostics'
```

#### レスポンス:

レスポンスは次のようになります:

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

> **Troubleshooting:** レスポンスがブランクの場合はどうしますか？
>
> -   Docker コンテナが実行されていることを確認するには、
>
> ```bash
> docker ps
> ```
>
> いくつかのコンテナが実行されているのが確認できるはずです。Draco が実行されていない場合は、
> 必要に応じてコンテナを再起動できます。

<a name="generating-context-data-2"></a>

### コンテキスト・データの生成

このチュートリアルでは、コンテキストが定期的に更新されているシステムを監視する必要があります。 ダミー IoT センサを
使用してこれを行うことができます。`http://localhost:3000/device/monitor` でデバイス・モニタ・ページを開き、
**Smart Door**のロックを解除して、**Smart Lamp**をオンにします。 これは、ドロップ・ダウン・リストから適切なコマンドを
選択し、`send` ボタンを押すことで実行できます。デバイスからの測定の流れは、同じページで見ることができます:

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/door-open.gif)

<a name="subscribing-to-context-changes-2"></a>

### コンテキスト変更のサブスクライブ

動的コンテキストシステムが稼働したら、コンテキストの変更を **Draco** に通知する必要があります。

これは、Orion Context Broker の `/v1/subscriptions` エンドポイントに POST リクエストを行うことによって行われます。

-   `fiware-service` と `fiware-servicepath` ヘッダは、これらの設定を使用してプロビジョニングされているため、
    接続された IoT センサからの測定値のみをリッスンするようにサブスクリプションをフィルタリングするために使用されます
-   リクエスト・ボディの `idPattern` により、Draco にすべてのコンテキスト・データの変更が通知されます
-   `throttling` 値は、変更がサンプリングされるレートを定義します

#### :seven: リクエスト:

```console
curl -L -X POST 'http://localhost:1026/ngsi-ld/v1/subscriptions/' \
-H 'Content-Type: application/ld+json' \
-H 'NGSILD-Tenant: openiot' \
--data-raw '{
  "description": "Notify Draco of all device and tractor changes",
  "type": "Subscription",
  "entities" : [{"type" :"Device"}, {"type": "Tractor"}],
  "notification": {
    "format": "normalized",
    "endpoint": {
      "uri": "http://draco:5050/v2/notify",
      "accept": "application/json"
    }
  },
   "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld"
}'
```

ご覧のとおり、コンテキスト・データの永続化に使用されるデータベースは、サブスクリプションの詳細に影響を与えません。
各データベースで同じです。レスポンスは、**201 - Created** になります。

<a name="mysql---reading-data-from-a-database"></a>

## MySQL - データベースからのデータの読み取り

コマンドラインから MySQL データを読み取るには、`mysql` クライアントにアクセスする必要があります。これを行うには、
次に示されているように接続文字列を提供する `mysql` イメージのインタラクティブ・インスタンスを実行して、
コマンドライン・プロンプトを起動します:

```console
docker exec -it  db-mysql mysql -h mysql-db -P 3306  -u root -p123
```

<a name="show-available-databases-on-the-mysql-server"></a>

### MySQL サーバで利用可能なデータベースを表示

使用可能なデータベースのリストを表示するには、次のようにステートメントを実行します:

#### クエリ:

```sql
SHOW DATABASES;
```

#### 結果:

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

使用可能なスキーマのリストを表示するには、次のようにステートメントを実行します:

#### クエリ:

```sql
SHOW SCHEMAS;
```

#### 結果:

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


Draco を Orion Context Broker にサブスクリプションした結果、`openiot` と呼ばれる新しいスキーマが作成されました。
スキーマの名前は `fiware-service` ヘッダと一致します。したがって、`openiot` は IoT デバイスの履歴コンテキストを
保持します。

<a name="read-historical-context-from-the-mysql-server"></a>

### MySQL サーバから履歴コンテキストの読み取り

ネットワーク内で Docker コンテナを実行すると、実行中のデータベースに関する情報を取得できます。

#### クエリ:

```sql
SHOW tables FROM openiot;
```

#### 結果:

```
 table_schema |    table_name
--------------+-------------------
 openiot      | door_001_door
 openiot      | lamp_001_lamp
 openiot      | motion_001_motion
(3 rows)
```

`table_schema` は、コンテキスト・データで提供される `fiware-service` ヘッダと一致します:

テーブル内のデータを読み取るには、次のように select ステートメントを実行します:

#### クエリ:

```sql
SELECT * FROM openiot.Motion_001_Motion limit 10;
```

#### 結果:

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

通常の **MySQL** クエリ構文を使用して、適切なフィールドと値をフィルタリングできます。たとえば、`id=Motion:001_Motion`
の **Motion Sensor** が蓄積する速度を読み取るには、次のようにクエリを実行します:

#### クエリ:

```sql
SELECT recvtime, attrvalue FROM openiot.Motion_001_Motion WHERE attrname ='count' LIMIT 10;
```

#### 結果:

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

MySQL クライアントを終了してインタラクティブ・モードを終了するには、次のコマンドを実行します:

```console
\q
```

その後、コマンド・ラインに戻ります。

<a name="multi-agent---persisting-context-data-into-a-multiple-databases"></a>

# Multi-Agent - コンテキスト・データを複数のデータベースに永続化

複数のデータベースに同時にデータを入力するように Draco を構成することもできます。 前の3つの例のアーキテクチャを
組み合わせて、複数のシンクにデータを格納するように Draco を構成できます。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/multiple-draco-tutorial.png)

これで、データ永続化のための PostgreSQL と MySQL およびデータ永続化と Orion Context Broker と IoT Agent に関連
するデータの保持の両方のための MongoDB の3つのデータベースを備えたシステムができました。

<a name="multi-agent---draco-configuration-for-multiple-databases"></a>

## Multi-Agent - 複数データベース向けの Draco の構成

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

`draco` コンテナは、2つのポートでリッスンしています:

-   Draco のサブスクリプション・ポート - `5050` はサービスが Orion Context Broker からの通知をリッスンするポート
-   Draco の Web インターフェイス- `9090` はプロセッサ (processors) を構成するためだけに公開されています

<a name="multi-agent---start-up"></a>

## Multi-Agent - 起動

**複数**のデータベースでシステムを起動するには、次のコマンドを実行します:

```console
./services multiple
```

次に、ブラウザに移動し、この URL `http://localhost:9090/nifi` を使用して Draco を開きます。

次に、NiFi GUI の上部にあるコンポーネント・ツールバーに移動し、テンプレート・アイコンを見つけて、Draco
ユーザ・スペース内にドラッグ・アンド・ドロップします。この時点で、使用可能なすべてのテンプレートのリストを含む
ポップアップが表示されます。テンプレート MULTIPLE-SINKS-TUTORIAL を選択してください。

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/multiple-tutorial-template.png)

ここで、MySQL と PostgreSQL の各接続のコントローラー DBCPConnectionPool でパスワードを設定するプロセスを
繰り返します。

すべてのプロセッサを選択し (Shift キーを押しながらすべてのプロセッサをクリック)、開始ボタンをクリックしてプロセッサを
起動します。これで、各プロセッサのステータス・アイコンが赤から緑に変わったことがわかります。

<a name="checking-the-draco-service-health-3"></a>

### Draco サービスの状態を確認

Draco が実行されたら、公開された draco ポートに対して `/system-diagnostics` への HTTP リクエストを行うことで、
ステータスを確認できます。レスポンスがブランクの場合、これは通常、Draco が実行されていないか、別のポートでリッスン
していることが原因です。

#### :one: リクエスト:

```console
curl -X GET \
  'http://localhost:9090/nifi-api/system-diagnostics'
```

#### レスポンス:

レスポンスは次のようになります:

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

> **Troubleshooting:** レスポンスがブランクの場合はどうしますか？
>
> -   Docker コンテナが実行されていることを確認するには、
>
> ```bash
> docker ps
> ```
>
> いくつかのコンテナが実行されているのが確認できるはずです。Draco が実行されていない場合は、
> 必要に応じてコンテナを再起動できます。

<a name="generating-context-data-3"></a>

### コンテキスト・データの生成

このチュートリアルでは、コンテキストが定期的に更新されているシステムを監視する必要があります。 ダミー IoT センサを
使用してこれを行うことができます。`http://localhost:3000/device/monitor` でデバイス・モニタ・ページを開き、
**Smart Door**のロックを解除して、**Smart Lamp**をオンにします。 これは、ドロップ・ダウン・リストから適切なコマンドを
選択し、`send` ボタンを押すことで実行できます。デバイスからの測定の流れは、同じページで見ることができます:

![](https://fiware.github.io/tutorials.Historic-Context-NIFI/img/door-open.gif)

<a name="subscribing-to-context-changes-3"></a>

### コンテキスト変更のサブスクライブ

動的コンテキストシステムが稼働したら、コンテキストの変更を **Draco** に通知する必要があります。

これは、Orion Context Broker の `/v1/subscriptions` エンドポイントに POST リクエストを行うことによって行われます。

-   `fiware-service` と `fiware-servicepath` ヘッダは、これらの設定を使用してプロビジョニングされているため、
    接続された IoT センサからの測定値のみをリッスンするようにサブスクリプションをフィルタリングするために使用されます
-   リクエスト・ボディの `idPattern` により、Draco にすべてのコンテキスト・データの変更が通知されます
-   `throttling` 値は、変更がサンプリングされるレートを定義します

#### :nine: リクエスト:

```console
curl -L -X POST 'http://localhost:1026/ngsi-ld/v1/subscriptions/' \
-H 'Content-Type: application/ld+json' \
-H 'NGSILD-Tenant: openiot' \
--data-raw '{
  "description": "Notify Draco of all device and tractor changes",
  "type": "Subscription",
  "entities" : [{"type" :"Device"}, {"type": "Tractor"}],
  "notification": {
    "format": "normalized",
    "endpoint": {
      "uri": "http://draco:5050/v2/notify",
      "accept": "application/json"
    }
  },
   "@context": "http://context-provider:3000/data-models/ngsi-context.jsonld"
}'
```

ご覧のとおり、コンテキスト・データの永続化に使用されるデータベースは、サブスクリプションの詳細に影響を与えません。
各データベースで同じです。レスポンスは、**201 - Created** になります。

<a name="multi-agent---reading-persisted-data"></a>

## Multi-Agent - 永続データの読み取り

接続したデータベースから永続データを読み取るには、このチュートリアルの前のセクションを参照してください。

<a name="next-steps"></a>

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか？ このシリーズの
[他のチュートリアル](https://www.letsfiware.jp/ngsi-ld-tutorials)を読むことで見つけることができます

---

## License

[MIT](LICENSE) © 2021 FIWARE Foundation e.V.
