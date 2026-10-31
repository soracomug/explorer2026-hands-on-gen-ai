---
title: 生成AI × SORACOM ハンズオン
---

## このハンズオンについて

Claude Code や Kiro CLI などの生成AIエージェントから SORACOM を操作し、「ドキュメントを調べる → 自分のアカウントを見る・変える → データを分析する」までを一通り体験します。

このドキュメントに載っている生成AIへの依頼内容は、あくまで一例です。言い回しや条件を変えて、いろいろ試してみましょう。

**ゴール**

- SORACOM Knowledge MCP サーバーを生成AIに接続し、公式ドキュメントに基づいた回答を得られる
- 生成AIから自分の SORACOM アカウントの情報（SIM、利用料金、通信量など）を参照し、設定変更を指示できる
- SORACOM Harvest Files に入れたデータを、生成AIと対話しながら分析できる

## タイムテーブル

| 時間 | 内容 |
| :-- | :-- |
| 10分 | イントロ・前提確認 |
| 15分 | STEP 1：SORACOM Knowledge MCP を設定する |
| 25分 | STEP 2：自分のアカウントを参照・設定変更する |
| 25分 | STEP 3：Harvest Files のデータを分析する |
| 15分 | まとめ・後片付け（Appendix は時間があれば） |

## 事前準備

### 必要なもの

- PC（macOS / Windows / Linux）とインターネット接続
- SORACOM アカウント（ユーザーコンソールにログインできること）
- 生成AIクライアント：Claude Code、Kiro CLI、Claude Desktop のいずれか（Claude Desktop は STEP 2 の SORACOM MCP Server を .mcpb でインストールします）
- Node.js v22 以上（STEP 2 の SORACOM MCP Server を npx で起動するため）
- SORACOM CLI（2-2 でインストールします）
- STEP 3 用：運営が配布するデータファイル（当日配布）。Appendix のみ GPS マルチユニット SORACOM Edition を使用

### 生成AIのプランについて（推奨）

入力内容がモデルの学習に使われないよう、Pro 以上のプランや、学習に利用されない設定のアカウントでの参加を推奨します。STEP 2・3 では自分のアカウント情報やデバイスデータを生成AIに渡すため、利用するAIサービスの規約・プライバシー設定を事前に確認してください。

## STEP 1：生成AIから SORACOM Knowledge MCP を設定する

SORACOM Knowledge MCP サーバーは、AIエージェントから SORACOM のユーザーガイド、料金、IoTレシピ、APIリファレンス、CLI コマンド、SAM 権限などを検索できる読み取り専用の MCP サーバーです。認証不要・無償で、アカウントやリソースを変更することはありません。

エンドポイント：`https://knowledge-mcp.soracom.com`（Streamable HTTP）

### 1-1. Claude Code の場合

```bash
claude mcp add --transport http soracom-knowledge https://knowledge-mcp.soracom.com
claude mcp list
```

`soracom-knowledge` が表示され、3つのツール（`search_soracom_docs` / `search_api_docs` / `get_document`）が使えればOKです。

### 1-2. Kiro CLI の場合

Kiro CLI の MCP 設定ファイル（例：`~/.kiro/settings/mcp.json`）に以下を追加します。

```json
{
  "mcpServers": {
    "soracom-knowledge": {
      "url": "https://knowledge-mcp.soracom.com"
    }
  }
}
```

### 1-3. 動作確認：聞いてみよう

- 「SORACOM Harvest Data の無料利用枠と料金を教えて」
- 「plan-D と plan01s の違いを、SORACOM のドキュメントを根拠に比較して」
- 「SIM のグループを変更する SORACOM CLI のコマンドと必要な SAM 権限は？」

回答に出典URLが付いているか、MCP のツールが呼ばれているかを確認しましょう。

> **注意：** このエンドポイントにはユーザーごとの認証がなく、入力した情報がそのまま送信されます。機密情報、アカウント識別子、認証情報、個人情報を含めないでください。

