# 実際にResourceGraphを使って、AzureFirewallのルール一覧を表示するためのクエリを作成してみよう

## 前提

事前にテスト用のAzureFirewallを、ARMテンプレートを使い、Azure Portalの「カスタム テンプレートからのデプロイ」からデプロイしておきましょう。これは料金がかかりません。

<details><summary>ARM Templateはこちら</summary>

```
{
    "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "firewallPolicies_testFirewall_name": {
            "defaultValue": "testFirewall",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "Microsoft.Network/firewallPolicies",
            "apiVersion": "2025-07-01",
            "name": "[parameters('firewallPolicies_testFirewall_name')]",
            "location": "japaneast",
            "properties": {
                "sku": {
                    "tier": "Standard"
                },
                "threatIntelMode": "Alert",
                "threatIntelWhitelist": {
                    "fqdns": [],
                    "ipAddresses": []
                }
            }
        },
        {
            "type": "Microsoft.Network/firewallPolicies/ruleCollectionGroups",
            "apiVersion": "2025-07-01",
            "name": "[concat(parameters('firewallPolicies_testFirewall_name'), '/DefaultApplicationRuleCollectionGroup')]",
            "location": "japaneast",
            "dependsOn": [
                "[resourceId('Microsoft.Network/firewallPolicies', parameters('firewallPolicies_testFirewall_name'))]"
            ],
            "properties": {
                "priority": 300,
                "ruleCollections": [
                    {
                        "ruleCollectionType": "FirewallPolicyFilterRuleCollection",
                        "action": {
                            "type": "Allow"
                        },
                        "rules": [
                            {
                                "ruleType": "ApplicationRule",
                                "name": "rc-3-1",
                                "protocols": [
                                    {
                                        "protocolType": "Http",
                                        "port": 80
                                    },
                                    {
                                        "protocolType": "Https",
                                        "port": 443
                                    }
                                ],
                                "fqdnTags": [],
                                "webCategories": [],
                                "targetFqdns": [
                                    "www.google.com"
                                ],
                                "targetUrls": [],
                                "terminateTLS": false,
                                "sourceAddresses": [
                                    "10.0.0.0/24"
                                ],
                                "destinationAddresses": [],
                                "sourceIpGroups": [],
                                "httpHeadersToInsert": []
                            },
                            {
                                "ruleType": "ApplicationRule",
                                "name": "rc-3-2",
                                "protocols": [
                                    {
                                        "protocolType": "Http",
                                        "port": 80
                                    },
                                    {
                                        "protocolType": "Https",
                                        "port": 443
                                    }
                                ],
                                "fqdnTags": [],
                                "webCategories": [],
                                "targetFqdns": [
                                    "www.yahoo.co.jp"
                                ],
                                "targetUrls": [],
                                "terminateTLS": false,
                                "sourceAddresses": [
                                    "10.0.0.0/24"
                                ],
                                "destinationAddresses": [],
                                "sourceIpGroups": [],
                                "httpHeadersToInsert": []
                            },
                            {
                                "ruleType": "ApplicationRule",
                                "name": "rc-3-3",
                                "protocols": [
                                    {
                                        "protocolType": "Http",
                                        "port": 80
                                    },
                                    {
                                        "protocolType": "Https",
                                        "port": 443
                                    }
                                ],
                                "fqdnTags": [],
                                "webCategories": [],
                                "targetFqdns": [
                                    "github.com"
                                ],
                                "targetUrls": [],
                                "terminateTLS": false,
                                "sourceAddresses": [
                                    "10.0.0.0/24"
                                ],
                                "destinationAddresses": [],
                                "sourceIpGroups": [],
                                "httpHeadersToInsert": []
                            }
                        ],
                        "name": "rc-03",
                        "priority": 100
                    },
                    {
                        "ruleCollectionType": "FirewallPolicyFilterRuleCollection",
                        "action": {
                            "type": "Deny"
                        },
                        "rules": [
                            {
                                "ruleType": "ApplicationRule",
                                "name": "rc-4-1",
                                "protocols": [
                                    {
                                        "protocolType": "Http",
                                        "port": 80
                                    }
                                ],
                                "fqdnTags": [],
                                "webCategories": [],
                                "targetFqdns": [
                                    "www.google.com"
                                ],
                                "targetUrls": [],
                                "terminateTLS": false,
                                "sourceAddresses": [
                                    "10.0.0.0/24"
                                ],
                                "destinationAddresses": [],
                                "sourceIpGroups": [],
                                "httpHeadersToInsert": []
                            },
                            {
                                "ruleType": "ApplicationRule",
                                "name": "rc-4-2",
                                "protocols": [
                                    {
                                        "protocolType": "Http",
                                        "port": 80
                                    }
                                ],
                                "fqdnTags": [],
                                "webCategories": [],
                                "targetFqdns": [
                                    "www.yahoo.co.jp"
                                ],
                                "targetUrls": [],
                                "terminateTLS": false,
                                "sourceAddresses": [
                                    "10.0.0.0/24"
                                ],
                                "destinationAddresses": [],
                                "sourceIpGroups": [],
                                "httpHeadersToInsert": []
                            },
                            {
                                "ruleType": "ApplicationRule",
                                "name": "rc-4-3",
                                "protocols": [
                                    {
                                        "protocolType": "Http",
                                        "port": 80
                                    }
                                ],
                                "fqdnTags": [],
                                "webCategories": [],
                                "targetFqdns": [
                                    "github.com"
                                ],
                                "targetUrls": [],
                                "terminateTLS": false,
                                "sourceAddresses": [
                                    "10.0.0.0/24"
                                ],
                                "destinationAddresses": [],
                                "sourceIpGroups": [],
                                "httpHeadersToInsert": []
                            }
                        ],
                        "name": "rc-04",
                        "priority": 200
                    }
                ]
            }
        },
        {
            "type": "Microsoft.Network/firewallPolicies/ruleCollectionGroups",
            "apiVersion": "2025-07-01",
            "name": "[concat(parameters('firewallPolicies_testFirewall_name'), '/DefaultNetworkRuleCollectionGroup')]",
            "location": "japaneast",
            "dependsOn": [
                "[resourceId('Microsoft.Network/firewallPolicies', parameters('firewallPolicies_testFirewall_name'))]"
            ],
            "properties": {
                "priority": 200,
                "ruleCollections": [
                    {
                        "ruleCollectionType": "FirewallPolicyFilterRuleCollection",
                        "action": {
                            "type": "Allow"
                        },
                        "rules": [
                            {
                                "ruleType": "NetworkRule",
                                "name": "rule1-1",
                                "ipProtocols": [
                                    "TCP",
                                    "UDP",
                                    "ICMP"
                                ],
                                "sourceAddresses": [
                                    "10.0.0.0/24"
                                ],
                                "sourceIpGroups": [],
                                "destinationAddresses": [
                                    "10.0.1.0/24"
                                ],
                                "destinationIpGroups": [],
                                "destinationFqdns": [],
                                "destinationPorts": [
                                    "80",
                                    "443",
                                    "22",
                                    "3389"
                                ]
                            },
                            {
                                "ruleType": "NetworkRule",
                                "name": "rule1-2",
                                "ipProtocols": [
                                    "TCP"
                                ],
                                "sourceAddresses": [
                                    "10.0.0.0/24"
                                ],
                                "sourceIpGroups": [],
                                "destinationAddresses": [
                                    "10.0.1.0/24"
                                ],
                                "destinationIpGroups": [],
                                "destinationFqdns": [],
                                "destinationPorts": [
                                    "80",
                                    "443"
                                ]
                            },
                            {
                                "ruleType": "NetworkRule",
                                "name": "rule1-3",
                                "ipProtocols": [
                                    "ICMP"
                                ],
                                "sourceAddresses": [
                                    "10.0.0.0/24"
                                ],
                                "sourceIpGroups": [],
                                "destinationAddresses": [
                                    "10.0.1.0/24"
                                ],
                                "destinationIpGroups": [],
                                "destinationFqdns": [],
                                "destinationPorts": [
                                    "*"
                                ]
                            }
                        ],
                        "name": "rc-01",
                        "priority": 100
                    },
                    {
                        "ruleCollectionType": "FirewallPolicyFilterRuleCollection",
                        "action": {
                            "type": "Deny"
                        },
                        "rules": [
                            {
                                "ruleType": "NetworkRule",
                                "name": "rule-2-1",
                                "ipProtocols": [
                                    "TCP",
                                    "UDP",
                                    "ICMP"
                                ],
                                "sourceAddresses": [
                                    "10.0.0.0/24"
                                ],
                                "sourceIpGroups": [],
                                "destinationAddresses": [
                                    "10.0.2.0/24"
                                ],
                                "destinationIpGroups": [],
                                "destinationFqdns": [],
                                "destinationPorts": [
                                    "80",
                                    "443",
                                    "22",
                                    "3389"
                                ]
                            },
                            {
                                "ruleType": "NetworkRule",
                                "name": "rule-2-2",
                                "ipProtocols": [
                                    "TCP"
                                ],
                                "sourceAddresses": [
                                    "10.0.0.0/24"
                                ],
                                "sourceIpGroups": [],
                                "destinationAddresses": [
                                    "10.0.2.0/24"
                                ],
                                "destinationIpGroups": [],
                                "destinationFqdns": [],
                                "destinationPorts": [
                                    "80",
                                    "443"
                                ]
                            },
                            {
                                "ruleType": "NetworkRule",
                                "name": "rule-2-3",
                                "ipProtocols": [
                                    "ICMP"
                                ],
                                "sourceAddresses": [
                                    "10.0.0.0/24"
                                ],
                                "sourceIpGroups": [],
                                "destinationAddresses": [
                                    "10.0.2.0/24"
                                ],
                                "destinationIpGroups": [],
                                "destinationFqdns": [],
                                "destinationPorts": [
                                    "*"
                                ]
                            }
                        ],
                        "name": "rc-02",
                        "priority": 200
                    }
                ]
            }
        }
    ]
}
```

