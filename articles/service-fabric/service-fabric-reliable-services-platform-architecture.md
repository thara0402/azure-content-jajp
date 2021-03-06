<properties
   pageTitle="Reliable Service のアーキテクチャ | Microsoft Azure"
   description="ステートフル サービスとステートレス サービスの Reliable Service アーキテクチャの概要"
   services="service-fabric"
   documentationCenter=".net"
   authors="AlanWarwick"
   manager="timlt"
   editor=""/>

<tags
   ms.service="Service-Fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="09/03/2015"
   ms.author="alanwar"/>

# ステートフルとステートレスの Reliable Service のアーキテクチャ

Azure Service Fabric の Reliable Service は、ステートフルまたはステートレスのいずれかに設定できます。いずれのサービスも、特定のアーキテクチャ内で実行されます。この記事では、そのアーキテクチャについて説明します。ステートフル サービスとステートレス サービスの違いについての詳細は「[Reliable Service の概要](service-fabric-reliable-services-introduction.md)」を参照してください。

## ステートフル Reliable Service

### ステートフル サービスのアーキテクチャ
![ステートフル サービスのアーキテクチャ図](./media/service-fabric-reliable-services-platform-architecture/reliable-stateful-service-architecture.png)

### ステートフル Reliable Service

ステートフル Reliable Service は、StatefulService または StatefulServiceBase のいずれかのクラスから派生します。Service Fabric には、いずれの基本クラスも用意されています。これらのクラスで、Service Fabric の多様なレベルのサポートが実現し、インターフェイスに対してステートフル サービスを抽象化することができます。また、Service Fabric クラスター内のサービスとして参加することができます。

StatefulService は StatefulServiceBase から派生します。StatefulServiceBase によってサービスの柔軟性が高まりますが、Service Fabric の内部を深く理解する必要があります。StatefulService クラスと StatefulServiceBase クラスを使用したサービス作成の詳細については、「[Reliable Service の概要](service-fabric-reliable-services-introduction.md)」と「[Reliable Service の詳細な使用方法](service-fabric-reliable-services-advanced-usage.md)」を参照してください 。

いずれの基本クラスでも、サービス実装の有効期間と役割が管理されます。サービス実装によって、いずれかの基本クラスの仮想メソッドがオーバーライドされることがあります。それは、サービス実装のライフサイクルにおけるこれらのポイントで、サービスで実行しなければならない処理がある場合や、通信リスナー オブジェクトを作成する場合です。上記の図で示すように、サービスが独自の通信リスナー オブジェクトを実装して ICommunicationListener を公開する場合がありますが、このサービス実装では Service Fabric が実装した通信リスナーが使用されるため、通信リスナーは Service Fabric が実装します。

ステートフル Reliable Service では、Reliable State Manager を使用して Reliable Collection が利用されます。Reliable Collection は、サービスに対する可用性が高いローカル データ構造であるため、サービスのフェールオーバーに関係なく常に利用可能です。Reliable Collection は、Reliable State Provider によって実装されます。Reliable Collection の詳細については「[ Reliable Collection の概要](service-fabric-reliable-services-reliable-collections.md)」を参照してください。

### Reliable State Manager および State Provider

Reliable State Manager は Reliable State Provider を管理するオブジェクトです。Reliable State Provider を作成、削除、列挙し、Reliable State Provider を永続化して高可用性を保つ機能があります。Reliable State Provider のインスタンスは、ディクショナリやキューなどの永続化された高可用性のデータ構造のインスタンスのことです。

Reliable State Provider はそれぞれ、Reliable State Provider と対話するためにステートフル サービスによって使用されるインターフェイスを公開します。たとえば、IReliableDictionary は Reliable Dictionary とインターフェイスをとるために、IReliableQueue は Reliable Queue とインターフェイスをとるためそれぞれ使用されています。すべての Reliable State Provider は IReliableState インターフェイスを実装します。

Reliable State Manager は、ステートフル サービスからアクセスを許可する IReliableStateManager という名前のインターフェイスを持っています。Reliable State Provider とのインターフェイスが IReliableStateManager を介して返されます。

Reliable State Manager はプラグイン アーキテクチャを使用し、新しい種類の Reliable Collection を動的にプラグインできます。

高パフォーマンスのバージョン付き差分ストアを実装すると、Reliable Dictionary および Reliable Queue が作成されます。

### トランザクション レプリケーター

トランザクション レプリケーターのコンポーネントによって、サービスの状態 (Reliable State Manager および Reliable State Collections に格納される状態) の整合性が、サービスを実行しているすべてのレプリカで維持されます。また、状態はログに永続化されます。Reliable State Manager は、プライベート メカニズムを介してトランザクション レプリケーターとインターフェースをとります。

トランザクション レプリケーターは、ネットワーク プロトコルを使用してサービス インスタンスの他のレプリカと状態について通信するため、すべてのレプリカに最新の状態情報が行き渡ります。

