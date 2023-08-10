# Merge Patch NGSI-LD[<img src="https://img.shields.io/badge/NGSI-LD-d6604d.svg" width="90"  align="left" />](https://www.etsi.org/deliver/etsi_gs/CIM/001_099/009/01.06.01_60/gs_CIM009v010601p.pdf)[<img src="https://fiware.github.io/tutorials.Merge-Patch-Put/img/fiware.png" align="left" width="162">](https://www.fiware.org/)<br/>

[![FIWARE Core Context Management](https://nexus.lab.fiware.org/repository/raw/public/badges/chapters/core.svg)](https://github.com/FIWARE/catalogue/blob/master/core/README.md)
[![License: MIT](https://img.shields.io/github/license/fiware/tutorials.Merge-Patch-Put.svg)](https://opensource.org/licenses/MIT)
[![Support badge](https://img.shields.io/badge/tag-fiware-orange.svg?logo=stackoverflow)](https://stackoverflow.com/questions/tagged/fiware)
<br/> [![JSON LD](https://img.shields.io/badge/JSON--LD-1.1-f06f38.svg)](https://w3c.github.io/json-ld-syntax/)
[![Documentation](https://img.shields.io/readthedocs/fiware-tutorials.svg)](https://fiware-tutorials.rtfd.io)

このチュートリアルでは、NGSI-LD Merge Patch エンドポイントを紹介します。 マージ・パッチ (`/entities/<id>`)
と部分更新パッチ (`/entities/<id>/attrs`) の違いを説明し、この機能の使用方法を示します。

チュートリアルでは全体で [cUrl](https://ec.haxx.se/) コマンドを使用していますが、
[Postman documentation](https://fiware.github.io/tutorials.Merge-Patch-Put/ngsi-ld.html)
のドキュメントとしても利用できます。

[![Run in Postman](https://run.pstmn.io/button.svg)](https://app.getpostman.com/run-collection/d24facc3c430bb5d5aaf)

## コンテンツ

<details>
<summary><strong>詳細</strong></summary>

-   [Merge-Patch と PUT](#merge-patch-and-put)
    -   [なぜ2種類の **PATCH** があるのか？](#why-two-kinds-of-patch-)
        -   [部分更新 **PATCH**](#partial-update-patch)
        -   [マージ **PATCH**](#merge-patch)
        -   [上書き **PUT**](#overwrite-put)
-   [アーキテクチャ](#architecture)
-   [前提条件](#prerequisites)
    -   [Docker と Docker Compose](#docker-and-docker-compose)
    -   [Cygwin for Windows](#cygwin-for-windows)
-   [起動](#start-up)
-   [マージ・パッチ操作](#merge-patch-operations)
    -   [プリフライト](#preflight)
    -   [Merge-Patch 操作](#merge-patch-operations-1)
        -   [マージ・パッチを使用した更新](#updating-using-merge-patch)
        -   [マージ・パッチを使用して新しい属性を追加](#adding-new-attributes-using-merge-patch)
        -   [マージ・パッチを使用して既存の属性を削除](#removing-existing-attributes-using-merge-patch)
        -   [サブ属性を使用してプロパティの値を修正](#amending-values-of-a-property-with-sub-attributes)
        -   [Key-Value フォーマットを使用した更新](#updating-using-key-values-format)
        -   [`observedAt` による Key-Value を使用した更新](#updating-using-key-values-with-observedat)
        -   [`lang` による Key-Value を使用した更新](#updating-using-key-values-with-lang)
        -   [**PUT** によるエンティティの上書き](#overwriting-an-entity-with-put)

</details>

<a name="merge-patch-and-put"></a>

# Merge-Patch と PUT (Merge-Patch and Put)

> "Last night I dreamed about you. What happened in detail I can hardly remember, all I know is that we kept merging
> into one another. I was you, you were me. Finally you somehow caught fire"
>
> — Franz Kafka, Letters to Milena

Merge-Patch は、リソースに対して行う一連の変更を記述するために使用される、明確に定義された
[IETF 仕様](https://www.rfc-editor.org/rfc/rfc7396.html)です。JSON ペイロードをフォレンジック・ナイフ
(a forensic knife) として使用して、エンティティ内で個別に指定された属性を作成、変更、または削除します。
インターネット全体で定義されているように、Merge-Patch は通常 JSON ペイロードを使用し、通常は HTTP PATCH
メソッドに割り当てられるアクションです。NGSI-LD は、代わりに JSON-LD ペイロードで使用するために概念を拡張します。

<a name="why-two-kinds-of-patch-"></a>

## なぜ2種類の **PATCH** があるのか？ (Why two kinds of **PATCH** ?)

<a name="partial-update-patch"></a>

### 部分更新 **PATCH** (Partial Update **PATCH**)

部分的な更新パッチ (Partial Update Patch) は、2 つのエンドポイント (`/entities/<id>/attrs` と
`/entities/<id>/attrs/<attr-name>`) でサポートされています。部分更新 **PATCH** のルールは、
実質的に属性レベルでの上書きです。

次の NGSI-LD _Property_ が与えられた場合:

```json
{
    "temperature": {
        "type": "Property",
        "value": 25,
        "unitCode": "CEL",
        "observedAt": "2022-03-14T01:59:26.535Z"
    }
}
```

そして、次のペイロードで部分更新 **PATCH** オペレーション `/entities/<id>/attrs/temperature` を適用します。

```json
{
    "type": "Property",
    "value": 100,
    "observedAt": "2022-03-14T13:00:00.000Z"
}
```

#### 結果 1

次のように、`value` および `observedAt` サブ属性が上書きされ、`unitCode` サブ属性は変更されません。

```json
{
    "temperature": {
        "type": "Property",
        "value": 100,
        "unitCode": "CEL",
        "observedAt": "2022-03-14T13:00:00.000Z"
    }
}
```

ただし、同じエンティティを指定し、`/entities/<id>/attrs` レベルで部分更新 **PATCH** 操作を適用します。

```json
{
    "temperature": {
        "type": "Property",
        "value": 100,
        "observedAt": "2022-03-14T13:00:00.000Z"
    }
}
```

#### 結果 2

```json
{
    "temperature": {
        "type": "Property",
        "value": 100,
        "observedAt": "2022-03-14T13:00:00.000Z"
    }
}
```

`temperature` プロパティ全体が上書きされます。2番目のケースでは `unitCode` も削除されていることに注意してください。

属性レベルでの部分更新 **PATCH** の考え方は、データの一貫性を目指すことです。`observedAt` などのサブ属性が省略された
場合、それは削除されず、既存の値が残ります。ユーザは、他の手段を使用して意図的にそのようなデータを削除することを
余儀なくされます。

エンティティ・レベルでの部分更新 **PATCH** の考え方は、一時的な一貫性を目指すことです。更新ごとに、最初の属性レイヤー
内のすべてのサブ属性を再供給する必要があります。`observedAt` などのサブ属性が省略されている場合、それは削除され、
既存の一時レコードは影響を受けません。

**PATCH** はこれらの操作の両方に適しています。どちらの場合も、_Properties_ または _Properties of Properties_
の選択の一部 (すべてではない) の更新であり、エンティティ自体、その `id` および `type` は変更されないためです。


<a name="merge-patch"></a>

### マージ **PATCH** (Merge **PATCH**)

マージ・パッチは `/entities/<id>` でサポートされており、ペイロードで見つかった属性をアップサートするだけです。
再び、次の NGSI-LD _Property_ から開始します:

```json
{
    "temperature": {
        "type": "Property",
        "value": 25,
        "unitCode": "CEL",
        "observedAt": "2022-03-14T01:59:26.535Z"
    }
}
```

次のペイロードでマージ **PATCH** 操作 `/entities/<id>` を適用します。

```json
{
    "temperature": {
        "type": "Property",
        "value": 100,
        "observedAt": "2022-03-14T13:00:00.000Z"
    }
}
```

示されているように、`value` と `observedAt` サブ属性のみが更新されます:

#### 結果 3

```json
{
    "temperature": {
        "type": "Property",
        "value": 100,
        "unitCode": "CEL",
        "observedAt": "2022-03-14T13:00:00.000Z"
    }
}
```

エンティティ・レベルでのマージ **PATCH** の考え方は、一時的な一貫性を考慮せずに既存のデータを修正することです。
変更されていないサブ属性を再供給する必要はありません。`observedAt` などのサブ属性が省略されている場合は、
変更されたままになります。注意しないと、`observedAt` を更新せずに `value` を誤って更新する可能性があるため、
一時的なインターフェイスで予期しない副作用が発生する可能性があります。

要約すると、両方のスタイルの **PATCH** 操作にそれぞれの役割がありますが、マージ **PATCH** はより焦点が絞られており、
より小さなペイロードを使用できます。部分更新 **PATCH** は、ペイロードが完全であることを要求し、ペイロードから省略された
第 2 レベルの属性を削除することで、データの一貫性を高めます。これは `/entityOperations/upsert` エンドポイントにも
当てはまります。つまり、第 2 レベルの _Property-of-a-Property_ メタデータ属性は、削除したくない場合は常にペイロードに
含める必要があります。したがって、通常、`/entityOperations/upsert` ペイロード内のプロパティには、更新された `value`
だけでなく、`unitCode` と `observedAt` も含まれます。

<a name="overwrite-put"></a>

### 上書き **PUT** (Overwrite **PUT**)

HTTP **PUT** は `/entities/<id>` でサポートされており、既存のエンティティ内のデータを上書きするだけです。この場合、
エンティティ全体が上書きされます。以下のようなペイロードは、単一の `temperature` 属性を持つエンティティになります。

```json
{
    "temperature": {
        "type": "Property",
        "value": 25,
        "unitCode": "CEL",
        "observedAt": "2022-03-14T01:59:26.535Z"
    }
}
```

**PUT** はべき等操作であることが保証されています。これを数回呼び出すと、同じシステム状態になりますが、**PATCH**
は必ずしもべき等ではなく、いずれかのフレーバー (マージ・パッチまたは部分更新パッチ) の **PATCH** ペイロードは、
毎回エンティティ・データ全体からすべての属性を提供する必要はありません。

<a name="architecture"></a>

# アーキテクチャ

必要なアーキテクチャは、次の3つの要素で構成されます:

-   [Orion Context Broker](https://fiware-orion.readthedocs.io/en/latest/) は、
    [NGSI-LD](https://forge.etsi.org/swagger/ui/?url=https://forge.etsi.org/rep/NGSI-LD/NGSI-LD/raw/master/spec/updated/generated/full_api.json)
    を使用してリクエストを受け取ります
-   基礎となる [MongoDB](https://www.mongodb.com/) データベース:
    -   データ エンティティ、サブスクリプション、レジストレーションなどのコンテキスト・データ情報を保持するために
        Orion Context Broker によって使用されます
-   HTTP **Web-Server** は、システム内のコンテキスト・エンティティを定義する静的な `@context` ファイルを提供します

3 つの要素間のすべての対話は HTTP リクエストによって開始されるため、要素をコンテナ化して、公開されたポートから
実行できます。

![](https://fiware.github.io/tutorials.CRUD-Operations/img/architecture-ld.png)


必要な構成情報は、関連する `docker-compose.yml` ファイルの services セクションで確認できます。
以前のチュートリアルで説明されています。

<a name="prerequisites"></a>

# 前提条件

<a name="docker-and-docker-compose"></a>

## Docker と Docker Compose

シンプルにするために、すべてのコンポーネントは [Docker](https://www.docker.com) を使用して実行されます。**Docker**
は、それぞれの環境に分離されたさまざまなコンポーネントを可能にするコンテナ・テクノロジです。

-   Windows に Docker をインストールするには、[こちら](https://docs.docker.com/docker-for-windows/)の指示に従ってください
-   Mac に Docker をインストールするには、[こちら](https://docs.docker.com/docker-for-mac/)の指示に従ってください
-   Linux に Docker をインストールするには、[こちら](https://docs.docker.com/install/)の手順に従ってください

**Docker Compose** は、マルチコンテナ Docker アプリケーションを定義して実行するためのツールです。
[YAMLファイル](https://raw.githubusercontent.com/FIWARE/tutorials.Merge-Patch-Put/NGSI-LD/docker-compose.yml)
を使用して、アプリケーションに必要なサービスを設定します。これは、すべてのコンテナ・サービスを単一のコマンドで起動
できることを意味します。Docker Compose は、Docker for Windows および Docker for Mac の一部としてデフォルトでインストール
されますが、Linux ユーザは[こちら](https://docs.docker.com/compose/install/)にある手順に従う必要があります。

次のコマンドを使用して、現在の **Docker** および **Docker Compose** のバージョンを確認できます:

```console
docker-compose -v
docker version
```

Docker バージョン 20.10 以降および Docker Compose 1.29 以降を使用していることを確認し、
必要に応じてアップグレードしてください。

<a name="cygwin-for-windows"></a>

## Cygwin for Windows

簡単な bash スクリプトを使ってサービスを開始します。Windows ユーザは、Windows 上の Linux ディストリビューションに
似たコマンドライン機能を提供するために [cygwin](http://www.cygwin.com/) をダウンロードするべきです。

<a name="start-up"></a>

# 起動

開始する前に、必要な Docker イメージをローカルで取得またはビルドしていることを確認する必要があります。
次のコマンドを実行して、リポジトリのクローンを作成し、必要なイメージを作成してください:

```console
git clone https://github.com/FIWARE/tutorials.Merge-Patch-Put.git
cd tutorials.Merge-Patch-Put
git checkout NGSI-LD

./services create
```

その後、リポジトリ内で提供される [services](https://github.com/FIWARE/tutorials.Merge-Patch-Put/blob/NGSI-LD/services)
Bash スクリプトを実行することにより、コマンドラインからすべてのサービスを初期化できます:

```console
./services start
```

この起動スクリプトは、2つの City エンティティも Context Broker にプリロードします。

> **注:** クリーンアップして最初からやり直す場合は、次のコマンドで実行できます:
>
> ```console
> ./services stop
> ```

---

<a name="merge-patch-operations"></a>

# マージ・パッチ操作 (Merge Patch Operations)

<a name="preflight"></a>

## プリフライト (Preflight)

Orion Context Broker は **OPTIONS** メソッドをサポートし、サポートされている操作をユーザがリクエストできるように
します。2種類の **PATCH** 操作は、Accept-Patch ヘッダを読み取ることで区別できます。

#### :one: リクエスト:

```console
curl -iX OPTIONS \
    'http://localhost:1026/ngsi-ld/v1/entities/'
```

#### レスポンス:

`/entities/` エンドポイントは **GET** および **POST** 操作のみをサポートします。

```text
HTTP/1.1 200 OK
Date: Tue, 06 Dec 2022 09:36:42 GMT
Allow: GET,POST,OPTIONS
Content-Length: 0
```

#### :two: リクエスト:

```console
curl -iX OPTIONS \
    'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001'
```

#### レスポンス:

`/entities/<entity-id>` エンドポイントは、**GET**, **PUT**, **DELETE** および **PATCH** 操作のみをサポートします。
**PATCH** がサポートされているため、適切なペイロード・タイプをリストする追加の `Accept-Patch` ヘッダが返されます。
`/entities/<entity-id>` はマージ・パッチ・エンドポイントであるため、このリストには `application/merge-patch+json`
が含まれていることに注意してください。

```text
HTTP/1.1 200 OK
Date: Tue, 06 Dec 2022 09:49:43 GMT
Accept-Patch: application/json, application/ld+json, application/merge-patch+json
Allow: GET,PUT,DELETE,PATCH,OPTIONS
Content-Length: 0
```

#### :three: リクエスト:

```console
curl -iX OPTIONS \
    'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001/attrs'
```

#### レスポンス:

`/entities/<entity-id>/attrs/temperature` エンドポイントは、**POST** および **PATCH** 操作のみをサポートします。
`/entities/<entity-id>/attrs` は部分更新パッチ・エンドポイントであるため、追加の `Accept-Patch` ヘッダには
`application/merge-patch+json` は含まれません。

```text
HTTP/1.1 200 OK
Date: Tue, 06 Dec 2022 09:49:27 GMT
Accept-Patch: application/json, application/ld+json
Allow: POST,PATCH,OPTIONS
Content-Length: 0
```

#### :four: リクエスト:

```console
curl -iX OPTIONS \
    'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001/attrs/temperature'
```

#### レスポンス:

`/entities/<entity-id>/attrs/<attr>` エンドポイントは、**DELETE** および **PATCH** メソッドのみをサポートします。
`/entities/<entity-id>/attrs/<attribute>` は部分更新パッチ・エンドポイントであるため、追加の `Accept-Patch`
ヘッダには `application/merge-patch+json` は含まれません。

```text
HTTP/1.1 200 OK
Date: Tue, 06 Dec 2022 09:49:10 GMT
Accept-Patch: application/json, application/ld+json
Allow: DELETE,PATCH,OPTIONS
Content-Length:
```

<a name="merge-patch-operations-1"></a>

## Merge-Patch 操作 (Merge-Patch Operations)

マージ・パッチ (Merge-Patch) による修正のために、2つの既存エンティティが作成されていることに注意してください。
エンティティの現在の状態は、`/entities/<entity-id>` エンドポイントに対して **GET** リクエストを行うことで
取得できます。これらのリクエストは、各操作後にエンティティがどのように変化したかを確認するために行うことができます。

#### :five: リクエスト:

```console
curl -L -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json'
```

#### レスポンス:

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 25,
        "unitCode": "CEL",
        "observedAt": "2022-06-30T00:00:00.000Z"
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [28.955, 41.0136]
        }
    },
    "population": {
        "type": "Property",
        "value": 15840900,
        "observedAt": "2022-12-31T00:00:00.000Z"
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Kanlıca İskele Meydanı",
            "addressRegion": "İstanbul",
            "addressLocality": "Beşiktaş",
            "postalCode": "12345"
        }
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
            "el": "Κωνσταντινούπολις",
            "en": "Constantinople",
            "tr": "İstanbul"
        }
    },
    "runBy": {
        "type": "Relationship",
        "object": "urn:ngsi-ld:Adminstration:Cumhuriyet_Halk_Partisi"
    }
}
```

#### :six: リクエスト:

```console
curl -L -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:002' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'options=concise'
```

#### レスポンス:

```json
{
    "id": "urn:ngsi-ld:City:002",
    "type": "City",
    "temperature": {
        "value": 25,
        "unitCode": "CEL",
        "observedAt": "2022-06-30T00:00:00.000Z"
    },
    "address": {
        "value": {
            "streetAddress": "Viale di Valle Aurelia",
            "addressRegion": "Lazio",
            "addressLocality": "Roma",
            "postalCode": "00138"
        }
    },
    "location": {
        "type": "Point",
        "coordinates": [12.482, 41.893]
    },
    "population": {
        "value": 4342212,
        "observedAt": "2021-01-01T00:00:00.000Z"
    },
    "name": {
        "languageMap": {
            "el": "Ρώμη",
            "en": "Rome",
            "it": "Roma"
        }
    },
    "runBy": {
        "object": "urn:ngsi-ld:Adminstration:Partito_Democratico"
    }
}
```

<a name="updating-using-merge-patch"></a>

### マージ・パッチを使用した更新 (Updating using Merge Patch)

この例では、都市の `location` を北緯52.5146、東経13.350に移動し、`temperature` を20に修正します。
ここのデータは正規化された形式 (normalized format) ですが、簡潔な形式 (concise format) もサポートされています:

#### :seven::A: リクエスト:

```console
curl -L -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
    "temperature": {
        "type": "Property",
        "value": 25
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [
                28.955,
                41.0136
            ]
        }
    }
}'
```

#### :seven::B: リクエスト:

```console
curl -L -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
    "temperature": 20,
    "location": {
        "type": "Point",
        "coordinates": [
            13.3505,
            52.5146
        ]
    }
}'
```

#### :eight: リクエスト:

`urn:ngsi-ld:City:001` を再取得すると、`location` と `temperature` が変更されていることがわかりますが、他のすべての
_Properties_ と、`unitCode` や `observedAt` などの _Properties of Properties_ は変更されていません:

```console
curl -L -X GET 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json'
```

#### レスポンス:

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 20,
        "unitCode": "CEL",
        "observedAt": "2022-06-30T00:00:00.000Z"
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [13.3505, 52.5146]
        }
    },
    "population": {
        "type": "Property",
        "value": 15840900,
        "observedAt": "2022-12-31T00:00:00.000Z"
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Kanlıca İskele Meydanı",
            "addressRegion": "İstanbul",
            "addressLocality": "Beşiktaş",
            "postalCode": "12345"
        }
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
            "el": "Κωνσταντινούπολις",
            "en": "Constantinople",
            "tr": "İstanbul"
        }
    },
    "runBy": {
        "type": "Relationship",
        "object": "urn:ngsi-ld:Adminstration:Cumhuriyet_Halk_Partisi"
    }
}
```

<a name="adding-new-attributes-using-merge-patch"></a>

### マージ・パッチを使用して新しい属性を追加 (Adding new attributes using Merge Patch)

マージ・パッチ操作では、属性がペイロードに含まれていてもエンティティにない場合、それが挿入されます。この例では、
`temperature` 属性が更新され、`value` と `observedAt` が更新されていますが、`precision` (精度) の
_Property of a Property_ が挿入されています。

いつものように、正規化された形式と簡潔な形式の両方がサポートされています。不明な属性のデフォルトは _Property_
で、簡潔な _Relationship_ または、_LanguageProperty_ を挿入して、`object` または `languageMap` を期待どおりに含めます。

#### :nine::A: リクエスト:

```console
curl -L -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
    "temperature": {
        "type": "Property",
        "value": 7,
        "observedAt": "2022-03-14T12:51:02.000Z",
        "precision": {
            "value": 0.95,
            "type": "Property",
            "unitCode": "C62"
        }
    }
}'
```

#### :nine::B: リクエスト:

```console
curl -L -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
    "temperature": {
        "value": 7,
        "observedAt": "2022-03-14T12:51:02.000Z",
        "precision": {
            "value": 0.95,
            "unitCode": "C62"
        }
    }
}'
```

#### :one::zero: リクエスト:

`urn:ngsi-ld:City:001` を再取得すると、`temperature` が変化し、新しい `precision` (精度) の _Property of a Property_
が挿入されていることがわかります。

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'attrs=temperature' \
```

#### レスポンス:

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 7,
        "unitCode": "CEL",
        "observedAt": "2022-03-14T12:51:02.000Z",
        "precision": {
            "type": "Property",
            "value": 0.95,
            "unitCode": "C62"
        }
    }
}
```

<a name="removing-existing-attributes-using-merge-patch"></a>

### マージ・パッチを使用して既存の属性を削除 (Removing existing attributes using Merge Patch)

通常、マージ・パッチ操作では削除を示すために `null` が使用されます。ただし、JSON-LD はこれをサポートしていません
(`null` を持つ属性は展開時にペイロードから常に削除されるため)、代わりに NGSI-LD がプレースホルダー値 `urn:ngsi-ld:null`
を使用します。`urn:ngsi-ld:null` は、_Property_, _Relationship_, または _LanguageProperty_ の削除にも同様に有効である
ことに注意してください。以下の簡潔な例では、挿入、更新、および削除を同時に適用できます。

#### :one::one: リクエスト:

```console
curl -L -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:002' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
    "humidity": 80,
    "name": {
        "languageMap": {
            "el": "Βερολίνο",
            "en": "Berlin",
            "it": "Berlino"
        }
    },
    "temperature": "urn:ngsi-ld:null"
}'

```

#### :one::two: リクエスト:

`urn:ngsi-ld:City:002` を再度取得すると、`temperature` が削除され、新しい `humidity` _Property_ が挿入され、
`name` が更新されていることがわかります。

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:002' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'options=concise'
```

#### レスポンス:

```json
{
    "id": "urn:ngsi-ld:City:002",
    "type": "City",
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Viale di Valle Aurelia",
            "addressRegion": "Lazio",
            "addressLocality": "Roma",
            "postalCode": "00138"
        }
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [12.482, 41.893]
        }
    },
    "population": {
        "type": "Property",
        "value": 4342212,
        "observedAt": "2021-01-01T00:00:00.000Z"
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
            "el": "Βερολίνο",
            "en": "Berlin",
            "it": "Berlino"
        }
    },
    "runBy": {
        "type": "Relationship",
        "object": "urn:ngsi-ld:Adminstration:Partito_Democratico"
    },
    "humidity": {
        "type": "Property",
        "value": 80
    }
}
```

<a name="amending-values-of-a-property-with-sub-attributes"></a>

### サブ属性を使用してプロパティの値を修正 (Amending values of a Property with sub-attributes)

#### :one::three: リクエスト:

簡潔な形式を使用すると、JSON オブジェクトの属性と _Properties of Properties_ を区別する必要があります。この場合、
`value` の使用は、更新される `address` オブジェクトの `addressLocality` および `postalCode` であり、`verified`
_Property of a Property_ がメタデータ属性であることを示しています。

```console
curl -L -X PATCH \
    'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
--data-raw '{
    "address": {
        "value": {
            "addressLocality": "Fenerbahçe",
            "postalCode": "34567"
        },
        "verified": "true"
    }
}'
```

#### :one::four: リクエスト:

`urn:ngsi-ld:City:001` を再度取得すると、`address` とそのプロパティ (properties) が更新されていることがわかります。

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'attrs=address' \
```

#### レスポンス:

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Kanlıca İskele Meydanı",
            "addressRegion": "İstanbul",
            "addressLocality": "Fenerbahçe",
            "postalCode": "34567"
        },
        "verified": {
            "type": "Property",
            "value": "true"
        }
    }
}
```

<a name="updating-using-key-values-format"></a>

### Key-Value フォーマットを使用した更新 (Updating using key-values format)

Merge-Patch は、キー・バリュー形式を使用して `values` を更新するための限定的なサポートも提供します。この場合、
既存の `value` は更新されますが、メタデータは変更されません。この場合も、オブジェクト値はサブ属性を送信して
更新するだけでよく、サブ属性を `urn:ngsi-ld:null` に設定すると、サブ属性が削除されます。

これは、キー・バリューのエンティティを **GET** (取得)し、値を **PATCH** （パッチ）して、それを Context Broker
に戻すことができることを意味します。

#### :one::five: リクエスト:

```console
curl -G -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-d 'options=keyValues' \
-H 'Content-Type: application/json' \
--data-raw '{
    "temperature": 19,
    "location": {
        "type": "Point",
        "coordinates": [
            13.3505,
            52.5146
        ]
    },
    "address": {
        "addressLocality": "Beyoğlu",
        "postalCode": "98765"
    },
    "runBy": "urn:ngsi-ld:Adminstration:Adalet_ve_Kalkınma_Partisi"
}'
```

#### :one::six: リクエスト:

`urn:ngsi-ld:City:001` エンティティを再度取得すると、属性が更新されていることがわかります。_Relationship_ `runBy`
はまだ _Relationship_ として定義されていることに注意してください。変更されたのは、`object` の値だけです。

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'attrs=address,temperature,location,runBy' \
```

#### レスポンス:

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 19,
        "unitCode": "CEL",
        "observedAt": "2022-06-30T00:00:00.000Z"
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [13.3505, 52.5146]
        }
    },
    "address": {
        "type": "Property",
        "value": {
            "streetAddress": "Kanlıca İskele Meydanı",
            "addressRegion": "İstanbul",
            "addressLocality": "Beyoğlu",
            "postalCode": "98765"
        }
    },
    "runBy": {
        "type": "Relationship",
        "object": "urn:ngsi-ld:Adminstration:Adalet_ve_Kalkınma_Partisi"
    }
}
```

<a name="updating-using-key-values-with-observedat"></a>

### `observedAt` による  Key-Value を使用した更新 (Updating using key-values with `observedAt`)

`observedAt` は Context Broker 内で一貫した一時データを維持するために非常に重要であるため、マージ・パッチ中に
キー・バリューを更新するときに追加パラメータとして提供されます。既存の _Property_ がすでに `observedAt`
の _Property of a Property_ を使用している場合、タイムスタンプも更新されます。

次の例では、`location` と `temperature` の両方の属性を更新します:

#### :one::seven: リクエスト:

```console
curl -G -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
-d 'options=keyValues' \
-d 'observedAt=2022-10-10T10:10:00.000Z' \
--data-raw '{
    "temperature": 19,
    "location": {
        "type": "Point",
        "coordinates": [
            13.3505,
            52.5146
        ]
    }
}'
```

`urn:ngsi-ld:City:001` エンティティを再度取得すると、属性が更新されていることがわかります。今回はタイムスタンプも
変更されています。

#### :one::eight: リクエスト:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'attrs=temperature,location' \
```

#### レスポンス:

`observedAt` は常に更新されるだけであることに注意してください。 以前に存在しなかった _Property_ には追加されません。

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 19,
        "unitCode": "CEL",
        "observedAt": "2022-10-10T10:10:00.000Z"
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [13.3505, 52.5146]
        }
    }
}
```

<a name="updating-using-key-values-with-lang"></a>

### `lang` による  Key-Value を使用した更新 (Updating using key-values with `lang`)

**GET** を使用してエンティティを取得する場合、`lang` パラメータは、属性の型を `languageMap` から単一の文字列または
文字列配列に切り替えます。これは明らかに損失の多い操作であり、キー・バリューのマージ・パッチが LanguageProperties
を持つエンティティを完全にサポートするためには、単純な文字列値を `languageMap` にマージできる必要があります。

#### :one::nine: リクエスト:

```console
curl -G -X PATCH \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Content-Type: application/json' \
-d 'options=keyValues' \
-d 'lang=en'
--data-raw '{
    "temperature": 19,
    "population": 15850000,
    "name": "Istanbul, not Constantinople"
}'
```

#### :two::zero: リクエスト:

```console
curl -G -X GET \
  'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:001' \
-H 'Link: <http://context/json-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
-H 'Accept: application/json' \
-d 'options=keyValues' \
-d 'attrs=temperature,population,name'
```

#### レスポンス:

ご覧のとおり、英語 (`en`) の `name` 属性は更新されていますが、ギリシャ語 (`el`) とトルコ語 (`tr`)
の値は変更されていません。

```json
{
    "id": "urn:ngsi-ld:City:001",
    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 19,
        "unitCode": "CEL",
        "observedAt": "2022-10-10T10:10:00.000Z"
    },
    "population": {
        "type": "Property",
        "value": 15850000,
        "observedAt": "2022-12-31T00:00:00.000Z"
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
            "el": "Κωνσταντινούπολις",
            "en": "Istanbul, not Constantinople",
            "tr": "İstanbul"
        }
    }
}
```

<a name="overwriting-an-entity-with-put"></a>

### **PUT** によるエンティティの上書き (Overwriting an entity with **PUT**)

上書き操作の場合、既存の属性がペイロード・エンティティに含まれていない場合は削除されます。この例では、
`temperature`, `population` および `name` が更新され、エンティティのその他の属性はすべて削除されます。

いつものように、正規化された形式と簡潔な形式の両方がサポートされています。

#### :two::one::A: リクエスト:

```console
curl -G -X PUT \
 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:002' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '{

    "type": "City",
    "temperature": {
        "type": "Property",
        "value": 25,
        "unitCode": "CEL",
        "observedAt": "2022-06-30T00:00:00.000Z"
    },
    "location": {
        "type": "GeoProperty",
        "value": {
            "type": "Point",
            "coordinates": [
                12.482,
                41.893
            ]
        }
    },
    "name": {
        "type": "LanguageProperty",
        "languageMap": {
            "el": "Ρώμη",
            "en": "Rome",
            "it": "Roma"
        }
    }
}'
```

#### :two::one::B: リクエスト:

```console
curl -G -X PUT \
 'http://localhost:1026/ngsi-ld/v1/entities/urn:ngsi-ld:City:002' \
-H 'Content-Type: application/json' \
-H 'Link: <http://context/ngsi-context.jsonld>; rel="http://www.w3.org/ns/json-ld#context"; type="application/ld+json"' \
--data-raw '{

    "type": "City",
    "temperature": {
        "value": 25,
        "unitCode": "CEL",
        "observedAt": "2022-06-30T00:00:00.000Z"
    },
    "location": {
        "type": "Point",
        "coordinates": [
            12.482,
            41.893
        ]
    },
    "name": {
        "languageMap": {
            "el": "Ρώμη",
            "en": "Rome",
            "it": "Roma"
        }
    }
}'
```

# 次のステップ

高度な機能を追加することで、アプリケーションに複雑さを加える方法を知りたいですか？ このシリーズの
[他のチュートリアル](https://www.letsfiware.jp/ngsi-ld-tutorials)を読むことで見つけることができます

---

## License

[MIT](LICENSE) © 2022 FIWARE Foundation e.V.