## STEP 2：生成AIからご自身のアカウントデータを参照・設定変更する

SORACOM MCP Server（Early Access）を使うと、生成AIから SORACOM API を呼び出してアカウント情報を取得できます。

### 2-1. 認証キーを用意する（SAM ユーザー推奨）

1. ユーザーコンソールで、ハンズオン用の SAM ユーザーを作成します。（参考：[SAM ユーザーを作成する](https://users.soracom.io/ja-jp/docs/sam/create-sam-user/)）
2. 必要な API だけを許可した権限を付与します（参照系＋STEP 2-4 で使う変更系＋STEP 3 で使う FileEntry:putFile / listFiles / getFile / deleteFile / deleteDirectory＋Appendix で使う SoracomAir への書き込みと DataEntry:getDataEntries）。（参考：[SAM ユーザーの権限を設定する](https://users.soracom.io/ja-jp/docs/sam/set-permissions/)。権限を変更した場合は API キーと API トークンの再発行が必要です）
3. SAM ユーザーの認証キー（AuthKey ID / AuthKey）を発行します。（参考：[認証キーを生成する](https://users.soracom.io/ja-jp/docs/sam/create-sam-user/#%E8%AA%8D%E8%A8%BC%E3%82%AD%E3%83%BC%E3%82%92%E7%94%9F%E6%88%90%E3%81%99%E3%82%8B)）

ルートユーザーの認証キーは使わないでください。ハンズオン終了後は認証キーを削除します。（参考：[ルートユーザーの認証情報を管理する](https://users.soracom.io/ja-jp/guides/basic-knowledge/manage-root-user-credentials/)、[SAM ユーザーの認証キーの削除](https://users.soracom.io/ja-jp/docs/sam/create-sam-user/#%E8%AA%8D%E8%A8%BC%E3%82%AD%E3%83%BC%E3%82%92%E7%94%9F%E6%88%90%E3%81%99%E3%82%8B)）

ハンズオン用の SAM 権限ポリシーです（STEP 2・3 と Appendix で使う API だけを許可）。SAM ユーザーの [権限設定] に貼り付けます。

```json
{
  "statements": [
    {
      "effect": "allow",
      "api": [
        "Sim:listSims",
        "Sim:getSim",
        "Sim:listSimSessionEvents",
        "Sim:listSimStatusHistory",
        "Query:searchSims",
        "Group:listGroups",
        "Group:getGroup",
        "Stats:getAirStatsOfOperator",
        "Stats:getAirStatsOfSim",
        "Stats:getAirStatsOfGroup",
        "Billing:getBillingHistory",
        "Billing:getBilling",
        "Billing:getLatestBilling",
        "Billing:getBillingSummaryOfBillItems",
        "Billing:getBillingSummaryOfSims",
        "CellLocation:batchGetCellLocations"
      ]
    },
    {
      "effect": "allow",
      "api": [
        "Sim:putSimTags",
        "Sim:setSimGroup",
        "Sim:unsetSimGroup",
        "Group:createGroup"
      ]
    },
    {
      "effect": "allow",
      "api": [
        "Group:putConfigurationParameters"
      ],
      "condition": "pathVariable('namespace') == 'SoracomHarvest'"
    },
    {
      "effect": "allow",
      "api": [
        "Group:putConfigurationParameters"
      ],
      "condition": "pathVariable('namespace') == 'SoracomAir'"
    },
    {
      "effect": "allow",
      "api": [
        "FileEntry:listFiles",
        "FileEntry:findFiles",
        "FileEntry:getFile",
        "FileEntry:getFileMetadata",
        "FileEntry:putFile",
        "FileEntry:deleteFile",
        "FileEntry:deleteDirectory"
      ]
    },
    {
      "effect": "allow",
      "api": [
        "DataEntry:getDataEntries"
      ]
    }
  ]
}
```

速度クラスの変更、SIM の解約・休止、他サービスのグループ設定など、課金や復旧に影響する操作は許可していません。グループ設定の変更は、Harvest Data の有効化（SoracomHarvest）と、GPS マルチユニットの動作モードの保存先（SoracomAir）に限っています。

### 2-2. SORACOM CLI と SORACOM MCP Server を設定する

2-1 で発行した認証キーを、SORACOM CLI のプロファイル「handson」としてローカル PC に保存します。このプロファイルは、MCP Server と、STEP 2-4・STEP 3 の CLI 操作で共通して使います。

#### (1) SORACOM CLI をインストールする

macOS / Linux（Homebrew を利用）：

```bash
brew tap soracom/soracom-cli
brew install soracom-cli
```

Windows：[Releases ページ](https://github.com/soracom/soracom-cli/releases)から `soracom_[バージョン]_windows_amd64.zip` をダウンロードして解凍し、`soracom.exe` を PATH の通ったフォルダに置きます。

インストールできたか確認します。

```bash
soracom version
```

詳しくは [SORACOM CLI をインストールする](https://users.soracom.io/ja-jp/tools/cli/getting-started/) を参照してください。

#### (2) 認証キーをプロファイルに保存する

```bash
LANG=ja soracom configure --profile handson
```

Windows（PowerShell）では `LANG=ja` を付けずに `soracom configure --profile handson` を実行します。質問には次のように答えます。

- カバレッジタイプ：2（Japan）を選択します。plan01s などグローバルカバレッジの SIM を使う場合は 1（Global）を選択します。
- 認証方法：1（AuthKeyId と AuthKey を入力する）を選択します。
- authKeyId / authKey：2-1 で発行した認証キー ID と認証キーシークレットを入力します。

プロファイルは `~/.soracom/handson.json`（Windows は `%HOMEPATH%\.soracom\handson.json`）に保存されます。認証情報が含まれるため、取り扱いに注意してください（参考：[プロファイルについて](https://users.soracom.io/ja-jp/tools/cli/profiles/)）。

#### (3) 動作確認

```bash
soracom sims list --profile handson
```

SIM の一覧が返れば OK です（SIM がない場合は `[]` が返ります）。

#### (4) SORACOM MCP Server を設定する

MCP Server の起動には Node.js v22 以上が必要です。認証情報とカバレッジタイプは、プロファイル handson の設定が使われます。

Claude Code の場合：

```bash
claude mcp add soracom -e SORACOM_PROFILE=handson -- npx @soracom-labs/soracom-mcp-server
```

Kiro CLI の場合は、MCP 設定ファイル（例：`~/.kiro/settings/mcp.json`）の `mcpServers` に以下を追加します。1-2 で追加した `soracom-knowledge` と同じ `mcpServers` の中に並べて書きます。

```json
{
  "mcpServers": {
    "soracom": {
      "command": "npx",
      "args": ["@soracom-labs/soracom-mcp-server"],
      "env": {
        "SORACOM_PROFILE": "handson"
      }
    }
  }
}
```

Claude Desktop の場合（Claude Code が使えない Free プランの方など）は、ハンズオン用の拡張機能パッケージ [soracom-handson.mcpb](soracom-handson.mcpb) をダウンロードして、Claude Desktop の [設定] → [拡張機能] の画面にドラッグ＆ドロップし、インストールします。プロファイル名を聞かれたら `handson` のままにします。認証情報はパッケージに含まれておらず、(2) で保存したプロファイル handson のものが使われます。

### 2-3. アカウントデータを参照してみよう

- 「自分の SIM を一覧で見せて。名前、ステータス、所属グループ、セッション状態を表にして」
- 「今月の利用料金の見込みと、過去数か月の推移をグラフにして」
- 「直近7日間で通信量が多い SIM はどれ？」
- 「オンラインの SIM が接続している基地局のおおよその位置を表にして」

### 2-4. 設定変更してみよう

> **重要：** SORACOM MCP Server（Early Access）で利用できるツールは、現時点では SIM・グループ・統計・請求などの参照系が中心です。設定変更は、生成AIに SORACOM CLI のコマンドを組み立てて実行させる形で行います。

1. 2-2 で作成したプロファイル handson を使います（SORACOM CLI のコマンドには `--profile handson` を付けます）。
2. 生成AIに変更を依頼します。STEP 1 の Knowledge MCP で正しいコマンドを調べさせると精度が上がります。
3. 実行前に、生成AIが提示したコマンドの内容を必ず自分で確認してから承認します。

依頼の例：

- 「SIM ○○ の名前を『handson-01』に変更して」
- 「『SORACOM UG Handson』という SIM グループを作って、SIM ○○ を所属させて」
- 「そのグループで SORACOM Harvest Data を有効化して」

変更後、ユーザーコンソールや STEP 2-3 の参照で結果を確認しましょう。

> **注意：** SIM の所属グループを変えると、元のグループの設定（Harvest Data、Beam、GPS マルチユニットの動作モードなど）は適用されなくなります。ふだん使っている SIM で試す場合は、変更前の名前と所属グループを控えておき、後片付けで元に戻しましょう。なお、速度クラスの変更など、2-1 のポリシーで許可していない操作を依頼すると権限エラーになります。

## STEP 3：SORACOM Harvest Files にデータを入れて分析してみる

### 3-1. 配布データを準備する

運営が配布するデータファイル（イベントで取得した位置情報データ）をダウンロードし、Claude Code / Kiro CLI を起動する作業フォルダに置きます。

### 3-2. 生成AIから Harvest Files にアップロードする

SORACOM MCP Server（Early Access）には Harvest Files を扱うツールがないため、STEP 2-4 と同じく生成AIに SORACOM CLI を実行させてアップロードします（プロファイル handson を使用）。

依頼の例：

- 「作業フォルダの `<配布ファイル名>` を SORACOM Harvest Files の /handson/ にアップロードして。SORACOM CLI のプロファイル handson を使って」

生成AIが実行するコマンドの例：

```bash
soracom files put --scope private --path /handson/<配布ファイル名> --content-type text/csv --body @<配布ファイル名> --profile handson
soracom files list --scope private --path /handson/ --profile handson
```

実行前に、アップロード先のパスなどコマンドの内容を確認してから承認します。アップロードしたファイルは、ユーザーコンソールの [SORACOM Harvest Files] 画面でも確認できます。

ハンズオン用 SAM ユーザーには FileEntry:putFile / listFiles / getFile / deleteFile / deleteDirectory の権限が必要です（2-1 のポリシーに含まれています）。

> **注意：** Harvest Files は書き込みと読み出しのそれぞれに料金が発生します。また、ファイルの中身は分析時に生成AIのサービスへ送信されます。

### 3-3. 生成AIで分析する

生成AIに Harvest Files からファイルを取得させ、分析を依頼します。

生成AIが実行するコマンドの例：

```bash
soracom files get --scope private --path /handson/<配布ファイル名> --profile handson > downloaded-<配布ファイル名>
```

分析の依頼例：

- 「Harvest Files の /handson/ にあるファイルを取得して、データの概要（期間、件数、カラム）を教えて」
- 「位置情報を地図上にプロットした HTML を作って」
- 「デバイスごとの移動距離と、滞在時間が長い場所を表にして」
- 「データが欠けている時間帯や、位置が飛んでいる異常値を見つけて」

生成AIがどのコマンドを実行し、どんな手順で分析したかもあわせて確認しましょう。

> **注意：** 依頼の仕方によっては、生成AIが Harvest Files ではなく、作業フォルダにあるローカルのファイルを使って分析することがあります。その場合は「SORACOM Harvest Files のファイルを使って」と指示するか、作業フォルダの配布ファイルを削除してから依頼してください。

### 3-4. アップロードしたファイルを削除する

分析が終わったら、生成AIに Harvest Files のファイルを削除させます。

依頼の例：

- 「Harvest Files の /handson/ にあるファイルの一覧を見せてから、すべて削除して。空になった /handson/ ディレクトリも削除して」

生成AIが実行するコマンドの例：

```bash
soracom files delete --scope private --path /handson/<配布ファイル名> --profile handson
soracom files delete-directory --scope private --path /handson/ --profile handson
soracom files list --scope private --path / --profile handson
```

> **注意：** 削除したファイルは元に戻せません。生成AIが提示したコマンドの対象パスを確認してから承認しましょう。削除後は、ユーザーコンソールの [SORACOM Harvest Files] 画面でもファイルが消えていることを確認します。作業フォルダに取得したファイルも、不要であれば削除します。

## Appendix：GPS マルチユニットを生成AIで設定・分析してみよう

GPS マルチユニット SORACOM Edition（以下、GPS マルチユニット）は、位置情報（GPS）・温度・湿度・加速度センサーとバッテリーを内蔵したデバイスです。GPS マルチユニットをお持ちの方向けの内容です。

GPS マルチユニットの動作モード（送信内容や送信間隔など）は、SIM が所属するグループの SORACOM Air for セルラー設定（userdata）に JSON で保存されています。この JSON を生成AIに読み書きさせて、設定を変更してみます。

### A-1. GPS マルチユニットがあるか確認する

GPS マルチユニットとして登録した SIM には、タグ `soracom.gadgets`（値：`GPS-Multiunit`）が付いています。

依頼の例：

- 「タグの値が GPS-Multiunit の SIM を探して、SIM ID・IMSI・名前・所属グループを表にして」

見つからない場合は、ユーザーコンソールの [メニュー] → [ガジェット管理] → [GPS マルチユニット] で一度登録してください（参考：[動作モードを SORACOM ユーザーコンソールで設定する](https://users.soracom.io/ja-jp/guides/iot-devices/gps-multiunit/setting-mode-on-console/)）。

### A-2. Knowledge MCP で設定 JSON の仕様を確認する

依頼の例：

- 「GPS マルチユニットの動作モードを SORACOM CLI で設定する方法と、userdata の JSON の仕様を SORACOM のドキュメントで調べて。送信間隔はどこで指定する？」

「[SORACOM CLI / SORACOM API で動作モードを設定する](https://users.soracom.io/ja-jp/guides/iot-devices/gps-multiunit/configure-with-cli/)」のページが参照されれば OK です。定期送信 - 手動モードの送信間隔は、`Common.C5` の 6 番目の要素（インデックス 5）に 1〜1440（分）で指定します。

### A-3. 現在の設定値をローカルに保存する

依頼の例：

- 「GPS マルチユニットの SIM が所属するグループの設定を取得して、SORACOM Air for セルラー設定の userdata を gps-config-before.json に保存して。中身も説明して」

生成AIが実行するコマンドの例：

```bash
soracom groups get --group-id <グループID> --profile handson
```

取得した結果の `configuration.SoracomAir` に `userdata` が含まれます。`gps-config-before.json` は、後片付けで元の設定に戻すときにも使います。

### A-4. 送信間隔などを変えてみる

依頼の例：

- 「gps-config-before.json をコピーして gps-config-after.json を作り、定期送信の送信間隔を 5 分に変えて。変更前後の差分も見せて」

> **注意：** ユーザーコンソールで設定できる範囲を超える値は指定しないでください。GPS マルチユニットが正常に動作しなくなることがあります。シナリオ 0〜3 の値と、C7 の組み合わせは変えないようにしましょう。

### A-5. 設定を反映する

依頼の例：

- 「gps-config-after.json の内容を、そのグループの SORACOM Air for セルラー設定の userdata に反映して。metadata の設定は変えないで」

生成AIが実行するコマンドの例：

```bash
soracom groups put-config --group-id <グループID> --namespace SoracomAir --body @put-body.json --profile handson
```

`put-body.json` は、`[{"key": "userdata", "value": <gps-config-after.json の内容>}]` の形式のファイルです（生成AIに作らせます）。

グループに保存しただけでは、本体には反映されません。GPS マルチユニットのファンクションボタンを 1 秒押して、設定を本体にダウンロードします（同時にデータが送信されます）。

### A-6. 設定値を再確認する

依頼の例：

- 「もう一度グループの設定を取得して、userdata が gps-config-after.json と一致しているか確認して」

しばらく待ってから A-7 でデータを取得すると、送信間隔が変わったことも確認できます。

### A-7. GPS マルチユニットのデータを生成AIに分析させる

GPS マルチユニットのデータは、送信先で Harvest Data を有効にしていれば Harvest Data に保存されています。生成AIに取得させて分析します（参考：[送信されるデータフォーマット](https://users.soracom.io/ja-jp/guides/iot-devices/gps-multiunit/format/)）。

生成AIが実行するコマンドの例：

```bash
soracom data get-entries --resource-type Subscriber --resource-id <IMSI> --limit 1000 --profile handson > gps-data.json
```

分析の依頼例：

- 「GPS マルチユニットの直近 24 時間の Harvest Data を取得して gps-data.json に保存し、データの概要を教えて」
- 「位置情報を地図上にプロットした HTML を作って。lat / lon が null のデータは除いて」
- 「温度・湿度・バッテリー残量の推移をグラフにして」
- 「A-5 で設定を変えた前後で、送信間隔が実際に変わったか確認して」

**ポイント**

- 送信間隔の最小は1分です。ハンズオン中は短く、終了後は長めに戻しましょう。
- GPS が取得しにくい屋内では、lat / lon が null になったり、送信間隔が延びたりします。窓際での実施がおすすめです。
- データ通信（上り・設定取得の下り）はすべて課金対象です。

## 後片付け

- ハンズオン用 SAM ユーザーの認証キーを削除する（参考：[API キーと API トークンを無効化する](https://users.soracom.io/ja-jp/tools/api/key-and-token/#api-%E3%82%AD%E3%83%BC%E3%81%A8-api-%E3%83%88%E3%83%BC%E3%82%AF%E3%83%B3%E3%82%92%E7%84%A1%E5%8A%B9%E5%8C%96%E3%81%99%E3%82%8B)）
- Harvest Files にアップロードした配布ファイルを削除する（STEP 3-4 参照）
- 2-4 で変更した SIM の名前と所属グループを元に戻す
- 2-4 で作成した「SORACOM UG Handson」グループの Harvest Data を無効化する（グループが不要な場合は、ユーザーコンソールから削除します）
- GPS マルチユニットの動作モードを元に戻す（A-3 で保存した `gps-config-before.json` を生成AIに反映させる）か、電源を切る
- 生成AIクライアントの MCP 設定から認証情報を削除する

## 参考リンク

- [SORACOM Knowledge MCP サーバー](https://users.soracom.io/ja-jp/tools/soracom-knowledge-mcp-server/)
- [SORACOM MCP Server（GitHub）](https://github.com/soracom-labs/soracom-mcp-server)
- [SORACOM CLI をインストールする](https://users.soracom.io/ja-jp/tools/cli/getting-started/)
- [SORACOM Harvest Files にファイルをアップロードする](https://users.soracom.io/ja-jp/docs/harvest/send-files/)
- [Harvest Data にデータを蓄積する](https://users.soracom.io/ja-jp/guides/getting-started/send-data-to-harvest-data/)
- [GPS マルチユニット：動作モードをユーザーコンソールで設定する](https://users.soracom.io/ja-jp/guides/iot-devices/gps-multiunit/setting-mode-on-console/)
- [GPS マルチユニット：SORACOM CLI / SORACOM API で動作モードを設定する](https://users.soracom.io/ja-jp/guides/iot-devices/gps-multiunit/configure-with-cli/)
