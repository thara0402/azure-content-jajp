<properties
   pageTitle="リソース ロックのリソース マネージャー テンプレート | Microsoft Azure"
   description="テンプレートを使ってリソース ロックをデプロイするためのリソース マネージャー スキーマを示します。"
   services="azure-resource-manager"
   documentationCenter="na"
   authors="tfitzmac"
   manager="wpickett"
   editor=""/>

<tags
   ms.service="azure-resource-manager"
   ms.devlang="na"
   ms.topic="article"
   ms.tgt_pltfrm="na"
   ms.workload="na"
   ms.date="01/21/2016"
   ms.author="tomfitz"/>

# リソース ロック テンプレート スキーマ

リソースとその子リソースに対応する新しいロックを作成します。

## スキーマの形式

ロックを作成するには、テンプレートのリソース セクションに次のスキーマを追加します。
    
    {
        "type": enum,
        "apiVersion": "2015-01-01",
        "name": string,
        "dependsOn": [ array values ],
        "properties":
        {
            "level": enum,
            "notes": string
        }
    }



## 値

次の表では、スキーマに設定する必要がある値について説明します。

| 名前 | 型 | 必須 | 使用できる値 | 説明 |
| ---- | ---- | -------- | ---------------- | ----------- |
| type | enum | はい | リソース: <br />{名前空間}/{型}/providers/locks/<br /><br />リソース グループ:<br />Microsoft.Authorization/locks | 作成するリソースの種類。 |
| apiVersion | enum | はい | **2015-01-01** | リソースの作成に使用する API バージョン。 |  
| name | string | はい | リソース:<br /> {リソース}/Microsoft.Authorization/{ロックの名前}**<br /><br />リソース グループ:<br />**{ロックの名前}**<br /><br />最大 64 文字<br /><、>、%、&、? などの制御文字を含めることはできません。| ロックするリソースおよびロックの名前の両方を指定する値。| 
|dependsOn | 配列 | いいえ | リソース名またはリソースの一意識別子のコンマ区切りリスト。| このロックが依存するリソースのコレクション。ロックしているリソースが同じテンプレートにデプロイされている場合、そのリソース名をこの要素に含めることでリソースが最初にデプロイされるようにします。| 
| プロパティ | オブジェクト | はい | (下記参照) | ロックの種類を識別するオブジェクト、およびロックに関するメモ。| 

### プロパティ オブジェクト

| 名前 | 型 | 必須 | 使用できる値 | 説明 |
| ------- | ---- | ---------------- | -------- | ----------- |
| level | enum | あり | **CannotDelete** | スコープに適用するロックの種類。CanNotDelete を指定すると、変更はできますが、削除できなくなります。 |
| notes | string | いいえ | 512 文字 | ロックの説明。 |


## ロック リソースの使用方法

リソースに対する指定したアクションを防ぐには、このリソースをテンプレートに追加します。ロックは、すべてのユーザーとグループに適用されます。通常は、プロセスの実行中に、組織内の他のユーザーが誤ってリソースを変更したり削除したりしないように、一定の期間のみロックを適用します。

管理ロックを作成または削除するには、**Microsoft.Authorization/*** または **Microsoft.Authorization/locks/*** アクションにアクセスできる必要があります。組み込みのロールのうち、**所有者**と**ユーザー アクセスの管理者**にのみこれらのアクションが許可されています。ロールベースのアクセス制御については、「[Azure のロールベースのアクセス制御](./active-directory/role-based-access-control-configure.md)」を参照してください。

ロックは、指定されたリソースとすべての子リソースに適用されます。

ロックは PowerShell コマンド **Remove-AzureRmResourceLock** または REST API の [削除操作](https://msdn.microsoft.com/library/azure/mt204562.aspx)で削除することができます。

## 例

次の例では、Web アプリに削除不可のロックが適用されます。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {
            "hostingPlanName": {
      			"type": "string"
            }
        },
        "variables": {
            "siteName": "[concat('site',uniqueString(resourceGroup().id))]"
        },
        "resources": [
            {
                "apiVersion": "2015-08-01",
                "name": "[variables('siteName')]",
                "type": "Microsoft.Web/sites",
                "location": "[resourceGroup().location]",
                "properties": {
                    "serverFarmId": "[parameters('hostingPlanName')]"
                },
            },
            {
                "type": "Microsoft.Web/sites/providers/locks",
                "apiVersion": "2015-01-01",
                "name": "[concat(variables('siteName'),'/Microsoft.Authorization/MySiteLock')]",
                "dependsOn": [ "[variables('siteName')]" ],
                "properties":
                {
                    "level": "CannotDelete",
                    "notes": "my notes"
                }
             }
        ],
        "outputs": {}
    }

次の例では、リソース グループに削除不可のロックが適用されます。

    {
        "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "parameters": {},
        "variables": {},
        "resources": [
            {
                "type": "Microsoft.Authorization/locks",
                "apiVersion": "2015-01-01",
                "name": "MyGroupLock",
                "properties":
                {
                    "level": "CannotDelete",
                    "notes": "my notes"
                }
            }
        ],
        "outputs": {}
    }

## 次のステップ

- テンプレートの構造の詳細については、「[Azure リソース マネージャーのテンプレートの作成](resource-group-authoring-templates.md)」を参照してください。
- ロックの詳細については、「[Azure リソース マネージャーによるリソースのロック](resource-group-lock-resources.md)」を参照してください。

<!---HONumber=AcomDC_0128_2016-->