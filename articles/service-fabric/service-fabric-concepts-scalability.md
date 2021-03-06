<properties
   pageTitle="Service Fabric サービスの拡張性 | Microsoft Azure"
   description="Service Fabric サービスの拡張方法を説明する"
   services="service-fabric"
   documentationCenter=".net"
   authors="appi101"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="01/20/2016"
   ms.author="aprameyr"/>

# Service Fabric アプリケーションのスケーリング
Azure Service Fabric により、クラスターのすべてのノードのサービス、パーティション、およびレプリカの負荷分散を実施することにより、スケーラブルなアプリケーションを簡単に構築できます。これにより、リソースを最大限活用できます。

Service Fabric アプリケーションの高い拡張性は、次の 2 つの方法で実現できます。

1. パーティション レベルでのスケーリング

2. サービス名レベルでのスケーリング

## パーティション レベルでのスケーリング
Service Fabric では、個々 のサービスを複数の小さいパーティションにパーティション分割することをサポートします。「[パーティション分割の概要](service-fabric-concepts-partitioning.md)」では、サポートされているパーティション構成の種類に関する情報を提供します。各パーティションのレプリカが、クラスター内のノードに分散します。低値キー 0、高値キー 99、および 4 つのパーティションを持つ、範囲指定されたパーティション分割構成を使用したサービスを検討してください。3 ノード クラスターでは、サービスを次に示すように、各ノード上のリソースを共有する 4 つのレプリカでレイアウトする可能性があります。

![3 つのノードでのパーティション レイアウト](./media/service-fabric-concepts-scalability/layout-three-nodes.png)

ノードの数を増やすと、レプリカの一部を空のノードに移動して、Service Fabric が新しいノード上のリソースを利用できるようになります。ノードの数を 4 つに増やすと、サービスには (別のパーティションの) 各ノードで実行されている 3 つのレプリカがあり、リソース使用率とパフォーマンスを向上できます。

![4 つのノードでのパーティション レイアウト](./media/service-fabric-concepts-scalability/layout-four-nodes.png)

## サービス名レベルでのスケーリング
サービス インスタンスとは、アプリケーション名とサービスの種類名の特定のインスタンスです (「[Service Fabric アプリケーション ライフサイクル](service-fabric-application-lifecycle.md)」を参照)。サービスの作成時に使用するパーティション構成を指定します (「[Service Fabric サービスのパーティション分割](service-fabric-concepts-partitioning.md)」を参照)。

スケーリングの最初のレベルは、サービス名です。古いサービス インスタンスがビジー状態になったら、さまざまなレベルのパーティション分割を使用して、サービスの新しいインスタンスを作成できます。これにより、新しいサービス コンシューマーは、ビジー状態のサービス インスタンスよりも、ビジー状態でないものを使用することができます。

容量を増やすか、パーティションの数を増減するオプションにより、新しいパーティション構成で新しいサービス インスタンスを作成します。使用中のクライアントが、別の名前のサービスを使用するタイミングと方法を知る必要がある場合は、作業が複雑になります。

### シナリオの例: 埋め込みの日付
可能性のあるシナリオの 1 つとして、サービス名の一部としての日付の情報を使用する場合があります。たとえば、2013 年に参加したすべての顧客に対して特定の名前のサービス インスタンスを使用し、2014 年に参加した顧客に対して別の名前のサービス インスタンスを使用できます。この名前付けスキームでは、日付に応じてプログラムによって名前を増やすことができます (2014 年の方法では、2014 年のサービス インスタンスを要求に応じて作成できます)。

ただし、この方法は、Service Fabric の知識の範囲外にあるアプリケーション固有の名前付け情報を使用するクライアントに基づきます。

- *名前付けの規則を使用する*: 2013 年のアプリケーション運用時に、fabric:/app/service2013 という 1 つのサービスを作成します。2013 年の第 2 四半期に近づくと、fabric:/app/service2014 という別のサービスを作成します。これらの両方のサービスは、同じサービス型です。この方法では、クライアントは、年に基づいて適切なサービス名を構築するロジックを使用する必要があります。

- *検索サービスを使用する*: 別のパターンでは、必要なキーに対してサービス名を提供できるようにするセカンダリ参照サービスを提供します。新しいサービス インスタンスは、参照サービスによって作成できます。参照サービス自体では、アプリケーション データは保持せず、作成するサービス名についてのデータのみ保持します。そのため、上記の年ベースの例では、クライアントは最初に参照サービスに問い合わせて、特定の年のデータを処理するサービスの名前を確認し、次にそのサービス名を使用して、実際の操作を実行します。最初の参照の結果は、キャッシュすることができます。

## 次のステップ

Service Fabric の概念の詳細については、次を参照してください。

- [Service Fabric サービスの可用性](service-fabric-availability-services.md)

- [Service Fabric サービスのパーティション分割](service-fabric-concepts-partitioning.md)

- [状態の定義と管理](service-fabric-concepts-state.md)

<!---HONumber=AcomDC_0128_2016-->