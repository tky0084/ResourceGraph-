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