トランザクション レプリケーターはログを使用して状態情報を永続化するため、プロセスまたはノードがクラッシュしても状態情報は失われません。ログとインターフェースをとるにはプライベート メカニズムを使用します。

### ログ

ログ コンポーネントによって永続的なストアを構築し、回転式または固定状態のディスクに書き込むために最適化することができます。また、ディスク領域を使用効率を最適化することもできます。ログの設計は、ステートフル サービスを実行しているノードのローカルにある永続的なストレージ (つまりハード ディスク) 向けです。ローカルにあるので、ノードのローカルにはない永続的なストレージと比較して、短い待機時間と高いスループットを実現できます。

ログ コンポーネントでは、 2 種類のログ ファイルが使用されます。ノード全体で共有されるログ ファイルは、そのログ ファイル専用のディスクに格納する必要があります。このファイルは、Service Fabric ノードの作業ディレクトリに配置されます。サービスの各レプリカにも専用ログ ファイルがあり、サービスの作業ディレクトリ内に配置されます。

共有ログは状態情報の移行領域である一方、専用ログ ファイルは状態情報が最終的に永続化される場所です。この設計では、状態情報がまず共有ログ ファイルに書き込まれてから、バック グラウンドで専用ログ ファイルに部分的にデステージされます。このようにして共有ログへの書き込みの待機時間は最小化され、スループットは最大化されるため、サービスの処理時間がより短くなります。

ただし OptimizeForLocalSSD 設定を使用してソリッド ステート ディスクに最適化されるようにログ コンポーネントが構成されている場合は、状態情報は専用ログ ファイルに直接書き込まれるため、共有ログ ファイルはバイパスされます。ソリッド ステート ディスクではヘッドの移動の競合による遅延は発生しないため、専用ログ ファイルに直接書き込むことでペナルティは発生しません。

ログ コンポーネントが、OptimizeLogForLowerDiskUsage 設定を使用してディスクの使用領域を最小限に抑えるように最適化されている場合、専用ログ ファイルが NTFS スパース ファイルとして作成されます。通常ログ ファイルは常に状態情報でいっぱいであるわけではないため、スパース ファイルを使用すると、さらに多くのレプリカに対するディスク領域の過度なプロビジョニングが可能になります。このように構成されていない場合は、ログ ファイルの領域は事前に割り当て済みで、ログ コンポーネントは最大パフォーマンスでファイルに直接書き込むことができます。

このログは、最小限のユーザー モード インターフェイスであるにもかかわらず、カーネル モード ドライバーとして書き込まれます。ログがカーネル モード ドライバーとして実行されると、このログを使用するすべてのサービスのパフォーマンスが最大化されます。

ログ構成の詳細については「[ステートフル Reliable Service の構成](service-fabric-reliable-services-configuration.md)」を参照してください。

## ステートレス Reliable Service

### ステートレス サービスのアーキテクチャ
![ステートレス サービスのアーキテクチャ図](./media/service-fabric-reliable-services-platform-architecture/reliable-stateless-service-architecture.png)

### ステートレス Reliable Service

ステートレス サービス実装は、StatelessService または StatelessServiceBase クラスから派生します。StatelessServiceBase クラスは、StatelessService クラスよりも柔軟です。いずれの基本クラスでも、サービスの有効期間と役割が管理されます。

サービス実装によって、いずれかの基本クラスの仮想メソッドがオーバーライドされることがあります。それは、サービス実装のライフサイクルにおけるこれらのポイントで、サービスで実行しなければならない処理がある場合や、通信リスナー オブジェクトを作成する場合です。上記の図で示すように、サービスが独自の通信リスナー オブジェクトを実装して ICommunicationListener を公開する場合がありますが、このサービス実装では Service Fabric が実装した通信リスナーが使用されるため、通信リスナーは Service Fabric が実装します。

StatelessService クラスおよび StatelessServiceBase クラスを使用したサービス作成の詳細については、「[Reliable Service の概要](service-fabric-reliable-services-introduction.md)」と「[Reliable Service の詳細な使用方法](service-fabric-reliable-services-advanced-usage.md)」を参照してください 。

<!--Every topic should have next steps and links to the next logical set of content to keep the customer engaged-->
## 次のステップ

Service Fabric の詳細については、次の項目を参照してください。

[Reliable Service の概要](service-fabric-reliable-services-introduction.md)

[クイック スタート](service-fabric-reliable-services-quick-start.md)

[Reliable Collection の概要](service-fabric-reliable-services-reliable-collections.md)

[Reliable Service の詳細な使用方法](service-fabric-reliable-services-advanced-usage.md)

[Reliable Service の構成](service-fabric-reliable-services-configuration.md)

<!---HONumber=AcomDC_1223_2015-->