<properties
   pageTitle="Service Fabric サービス エンドポイントの指定 | Microsoft Azure"
   description="サービス マニフェストにエンドポイント リソースを記述する方法 (HTTPS エンドポイントの設定方法を含みます)"
   services="service-fabric"
   documentationCenter=".net"
   authors="mani-ramaswamy"
   manager="timlt"
   editor=""/>

<tags
   ms.service="service-fabric"
   ms.devlang="dotnet"
   ms.topic="article"
   ms.tgt_pltfrm="NA"
   ms.workload="NA"
   ms.date="01/26/2016"
   ms.author="sumukhs"/>

# サービス マニフェストにリソースを指定する

## 概要

サービス マニフェストを使用すると、コンパイルしたコードを変更することなく、サービスで使用するリソースを宣言/変更できます。Azure Service Fabric は、サービスで使用するエンドポイント リソースの構成をサポートします。サービス マニフェストで指定したリソースへのアクセスは、SecurityGroup を使用してアプリケーション マニフェスト内で制御できます。リソースを宣言すると、宣言したリソースをデプロイメント時に変更できるため、サービスに新しい構成メカニズムを導入する必要がありません。

## エンドポイント

サービス マニフェストにエンドポイント リソースが定義されているが、明示的なポートが指定されていない場合、Service Fabric は、予約済みのアプリケーション ポートの範囲からポートを割り当てます (例: エンドポイント *ServiceEndpoint1* を参照してください)。さらに、サービスでリソースの特定のポートを要求することもできます。別のクラスター ノードで実行しているサービスのレプリカに異なるポート番号を割り当てることができます。一方、同じノードで実行している同じサービスのレプリカは、同じポートを共有します。このようなポートは、レプリケーションやクライアント要求のリッスンなどのさまざまな目的のために、サービスのレプリカで使用できます。

```xml
<Resources>
  <Endpoints>
    <Endpoint Name="ServiceEndpoint1" Protocol="http"/>
    <Endpoint Name="ServiceEndpoint2" Protocol="http" Port="80"/>
    <Endpoint Name="ServiceEndpoint3" Protocol="https"/>
  </Endpoints>
</Resources>
```

構成パッケージ設定ファイル (settings.xml) からエンドポイントを参照する方法の詳細については、「[ステートフル Reliable Services の構成](service-fabric-reliable-services-configuration.md)」をご覧ください。

## 例: サービスの HTTP エンドポイントを指定する

次のサービス マニフェストは、1 つの TCP エンドポイント リソースと 2 つの HTTP エンドポイント リソース を &lt;Resources&gt; 要素内に定義しています。

HTTP エンドポイントは Service Fabric によって自動的に ACL に登録されます。

```xml
<?xml version="1.0" encoding="utf-8"?>
<ServiceManifest Name="Stateful1Pkg"
                 Version="1.0.0"
                 xmlns="http://schemas.microsoft.com/2011/01/fabric"
                 xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                 xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <ServiceTypes>
    <!-- This is the name of your ServiceType.
         This name must match the string used in the RegisterServiceType call in Program.cs. -->
    <StatefulServiceType ServiceTypeName="Stateful1Type" HasPersistedState="true" />
  </ServiceTypes>

  <!-- Code package is your service executable. -->
  <CodePackage Name="Code" Version="1.0.0">
    <EntryPoint>
      <ExeHost>
        <Program>Stateful1.exe</Program>
      </ExeHost>
    </EntryPoint>
  </CodePackage>

  <!-- Config package is the contents of the Config directoy under PackageRoot that contains an
       independently updateable and versioned set of custom configuration settings for your service. -->
  <ConfigPackage Name="Config" Version="1.0.0" />

  <Resources>
    <Endpoints>
      <!-- This endpoint is used by the communication listener to obtain the port number on which to
           listen. Note that if your service is partitioned, this port is shared with
           replicas of different partitions that are placed in your code. -->
      <Endpoint Name="ServiceEndpoint1" Protocol="http"/>
      <Endpoint Name="ServiceEndpoint2" Protocol="http" Port="80"/>
      <Endpoint Name="ServiceEndpoint3" Protocol="https"/>

      <!-- This endpoint is used by the replicator for replicating the state of your service.
           This endpoint is configured through the ReplicatorSettings config section in the Settings.xml
           file under the ConfigPackage. -->
      <Endpoint Name="ReplicatorEndpoint" />
    </Endpoints>
  </Resources>
</ServiceManifest>
```

## 例: サービスの HTTPS エンドポイントを指定する

HTTPS プロトコルはサーバー認証を提供し、また、クライアント/サーバー間通信の暗号化に使用されます。Service Fabric サービスでこれを有効にするには、サービスを定義するときに、サービス マニフェストの *<Resources>、<Endpoints>、<Endpoint>* セクションにプロトコルを指定します (前述のエンドポイント *ServiceEndpoint3* を参照してください)。

>[AZURE.NOTE] アプリケーションのアップグレード中はサービスのプロトコルを変更することはできません。これは、重大な変更であるためです。


HTTPS で設定する必要がある ApplicationManifest の例を次に示します (証明書のサムプリントを指定する必要があります)。 EndpointRef は、HTTPS プロトコルを設定する ServiceManifest 内の EndpointResource への参照です。1 つ以上の Endpointcertificate を追加することができます。

```
<?xml version="1.0" encoding="utf-8"?>
<ApplicationManifest ApplicationTypeName="Application1Type"
                     ApplicationTypeVersion="1.0.0"
                     xmlns="http://schemas.microsoft.com/2011/01/fabric"
                     xmlns:xsd="http://www.w3.org/2001/XMLSchema"
                     xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance">
  <Parameters>
    <Parameter Name="Stateful1_MinReplicaSetSize" DefaultValue="2" />
    <Parameter Name="Stateful1_PartitionCount" DefaultValue="1" />
    <Parameter Name="Stateful1_TargetReplicaSetSize" DefaultValue="3" />
  </Parameters>
  <!-- Import the ServiceManifest from the ServicePackage. The ServiceManifestName and ServiceManifestVersion
       should match the Name and Version attributes of the ServiceManifest element defined in the
       ServiceManifest.xml file. -->
  <ServiceManifestImport>
    <ServiceManifestRef ServiceManifestName="Stateful1Pkg" ServiceManifestVersion="1.0.0" />
    <ConfigOverrides />
    <Policies>
      <EndpointBindingPolicy CertificateRef="TestCert1" EndpointRef="ServiceEndpoint3"/>
    </Policies>
  </ServiceManifestImport>
  <DefaultServices>
    <!-- The section below creates instances of service types when an instance of this
         application type is created. You can also create one or more instances of service type by using the
         Service Fabric PowerShell module.

         The attribute ServiceTypeName below must match the name defined in the imported ServiceManifest.xml file. -->
    <Service Name="Stateful1">
      <StatefulService ServiceTypeName="Stateful1Type" TargetReplicaSetSize="[Stateful1_TargetReplicaSetSize]" MinReplicaSetSize="[Stateful1_MinReplicaSetSize]">
        <UniformInt64Partition PartitionCount="[Stateful1_PartitionCount]" LowKey="-9223372036854775808" HighKey="9223372036854775807" />
      </StatefulService>
    </Service>
  </DefaultServices>
  <Certificates>
    <EndpointCertificate Name="TestCert1" X509FindValue="FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF FF F0" X509StoreName="MY" />  
  </Certificates>
</ApplicationManifest>
```

<!---HONumber=AcomDC_0211_2016-->