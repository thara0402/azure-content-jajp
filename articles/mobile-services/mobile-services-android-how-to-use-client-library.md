<properties
	pageTitle="Mobile Services 向け Android クライアント ライブラリの操作"
	description="Azure Mobile Services 向け Android クライアントを使用する方法について説明します。"
	services="mobile-services"
	documentationCenter="android"
	authors="RickSaling"
	manager="dwrede"
	editor=""/>

<tags
	ms.service="mobile-services"
	ms.workload="mobile"
	ms.tgt_pltfrm="mobile-android"
	ms.devlang="java"
	ms.topic="article"
	ms.date="01/20/2016"
	ms.author="ricksal"/>


# Mobile Services 向け Android クライアント ライブラリの使用方法

[AZURE.INCLUDE [mobile-service-note-mobile-apps](../../includes/mobile-services-note-mobile-apps.md)]

&nbsp;

[AZURE.INCLUDE [mobile-services-selector-client-library](../../includes/mobile-services-selector-client-library.md)]

このガイドでは、AzureMobile Services 向け Android クライアントを使用して一般的なシナリオを実行する方法について説明します。紹介するシナリオは、データの照会、挿入、更新、および削除、ユーザーの認証、エラー処理、クライアントのカスタマイズなどです。

Mobile Services を初めて使用する場合は、クイックスタート チュートリアル「[Mobile Services を使い始める]」を先に完了しておく必要があります。このチュートリアルを完了すると、Android Studio がインストールされ、アカウントの構成、初めてのモバイル サービスの作成、Mobile Services SDK のインストールに役立ちます。Mobile Services SDK は、Android 2.2 以降をサポートしていますが、Android 4.2 以降にビルドすることをお勧めします。

