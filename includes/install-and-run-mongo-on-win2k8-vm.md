次のステップに従って、MongoDB を Windows Server が実行されている仮想マシンにインストールして実行します。

> [AZURE.IMPORTANT]認証、IP アドレス バインドなどの MongoDB セキュリティ機能は既定では有効になっていません。MongoDB を運用環境に展開する前に、セキュリティ機能を有効にすることをお勧めします。詳細については、「[Security and Authentication (セキュリティと認証)](http://www.mongodb.org/display/DOCS/Security+and+Authentication)」を参照してください。

1. リモート デスクトップを使用して仮想マシンに接続したら、仮想マシンで **[スタート]** メニューから Internet Explorer を開きます。

2. 右上にある **[ツール]** を選択します。**[インターネット オプション]** で、**[セキュリティ]** タブ、**[信頼済みサイト]** アイコンの順に選択し、**[サイト]** をクリックします。信頼済みサイトの一覧に _http://*.mongodb.org_ を追加します。

3. [MongoDB のダウンロード ページ][MongoDownloads]にアクセスします。

4. **[Current Stable Release]** を見つけ、[Windows] 列で最新の **64 ビット** バージョンを選択し、MSI インストーラーをダウンロードして実行します。

5. MongoDB は、通常は C:\\Program Files\\MongoDB にインストールされます。デスクトップで環境変数を検索し、MongoDB のバイナリ パスを PATH 変数に追加します。たとえば、バイナリはコンピューターの C:\\Program Files\\MongoDB\\Server\\3.0\\bin に入っている可能性があります。

6. MongoDB データとログ ディレクトリを、前の手順で作成したデータ ディスク (ドライブ **F:** など) に作成します。**[スタート]** で **[コマンド プロンプト]** を選択し、コマンド プロンプト ウィンドウを開きます。型:

		C:\> F:
		F:> mkdir \MongoData
		F:> mkdir \MongoLogs

7. データベースを実行するには、次を実行します。

		F:> C:
		C:\> mongod --dbpath F:\MongoData\ --logpath F:\MongoLogs\mongolog.log

	mongod.exe サーバーがジャーナル ファイルを開始して事前に割り当てると、すべてのログ メッセージが *F:\\MongoLogs\\mongolog.log* ファイルにダイレクトされます。MongoDB がジャーナル ファイルを事前に割り当てて、接続のリッスンを開始するには、数分かかる場合があります。

8. MongoDB 管理シェルを開始するには、**[スタート]** から他のコマンド ウィンドウを開き、次のように入力します。

		C:\> cd \my_mongo_dir\bin  
		C:\my_mongo_dir\bin> mongo  
		>db  
		test
		> db.foo.insert( { a : 1 } )  
		> db.foo.find()  
		{ _id : ..., a : 1 }  
		> show dbs  
		...  
		> show collections  
		...  
		> help  

	データベースは挿入によって作成されます。

9. または、mongod.exe をサービスとしてインストールできます。

		C:\mongodb\bin>mongod --logpath F:\MongoLogs\mongolog.log --logappend --dbpath F:\MongoData\ --install

	これにより、"Mongo DB" という名前で、説明に "Mongo DB" が指定されたサービスが作成されます。実行中のサービスには出力を表示するコマンド ウィンドウがないので、**--logpath** オプションを使用して、ログ ファイルを指定する必要があります。**--logappend** オプションを指定すると、サービスを再起動することで、既存のログ ファイルに出力が追加されるようになります。**--dbpath** オプションは、データ ディレクトリの場所を指定します。サービス関連のコマンド ライン オプションの詳細については、「[Service-related command line options (サービス関連のコマンド ライン オプション)][MongoWindowsSvcOptions]」を参照してください。

	サービスを開始するには、次のコマンドを実行します。

		C:\mongodb\bin>net start MongoDB

10. これで MongoDB がインストールされました。この MongoDB は現在実行されています。次は MongoDB にリモート接続するために、Windows ファイアウォールのポートを開く必要があります。**[スタート]** メニューから、**[管理ツール]**、**[セキュリティが強化された Windows ファイアウォール]** の順に選択します。

11. 左側のウィンドウで、**[受信の規則]** を選択します。右側の **[操作]** ウィンドウで、**[新しい規則]** を選択します。

	![Windows Firewall][Image1]

	**新規の受信の規則ウイザード**で、**[ポート]** を選択し、**[次へ]** をクリックします。

	![Windows Firewall][Image2]

	**[TCP]**、**[特定のローカル ポート]** の順に選択します。ポートとして「27017」(MongoDB がリッスンする規定のポート) を指定し、**[次へ]** をクリックします。

	![Windows Firewall][Image3]

	**[接続を許可する]** を選択し、**[次へ]** をクリックします。

	![Windows Firewall][Image4]

	もう一度 **[次へ]** をクリックします。

	![Windows Firewall][Image5]

	規則の名前 ("MongoPort" など) を指定し、**[完了]** をクリックします。

	![Windows Firewall][Image6]

12. 仮想マシンを作成したときに MongoDB のエンドポイントを構成しなかった場合は、ここで構成できます。ファイアウォール ルールとエンドポイントの両方が MongoDB にリモートで接続できる必要があります。管理ポータルで、**[仮想マシン]**、新しい仮想マシンの名前、**[エンドポイント]** の順にクリックします。

	![Endpoints][Image7]

13. ページの下部にある **[追加]** をクリックします。**[スタンドアロン エンドポイントの追加]** を選択し、**[次へ]** をクリックします。

	![Endpoints][Image8]

14. エンドポイントを追加します。名前に「Mongo」、プロトコルに **[TCP]** を指定し、**[パブリック ポート]** と **[プライベート ポート]** に「27017」を指定します。これにより、MongoDB へのリモート アクセスが可能になります。

	![Endpoints][Image9]

> [AZURE.NOTE]ポート 27017 は MongoDB によって使用される既定のポートです。これは、mongod.exe サーバーの起動時に _--port_ サブコマンドによって変更することができます。ファイアウォールでは、上記の手順の「Mongo」エンドポイントと同じポート番号を指定してください。


[MongoDownloads]: http://www.mongodb.org/downloads

[MongoWindowsSvcOptions]: http://www.mongodb.org/display/DOCS/Windows+Service


[Image1]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall1.png
[Image2]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall2.png
[Image3]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall3.png
[Image4]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall4.png
[Image5]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall5.png
[Image6]: ./media/install-and-run-mongo-on-win2k8-vm/WinFirewall6.png
[Image7]: ./media/install-and-run-mongo-on-win2k8-vm/WinVmAddEndpoint.png
[Image8]: ./media/install-and-run-mongo-on-win2k8-vm/WinVmAddEndpoint2.png
[Image9]: ./media/install-and-run-mongo-on-win2k8-vm/WinVmAddEndpoint3.png

<!---HONumber=Oct15_HO3-->