<properties
	pageTitle="Azure App Service の Web ジョブ"
	description="Web ジョブを作成し、バックグラウンドでのテストの実行、Storage や Service Bus などのサービスとの対話、スケジュールされたタスクの作成を行う方法について説明します。"
	services="app-service"
	documentationCenter=""
	authors="christopheranderson"
	manager="wpickett"
	editor="mollybos"/>

<tags
	ms.service="app-service"
	ms.workload="web"
	ms.tgt_pltfrm="na"
	ms.devlang="na"
	ms.topic="article"
	ms.date="12/10/2015"
	ms.author="chrande"/>

# Azure App Service での Web ジョブの使用

このトピックでは、Azure Web ジョブと Azure Web ジョブ SDK の使用方法に関するドキュメント リソースへのリンクを掲載しています。Azure Web ジョブは、[App Service Web Apps](http://go.microsoft.com/fwlink/?LinkId=529714) でバック グラウンド プロセスとしてスクリプトやプログラムを実行する簡単な方法を提供します。cmd、bat、exe (.NET)、ps1、sh、php、py、js、jar などの実行可能ファイルをアップロードして、実行できます。これらのプログラムは、スケジュールに従って (cron)、または継続的に Web ジョブとして実行されます。

Web ジョブ SDK を使用すると、簡単に Azure Storage を使用できます。Web ジョブ SDK には、Microsoft Azure Storage の BLOB、キュー、テーブルおよび Service Bus のキューと共に動作するバインドおよびトリガー システムがあります。

Web ジョブの作成、デプロイ、管理は、Visual Studio の統合ツールでシームレスに行うことができます。テンプレートから Web ジョブを作成し、発行、および管理 (実行、停止、監視、デバッグ) できます。

Azure ポータルの Web ジョブ ダッシュボードは、Web ジョブ内の個々の関数を呼び出す機能など、強力な管理機能を提供して Web ジョブの実行をきめ細かく制御できます。また、ダッシュボードでは、関数の実行時間とログ出力も表示されます。

[AZURE.INCLUDE [app-service-blueprint-webjobs](../../includes/app-service-blueprint-webjobs.md)]

<!---HONumber=AcomDC_0121_2016-->