[ここ](http://go.microsoft.com/fwlink/p/?LinkId=298735)で、Android クライアント ライブラリの Javadoc API リファレンスを検索できます。

[AZURE.INCLUDE [mobile-services-concepts](../../includes/mobile-services-concepts.md)]

##<a name="setup"></a>セットアップと前提条件

前提条件として、モバイル サービスとテーブルを作成してあるとします。詳細については、「[テーブルの作成](http://go.microsoft.com/fwlink/p/?LinkId=298592)」を参照してください。このトピックで使用するコードでは、次の列を含むテーブル ToDoItem を想定しています。

- id
- text
- 完了

これに対応する型指定されたクライアント側オブジェクトを次に示します。

	public class ToDoItem {
		private String id;
		private String text;
		private Boolean complete;
	}

動的スキーマが有効な場合、挿入または更新の要求に含まれるオブジェクトに基づいて、Azure Mobile Services によって自動的に新しい列が生成されます。詳細については、「[動的スキーマ](http://go.microsoft.com/fwlink/p/?LinkId=296271)」を参照してください。

##<a name="create-client"></a>方法: Mobile Services クライアントを作成する
次のコードは、モバイル サービスにアクセスするために使用される [MobileServiceClient](http://dl.windowsazure.com/androiddocs/com/microsoft/windowsazure/mobileservices/MobileServiceClient.html) オブジェクトを生成します。このコードは、**MAIN** アクションと **LAUNCHER** カテゴリとして AndroidManifest.xml で指定された Activity クラスの `onCreate` メソッドにあります。

		MobileServiceClient mClient = new MobileServiceClient(
				"MobileServiceUrl", // Replace with the above Site URL
				"AppKey", 			// replace with the Application Key
				this)

前のコードの `MobileServiceUrl` と `AppKey` を、モバイル サービスの URL とアプリケーション キーで順に置き換えます。どちらも Azure クラシック ポータルで確認できます。モバイル サービスを選択し、[ダッシュボード] をクリックしてください。

##<a name="instantiating"></a>方法: テーブル参照を作成する

Java は厳密に型指定された言語であるため、モバイル サービスのデータを照会または変更する最も簡単な方法は型指定されたプログラミング モデルを使用する方法です (後で型指定されないモデルについて説明します)。このモデルは、クライアントとモバイル サービスの間でデータを送信するときに [gson](http://go.microsoft.com/fwlink/p/?LinkId=290801) ライブラリを使用して JSON にシームレスなシリアル化と非シリアル化を提供します。開発者は何も作業する必要がありません。フレームワークによってすべてが処理されます。

データを照会または変更するためには、最初に [**MobileServiceClient**](http://go.microsoft.com/fwlink/p/?LinkId=296835) の [getTable](http://dl.windowsazure.com/androiddocs/com/microsoft/windowsazure/mobileservices/MobileServiceClient.html) メソッドを呼び出して **MobileServiceTable** オブジェクトを作成します。ここでは、このメソッドの 2 つのオーバーロードに注目します。

	public class MobileServiceClient {
	    public <E> MobileServiceTable<E> getTable(Class<E> clazz);
	    public <E> MobileServiceTable<E> getTable(String name, Class<E> clazz);
	}

次のコードで、mClient は、モバイル サービス クライアントへの参照です。

[1 つ目のオーバーロード](http://go.microsoft.com/fwlink/p/?LinkId=296839)は、クラス名とテーブル名が同じ場合に使用されています。

		MobileServiceTable<ToDoItem> mToDoTable = mClient.getTable(ToDoItem.class);


[2 つ目のオーバーロード](http://go.microsoft.com/fwlink/p/?LinkId=296840)は、テーブル名と型名が異なる場合に使用されています。

		MobileServiceTable<ToDoItem> mToDoTable = mClient.getTable("ToDoItemBackup", ToDoItem.class);

## <a name="api"></a>API の構造

クライアント ライブラリのバージョン 2.0 以降、モバイル サービスのテーブル操作は、クエリや挿入、更新、削除の操作を含むメソッドなどのすべての非同期操作で、[Future](http://developer.android.com/reference/java/util/concurrent/Future.html) オブジェクトと [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) オブジェクトを使用します。これにより、複数の入れ子になったコールバックを処理しなくても (バックグラウンド スレッドで) 簡単に複数の操作を実行できます。


##<a name="querying"></a>方法: モバイル サービスのデータを照会する

このセクションでは、モバイル サービスにクエリを発行する方法について説明します。サブセクションでは、並べ替え、フィルター処理、ページングなど、さまざまな処理について説明します。最後に、これらの操作を連結する方法について説明します。

### <a name="showAll"></a>方法: テーブルからすべての項目を返す

次のコードは、ToDoItem テーブル内のすべての項目を返します。この項目をアダプターに追加して、UI に表示します。このコードは、クイックスタート チュートリアル「[Mobile Services を使い始める]」で説明されているコードに似ています。

		new AsyncTask<Void, Void, Void>() {

            @Override
            protected Void doInBackground(Void... params) {
                try {

					final MobileServiceList<ToDoItem> result = mToDoTable.execute().get();
                    runOnUiThread(new Runnable() {

                        @Override
                        public void run() {
                            mAdapter.clear();
                            for (ToDoItem item : result) {
                                mAdapter.add(item);
                            }
                        }
                    });
               } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
               }
               return result;
            }
		}.execute();


このようなクエリでは [AsyncTask](http://developer.android.com/reference/android/os/AsyncTask.html) オブジェクトを使用します。

結果の変数は、クエリの結果セットを返します。`mToDoTable.execute().get()` ステートメントの次のコードは、個々の行を表示する方法を示します。


### <a name="filtering"></a>方法: 返されるデータをフィルター処理する

次のコードは、ToDoItem テーブルから、complete フィールドが false に等しいすべての項目を返します。mToDoTable は、以前に作成したモバイル サービス テーブルへの参照です。

        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                try {
                    final MobileServiceList<ToDoItem> result =
						mToDoTable.where().field("complete").eq(false).execute().get();
					for (ToDoItem item : result) {
                		Log.i(TAG, "Read object with ID " + item.id);
					}
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        }.execute();



フィルターは、テーブル参照での [**where**](http://go.microsoft.com/fwlink/p/?LinkId=296867) メソッドの呼び出しで開始します。その次に [**field**](http://go.microsoft.com/fwlink/p/?LinkId=296869) メソッド呼び出し、その次に論理述語を指定するメソッド呼び出しを続けます。使用できる述語メソッドには、[**eq**](http://go.microsoft.com/fwlink/p/?LinkId=298461)、[**ne**](http://go.microsoft.com/fwlink/p/?LinkId=298462)、[**gt**](http://go.microsoft.com/fwlink/p/?LinkId=298463)、[**ge**](http://go.microsoft.com/fwlink/p/?LinkId=298464)、[**lt**](http://go.microsoft.com/fwlink/p/?LinkId=298465)、[**le**](http://go.microsoft.com/fwlink/p/?LinkId=298466) などがあります。

数値フィールドと文字列フィールドを特定の値と比較するにはこれだけで十分ですが、それ以外のさまざまな操作を行うことができます。

たとえば、日付をフィルター処理できます。日付フィールド全体を比較できるほか、[**year**](http://go.microsoft.com/fwlink/p/?LinkId=298467)、[**month**](http://go.microsoft.com/fwlink/p/?LinkId=298468)、[**day**](http://go.microsoft.com/fwlink/p/?LinkId=298469)、[**hour**](http://go.microsoft.com/fwlink/p/?LinkId=298470)、[**minute**](http://go.microsoft.com/fwlink/p/?LinkId=298471)、[**second**](http://go.microsoft.com/fwlink/p/?LinkId=298472) などのメソッドを使用して日付の一部を比較することもできます。次の部分的なコードでは、期限 (due) が 2013 に等しい項目のフィルターを追加しています。

		mToDoTable.where().year("due").eq(2013).execute().get();

[**startsWith**](http://go.microsoft.com/fwlink/p/?LinkId=298473)、[**endsWith**](http://go.microsoft.com/fwlink/p/?LinkId=298474)、[**concat**](http://go.microsoft.com/fwlink/p/?LinkId=298475)、[**subString**](http://go.microsoft.com/fwlink/p/?LinkId=298477)、[**indexOf**](http://go.microsoft.com/fwlink/p/?LinkId=298488)、[**replace**](http://go.microsoft.com/fwlink/p/?LinkId=298491)、[**toLower**](http://go.microsoft.com/fwlink/p/?LinkId=298492)、[**toUpper**](http://go.microsoft.com/fwlink/p/?LinkId=298493)、[**trim**](http://go.microsoft.com/fwlink/p/?LinkId=298495)、[**length**](http://go.microsoft.com/fwlink/p/?LinkId=298496) などのメソッドを使用すると、文字列フィールドに対してさまざまな複雑なフィルターを適用できます。次の部分的なコードでは、text 列が "PRI0" で始まるテーブル行をフィルター処理します。

		mToDoTable.where().startsWith("text", "PRI0").execute().get();

数値フィールドに対しては、[**add**](http://go.microsoft.com/fwlink/p/?LinkId=298497)、[**sub**](http://go.microsoft.com/fwlink/p/?LinkId=298499)、[**mul**](http://go.microsoft.com/fwlink/p/?LinkId=298500)、[**div**](http://go.microsoft.com/fwlink/p/?LinkId=298502)、[**mod**](http://go.microsoft.com/fwlink/p/?LinkId=298503)、[**floor**](http://go.microsoft.com/fwlink/p/?LinkId=298505)、[**ceiling**](http://go.microsoft.com/fwlink/p/?LinkId=298506)、[**round**](http://go.microsoft.com/fwlink/p/?LinkId=298507) などのメソッドを使用してさまざまな複雑なフィルターを適用できます。次の部分的なコードでは、duration が偶数のテーブル行をフィルター処理します。

		mToDoTable.where().field("duration").mod(2).eq(0).execute().get();


[**and**](http://go.microsoft.com/fwlink/p/?LinkId=298512)、[**or**](http://go.microsoft.com/fwlink/p/?LinkId=298514)、[**not**](http://go.microsoft.com/fwlink/p/?LinkId=298515) のようなメソッドを使用すると、述語を結合できます。次の部分的なコードでは、前の 2 つの例を結合しています。

		mToDoTable.where().year("due").eq(2013).and().startsWith("text", "PRI0")
					.execute().get();

次の部分的なコードに示すように、論理演算子をグループ化したり入れ子にしたりすることができます。

		mToDoTable.where()
					.year("due").eq(2013)
						.and
					(startsWith("text", "PRI0").or().field("duration").gt(10))
					.execute().get();

フィルター処理の詳細と例については、[Mobile Services の Android クライアント クエリ モデルの機能に関する記事](http://hashtagfail.com/post/46493261719/mobile-services-android-querying)を参照してください。

### <a name="sorting"></a>方法: 返されるデータを並べ替える

次のコードは、ToDoItems テーブルから、text フィールドの値に基づいて昇順に並べ替えられたすべての項目を返します。mToDoTable は、以前に作成したモバイル サービス テーブルへの参照です。

		mToDoTable.orderBy("text", QueryOrder.Ascending).execute().get();

[**orderBy**](http://go.microsoft.com/fwlink/p/?LinkId=298519) メソッドの 1 つ目のパラメーターは、並べ替えに使用するフィールドの名前に等しい文字列です。

2 番目のパラメーターでは、[**QueryOrder**](http://go.microsoft.com/fwlink/p/?LinkId=298521) 列挙を使用して、昇順または降順のどちらで並べ替えを行うかを指定します。

**where** メソッドを使用してフィルター処理を行う場合、**where** メソッドは **orderBy** メソッドの前に呼び出す必要があります。

### <a name="paging"></a>方法: ページにデータを返す

最初の例は、テーブルから最初の 5 つの項目を選択する方法を示しています。このクエリは、ToDoItems テーブルから項目を返します。mToDoTable は、以前に作成したモバイル サービス テーブルへの参照です。

       final MobileServiceList<ToDoItem> result = mToDoTable.top(5).execute().get();


次に、最初の 5 つの項目をスキップしてその次の 5 つの項目を返すクエリを定義します。

		mToDoTable.skip(5).top(5).execute().get();


### <a name="selecting"></a>方法: 特定の列を選択する

次のコードは、ToDoItems テーブルからすべての項目を返したうえで complete フィールドと text フィールドのみを表示する方法を示しています。mToDoTable は、以前に作成したモバイル サービス テーブルへの参照です。

		mToDoTable.select("complete", "text").execute().get();


select 関数のパラメーターは、取得するテーブルの列の文字列名です。

[**select**](http://go.microsoft.com/fwlink/p/?LinkId=290689) メソッドは、[**where**](http://go.microsoft.com/fwlink/p/?LinkId=296296) や [**orderBy**](http://go.microsoft.com/fwlink/p/?LinkId=296313) のようなメソッドがある場合はその後に記述する必要があります。このメソッドの後に [**top**](http://go.microsoft.com/fwlink/p/?LinkId=298731) のようなメソッドを記述することができます。

### <a name="chaining"></a>方法: クエリ メソッドを連結する

モバイル サービス テーブルの照会に使用されているメソッドは連結できます。これにより、並べ替えとページングが行われたフィルター処理された行の特定の列を選択するなど、さまざまな操作を行うことができます。複雑な論理フィルターを作成できます。

そのしくみは、使用しているクエリ メソッドによって [**MobileServiceQuery&lt;T&gt;**](http://go.microsoft.com/fwlink/p/?LinkId=298551) オブジェクトを返し、次にこのオブジェクトで追加のメソッドを呼び出します。一連のメソッドを終了し、クエリを実際に実行するには、[**execute**](http://go.microsoft.com/fwlink/p/?LinkId=298554) メソッドを呼び出します。

次にコード サンプルを示します。ここで、mToDoTable は、モバイル サービスの ToDoItem テーブルへの参照です。

		mToDoTable.where().year("due").eq(2013)
						.and().startsWith("text", "PRI0")
						.or().field("duration").gt(10)
					.select("id", "complete", "text", "duration")
					.orderBy(duration, QueryOrder.Ascending).top(20)
					.execute().get();

メソッドを連結する場合の主な要件として、where メソッドと述語を最初に記述する必要があります。それ以降、アプリケーションのニーズに最も合う順番で後続のメソッドを呼び出すことができます。


##<a name="inserting"></a>方法: モバイル サービスにデータを挿入する

次のコードは、テーブルに新しい行を挿入する方法を示しています。

最初に、ToDoItem クラスのインスタンスをインスタンス化し、そのプロパティを設定します。

		ToDoItem mToDoItem = new ToDoItem();
		mToDoItem.text = "Test Program";
		mToDoItem.complete = false;

 次に、次のコードを実行します。

		// Insert the new item
	    new AsyncTask<Void, Void, Void>() {

	        @Override
	        protected Void doInBackground(Void... params) {
	            try {
	                mToDoTable.insert(item).get();
	                if (!item.isComplete()) {
	                    runOnUiThread(new Runnable() {
	                        public void run() {
	                            mAdapter.add(item);
	                        }
	                    });
	                }
	            } catch (Exception exception) {
	                createAndShowDialog(exception, "Error");
	            }
	            return null;
	        }
	    }.execute();


このコードが新しい項目を挿入し、これをアダプターに追加すると、UI に項目が表示されます。

Mobile Services では、テーブル ID として一意のカスタム文字列値がサポートされています。これによって、アプリケーションは Mobile Services テーブルの ID 列に電子メール アドレスやユーザー名などのカスタム値を使用できます。たとえば、各レコードを電子メール アドレスで識別する場合は、次の JSON オブジェクトを使用できます。

		ToDoItem mToDoItem = new ToDoItem();
		mToDoItem.id = "myemail@mydomain.com";
		mToDoItem.text = "Test Program";
		mToDoItem.complete = false;

新しいレコードをテーブルに挿入するときに文字列 ID 値が指定されない場合は、Mobile Services によって ID 用の一意の値が生成されます。

文字列 ID のサポートは、開発者にとって次のような利点があります。

+ データベースと往復することなく ID が生成されます。
+ 他のテーブルやデータベースのレコードをより簡単にマージできます。
+ ID 値をより適切にアプリケーションのロジックに統合できます。

サーバー スクリプトを使用して ID 値を設定することもできます。次のスクリプト例は、カスタム GUID を生成し、新しいレコードの ID に割り当てます。これは、レコードの ID として値を渡さなかった場合に、Mobile Services によって生成される ID 値に似ています。

	//Example of generating an id. This is not required since Mobile Services
	//will generate an id if one is not passed in.
	item.id = item.id || newGuid();
	request.execute();

	function newGuid() {
		var pad4 = function(str) { return "0000".substring(str.length) + str; };
		var hex4 = function () { return pad4(Math.floor(Math.random() * 0x10000 /* 65536 */ ).toString(16)); };
		return (hex4() + hex4() + "-" + hex4() + "-" + hex4() + "-" + hex4() + "-" + hex4() + hex4() + hex4());
	}


アプリケーションが ID の値を指定すると、Mobile Services はそれをそのまま格納します。これには、前後の空白文字も含まれます。値から空白文字が除去されることはありません。

`id` の値は一意である必要があり、次のセット内の文字を含まないようにする必要があります。

+ 制御文字: [0x0000-0x001F] および [0x007F-0x009F]。詳細については、[ASCII 制御コード C0 および C1 に関するページ]を参照してください。
+  印刷可能文字: **"** (0x0022)、**+** (0x002B)、**/** (0x002F)、**?** (0x003F)、**\** (0x005C)、**`** (0x0060)
+  ID "." および ".."

また、テーブルに整数 ID を使用することもできます。整数 ID を使用するには、`mobile table create` コマンドで `--integerId` オプションを使用してテーブルを作成する必要があります。このコマンドは、Azure のコマンド ライン インターフェイス (CLI) で使用されます。CLI の使い方の詳細については、「Mobile Services テーブルを管理するための CLI」を参照してください。


##<a name="updating"></a>方法: モバイル サービスのデータを更新する

次のコードは、テーブルのデータを更新する方法を示しています。この例では、item は、いくつかの変更が加えられた ToDoItem テーブルの行への参照です。次のメソッドは、テーブルと UI アダプターを更新します。

	private void updateItem(final ToDoItem item) {
	    if (mClient == null) {
	        return;
	    }

	    new AsyncTask<Void, Void, Void>() {

	        @Override
	        protected Void doInBackground(Void... params) {
	            try {
	                mToDoTable.update(item).get();
	                runOnUiThread(new Runnable() {
	                    public void run() {
	                        if (item.isComplete()) {
	                            mAdapter.remove(item);
	                        }
	                        refreshItemsFromTable();
	                    }
	                });
	            } catch (Exception exception) {
	                createAndShowDialog(exception, "Error");
	            }
	            return null;
	        }
	    }.execute();
	}

##<a name="deleting"></a>方法: モバイル サービスのデータを削除する

次のコードは、テーブルのデータを削除する方法を示しています。これは、UI で **[完了]** チェック ボックスがオンになっている ToDoItem テーブルから、既存の項目を削除します。

	public void checkItem(final ToDoItem item) {
		if (mClient == null) {
			return;
		}

		// Set the item as completed and update it in the table
		item.setComplete(true);

		new AsyncTask<Void, Void, Void>() {

            @Override
            protected Void doInBackground(Void... params) {
                try {
                    mToDoTable.delete(item);
                    runOnUiThread(new Runnable() {
                        public void run() {
                            if (item.isComplete()) {
                                mAdapter.remove(item);
                            }
                            refreshItemsFromTable();
                        }
                    });
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        }.execute();
	}


次のコードは、別の方法でこの操作を行う方法を示しています。このコードでは、削除する行の id フィールドの値 ("2FA404AB-E458-44CD-BC1B-3BC847EF0902" に等しいと想定) を指定して、ToDoItem テーブル内の既存の項目を削除します。実際のアプリケーションでは、何らかの方法で ID を選択し、これを変数として渡すことができます。ここでは、テストを簡単にするため、Azure クラシック ポータルのサービスに移動して、**[データ]** タブをクリックし、テストする ID をコピーします。

    public void deleteItem(View view) {

        final String ID = "2FA404AB-E458-44CD-BC1B-3BC847EF0902";
        new AsyncTask<Void, Void, Void>() {

            @Override
            protected Void doInBackground(Void... params) {
                try {
                    mToDoTable.delete(ID);
                    runOnUiThread(new Runnable() {
                        public void run() {
                            refreshItemsFromTable();
                        }
               });
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        }.execute();
    }

##<a name="lookup"></a>方法: 特定の項目を検索する
一般的にクエリを使って特定の条件を満たした項目のコレクションを取得する代わりに、特定の項目を id で検索する必要がある場合があります。その方法を次のコードに示します (ここでは id 値として `0380BAFB-BCFF-443C-B7D5-30199F730335` を想定しています)。実際のアプリケーションでは、何らかの方法で ID を選択し、これを変数として渡すことができます。ここでは、テストを簡単にするため、Azure クラシック ポータルのサービスに移動して、**[データ]** タブをクリックし、テストする ID をコピーします。

    /**
     * Lookup specific item from table and UI
     */
    public void lookup(View view) {

        final String ID = "0380BAFB-BCFF-443C-B7D5-30199F730335";
        new AsyncTask<Void, Void, Void>() {

            @Override
            protected Void doInBackground(Void... params) {
                try {
                    final ToDoItem result = mToDoTable.lookUp(ID).get();
                    runOnUiThread(new Runnable() {
                        public void run() {
                            mAdapter.clear();
                            mAdapter.add(result);
                        }
               });
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        }.execute();
    }

##<a name="untyped"></a>方法: 型指定のないデータを処理する

型指定のないプログラミング モデルでは、JSON シリアル化を的確に制御できます。このモデルを使用するシナリオとしては、モバイル サービス テーブルに大量の列が含まれ、参照する必要があるのはそのいくつかのみである場合などが想定されます。型指定されたモデルを使用するには、モバイル サービス テーブルのすべての列をデータ クラスに定義する必要があります。一方、型指定のないモデルでは、使用する列を定義するだけで済みます。

データにアクセスするための API 呼び出しのほとんどは、型指定されたプログラミングでの呼び出しに似ています。主な違いとして、型指定のないモデルでは、**MobileServiceTable** オブジェクトの代わりに **MobileServiceJsonTable** オブジェクトのメソッドを呼び出します。


### <a name="json_instance"></a>方法: 型指定のないテーブルのインスタンスを作成する

型指定されたモデルと同様、テーブル参照を取得することから始めますが、このケースでのオブジェクトは[MobileServicesJsonTable](http://go.microsoft.com/fwlink/p/?LinkId=298733) です。Mobile Services クライアントのインスタンスで [getTable()](http://go.microsoft.com/fwlink/p/?LinkId=298734) メソッドを呼び出して、参照を取得します。

まず、変数を定義します。

    /**
     * Mobile Service Json Table used to access untyped data
     */
    private MobileServiceJsonTable mJsonToDoTable;



**onCreate** メソッドの Mobile Services クライアントのインスタンス (ここでは、mClient 変数) を作成した後は、次のコードを使用して **MobileServiceJsonTable** のインスタンスを作成します。


            // Get the Mobile Service Json Table to use
            mJsonToDoTable = mClient.getTable("ToDoItem");

**MobileServiceJsonTable** のインスタンスを作成すると、型指定されたプログラミング モデルで呼び出すことができるほぼすべてのメソッドをそのインスタンスで呼び出すことができます。ただし、次の例に示すように、メソッドが型指定のないパラメーターを受け取る場合もあります。

### <a name="json_insert"></a>方法: 型指定のないテーブルに挿入する

次のコードは、挿入操作を行う方法を示しています。最初に、[**JsonObject**](http://google-gson.googlecode.com/svn/trunk/gson/docs/javadocs/com/google/gson/JsonObject.html) を作成します。これは、<a href=" http://go.microsoft.com/fwlink/p/?LinkId=290801" target="_blank">gson</a> ライブラリの一部です。

		JsonObject item = new JsonObject();
		item.addProperty("text", "Wake up");
		item.addProperty("complete", false);

次に、このオブジェクトを挿入します。[**insert**](http://go.microsoft.com/fwlink/p/?LinkId=298535) メソッドに渡されるコールバック関数は、[**TableJsonOperationCallback**](http://go.microsoft.com/fwlink/p/?LinkId=298532) クラスのインスタンスです。insert メソッドのパラメーターが JsonObject である点に注目してください。

        // Insert the new item
        new AsyncTask<Void, Void, Void>() {

            @Override
            protected Void doInBackground(Void... params) {
                try {
                    mJsonToDoTable.insert(item).get();
                    refreshItemsFromTable();
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        }.execute();


挿入されたオブジェクトの ID を取得する必要がある場合は、このメソッド呼び出しを使用します。

		        jsonObject.getAsJsonPrimitive("id").getAsInt());


### <a name="json_delete"></a>方法: 型指定のないテーブルから削除する

次のコードでは、インスタンス (このケースでは前の **insert** の例で作成したのと同じ JsonObject のインスタンス) を削除する方法を示します。このコードは、型指定されたものと同じですが、このメソッドは **JsonObject** を参照するため、署名が異なります。


         mToDoTable.delete(item);


ID を使用してインスタンスを直接削除することもできます。

		 mToDoTable.delete(ID);



### <a name="json_get"></a>方法: 型指定のないテーブルからすべての行を返す

次のコードは、テーブル全体を取得する方法を示しています。JSON テーブルを使用しているため、テーブルの列の一部のみを選択的に取得できます。

    public void showAllUntyped(View view) {
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                try {
                    final JsonElement result = mJsonToDoTable.execute().get();
                    final JsonArray results = result.getAsJsonArray();
                    runOnUiThread(new Runnable() {

                        @Override
                        public void run() {
                            mAdapter.clear();
                            for (JsonElement item : results) {
                                String ID = item.getAsJsonObject().getAsJsonPrimitive("id").getAsString();
                                String mText = item.getAsJsonObject().getAsJsonPrimitive("text").getAsString();
                                Boolean mComplete = item.getAsJsonObject().getAsJsonPrimitive("complete").getAsBoolean();
                                ToDoItem mToDoItem = new ToDoItem();
                                mToDoItem.setId(ID);
                                mToDoItem.setText(mText);
                                mToDoItem.setComplete(mComplete);
                                mAdapter.add(mToDoItem);
                            }
                        }
                    });
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        }.execute();
    }

型指定されたプログラミング モデルで使用されたのと同じ名前を持つメソッドを連結して、フィルター処理、並べ替え、およびページングを行うことができます。


##<a name="binding"></a>方法: データをユーザー インターフェイスにバインドする

データ バインドには、次の 3 つのコンポーネントが関係します。

- データ ソース
- 画面レイアウト
- これらの 2 つを連結するアダプター

次のサンプル コードでは、モバイル サービス テーブル ToDoItem のデータを配列に返します。これは、データ アプリケーションで一般的なパターンの 1 つです。一般に、データベース クエリでは、行のコレクションが返されます。クライアントは、これをリストまたは配列に取得します。この例では、配列はデータ ソースです。

コードで、デバイスに表示されるデータのビューを定義する画面レイアウトを指定します。

これらの 2 つをアダプターによってバインドします。アダプターは、このコードでは ArrayAdapter&lt;ToDoItem&gt; クラスの拡張です。

### <a name="layout"></a>方法: レイアウトを定義する

レイアウトは、XML コードの複数のスニペットで定義されます。既存のレイアウトがあり、次のコードがサーバーのデータを設定する **ListView** を表すと想定します。

    <ListView
        android:id="@+id/listViewToDo"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        tools:listitem="@layout/row_list_to_do" >
    </ListView>


前のコードで、listitem 属性は、リスト内の個々の行のレイアウトの ID を指定します。次にそのコードを示します。チェック ボックスとそれに関連付けられたテキストを指定しています。これは、リスト内のそれぞれの項目に対して 1 回インスタンス化されます。より複雑なレイアウトでは、表示する追加フィールドを指定することもできます。このコードは、row\_list\_to\_do.xml ファイルに含まれています。

	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:orientation="horizontal">
	    <CheckBox
	        android:id="@+id/checkToDoItem"
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:text="@string/checkbox_text" />
	</LinearLayout>


### <a name="adapter"></a>方法: アダプターを定義する

このビューのデータ ソースは ToDoItem の配列であるため、ArrayAdapter<ToDoItem> クラスからアダプターをサブクラス化します。このサブクラスでは、row\_list\_to\_do レイアウトを使用してすべての ToDoItem のビューを生成します。

コードでは、ArrayAdapter&lt;E&gt; クラスの拡張である次のクラスを定義します。

	public class ToDoItemAdapter extends ArrayAdapter<ToDoItem> {


アダプターの getView メソッドはオーバーライドする必要があります。その方法の 1 つを次のサンプル コードに示します。詳細はアプリケーションによって異なります。

	public View getView(int position, View convertView, ViewGroup parent) {
		View row = convertView;

		final ToDoItem currentItem = getItem(position);

		if (row == null) {
			LayoutInflater inflater = ((Activity) mContext).getLayoutInflater();
			row = inflater.inflate(R.layout.row_list_to_do, parent, false);
		}

		row.setTag(currentItem);

		final CheckBox checkBox = (CheckBox) row.findViewById(R.id.checkToDoItem);
		checkBox.setText(currentItem.getText());
		checkBox.setChecked(false);
		checkBox.setEnabled(true);

		return row;
	}

次に示すように、このクラスのインスタンスを Activity 内で作成します。

	ToDoItemAdapter mAdapter;
	mAdapter = new ToDoItemAdapter(this, R.layout.row_list_to_do);

ToDoItemAdapter コンストラクターの 2 つ目のパラメーターはレイアウトへの参照であることに注意してください。コンストラクターの呼び出しに続けて、次のコードを記述します。このコードは、最初に **ListView** への参照を取得します。次に、setAdapter を呼び出して、作成したアダプターを使用するようにそれ自体を構成します。

	ListView listViewToDo = (ListView) findViewById(R.id.listViewToDo);
	listViewToDo.setAdapter(mAdapter);


### <a name="use-adapter"></a>方法: アダプターを使用する

これで、データ バインドを使用する準備が整いました。次のコードでは、モバイル サービス テーブルの項目を取得した後、アダプターをクリアしています。次に、アダプターの add メソッドを呼び出して、返された項目をアダプターに設定しています。

    public void showAll(View view) {
        new AsyncTask<Void, Void, Void>() {
            @Override
            protected Void doInBackground(Void... params) {
                try {
                    final MobileServiceList<ToDoItem> result = mToDoTable.execute().get();
                    runOnUiThread(new Runnable() {

                        @Override
                        public void run() {
                            mAdapter.clear();
                            for (ToDoItem item : result) {
                                mAdapter.add(item);
                            }
                        }
                    });
                } catch (Exception exception) {
                    createAndShowDialog(exception, "Error");
                }
                return null;
            }
        }.execute();
    }

ToDoItem テーブルを変更したときにその結果を表示するには、その都度アダプターを呼び出す必要があります。変更はレコード単位で加えられるため、操作の対象はコレクションではなく行になります。項目を挿入する場合は、アダプターの add メソッドを呼び出します。項目を削除する場合は、remove メソッドを呼び出します。

##<a name="custom-api"></a>方法: カスタム API の呼び出し

カスタム API を使用してカスタム エンドポイントを定義することにより、insert、update、delete、read のいずれの操作にも関連しないサーバー機能を公開することができます。カスタム API を使用することによって、HTTP メッセージ ヘッダーの読み取りや設定、JSON 以外のメッセージ本文形式の定義など、メッセージングをより柔軟に制御することができます。モバイル サービスでカスタム API を作成する方法の例については、「[カスタム API を定義する方法](mobile-services-dotnet-backend-define-custom-api.md)」を参照してください。

[AZURE.INCLUDE [mobile-services-android-call-custom-api](../../includes/mobile-services-android-call-custom-api.md)]


##<a name="authentication"></a>方法: ユーザーを認証する

Mobile Services は、Facebook、Google、Microsoft アカウント、Twitter、Azure Active Directory などのさまざまな外部 ID プロバイダーを使用したアプリケーション ユーザーの認証と承認をサポートします。テーブルのアクセス許可を設定することにより、特定の操作へのアクセスを認証されたユーザーのみに制限できます。さらに、認証されたユーザーの ID を使用することにより、バックエンドで承認ルールを実装することもできます。詳細については、「[認証の使用](http://go.microsoft.com/fwlink/p/?LinkId=296316)」を参照してください。

サーバー フローとクライアント フローという 2 つの認証フローがサポートされます。サーバー フローには、プロバイダーの Web 認証のインターフェイスを利用する、最も簡単な認証方法が用意されています。クライアント フローでは、プロバイダー固有およびデバイス固有の SDK を利用することから、シングル サインオンなどのデバイス固有の機能との統合がさらに進みます。

アプリケーションで認証を有効にするには、次の 3 つの手順を実行する必要があります。

- 認証するアプリケーションをプロバイダーに登録し、Mobile Services を構成する
- テーブルのアクセス許可を、認証されたユーザーのみに制限する
- アプリケーションに認証コードを追加する


Mobile Services は、ユーザーを認証するために使用できる次の既存の ID プロバイダーをサポートしています。

- Microsoft アカウント
- Facebook
- Twitter
- Google
- Azure Active Directory

テーブルのアクセス許可を設定することにより、特定の操作へのアクセスを認証されたユーザーのみに制限できます。さらに、認証されたユーザーの ID を使用して要求を変更することもできます。

これらの最初の 2 つのタスクは、[Azure クラシック ポータル](https://manage.windowsazure.com/)を使用して実行します。詳細については、「[認証の使用](http://go.microsoft.com/fwlink/p/?LinkId=296316)」を参照してください。

### <a name="caching"></a>方法: アプリケーションに認証コードを追加する

1.  アプリケーションのアクティビティ ファイルに次の import ステートメントを追加します。

		import java.util.concurrent.ExecutionException;
		import java.util.concurrent.atomic.AtomicBoolean;

		import android.content.Context;
		import android.content.SharedPreferences;
		import android.content.SharedPreferences.Editor;

		import com.microsoft.windowsazure.mobileservices.authentication.MobileServiceAuthenticationProvider;
		import com.microsoft.windowsazure.mobileservices.authentication.MobileServiceUser;

2. アクティビティ クラスの **onCreate** メソッドで、`MobileServiceClient` オブジェクトを作成するコードの後に、次のコード行を追加します。`MobileServiceClient` オブジェクトへの参照は mClient であると想定しています。

	    // Login using the Google provider.

		ListenableFuture<MobileServiceUser> mLogin = mClient.login(MobileServiceAuthenticationProvider.Google);

    	Futures.addCallback(mLogin, new FutureCallback<MobileServiceUser>() {
    		@Override
    		public void onFailure(Throwable exc) {
    			createAndShowDialog((Exception) exc, "Error");
    		}
    		@Override
    		public void onSuccess(MobileServiceUser user) {
    			createAndShowDialog(String.format(
                        "You are now logged in - %1$2s",
                        user.getUserId()), "Success");
    			createTable();
    		}
    	});

    このコードでは、Google ログインを使用してユーザーを認証します。認証されたユーザーの ID を示すダイアログが表示されます。認証が成功しないと、次に進むことはできません。

    > [AZURE.NOTE]Google 以外の ID プロバイダーを使用している場合は、上の **login** メソッドに渡す値を、 MicrosoftAccount、Facebook、Twitter、WindowsAzureActiveDirectory のいずれかに変更します。


3. アプリケーションを実行したときに、選択した ID プロバイダーを使ってサインインします。


### <a name="caching"></a>方法: 認証トークンをキャッシュする

このセクションでは、認証トークンをキャッシュする方法について説明します。これにより、トークンが有効なときにアプリケーションが "休止状態" になった場合にユーザーに認証のやり直しを求めることを回避できます。

認証トークンをキャッシュするには、ユーザー ID と認証トークンをデバイスにローカルで保存する必要があります。アプリケーションが次回起動されたときに、キャッシュを確認し、これらの値が存在する場合はログイン手順をスキップし、クライアントに再度このデータを渡します。ただし、このデータは慎重な扱いを要する情報であり、電話の盗難に備えて安全のために暗号化して保存する必要があります。

次のコード スニペットは、Microsoft アカウントのログイン用のトークンを取得する方法を示します。キャッシュが見つかるとトークンはキャッシュされ、再度読み込まれます。

	private void authenticate() {
		if (LoadCache())
		{
			createTable();
		}
		else
		{
			    // Login using the Google provider.
				ListenableFuture<MobileServiceUser> mLogin = mClient.login(MobileServiceAuthenticationProvider.Google);

		    	Futures.addCallback(mLogin, new FutureCallback<MobileServiceUser>() {
		    		@Override
		    		public void onFailure(Throwable exc) {
		    			createAndShowDialog("You must log in. Login Required", "Error");
		    		}
		    		@Override
		    		public void onSuccess(MobileServiceUser user) {
		    			createAndShowDialog(String.format(
		                        "You are now logged in - %1$2s",
		                        user.getUserId()), "Success");
		    			cacheUserToken(mClient.getCurrentUser());
		    			createTable();
		    		}
		    	});		}
	}


	private boolean LoadCache()
	{
		SharedPreferences prefs = getSharedPreferences("temp", Context.MODE_PRIVATE);
		String tmp1 = prefs.getString("tmp1", "undefined");
		if (tmp1 == "undefined")
			return false;
		String tmp2 = prefs.getString("tmp2", "undefined");
		if (tmp2 == "undefined")
			return false;
		MobileServiceUser user = new MobileServiceUser(tmp1);
		user.setAuthenticationToken(tmp2);
		mClient.setCurrentUser(user);
		return true;
	}


	private void cacheUser(MobileServiceUser user)
	{
		SharedPreferences prefs = getSharedPreferences("temp", Context.MODE_PRIVATE);
		Editor editor = prefs.edit();
		editor.putString("tmp1", user.getUserId());
		editor.putString("tmp2", user.getAuthenticationToken());
		editor.commit();
	}


トークンが期限切れになった場合はどのような処理が行われるでしょうか。 その場合、トークンを使用して接続を試みると、"401 許可されていません" 応答が返されます。ユーザーは、ログインして新しいトークンを取得する必要があります。Mobile Services の呼び出しと Mobile Services からの応答を取得するフィルターを使用すると、アプリケーション内でモバイル サービスを呼び出すすべての場所にこれを処理するコードを書かずに済みます。フィルター コードは、401 の応答の有無をテストし、必要に応じてログイン プロセスをトリガーし、401 を生成した要求を再開します。


##<a name="customizing"></a>方法: クライアントをカスタマイズする

Mobile Services クライアントの既定の動作は、さまざまな方法でカスタマイズできます。

### <a name="headers"></a>方法: 要求ヘッダーをカスタマイズする

すべての出力要求にカスタム ヘッダーを接続する場合があります。次のような **ServiceFilter** を構成することによって実現できます。

	private class CustomHeaderFilter implements ServiceFilter {

        @Override
        public ListenableFuture<ServiceFilterResponse> handleRequest(
                	ServiceFilterRequest request,
					NextServiceFilterCallback next) {

            runOnUiThread(new Runnable() {

                @Override
                public void run() {
	        		request.addHeader("My-Header", "Value");	                }
            });

            SettableFuture<ServiceFilterResponse> result = SettableFuture.create();
            try {
                ServiceFilterResponse response = next.onNext(request).get();
                result.set(response);
            } catch (Exception exc) {
                result.setException(exc);
            }
        }

### <a name="serialization"></a>方法: シリアル化をカスタマイズする

Mobile Services では、サーバー上のテーブル名、列名、データ型のすべてがクライアント上の対応する名前や型と一致するものと既定で想定しています。ただし、サーバーとクライアントとの間で名前が一致しない理由はいくつかあります。1 つの例として、既存のクライアントがあり、これを競合企業の製品の代わりに Mobile Services を使用するように変更することを検討しているとします。

次の種類のカスタマイズを行うことができます。

- モバイル サービス テーブルで使用されている列名がクライアントで使用されている名前と一致しない
- クライアントでマップされているクラスと異なる名前を持つモバイル サービス テーブルを使用する
- プロパティの大文字の自動設定を有効にする
- 複合プロパティをオブジェクトに追加する

### <a name="columns"></a>方法: クライアントとサーバーの間で異なる名前をマップする

Java クライアント コードで、ToDoItem オブジェクト プロパティに次のような標準 Java 形式の名前を使用しているとします。

- mId
- mText
- mComplete
- mDuration


クライアントの名前は、サーバー上のToDoItem テーブルの列名と一致する JSON 名にシリアル化する必要があります。<a href=" http://go.microsoft.com/fwlink/p/?LinkId=290801" target="_blank">gson</a> ライブラリを使用してこの操作を行うコードを次に示します。

	@com.google.gson.annotations.SerializedName("text")
	private String mText;

	@com.google.gson.annotations.SerializedName("id")
	private int mId;

	@com.google.gson.annotations.SerializedName("complete")
	private boolean mComplete;

	@com.google.gson.annotations.SerializedName("duration")
	private String mDuration;

### <a name="table"></a>方法: クライアントとモバイル サービスの間で異なるテーブル名をマップする

クライアントのテーブル名を異なるモバイル サービス テーブル名にマップするのは簡単です。次のコードに示すように、<a href="http://go.microsoft.com/fwlink/p/?LinkId=296840" target="_blank">getTable()</a> 関数のオーバーライドの 1 つを使用するだけです。

	mToDoTable = mClient.getTable("ToDoItemBackup", ToDoItem.class);


### <a name="conversions"></a>方法: 列名のマッピングを自動化する

列が数個しかない狭いテーブルの列名のマッピングは、前のセクションに示したように簡単に行うことができます。ここで、テーブルに多数の列 (たとえば、20 ～ 30) があるとします。その場合は、<a href=" http://go.microsoft.com/fwlink/p/?LinkId=290801" target="_blank">gson</a> API を呼び出して、すべての列に適用される変換戦略を指定することにより、それぞれの列名に注釈を付けずに済みます。

そのためには、<a href=" http://go.microsoft.com/fwlink/p/?LinkId=290801" target="_blank">gson</a> ライブラリを使用します。Android クライアント ライブラリは、このライブラリをバックグラウンドで使用して Java オブジェクトを Azure Mobile Services に送信される JSON データにシリアル化します。

次のコードでは、FieldNamingStrategy() メソッドが定義された setFieldNamingStrategy() メソッドを使用しています。このメソッドでは、すべてのフィールド名について、最初の文字 ("m") を削除し、その次の文字を小文字にすることを指定しています。さらに、このコードは、出力 JSON の再フォーマットを有効にしています。

	client.setGsonBuilder(
	    MobileServiceClient
	    .createMobileServiceGsonBuilder()
	    .setFieldNamingStrategy(new FieldNamingStrategy() {
	        public String translateName(Field field) {
	            String name = field.getName();
	            return Character.toLowerCase(name.charAt(1))
	                + name.substring(2);
	            }
	        })
	        .setPrettyPrinting());



このコードは、Mobile Services クライアント オブジェクトのすべてのメソッド呼び出しの前に実行する必要があります。

### <a name="complex"></a>方法: オブジェクトまたは配列プロパティをテーブルに保存する

これまでのシリアル化のすべての例には、JSON およびモバイル サービス テーブルに簡単にシリアル化できるプリミティブ型 (整数、文字列など) が使用されていました。ここで、JSON やテーブルに自動的にシリアル化されない複合オブジェクトをクライアント型に追加する場合を想定します。たとえば、文字列の配列をクライアント オブジェクトに追加するとします。この段階で、シリアル化を行う方法と配列をモバイル サービス テーブルに格納する方法は自由に選ぶことができます。

これを行う方法の例については、<a href="http://hashtagfail.com/post/44606137082/mobile-services-android-serialization-gson" target="_blank">Mobile Services Android クライアントでの <a href=" http://go.microsoft.com/fwlink/p/?LinkId=290801" target="_blank">gson</a> ライブラリを使用したシリアル化のカスタマイズに関するブログ記事を参照してください。</a>

この汎用的な方法は、JSON やモバイル サービス テーブルに自動的にシリアル化されない複合オブジェクトがある場合にいつでも使用できます。

<!-- Anchors. -->

[What is Mobile Services]: #what-is
[Concepts]: #concepts
[How to: Create the Mobile Services client]: #create-client
[How to: Create a table reference]: #instantiating
[The API structure]: #api
[How to: Query data from a mobile service]: #querying
[Return all Items]: #showAll
[Filter returned data]: #filtering
[Sort returned data]: #sorting
[Return data in pages]: #paging
[Select specific columns]: #selecting
[How to: Concatenate query methods]: #chaining
[How to: Bind data to the user interface]: #binding
[How to: Define the layout]: #layout
[How to: Define the adapter]: #adapter
[How to: Use the adapter]: #use-adapter
[How to: Insert data into a mobile service]: #inserting
[How to: update data in a mobile service]: #updating
[How to: Delete data in a mobile service]: #deleting
[How to: Look up a specific item]: #lookup
[How to: Work with untyped data]: #untyped
[How to: Authenticate users]: #authentication
[Cache authentication tokens]: #caching
[How to: Handle errors]: #errors
[How to: Design unit tests]: #tests
[How to: Customize the client]: #customizing
[Customize request headers]: #headers
[Customize serialization]: #serialization
[Next Steps]: #next-steps
[Setup and Prerequisites]: #setup

<!-- Images. -->



<!-- URLs. -->
[Mobile Services を使い始める]: mobile-services-android-get-started.md
[ASCII 制御コード C0 および C1 に関するページ]: http://en.wikipedia.org/wiki/Data_link_escape_character#C1_set

<!---HONumber=AcomDC_0121_2016-->