</details>


## AzureFirewallのルールのリソース構成について
AzureFirewallは主にAzureFirewall本体と、AzureFirewallポリシーで成り立っています。

一般的にportalで穴あけルールを作る際、Azurefirewallポリシーに対して作業をするように見えます。<br>
しかし、JSONファイルを見てみましょう。<br>
どこにも穴あけルールについての記載がありません。

![alt text](<img/スクリーンショット 2026-08-18 085810.png>)

<br>

では、どこにAzureFirewallの穴あけが記載されているのでしょうか。

ARMテンプレートを見てみましょう。すると、リソースが3つ存在します。<br>
その中には、"type": "Microsoft.Network/firewallPolicies/ruleCollectionGroupsという記載が2つ存在しますね。

![alt text](<img/スクリーンショット 2026-08-18 090516.png>)

<br>

そう、AzureFirewallの穴あけルールは、GUI上ではFirewallポリシーに対して作業をしているように見えても、<br>
隠れたリソースであるruleCollectionGroupの中に記載されているのです。<br>
ResourceGraphでは、これに対して検索をする必要があります。<br>

---
## ResourceGraphで検索

それでは、ResourceGraphで以下のクエリを打ってみましょう。

```
resources
| where type =~ "Microsoft.Network/firewallPolicies/ruleCollectionGroups"
```
<br>
しかし、結果は何も表示されません。

