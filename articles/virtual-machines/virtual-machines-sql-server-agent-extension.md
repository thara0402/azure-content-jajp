<properties
	pageTitle="SQL Server IaaS Agent 拡張機能 | Microsoft Azure"
	description="このトピックでは、クラシック デプロイ モデルで作成されたリソースを使用し、Azure で SQL Server を実行する VM が自動化機能を使用できるようにする SQL Server Agent 拡張機能について説明します。"
	services="virtual-machines"
	documentationCenter=""
	authors="jeffgoll"
	manager="jeffreyg"
   editor="monicar"    
   tags="azure-service-management"/>

<tags
	ms.service="virtual-machines"
	ms.devlang="na"
	ms.topic="article"
	ms.tgt_pltfrm="vm-windows-sql-server"
	ms.workload="infrastructure-services"
	ms.date="10/02/2015"
	ms.author="jeffreyg"/>

# SQL Server IaaS Agent 拡張機能

この拡張機能を使用すると、Azure Virtual Machines の SQL Server で、この記事に掲載されている特定のサービスを使用できるようになります。これらのサービスは、この拡張機能がインストールされている場合にのみ使用できます。この拡張機能は、Azure ポータルの SQL Server Gallery Images の場合に自動的にインストールされます。Azure VM Guest Agent がインストールされている Azure の任意の SQL Server VM にインストールできます。

[AZURE.INCLUDE [learn-about-deployment-models](../../includes/learn-about-deployment-models-classic-include.md)]リソース マネージャー モデル。


## 前提条件
Powershell コマンドレットを使用するための要件:

- 最新の Azure コマンドライン SDK は、[こちらで入手できます](https://azure.microsoft.com/downloads/)

VM で拡張機能を使用するための要件:

- Azure VM Guest Agent
- Windows Server 2012、Windows Server 2012 R2 以降
- SQL Server 2012、SQL Server 2014 以降

## 拡張機能で使用できるサービス

- **SQL の自動化されたバックアップ**: このサービスで、VM 内の SQL Server の既定インスタンスについて、すべてのデータベースのバックアップが自動的にスケジュールされます。このサービスの詳細については、「[Azure Virtual Machines における SQL Server の自動化されたバックアップ](virtual-machines-sql-server-automated-backup.md)」を参照してください。
- **SQL 自動修正プログラム適用**: このサービスを使用すると、VM の更新プログラムを実行できるメンテナンス期間を構成できるので、ワークロードのピーク時の更新プログラムを回避できます。このサービスの詳細については、「[Azure Virtual Machines における SQL Server の自動化された修正プログラムの適用](virtual-machines-sql-server-automated-patching.md)」を参照してください。

## Powershell を使用した拡張機能の追加
[Azure ポータル](https://portal.azure.com/)を使用して SQL Server VM をプロビジョニングすると、拡張機能は自動的にインストールされます。[Azure クラシック ポータル](https://manage.windowsazure.com)で SQL Server VM をプロビジョニングした場合、または SQL ライセンスを持っている VM の場合、次の Azure PowerShell コマンドレットを使用して、既存の VM にこの拡張機能を追加できます。

**Set-AzureVMSqlServerExtension**

### 構文

Set-AzureVMSqlServerExtension [-VM] <IPersistentVM> [[-Version] <string>] [-AutoBackupSettings <AutoBackupSettings>] [-AutoPatchingSetttings <AutoPatchingSetttings>] [-Confirm] [-WhatIf] [<CommonParameters>]

> [AZURE.NOTE] –Version パラメーターは、省略することをお勧めします。省略すると、最新バージョンの拡張機能が既定値になります。

### 例
	Get-AzureVM –ServiceName serviceName –Name vmName | Set-AzureVMSqlServerExtension –AutoBackupSettings $abs | Update-AzureVM**

## 拡張機能の状態を確認します。
この拡張機能とそれに関連付けられているサービスの状態を確認する場合は、いずれかのポータルを使用できます。既存の VM の詳細で、**[設定]** の** [拡張機能]** を確認します。

また、次の Azure PowerShell コマンドレットを使用することもできます。

**Get-AzureVMSqlServerExtension**

### 構文

Get-AzureVMSqlServerExtension [[-VM] <IPersistentVM>] [[-Version] <string>] [<CommonParameters>]

> [AZURE.NOTE] –Version パラメーターは省略できます。省略すると、最新バージョンの拡張機能が既定値になります。

### 例
	Get-AzureVM –ServiceName "service" –Name "vmname" | Get-AzureVMSqlServerExtension

## Powershell を使用した拡張機能の削除   
VM からこの拡張機能を削除する場合は、次の Azure Powershell コマンドレットを使用します。

**Remove-AzureVMSqlServerExtension**

### 構文
Remove-AzureVMSqlServerExtension -VM <IPersistentVM> [<CommonParameters>]

<!---HONumber=AcomDC_0128_2016-->