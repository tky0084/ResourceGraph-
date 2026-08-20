実際にResourceGraphを使って、AzureFirewallのルール一覧を表示するためのクエリを作成してみよう

# AzureFirewallのルールのリソース構成について
AzureFirewallは主にAzureFirewall本体と、AzureFirewallポリシーで成り立っています。

一般的にportalで穴あけルールを作る際、Azurefirewallポリシーに対して作業をするように見えます。
しかし、JSONファイルを見てみましょう。
どこにも穴あけルールについての記載がありません。
![alt text](<スクリーンショット 2026-08-18 085810.png>)

では、どこにAzureFirewallの穴あけが記載されているのでしょうか。

ARMテンプレートを見てみましょう。
すると、リソースが3つ存在します。
その中には、
"type": "Microsoft.Network/firewallPolicies/ruleCollectionGroups
という記載が2つ存在しますね。

![alt text](<スクリーンショット 2026-08-18 090516.png>)

そう、AzureFirewallの穴あけルールは、GUI上ではFirewallポリシーに対して作業をしているように見えても、
隠れたリソースであるruleCollectionGroupの中に記載されているのです。

ResourceGraphでは、これに対して検索をする必要があります。

---

# ResourceGraphで検索

それでは、ResourceGraphで以下のクエリを打ってみましょう。

```
resources
| where type =~ "Microsoft.Network/firewallPolicies/ruleCollectionGroups"
```

しかし、結果は何も表示されません。
![alt text](<スクリーンショット 2026-08-18 090913.png>)

では、テーブル検索欄に、リソースタイプ「Microsoft.Network/firewallPolicies/ruleCollectionGroups」を検索してみましょう。
すると、規則コレクショングループは、networkresourcesというテーブルに入っていることがわかります。

![alt text](<スクリーンショット 2026-08-18 091042.png>)

では、テーブル名を直して、もう一度実行してみましょう。
これで検索がかかったと思います。
![alt text](<スクリーンショット 2026-08-18 091253.png>)

# ルールが記載されたプロパティを列に外出しする
このクエリ結果には、直接記載ルールが1行ずつ出るわけではありません。
必要な項目を外に出すという方法をとります。

では、1行目を見てみましょう。
これを見ると、propertiesという列の中の、ruleCollectionsという配列がいます。
ruleCollectionsは穴あけルールをグループ化するものですので、まずはこれを外に出します。

![alt text](<スクリーンショット 2026-08-18 091545.png>)

では、クエリを以下のように追記してみましょう。
extendで列内の特定の項目を別の列に拡張します。
今回は、右辺のproperties.ruleCollections(propertiesの中のruleCollections)を、左辺のruleCollectionsという名前(任意)として拡張します。

すると、ruleCollectionsという名前の列が作られて、ruleCollectionsの中身だけが記載されていると思います。
```
networkresources
| where type =~ "Microsoft.Network/firewallPolicies/ruleCollectionGroups"
| extend ruleCollections = properties.ruleCollections
```

![alt text](<スクリーンショット 2026-08-18 092038.png>)

しかし、これでは不十分です。
なぜなら、ExtendはpropertiesのruleCollectionsをそのまま外に出しているだけです。

```
ruleCollections: [
     { "name": "NetworkRuleCollection01", "ruleCollectionType": "FirewallPolicyFilterRuleCollection" },
     { "name": "NetworkRuleCollection02", "ruleCollectionType": "FirewallPolicyFilterRuleCollection" } 
]
```

このままでは、複数の規則コレクションが1つのセルにまとまっているため、それぞれの情報を扱いにくい状態です。

そこで使用するのが mv-expand です。
mv-expandとは

mv-expand は、配列に格納されている複数の要素を、それぞれ別の行に展開するための演算子です。

mv は multi-value（複数の値） を意味します。

たとえば、以下のように1つの行に配列が格納されていたとします。

["RuleCollection01", "RuleCollection02", "RuleCollection03"]

これに mv-expand を使用すると、

RuleCollection01
RuleCollection02
RuleCollection03

というように、配列の要素ごとに行が分割されます。

Azure Firewallの場合、1つの規則コレクショングループの中に複数の ruleCollections が格納されています。

そのため、

| mv-expand ruleCollections

とすることで、それぞれの規則コレクションを1行ずつに展開できます。

では、extendを書き換えましょう。
すると、ruleCollectionsの中にある{}の内容が、ruleCollectionsの列名で、1つあたり1行ずつ展開されていることがわかります。

```
networkresources
| where type =~ "Microsoft.Network/firewallPolicies/ruleCollectionGroups"
| mv-expand ruleCollections = properties.ruleCollections
```

![alt text](<スクリーンショット 2026-08-21 083317.png>)

さらに、規則コレクションにはルールが内包されていますね。
これも同じように展開しましょう。、ただし展開元はpropertiesではなく、ruleCollectionsです。

```
networkresources
| where type =~ "Microsoft.Network/firewallPolicies/ruleCollectionGroups"
| mv-expand ruleCollections = properties.ruleCollections
| mv-expand rules = ruleCollections.rules
| project name, ruleCollections, rules
```

ルールが1行ずつ展開されました。
ここから、さらに細かくextendやmv-expandを使って列を作って展開することもできます。
![alt text](<スクリーンショット 2026-08-21 084408.png>)

アプリケーションルールに対して必要な行を展開しました。

```
networkresources
| where type =~ "Microsoft.Network/firewallPolicies/ruleCollectionGroups"
| mv-expand ruleCollections = properties.ruleCollections
| mv-expand rules = ruleCollections.rules
| project name, ruleCollections, rules
| extend ruleCollection = ruleCollections.name
| extend priority = ruleCollections.priority
| extend action = ruleCollections.action.type
| extend ruleName = rules.name
| extend sourceAddresses = rules.sourceAddresses
| extend protocols = rules.protocols
| extend targetFqdns = rules.targetFqdns
| extend ruleType = rules.ruleType
| extend ruleType = rules.targetUrls
| extend ruleType = rules.fqdnTags
```