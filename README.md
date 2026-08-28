# Resource Graphで効率的にリソース情報を取得する

## 概要

Azure環境の調査や作業を行う際、以下のようなケースがあります。

- 複数のサブスクリプションから情報を集めたい
- Azure Portalで1つずつリソースを確認するのが大変
- NSGやAzure Firewallなどの設定情報をまとめて確認したい

このような、**Azure上の大量のリソース情報を横断的に検索・取得したい場合**に便利なのが  
**Azure Resource Graph**です。

Azure Resource Graphでは、Azure上のリソース情報を  **KQL（Kusto Query Language）**を使って検索できます。

---

# Resource Graphを使うとよい場面

## 1. Azure Portalから1つずつ確認するのが大変なとき

Azure Portalからリソース情報を確認する場合、対象のサブスクリプションやリソースを開いて確認します。

対象が数個であれば問題ありませんが、例えば以下のような環境では確認に時間がかかります。

- サブスクリプションが20個ある
- 仮想マシンが100台ある
- NSGが50個ある

Resource Graphを利用すると、KQLを使って必要な情報をまとめて検索できます。

### 例：仮想マシンの一覧を取得する

```kusto
Resources
| where type =~ "microsoft.compute/virtualmachines"
| project subscriptionId, resourceGroup, name, location
```

このように、必要な情報だけを表形式で取得できます。

### Portalでの確認

```text
サブスクリプションを開く
        ↓
リソースを探す
        ↓
対象リソースを開く
        ↓
設定を確認する
        ↓
次のリソースを開く
        ↓
・・・
```

対象リソースが増えるほど、確認作業も増えていきます。

一方、Resource Graphでは以下のように検索できます。

```text
Resource Graph Explorer
        ↓
KQLを実行
        ↓
条件に一致するリソースを一覧表示
```

そのため、**「Portalで1つずつ確認するのは大変」**と感じた場合は、Resource Graphの利用を検討できます。

---

## 2. 複数サブスクリプションを横断して検索したいとき

Resource Graphの大きなメリットの1つが、**複数のサブスクリプションを横断して検索できること**です。

通常のAzure CLIやAzure PowerShellでは、操作対象となるサブスクリプションを意識してコマンドを実行します。

例えば、

```text
サブスクリプションA
        ↓
リソース情報取得

サブスクリプションB
        ↓
リソース情報取得

サブスクリプションC
        ↓
リソース情報取得
```

のように、サブスクリプションごとに情報を取得する必要が出てくる場合があります。

これだと、繰り返し処理でサブスクリプションを切り替えて検索をするということが必要になるので、スクリプト全t内の実行時間が長くなるということがデメリットになります。

Resource Graphを利用すると、アクセス権を持っている複数のサブスクリプションを対象として検索できます。
例えば、以下のような調査に向いています。
- 全サブスクリプションの仮想マシンを一覧化したい
- 特定の名前を持つリソースを探したい
- 特定のIPアドレスを使用しているリソースを探したい
- 特定のタグが設定されているリソースを探したい

### イメージ

```text
Subscription A ─┐
                │
Subscription B ─┼──→ Resource Graph ──→ 検索結果
                │
Subscription C ─┘
```

複数のサブスクリプションを管理している環境ほど、Resource Graphを利用するメリットが大きくなります。

---

## 3. NSGやAzure Firewallなどの設定情報をまとめて確認したいとき

Resource Graphは、単純なリソース一覧だけではなく、  **リソースが持っている設定情報を調査する用途**にも利用できます。

例えば、以下のような情報を調査できます。

- NSG：セキュリティルール
- Azure Firewall / Firewall Policyの穴あけルール
- VNet：
    - Subnet
    - ピアリング
    - IP範囲
- 各リソースのSKU
- タグ

### 例：NSGを検索する

```kusto
Resources
| where type =~ "microsoft.network/networksecuritygroups"
| project subscriptionId, resourceGroup, name, location, properties
```

さらに、`properties` の中から必要な情報を取り出すことで、設定値を一覧化できます。

例えば、Azure PortalでNSGのルールを確認する場合、

```text
NSG-A
 ├─ Rule01
 ├─ Rule02
 └─ Rule03

NSG-B
 ├─ Rule01
 ├─ Rule02
 └─ Rule03
```

のように、それぞれのNSGを開いて確認することになります。

Resource Graphを利用すると、

```text
NSG名 | Rule名 | Source | Destination | Port | Action
------------------------------------------------------
NSG-A | Rule01 | xxx    | xxx         | 443  | Allow
NSG-A | Rule02 | xxx    | xxx         | 22   | Deny
NSG-B | Rule01 | xxx    | xxx         | 443  | Allow
```

のように、複数リソースの設定を一覧として確認することもできます。

そのため、Resource Graphは

**「Azure上に現在どのような設定が存在しているのかを調査する」**

という用途でも便利です。

---

## 4. 条件を指定して対象リソースを絞り込みたいとき

Resource GraphではKQLを利用するため、さまざまな条件でリソースを検索できます。

例えば、

- 特定のリソースタイプ
- 特定のリージョン
- 特定のリソース名
- 特定のタグ
- 特定の設定値

などを条件として検索できます。

### 例：Japan Eastの仮想マシンを検索