![alt text](<img/スクリーンショット 2026-08-18 090913-1.png>)

<br>
では、テーブル検索欄に、リソースタイプ「Microsoft.Network/firewallPolicies/ruleCollectionGroups」を検索してみましょう。<br>
すると、規則コレクショングループは、networkresourcesというテーブルに入っていることがわかります。
<br>

![alt text](<img/スクリーンショット 2026-08-18 091042-1.png>)

では、テーブル名を直して、もう一度実行してみましょう。<br>
これで検索がかかったと思います。

![alt text](<img/スクリーンショット 2026-08-18 091253.png>)

---

## ルールが記載されたプロパティを列に外出しする
このクエリ結果には、直接記載ルールが1行ずつ出るわけではありません。<br>
必要な項目を外に出すという方法をとります。<br>

では、1行目を見てみましょう。<br>
これを見ると、propertiesという列の中の、ruleCollectionsという配列がいます。<br>
ruleCollectionsは穴あけルールをグループ化するものですので、まずはこれを外に出します。<br>

![alt text](<img/スクリーンショット 2026-08-18 091545.png>)

<br>

では、クエリを以下のように追記してみましょう。<br>
extendで列内の特定の項目を別の列に拡張します。<br>
今回は、右辺のproperties.ruleCollections(propertiesの中のruleCollections)を、左辺のruleCollectionsという名前(任意)として拡張します。