```kusto
Resources
| where type =~ "microsoft.compute/virtualmachines"
| where location =~ "japaneast"
| project subscriptionId, resourceGroup, name, location
```

単純に「VMを全部取得する」のではなく、

**「条件に該当するリソースだけを探す」**

という使い方ができます。

---

## 5. 取得した情報を後続処理で利用したいとき

Resource Graphで取得した情報は、調査だけでなく、  
後続のAzure CLIやAzure PowerShellの処理にも利用できます。

例えば、

1. Resource Graphで対象リソースを検索
2. Resource IDやリソース名を取得
3. 取得結果を変数に格納
4. Azure CLI / Azure PowerShellのパラメータとして利用

という使い方ができます。

例えば、

**「特定条件に該当するVMをResource Graphで探して、そのVMに対して処理を行う」**

といった自動化ができます。

この場合、

```text
Resource Graph
    ↓
リソースを「探す」

Azure CLI / Azure PowerShell
    ↓
リソースを「削除する」※コマンドのパラメータに、リソース名、idなどを指定する。
```

という役割分担ができます。

---

# Resource Graphのメリット

Resource Graphを利用する主なメリットは以下です。

| メリット | 内容 |
|---|---|
| 高速に検索できる | 大量のAzureリソースから必要な情報を検索できる |
| 複数サブスクリプションを横断できる | サブスクリプションを1つずつ切り替えて調査する必要がない |
| KQLで条件を指定できる | リソース種類、名前、タグ、設定値などから対象を絞り込める |
| 必要な情報だけ取得できる | `project` などを利用して必要な列だけ表示できる |
| 設定情報を一覧化できる | NSGやAzure Firewallなど、大量の設定をまとめて確認できる |
| 後続処理に利用できる | Resource IDなどを取得してCLIやPowerShellのパラメータに利用できる |
| 調査を再現できる | 同じKQLを実行することで、同じ条件で再調査できる |

---

# Azure Portalとの使い分け

Resource Graphですべてを行う必要はありません。
対象リソースが少ない場合や、1つのリソースの詳細を確認したい場合はAzure Portalの方が簡単です。

一方で、対象が増えるほどResource Graphのメリットが大きくなります。

| やりたいこと | 向いている方法 |
|---|---|
| 1台のVMの設定を確認したい | Azure Portal |
| 1つのNSGルールを確認したい | Azure Portal |
| 全VMを一覧化したい | Resource Graph |
| 複数サブスクリプションからリソースを探したい | Resource Graph |
| 条件に一致するリソースを探したい | Resource Graph |
| NSGのルールをまとめて確認したい | Resource Graph |
| Azure Firewallの設定を一覧化したい | Resource Graph |
| リソースを作成・変更・削除したい | Azure CLI / Azure PowerShell |

---

# Azure CLI / Azure PowerShellとの違い

Resource GraphとAzure CLI / Azure PowerShellは、競合するものではなく、用途によって使い分けます。

大まかには、

```text
Resource Graph
    ↓
検索・調査が得意

Azure CLI / Azure PowerShell
    ↓
操作・変更が得意
```

と考えると分かりやすいです。

例えば、

```text
① Resource Graph

「条件に該当するリソースはどれ？」

            ↓

② Resource IDなどを取得

            ↓

③ Azure CLI / Azure PowerShell

「見つけたリソースに対して処理を実行」
```

という組み合わせができます。

---

# Resource Graphの考え方

Resource Graphは、

**「Azureリソースを効率的に検索・調査するためのもの」**

と考えると分かりやすいです。

```text
                    Azure Portal
                         │
                  画面から確認
                         │
                         ▼
                ┌─────────────────┐
                │ Azure Resources │
                └─────────────────┘
                    ▲          ▲
                    │          │
                検索・調査    操作・変更
                    │          │
             Resource Graph   │
                               │
                      CLI / PowerShell
```

特に、

**「Portalで1つずつ見るのは面倒だな」**

と思ったときは、

**「Resource Graphでまとめて取得できないか？」**

と考えてみるとよいです。

---

#
# まとめ

Azure Resource Graphは、Azure上のリソース情報をKQLで効率的に検索するためのサービスです。

特に以下のような場面で有効です。

- Azure Portalで1つずつ確認するのが大変
- 大量のリソースから条件に一致するものを探したい
- 複数サブスクリプションを横断して検索したい
- NSGやAzure Firewallなどの設定を一覧化したい
- Resource IDなどを取得して後続処理に利用したい

Azure環境の規模が大きくなるほど、Portalだけですべてを確認するのは大変になります。

そのため、

**Resource Graphで「検索・調査」し、  
Azure CLI / Azure PowerShellで「操作する」**

という使い分けが有効です。

---

# 次に覚えること

Resource Graphの用途を理解したら、次は実際にResource Graph Explorerを使ってKQLを書いてみます。

まずは、以下の基本的なKQLの使い方を覚えると便利です。

| KQL | 用途 |
|---|---|
| `where` | 条件を指定して絞り込む |
| `project` | 表示する列を指定する |
| `extend` | 新しい列を作成する |
| `sort by` | 結果を並び替える |
| `summarize` | 集計する |
| `mv-expand` | 配列になっている情報を行に展開する |

これらを組み合わせることで、

**「大量のAzureリソースから必要な情報だけを取り出す」**

ことができるようになります。