すると、ruleCollectionsという名前の列が作られて、ruleCollectionsの中身だけが記載されていると思います。

```
networkresources
| where type =~ "Microsoft.Network/firewallPolicies/ruleCollectionGroups"
| extend ruleCollections = properties.ruleCollections
```

![alt text](<img/スクリーンショット 2026-08-18 092038.png>)

<br>

しかし、これでは不十分です。<br>
なぜなら、ExtendはpropertiesのruleCollectionsをそのまま外に出しているだけです。

```
ruleCollections: [
     { "name": "NetworkRuleCollection01", "ruleCollectionType": "FirewallPolicyFilterRuleCollection" },
     { "name": "NetworkRuleCollection02", "ruleCollectionType": "FirewallPolicyFilterRuleCollection" } 
]
```

このままでは、複数の規則コレクションが1つのセルにまとまっているため、それぞれの情報を扱いにくい状態です。

そこで使用するのが mv-expand です。

##  mv-expandで複数の要素を展開する

mv-expand は、配列に格納されている複数の要素を、それぞれ別の行に展開するための演算子です。<br>
mv は multi-value（複数の値） を意味します。

たとえば、以下のように1つの行に配列が格納されていたとします。<br>
["RuleCollection01", "RuleCollection02", "RuleCollection03"]<br>

これに mv-expand を使用すると、<br>
RuleCollection01<br>
RuleCollection02<br>
RuleCollection03<br>
というように、配列の要素ごとに行が分割されます。

Azure Firewallの場合、1つの規則コレクショングループの中に複数の ruleCollections が格納されています。<br>
そのため、<br>
| mv-expand ruleCollections<br>
とすることで、それぞれの規則コレクションを1行ずつに展開できます。

では、extendを書き換えましょう。<br>
すると、ruleCollectionsの中にある{}の内容が、ruleCollectionsの列名で、1つあたり1行ずつ展開されていることがわかります。

```
networkresources
| where type =~ "Microsoft.Network/firewallPolicies/ruleCollectionGroups"
| mv-expand ruleCollections = properties.ruleCollections
```

![alt text](<img/スクリーンショット 2026-08-21 083317.png>)

<br>

さらに、規則コレクションにはルールが内包されていますね。<br>
これも同じように展開しましょう。、ただし展開元はpropertiesではなく、ruleCollectionsです。

```
networkresources
| where type =~ "Microsoft.Network/firewallPolicies/ruleCollectionGroups"
| mv-expand ruleCollections = properties.ruleCollections
| mv-expand rules = ruleCollections.rules
| project name, ruleCollections, rules
```

ルールが1行ずつ展開されました。<br>
ここから、さらに細かくextendやmv-expandを使って列を作って展開することもできます。

![alt text](<img/スクリーンショット 2026-08-21 084408.png>)

<br>

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
| extend ruleType = rules.ruleType
| extend sourceAddresses = rules.sourceAddresses
| extend protocols = rules.protocols
| extend targetFqdns = rules.targetFqdns
| extend targetUrls = rules.targetUrls
| extend fqdnTags = rules.fqdnTags
```

ところが、ネットワークルールは展開されていないですね。

![alt text](<img/スクリーンショット 2026-08-21 083317-1.png>)

<br>

ネットワークルールとアプリケーションルールは、プロパティの構造が異なるため、必要なプロパティを指定して展開する必要があります。<br>
また、extendは、コンマで区切れば1度に複数指定できることを覚えておきましょう。

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
| extend ruleType = rules.ruleType
| extend sourceAddresses = rules.sourceAddresses
| extend protocols = rules.protocols
| extend targetFqdns = rules.targetFqdns
| extend targetUrls = rules.targetUrls
| extend fqdnTags = rules.fqdnTags
| extend
    sourceIpGroups = rules.sourceIpGroups,
    ipProtocols = rules.ipProtocols,
    destinationPorts = rules.destinationPorts,
    destinationAddresses = rules.destinationAddresses,
    destinationIpGroups = rules.destinationIpGroups,
    destinationFqdns = rules.destinationFqdns
```

## 2つの列を一つにまとめる方法
ここまで出来たら、必要な列だけに絞り、ルールごとに表示の異なる同じ意味の項目の列はまとめて表示するようにしてみましょう。

![alt text](<img/スクリーンショット 2026-08-23 092159.png>)

<br>

まずはextendした列を並び替えましょう。extendはいったん各行に出すようにします。

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
| extend ruleType = rules.ruleType
| extend sourceAddresses = rules.sourceAddresses
| extend sourceIpGroups = rules.sourceIpGroups
| extend ipProtocols = rules.ipProtocols
| extend destinationPorts = rules.destinationPorts
| extend protocols = rules.protocols
| extend destinationAddresses = rules.destinationAddresses
| extend destinationIpGroups = rules.destinationIpGroups
| extend destinationFqdns = rules.destinationFqdns
| extend targetFqdns = rules.targetFqdns
| extend targetUrls = rules.targetUrls
| extend fqdnTags = rules.fqdnTags 
```
<br>
ここで、プロトコル（ipProtocols/protocols)と、宛先fqdn(destinationFqdns/targetFqdns)についてを1行にまとめてみましょう。

ruleTypeの値に応じて、case構文を使って分岐します。
```
| extend protocol = case(
    ruleType == "ApplicationRule", protocols,
    ruleType == "NetworkRule", ipProtocols,
    dynamic([])
    )

| extend destinationFqdns = case(
    ruleType == "ApplicationRule", rules.targetFqdns,
    ruleType == "NetworkRule", rules.destinationFqdns,
    dynamic([])
)
```

<br>

このようにまとめることができました。
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
| extend ruleType = rules.ruleType
| extend sourceAddresses = rules.sourceAddresses
| extend sourceIpGroups = rules.sourceIpGroups
| extend ipProtocols = rules.ipProtocols
| extend destinationPorts = rules.destinationPorts
| extend protocols = rules.protocols
| extend protocol = case(
    ruleType == "ApplicationRule", protocols,
    ruleType == "NetworkRule", ipProtocols,
    dynamic([])
)
| extend destinationAddresses = rules.destinationAddresses
| extend destinationIpGroups = rules.destinationIpGroups
| extend destinationFqdns = rules.destinationFqdns
| extend targetFqdns = rules.targetFqdns
| extend destinationFqdns = case(
    ruleType == "ApplicationRule", rules.targetFqdns,
    ruleType == "NetworkRule", rules.destinationFqdns,
    dynamic([])
)
| extend targetUrls = rules.targetUrls
| extend fqdnTags = rules.fqdnTags 
```
<br>

最後に、project構文を末尾に追記して、必要な項目だけを表示させて完成です。

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
| extend ruleType = rules.ruleType
| extend sourceAddresses = rules.sourceAddresses
| extend sourceIpGroups = rules.sourceIpGroups
| extend ipProtocols = rules.ipProtocols
| extend destinationPorts = rules.destinationPorts
| extend protocols = rules.protocols
| extend protocol = case(
    ruleType == "ApplicationRule", protocols,
    ruleType == "NetworkRule", ipProtocols,
    dynamic([])
)
| extend destinationAddresses = rules.destinationAddresses
| extend destinationIpGroups = rules.destinationIpGroups
| extend destinationFqdns = rules.destinationFqdns
| extend targetFqdns = rules.targetFqdns
| extend destinationFqdns = case(
    ruleType == "ApplicationRule", rules.targetFqdns,
    ruleType == "NetworkRule", rules.destinationFqdns,
    dynamic([])
)
| extend targetUrls = rules.targetUrls
| extend fqdnTags = rules.fqdnTags 
| project name, ruleCollection, priority, action, ruleName, ruleType, sourceAddresses, sourceIpGroups, protocol, destinationPorts, destinationAddresses, destinationIpGroups, destinationFqdns, targetUrls, fqdnTags
```

## まとめ
このように、クエリの構文を駆使して、必要な情報を取り出すことができます。
ほかにも使えるAzureサービスはたくさんあるので、ぜひ試してみてください。